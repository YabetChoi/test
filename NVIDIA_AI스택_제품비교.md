# NVIDIA AI 스택 — AI Enterprise · NIM · Omniverse · DGX Cloud 비교

> **핵심:** 네 가지는 서로 경쟁 제품이 아니라 **스택의 서로 다른 층**이다.
>
> - **DGX Cloud** = 인프라 (어디서 돌리나)
> - **NVIDIA AI Enterprise** = 소프트웨어 우산 (전체 라이선스 묶음)
> - **NIM** = 그 우산 안의 추론 부품
> - **Omniverse (+ Kit App Streaming)** = 별도 도메인의 애플리케이션 (3D/시뮬레이션)

---

## 1. NVIDIA AI Enterprise — "전체 소프트웨어 우산"

가장 상위 개념. NVIDIA가 GPU 하드웨어 위에 소프트웨어·서비스를 파는 전략의 **상업적 라이선스 프레임워크**로, AI 개발·배포에 필요한 프레임워크·라이브러리·툴을 묶은 엔터프라이즈 소프트웨어 스위트.

- 단일 제품이 아니라 **묶음(suite)** — NIM도, 다른 도구들도 이 안에 포함
- NIM은 NVIDIA AI Enterprise 스택을 구성하는 포트폴리오의 일부
- **영업 관점:** Azure Marketplace에서 라이선스로 판매. GPU 워크로드를 "엔터프라이즈 지원·보안 패치·검증된 버전"으로 쓰게 해주는 구독 계층

---

## 2. NVIDIA NIM — "추론(Inference) 전용 컨테이너 부품"

AI Enterprise 안의 구체적인 한 컴포넌트. 학습이 아니라 **배포(추론)**에 특화.

- AI 모델을 컨테이너화된 마이크로서비스로 패키징 → 모델 가중치 + 최적화 추론 엔진 + OpenAI 호환 REST API를 단일 Docker 컨테이너에 묶어 몇 분 만에 서비스 구동
- 컨테이너 구성: TensorRT-LLM 엔진 프로파일, vLLM/SGLang 런타임, OpenAI 호환 API 서버, CUDA 라이브러리, 엔터프라이즈 보안 레이어
- **성능:** H100에서 Llama 3.1 8B 기준 NIM 적용 시 미적용 대비 약 2.6배 처리량 (1,201 vs 613 tokens/s)
- **이식성:** DGX 시스템, DGX Cloud, NVIDIA Certified Systems, RTX 워크스테이션에서 동작

> 한 줄: "학습된 모델을 빠르게 프로덕션 API로 만드는 표준 추론 부품."

---

## 3. Omniverse (VDI / Kit App Streaming) — "3D·시뮬레이션 애플리케이션"

위 둘이 LLM·생성형 AI 영역이라면, Omniverse는 **물리 AI·산업 디지털화·3D 시뮬레이션** 도메인의 별도 플랫폼. AVOps에서 가장 중요한 부분.

- **Omniverse 자체** = Azure에서 호스팅되는 산업 디지털화 플랫폼. AV·로보틱스 개발자가 폐루프 가상 환경에서 물리 기반 센서 시뮬레이션 수행
- **Kit App Streaming** = Omniverse 앱을 사용자에게 전달하는 방식
  - OpenUSD 기반 애플리케이션을 NVIDIA RTX GPU로 저지연 렌더링해 브라우저/웹 앱에 스트리밍하는 API·확장 모음
  - 무거운 3D 앱을 설치 없이 **서버에서 렌더링 → 화면만 스트리밍** (전통적 VDI 개념의 GPU 가속 버전)
  - 배포 옵션: ① 자체 인프라/CSP 자체 관리 ② Azure Marketplace 솔루션 템플릿 원클릭 프로비저닝 ③ NVIDIA Omniverse on DGX Cloud 완전 관리형 인프라

> 한 줄: "AV 시뮬레이션·디지털 트윈을 만들고(Omniverse), 그걸 브라우저로 가볍게 전달하는(Kit App Streaming) 3D 스택."

---

## 4. DGX Cloud — "이 모든 걸 돌리는 관리형 인프라"

소프트웨어가 아니라 **인프라/컴퓨트 계층**.

- NVIDIA가 직접 관리하는 클라우드 네이티브 AI 플랫폼. 최신 GPU·소프트웨어·전문 인력을 묶어 모델 학습·배포 가속
- NIM도(DGX Cloud에서 실행), Omniverse도(Omniverse on DGX Cloud 완전 관리형) **DGX Cloud 위에서 구동 가능**

> 한 줄: "NIM·Omniverse·AI Enterprise를 올려놓고 돌리는 NVIDIA 관리형 AI 슈퍼컴퓨터."

---

## 한눈에 비교

| 제품 | 계층 | 무엇인가 | 핵심 용도 | 관계 |
|---|---|---|---|---|
| **AI Enterprise** | 소프트웨어 우산 | 라이선스 묶음(suite) | 전체 AI SW 스택 + 엔터프라이즈 지원 | NIM 등을 포함 |
| **NIM** | SW 부품 | 추론 마이크로서비스 컨테이너 | 모델 → 프로덕션 API 배포 | AI Enterprise의 일부 |
| **Omniverse / Kit App Streaming** | 애플리케이션 | 3D·시뮬레이션 플랫폼 + 스트리밍 전달 | AV 시뮬·디지털 트윈, 브라우저 전달 | DGX Cloud 위에서 구동 가능 |
| **DGX Cloud** | 인프라 | 관리형 GPU 컴퓨트 | 위 3가지를 돌리는 기반 | 나머지를 호스팅 |

---

## AVOps 영업 관점 정리

자율주행 개발 흐름에 매핑한 각 제품의 역할:

| 제품 | AVOps에서의 역할 |
|---|---|
| **DGX Cloud** | 학습·시뮬레이션을 돌리는 GPU 기반 (Azure ND H100 v5 위) |
| **AI Enterprise** | 그 위에서 쓰는 MLOps·프레임워크 라이선스 |
| **NIM** | 학습한 인지 모델을 추론 API로 배포 (차량/클라우드 서빙) |
| **Omniverse + Kit App Streaming** | 센서 시뮬레이션·디지털 트윈 제작 및 엔지니어에게 브라우저 전달 |

**공통 셀링 레버리지:** 네 가지 모두 Azure Marketplace를 통해 소비되므로 Azure 소비 약정(MACC)에 묶을 수 있다.

---

### 참고 출처
- NVIDIA — NIM Microservices for AI Inference
- Dell Technologies Info Hub — Introduction to NVIDIA Inference Microservices (NIM)
- NVIDIA Developer Blog — Deploying Your Omniverse Kit Apps at Scale
- NVIDIA Blog — Omniverse Cloud Services on Microsoft Azure
- Azure Marketplace — NVIDIA DGX Cloud
