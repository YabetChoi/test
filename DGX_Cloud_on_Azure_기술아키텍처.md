# DGX Cloud on Azure — 기술 아키텍처

> **기본 개념:** DGX Cloud는 NVIDIA의 풀스택 AI 소프트웨어를 Azure의 GPU 인프라(ND H100 v5) 위에 얹어, NVIDIA가 관리형(managed)으로 제공하는 서비스입니다. 별도 클라우드가 아니라 **Azure Marketplace를 통해 Azure 환경 안에서 소비**되는 구조입니다.

## 인프라 스택 개요

```
 ┌─────────────────────────────────────────────────────┐
 │  L5  소비 모델     Azure Marketplace Private Offer     │
 │  L4  SW 스택       Base Command · AI Enterprise · Run:ai │
 │  L3  오케스트레이션  Slurm (CycleCloud) · 컨테이너        │
 │  L2  스토리지       Azure Managed Lustre (AMLFS)        │
 │  L1  네트워킹       Quantum-2 InfiniBand · NVLink       │
 │  L0  컴퓨트         ND H100 v5 (8× H100 GPU)            │
 └─────────────────────────────────────────────────────┘
```

---

## L0. 컴퓨트 — Azure ND H100 v5

DGX Cloud가 Azure에서 도는 물리적 기반은 `Standard_ND96isr_H100_v5` VM SKU입니다.

| 구성 요소 | 사양 |
|---|---|
| GPU | NVIDIA H100 Tensor Core GPU 8개 (각 80GB, 총 640GB) |
| CPU | 96 vCPU, 4세대 Intel Xeon (Sapphire Rapids) |
| 메모리 | 1,900 GiB |
| 로컬 스토리지 | 로컬 1,024 GiB + NVMe 8개 × 28 TiB |

고성능 딥러닝 학습과 긴밀하게 결합된(tightly-coupled) scale-up·scale-out 생성형 AI·HPC 워크로드를 위해 설계되었으며, 단일 VM 8 GPU에서 시작합니다.

---

## L1. 네트워킹 — 멀티노드 학습의 핵심

DGX Cloud의 성능을 좌우하는 가장 중요한 기술 레이어. 두 종류의 인터커넥트가 있습니다.

### ① VM 내부 (intra-node) — NVLink 4.0
VM 내부 GPU 간 통신은 NVLink 4.0로 연결되며, GPU끼리 900GB/s 속도로 통신합니다.

### ② VM 간 (inter-node) — Quantum-2 InfiniBand
- 각 GPU에 토폴로지 독립적인 전용 **400 Gb/s** NVIDIA Quantum-2 CX7 InfiniBand 연결
- VM당 **3.2 Tbps** 인터커넥트 대역폭으로 수천 개 GPU까지 확장
- 동일 VM scale set 내 VM 간 자동 구성, **GPUDirect RDMA** 지원
- NVIDIA **NCCL** 통신 라이브러리 기반 도구 지원 → GPU 클러스터링을 매끄럽게 처리

> **AVOps 관점:** 자율주행 인지(perception) 모델은 대용량 멀티노드 학습이 필수인데, 이 InfiniBand 패브릭이 "수천 GPU에서도 거의 선형 확장"을 가능케 하는 기술적 근거입니다.

---

## L2. 스토리지 — Azure Managed Lustre (AMLFS)

대규모 GPU 학습에서는 GPU를 굶기지 않는 I/O가 관건입니다.

- ND H100 v5 SKU에 Azure Managed Lustre File System(AMLFS)을 고성능 공유 스토리지 백엔드로 결합
- AMLFS는 멀티 GPU 학습 잡에 데이터를 대규모로 공급하는 데 필요한 I/O 처리량 제공
- **단, AMLFS는 대상 리전에서 사용 가능해야 설정 가능** → 고객 리전 가용성 체크가 영업 시 필수 확인 항목

---

## L3. 오케스트레이션 — Slurm + 컨테이너

DGX Cloud의 벤치마킹/학습 워크로드는 Slurm 기반으로 동작합니다.

- DGX Cloud 성능 레시피는 Slurm 기반 환경을 요구
- **Azure CycleCloud Workspace for Slurm(CCWS)** — Azure Marketplace 솔루션 템플릿으로 가상 네트워크 생성·노드 풀 정의·파일시스템 통합·Slurm 스케줄러 구성 등 인프라 셋업을 자동화
- 결과 환경에는 컨테이너 워크플로용 **PMIx v4, Pyxis, Enroot** 사전 구성
- Azure HPC Ubuntu-HPC 22.04 이미지 사용 (GPU·네트워킹 드라이버 사전 설치)

**SW 전제조건(참고):** SLURM 22.x+, CUDA 12.3+, NVIDIA 드라이버 535.129.03+, OFED 5.9+, NCCL 2.19.4+, Enroot, NGC 레지스트리 접근

---

## L4. 소프트웨어 스택 — NVIDIA가 관리

순수 IaaS와 DGX Cloud를 가르는 핵심 차별점입니다.

| 컴포넌트 | 역할 |
|---|---|
| **NVIDIA Base Command Platform** | AI 워크로드 구성·관리, 통합 데이터셋 관리, 단일 GPU~대규모 멀티노드까지 적정 자원에 잡 실행 |
| **NVIDIA AI Enterprise** | 가속 데이터 사이언스 라이브러리·최적화 프레임워크·사전학습 모델. Azure Machine Learning과 MLOps 통합 |
| **NVIDIA Run:ai** | 하이브리드 AI 인프라 전반의 워크로드 스케일링·자원 관리 자동화로 GPU 활용도 극대화 |
| **NVIDIA Cloud Functions (NVCF)** | 즉각적 확장성·비용 효율적 GPU 활용의 서버리스 추론 서비스 |

---

## L5. 검증된 성능 & 소비 모델

**성능 검증**
Azure에서 8개 → 1024개 NDv5 H100 GPU까지 확장한 벤치마크에서, LLM 학습 성능이 NVIDIA 공개 벤치마크와 동등한 수준으로 나타나 Azure 인프라의 확장성과 효율성을 입증.

**구매 방식**
Azure Marketplace를 통한 NVIDIA private offer로 계약 → Azure 소비 약정(MACC)에 카운트 가능 (영업 레버리지)

**포함된 지원**
모든 구독에 전담 NVIDIA TAM과 24/7 비즈니스 크리티컬 지원 포함

---

## 정리: "왜 IaaS가 아니라 DGX Cloud인가"

| 구분 | 일반 GPU VM (IaaS) | DGX Cloud on Azure |
|---|---|---|
| 컴퓨트 | ND H100 v5 직접 구성 | 동일 ND H100 v5 |
| 클러스터 구축 | 고객이 직접 | CCWS로 자동화 |
| SW 스택 | 직접 설치 | Base Command + AI Enterprise 번들 |
| 운영/지원 | 고객 책임 | NVIDIA 관리형 + TAM |

**한 줄 요약:** DGX Cloud는 "Azure ND H100 v5 인프라 + NVIDIA InfiniBand 패브릭 + NVIDIA 풀스택 SW + NVIDIA 운영"을 Marketplace 한 건으로 묶은 관리형 AI 팩토리입니다.

---

### 참고 출처
- Microsoft Learn — ND-H100-v5 size series
- NVIDIA Blog — H100 Tensor Core GPU on Microsoft Azure ND H100 v5
- Argon Systems — DGX Cloud Benchmarking on Azure (8~1024 H100)
- Azure Marketplace — NVIDIA DGX Cloud
- NVIDIA DGX Cloud Datasheet
