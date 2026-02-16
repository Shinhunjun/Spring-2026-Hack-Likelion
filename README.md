# Spring 2026 PhysioNet Challenge - LikeLion Team

> **PhysioNet Challenge 2026**: 수면 검사(PSG) 데이터를 활용한 인지 장애 스크리닝

---

## 챌린지 개요

[PhysioNet Challenge 2026](https://moody-challenge.physionet.org/2026/)은 **수면다원검사(Polysomnography, PSG)** 데이터를 기반으로 향후 **인지 장애(MCI, 알츠하이머, 치매)** 발생 여부를 예측하는 과제입니다.

### 핵심 정보

| 항목 | 내용 |
|------|------|
| **목표** | PSG 기록으로부터 향후 인지 장애 발병 예측 |
| **데이터** | ~100GB EDF 파일 (EEG, EOG, EMG, ECG, 호흡 신호) |
| **학습 데이터** | 780개 녹화 (3개 사이트) + CAISR 주석 + 수동 주석 |
| **검증/테스트** | 숨겨진 사이트의 데이터 (일반화 성능 평가) |
| **평가 지표** | **AUROC** - Group 1 vs Group 2 |
| **제출 형식** | GitHub을 통한 Python 코드 제출 |

### 예측 그룹 정의

- **Group 1 (양성)**: PSG 검사 후 3~7년 이내에 인지 장애 진단을 받은 환자
- **Group 2 (음성)**: 7년 이상 추적 관찰 후에도 인지 장애 진단이 없는 환자
- **Group 3 (제외)**: 그 외 환자 (3년 미만 진단, 추적 기간 부족 등)

---

## 협업 방식

### 원칙: 전원 참여, 함께 성장

모든 팀원이 **모든 분야에 참여**합니다. 역할을 고정하지 않고, 매주 공통 과제를 조사/구현하고 Git에 push하여 기여합니다.

### 주간 사이클

```
월요일     →  주간 과제 확인 & 방향 논의
화~수요일  →  각자 조사/구현 후 개인 브랜치에 push
목요일     →  미팅: 조사 내용 공유 & 리뷰 & 다음 주 계획
금~일요일  →  피드백 반영 & 추가 작업
```

### 기여 방식

각 팀원은 매주 자신의 **개인 폴더**에 조사/코드를 정리해서 push합니다.

```
weekly/
├── week01-research/          # 주차별 폴더
│   ├── member1/              # 팀원별 폴더
│   │   └── research.md       # 조사 내용
│   ├── member2/
│   │   └── research.md
│   ├── member3/
│   │   └── research.md
│   ├── member4/
│   │   └── research.md
│   └── member5/
│       └── research.md
├── week02-data/
│   ├── member1/
│   ...
```

---

## 타임라인

총 **9주** 계획 (2026.02.16 ~ 2026.04.15)

---

### Week 1 (2/16 ~ 2/22): 챌린지 이해 + 리서치

> **마감: 목요일 (2/19) 미팅 전까지 push**

#### 전원 공통 과제

각자 아래 4가지를 조사해서 `weekly/week01-research/본인이름/research.md`에 정리 후 push

| # | 조사 항목 | 설명 |
|---|----------|------|
| 1 | **챌린지 이해** | 이 대회가 정확히 뭘 하는 건지, 규칙, 평가 방식, 제출 형식 정리 |
| 2 | **데이터셋 분석** | PSG 데이터란 무엇인지, EDF 파일 포맷, 포함된 신호(EEG/EOG/EMG/ECG) 각각의 의미 |
| 3 | **관련 연구 조사** | 수면 데이터로 인지 장애를 예측한 기존 논문/연구 1~2개 찾아서 요약 |
| 4 | **방법론 조사** | 찾은 연구에서 어떤 방법(ML/DL)을 썼는지, 어떤 피처를 추출했는지 정리 |

#### research.md 템플릿

```markdown
# Week 1 리서치 - [이름]

## 1. 챌린지 이해
- 대회 목표:
- 평가 지표:
- 제출 방식:
- 주요 규칙:

## 2. 데이터셋 분석
- PSG(수면다원검사)란:
- EDF 파일 포맷:
- 포함된 신호 종류와 의미:
  - EEG:
  - EOG:
  - EMG:
  - ECG:
  - 호흡 신호:
- CAISR 주석이란:

## 3. 관련 연구 (1~2편)
### 논문 1: [제목]
- 저자/연도:
- 목표:
- 데이터:
- 방법:
- 결과:

### 논문 2: [제목]
- 저자/연도:
- 목표:
- 데이터:
- 방법:
- 결과:

## 4. 방법론 정리
- 사용된 모델:
- 주요 피처:
- 우리 프로젝트에 적용 가능한 아이디어:
```

#### 리서치 팁

논문 검색:
- [Google Scholar](https://scholar.google.com/) - `"sleep" "cognitive impairment" "polysomnography" prediction`
- [PubMed](https://pubmed.ncbi.nlm.nih.gov/) - 의학 논문 검색
- [PhysioNet 이전 챌린지](https://physionet.org/about/challenge/) - 과거 우승 솔루션 참고

키워드:
- `sleep EEG cognitive decline prediction`
- `polysomnography dementia screening`
- `sleep architecture Alzheimer's biomarker`
- `PSG machine learning MCI`

---

### Week 2 (2/23 ~ 3/1): 데이터 탐색 + 환경 세팅

> **전원**: 직접 데이터를 만져보기

- [ ] Python 환경 세팅 (requirements.txt)
- [ ] 샘플 EDF 파일 로딩해보기 (MNE 또는 PyEDFlib)
- [ ] 신호 시각화 (EEG, ECG 등 각 채널 plot)
- [ ] CAISR 주석 파일 구조 파악
- [ ] 메타데이터 (나이, 성별, BMI 등) 분포 확인
- [ ] GCP 환경 구성

각자 `weekly/week02-data/본인이름/` 에 EDA 노트북 push

---

### Week 3-4 (3/2 ~ 3/15): 전처리 + Feature Engineering

- [ ] 신호 전처리 파이프라인 (노이즈 제거, 필터링)
- [ ] 시간 도메인 피처 추출 (통계량, entropy)
- [ ] 주파수 도메인 피처 추출 (PSD, band power)
- [ ] 수면 구조 피처 (수면 효율, 단계 전환, REM 특성)
- [ ] 심박 변이도(HRV) 피처 (ECG 기반)
- [ ] Baseline 모델 (Logistic Regression, Random Forest)
- [ ] 초기 AUROC 측정

---

### Week 5-6 (3/16 ~ 3/29): 모델 개선 + 실험

- [ ] XGBoost / LightGBM 튜닝
- [ ] Deep Learning 실험 (CNN, LSTM, Transformer)
- [ ] 피처 중요도 분석 및 선택
- [ ] 사이트 간 일반화 분석
- [ ] 실험 추적 (W&B)

---

### Week 7 (3/30 ~ 4/5): 앙상블 + 최적화

- [ ] 최종 모델 조합 (Stacking / Blending)
- [ ] 제출 코드 패키징 (PhysioNet 형식)
- [ ] 전체 학습 데이터로 최종 학습

---

### Week 8 (4/6 ~ 4/9): 최종 제출

- [ ] 제출 코드 검증
- [ ] **4/9 Unofficial Phase 제출 완료**

---

### Week 8-9 (4/10 ~ 4/15): Abstract 작성

- [ ] 방법론 + 결과 정리
- [ ] CinC Abstract 초안 (전원 참여)
- [ ] **4/15 Abstract 제출**

---

## 프로젝트 구조

```
Spring-2026-Hack-Likelion/
├── README.md
├── requirements.txt
├── .gitignore
│
├── weekly/                       # 주차별 팀원 기여
│   ├── week01-research/          # Week 1: 리서치
│   │   ├── member1/
│   │   ├── member2/
│   │   ├── member3/
│   │   ├── member4/
│   │   └── member5/
│   ├── week02-data/              # Week 2: 데이터 탐색
│   └── ...
│
├── data/                         # 데이터 (.gitignore)
│   ├── raw/
│   ├── processed/
│   └── features/
│
├── src/                          # 공용 소스 코드
│   ├── data/                     # 데이터 로딩/전처리
│   ├── features/                 # 피처 엔지니어링
│   ├── models/                   # 모델 정의
│   ├── training/                 # 학습 파이프라인
│   └── utils/                    # 유틸리티
│
├── notebooks/                    # 공용 노트북
│   ├── eda/
│   └── experiments/
│
├── configs/                      # 실험 설정
├── experiments/                  # 실험 결과 로그
├── submission/                   # 제출용 코드
└── docs/                         # Abstract 등 문서
```

---

## Git 워크플로우

### 브랜치 전략

```
main              ← 안정 브랜치 (제출용)
  └── dev         ← 개발 브랜치
       ├── [이름]/week01    ← 개인 주차별 브랜치
       ├── [이름]/week02
       └── feature/xxx      ← 공용 기능 브랜치
```

### 작업 흐름 (매주)

```bash
# 1. dev에서 개인 브랜치 생성
git checkout dev
git pull origin dev
git checkout -b 본인이름/week01

# 2. 작업 후 push
git add weekly/week01-research/본인이름/
git commit -m "docs: Week 1 리서치 - 본인이름"
git push origin 본인이름/week01

# 3. GitHub에서 PR 생성 (본인이름/week01 → dev)
```

### 커밋 메시지 규칙

```
<type>: <description>

타입:
docs:    조사/문서 작업
feat:    새로운 기능/코드
fix:     버그 수정
data:    데이터 처리 관련
model:   모델 관련
exp:     실험 결과
```

---

## 기술 스택

| 분류 | 도구 | 용도 |
|------|------|------|
| **신호 처리** | MNE, PyEDFlib, SciPy | PSG 신호 처리/분석 |
| **ML** | scikit-learn, XGBoost, LightGBM | 전통 ML 모델 |
| **DL** | PyTorch | 딥러닝 모델 |
| **실험 관리** | Weights & Biases | 실험 추적 |
| **데이터** | NumPy, Pandas | 데이터 처리 |
| **시각화** | Matplotlib, Seaborn | 그래프 |
| **클라우드** | GCP (Compute Engine + Storage) | GPU 학습 |

---

## GCP 예산 계획 ($300)

| 항목 | 예상 비용 | 비고 |
|------|----------|------|
| Cloud Storage (100GB) | ~$4 | 2개월 |
| Compute Engine (T4 Spot) | ~$100-150 | 실제 학습 시에만 |
| 기타 | ~$10-20 | 네트워크 등 |
| **합계** | **~$150-200** | 여유 있음 |

---

## 시작하기

```bash
# 1. 레포 클론
git clone https://github.com/Shinhunjun/Spring-2026-Hack-Likelion.git
cd Spring-2026-Hack-Likelion

# 2. Python 환경 (conda)
conda create -n physionet2026 python=3.10
conda activate physionet2026
pip install -r requirements.txt

# 3. dev 브랜치로 이동
git checkout dev
```

---

## 주요 참고 자료

### 챌린지 공식
- [PhysioNet Challenge 2026](https://moody-challenge.physionet.org/2026/)
- [Kaggle 데이터셋](https://www.kaggle.com/datasets/physionet/physionetchallenge2026data)
- [Python 예제 코드](https://github.com/physionetchallenges/python-example-2026)

### 도메인 지식
- [수면다원검사(PSG)](https://en.wikipedia.org/wiki/Polysomnography)
- [EEG 주파수 밴드](https://en.wikipedia.org/wiki/Electroencephalography#Normal_activity)
- [AASM 수면 단계 분류](https://aasm.org/)

### 논문 검색
- [Google Scholar](https://scholar.google.com/)
- [PubMed](https://pubmed.ncbi.nlm.nih.gov/)

---

*LikeLion Coding Club - Spring 2026*
