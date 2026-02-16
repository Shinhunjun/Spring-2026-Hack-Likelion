# Week 1 리서치 - 신현준

## 1. 챌린지 이해

### 대회 목표
- **수면다원검사(PSG) 데이터**를 분석하여 향후 **인지 장애(MCI, 알츠하이머, 치매)** 발병 여부를 예측
- 수면 중 뇌파, 심박, 호흡 등의 미세한 변화가 인지 장애 발병 수 년 전에 나타난다는 가설에 기반

### 평가 지표
- **AUROC** (Area Under the Receiver Operating Characteristic Curve)
- Group 1 (양성) vs Group 2 (음성) 간의 분류 성능을 측정
- 추가로 **Challenge Score** 계산: 상위 5% 용량(capacity) 내에서의 True Positive Rate

### 환자 그룹 분류

| 그룹 | 정의 | 용도 |
|------|------|------|
| **Group 1 (양성)** | PSG 후 3~7년 내 인지 장애 진단 2회 이상 (최소 1주 간격) | 평가 대상 |
| **Group 2 (음성)** | 진단 없음 + 7년 이상 추적 관찰 (일부 사이트 6년) | 평가 대상 |
| **Group 3 (제외)** | 3년 미만 진단, 추적 부족 등 나머지 | 평가에서 제외 |

### 제출 방식
- **GitHub 레포지토리**에 Python 코드 제출
- 필수 파일: `train_model.py`, `run_model.py`, `helper_code.py`, `team_code.py`
- Docker 환경에서 실행됨 (Dockerfile 포함)
- 학습 코드 + 추론 코드 모두 제출해야 함

### 주요 규칙
- Unofficial Phase: 2/10 ~ 4/9 (1~5회 제출 가능)
- Official Phase: 5/11 ~ 8월말 (1~10회 제출 가능)
- Abstract 마감: 4/15
- 다른 팀과 코드/방법론 공유 금지
- 대회 중 블로그, preprint 등으로 방법론 공개 금지
- 오픈소스 코드만 랭킹에 포함 (BSD 3-Clause 기본)

---

## 2. 데이터셋 분석

### PSG(수면다원검사)란
- **Polysomnography**: 수면 중 다양한 생체 신호를 동시에 기록하는 검사
- 수면 장애 진단의 Gold Standard
- 보통 병원 수면검사실에서 하룻밤 동안 진행 (6~8시간)

### EDF 파일 포맷
- **European Data Format**: 생체 신호 저장을 위한 국제 표준 포맷
- 헤더 + 데이터 레코드로 구성
- 채널별 서로 다른 샘플링 레이트 지원 (EEG: 200~500Hz, ECG: 250~512Hz 등)
- Python에서 `edfio` 또는 `MNE` 라이브러리로 로딩

### 데이터 디렉토리 구조

```
training_set/
├── demographics.csv                              # 메타데이터 (나이, 성별, BMI 등)
├── physiological_data/                           # 원시 PSG 신호
│   ├── S0001/                                    # 사이트별 폴더 (572건)
│   │   └── sub-S0001XXXXXX-ses2.edf             # 환자별 EDF 파일
│   ├── I0002/                                    # (54건)
│   └── I0006/                                    # (154건)
├── algorithmic_annotations/                      # CAISR 자동 주석
│   └── [사이트]/[환자]_caisr_annotations.edf
└── human_annotations/                            # 전문가 수동 주석 (training만)
    └── [사이트]/[환자]_expert_annotations.edf
```

총 **780건** 학습 데이터 (3개 사이트), 검증/테스트는 숨겨진 다른 사이트

### 포함된 신호 종류와 의미

| 신호 | 약자 | 설명 | 주요 채널 |
|------|------|------|----------|
| **EEG** | 뇌파 | 대뇌 피질의 전기적 활동. 수면 단계 판별의 핵심 | F3-M2, F4-M1, C3-M2, C4-M1, O1-M2, O2-M1 |
| **EOG** | 안전도 | 안구 운동 기록. REM 수면 탐지 | E1-M2, E2-M1 (LOC, ROC) |
| **EMG** | 근전도 | 근육 긴장도. REM atonia 확인 | Chin1-Chin2, LAT, RAT (다리) |
| **ECG** | 심전도 | 심장 전기 활동. 심박 변이도(HRV) 분석 | ECG (단일 리드) |
| **호흡** | Resp | 호흡 노력 + 기류 | Airflow, PTAF(비강압력), Chest, Abd |
| **SpO2** | 산소포화도 | 혈중 산소 농도 | SpO2/SaO2 |

**참고**: 사이트별로 채널 이름과 샘플링 레이트가 다름 → `channel_table.csv`로 표준화 필요

### EEG 주파수 밴드와 의미

| 밴드 | 주파수 (Hz) | 관련 상태 | 인지 장애 관련성 |
|------|------------|----------|----------------|
| **Delta** | 0.5 ~ 4 | 깊은 수면 (N3) | 인지 장애 시 Delta 파워 감소 |
| **Theta** | 4 ~ 8 | 얕은 수면 (N1), 졸림 | Theta 증가 → 인지 저하 신호 |
| **Alpha** | 8 ~ 12 | 편안한 각성, 명상 | Alpha 감소 → 주의력 저하 |
| **Beta** | 12 ~ 30 | 활발한 사고, 집중 | Beta 변화 → 전두엽 기능 변화 |
| **Gamma** | 30+ | 고차 인지 처리 | Gamma 감소 → 기억 처리 저하 |

### CAISR 주석이란
- **Complete AI Sleep Report**: 자동 수면 분석 알고리즘
- AASM 가이드라인에 따라 다음을 자동 판별:

| 주석 종류 | 해상도 | 클래스 |
|----------|--------|--------|
| **수면 단계** | 30초 epoch | 1=N3, 2=N2, 3=N1, 4=REM, 5=Wake, 9=Unavailable |
| **각성 반응** | 0.5초 | 0=없음, 1=각성 (+ softmax 확률) |
| **호흡 이벤트** | 1초 | 0=없음, 1=폐쇄성 무호흡, 2=중추성, 3=혼합, 4=저호흡, 5=RERA |
| **사지 움직임** | 1초 | 0=없음, 1=고립 움직임, 2=주기적 사지 운동(PLM) |

- 학습/검증/테스트 모두에서 사용 가능 (human annotations은 training만)

### demographics.csv 필드

| 필드 | 설명 | 타입 |
|------|------|------|
| SiteID | 수집 기관 ID | 범주형 |
| BDSPPatientID | 환자 고유 ID | 문자열 |
| BidsFolder | 파일 경로 매칭용 폴더명 | 문자열 |
| SessionID | 수면검사 세션 번호 | 정수 |
| Age | 나이 (90세 이상은 "90") | 연속형 |
| Sex | 성별 | 범주형 |
| Race | 인종 (Asian, Black, White, Others, Unavailable) | 범주형 |
| Ethnicity | 민족 (Hispanic, Not Hispanic, Unavailable) | 범주형 |
| BMI | 체질량지수 | 연속형 |
| Time_to_Event | PSG → 인지 장애 진단까지 일수 | 연속형 (training만) |
| Cognitive_Impairment | TRUE/FALSE | 이진 (training만) |
| Time_to_Last_Visit | PSG → 마지막 방문까지 일수 | 연속형 (training만) |

---

## 3. 관련 연구 (3편)

### 논문 1: Sleep EEG-Based Approach to Detect Mild Cognitive Impairment

- **저자/연도**: Liguori et al., 2020 (Frontiers in Aging Neuroscience)
- **목표**: 수면 EEG를 이용하여 경도인지장애(MCI) 환자를 건강한 대조군과 구별
- **데이터**: 43명 MCI 환자 + 33명 건강 대조군의 PSG 기록
- **방법**:
  - 수면 구조 분석 (수면 효율, 각 단계 비율, REM latency)
  - EEG 주파수 분석 (Delta, Theta, Alpha, Beta power)
  - 수면 스핀들(spindle) 특성 분석
  - 통계적 비교 (t-test, ANOVA)
- **결과**:
  - MCI 환자에서 **N3 수면 감소**, **수면 효율 저하**
  - **Delta power 감소**, **Theta/Alpha ratio 증가**
  - 수면 스핀들 밀도와 진폭 감소
- **우리에게 주는 시사점**: Delta power, 수면 효율, 스핀들 특성이 유용한 피처가 될 수 있음

### 논문 2: Association of Sleep Architecture and Physiology with Cognitive Decline and Dementia

- **저자/연도**: Pase et al., 2023 (Sleep Medicine Reviews)
- **목표**: 대규모 수면 연구에서 수면 구조/생리가 인지 저하 및 치매와 어떤 연관이 있는지 종합 분석
- **데이터**: Framingham Heart Study 등 대규모 코호트 연구 메타분석
- **방법**:
  - 수면 구조 지표 (수면 단계 비율, 수면 분절)
  - 호흡 지표 (AHI, 산소포화도)
  - HRV (심박 변이도) 분석
  - Cox proportional hazards regression (생존 분석)
- **결과**:
  - **REM 수면 감소**가 치매 발병의 독립적 예측인자
  - 수면 분절(fragmentation) 증가 → 인지 저하 위험 증가
  - 수면무호흡(높은 AHI) → 인지 장애 위험 상승
  - 야간 저산소증(SpO2 저하) → 백질 변성 및 인지 저하
- **우리에게 주는 시사점**: REM% 감소, 수면 분절, AHI, SpO2 drop이 핵심 피처

---

### 논문 3: Morphometric Similarity Network-based Graph Convolutional Networks for Schizophrenia Classification

- **저자/연도**: Park & Lee, 2025 (Scientific Reports)
- **논문 링크**: https://www.nature.com/articles/s41598-025-19894-8
- **목표**: MRI 기반 형태학적 유사도 네트워크(MSN)와 GCN을 결합하여 조현병 환자를 분류
- **데이터**: 9개 사이트에서 수집, 조현병 366명 + 건강 대조군 590명 (총 956명)
- **방법**:
  - **개인 그래프 (Individual MSN)**: 각 환자의 MRI에서 뇌 영역 간 형태학적 유사도를 계산하여 개인별 그래프 생성
    - 피처: 피질 두께, 표면적, 회백질 부피, 평균 곡률, 가우시안 곡률
    - 영역 간 피처 벡터의 상관관계 → 엣지 가중치
  - **Population Graph 구축**: 모든 환자를 노드로, 환자 간 유사도를 엣지로 하는 그래프 생성
    - 노드 피처 = 개인 MSN의 토폴로지 특성 (degree, clustering coefficient, betweenness 등)
    - 엣지 가중치 = 토폴로지 피처 유사도 + 인구통계(나이, 성별 등) 정보
    - Adaptive edge optimization으로 엣지 가중치를 학습 중 동적 최적화
  - **GCN 분류**: Population graph 위에서 GCN을 통해 환자 vs 건강인 분류
    - Variational edges로 학습 과정 강화
- **결과**:
  - 분류 정확도 **81.8%** (9개 사이트 cross-validation)
  - 상측두이랑(superior temporal gyrus)이 조현병 판별에 가장 중요한 영역으로 확인
  - 기존 ML 방법 대비 유의미한 성능 향상
- **우리 프로젝트에 적용 가능한 아이디어**:

  이 논문의 핵심은 **"환자 간 유사도 그래프를 만들어서 GNN으로 분류"**하는 프레임워크. 원본은 MRI 기반이지만 **PSG 데이터에도 동일한 구조를 적용 가능**:

  | 원본 (MRI) | 우리 적용 (PSG) |
  |-----------|----------------|
  | 개인 MSN (뇌 영역 간 형태 유사도) | 개인 PSG 그래프 (EEG 채널 간 functional connectivity) |
  | 노드 피처: 토폴로지 특성 | 노드 피처: PSG에서 추출한 피처 벡터 (Hjorth, PSD, HRV, 수면구조 등) |
  | 엣지: 토폴로지 유사도 + 인구통계 | 엣지: PSG 피처 유사도 + 인구통계 (나이, 성별, BMI) |
  | GCN 분류 (조현병 vs 건강) | GCN 분류 (인지장애 vs 정상) |

  **장점**: 환자 간 관계를 활용하여 소수 샘플에서도 유사한 환자의 정보를 공유 → 클래스 불균형 문제에 도움이 될 수 있음. 또한 다중 사이트 데이터에서 사이트 정보를 엣지에 반영하면 domain adaptation 효과도 기대.

---

## 4. 방법론 정리

### PhysioNet 예제 코드에서 사용된 방법

예제 코드(`team_code.py`) 분석 결과:

**모델**: RandomForestClassifier (n_estimators=12, max_leaf_nodes=34)

**피처 구성** (총 71차원):
1. **인구통계 피처** (10차원): Age, Sex(3), Race(5), BMI
2. **생리 신호 피처** (49차원 = 7채널 x 7피처/채널):
   - 7개 신호 그룹: EEG, EOG, Chin EMG, Leg EMG, ECG, Resp, SpO2
   - 각 채널별 7개 시간 도메인 피처:
     - STD (표준편차)
     - MAV (평균 절대값)
     - ZCR (영교차율 - 주파수 프록시)
     - RMS (제곱평균제곱근)
     - Activity (분산 - Hjorth 1)
     - Mobility (이동성 - Hjorth 2, 평균 주파수 프록시)
     - Complexity (복잡성 - Hjorth 3, 대역폭 프록시)
3. **CAISR 주석 피처** (12차원):
   - AHI (무호흡-저호흡 지수), 각성 지수, 사지운동 지수
   - 수면 단계 비율 (W%, N1%, N2%, N3%, REM%, 수면 효율)
   - CAISR 확률값 (Wake prob, N3 prob, Arousal prob)

### 우리 프로젝트에 적용 가능한 아이디어

**기본 (예제 코드 기반)**:
- 인구통계 + Hjorth 파라미터 + CAISR 주석 기반 Random Forest → Baseline

**개선 방향**:

| 분야 | 아이디어 | 근거 |
|------|---------|------|
| **피처 추가** | 주파수 도메인 (PSD, Band Power) | Delta/Theta ratio가 인지 장애 바이오마커 |
| **피처 추가** | HRV 피처 (RMSSD, LF/HF ratio) | 자율신경계 변화가 인지 장애와 연관 |
| **피처 추가** | 수면 스핀들 탐지 및 특성 | MCI에서 스핀들 감소 확인됨 |
| **피처 추가** | 수면 분절 지수 (fragmentation) | 수면 분절 → 인지 저하 예측 |
| **피처 추가** | SpO2 야간 저하 통계 | 저산소증 → 인지 저하 |
| **모델** | XGBoost/LightGBM | Random Forest 대비 성능 향상 기대 |
| **모델** | 1D-CNN on raw EEG | End-to-End 학습으로 수동 피처 한계 극복 |
| **모델** | Transformer | 장기 시계열 패턴 포착 |
| **모델** | Population Graph + GCN | 환자 간 유사도 그래프 기반 분류 (Park 2025 방법론 적용) |
| **전략** | 사이트 간 Domain Adaptation | 검증/테스트가 다른 사이트 → 일반화 핵심 |
| **전략** | 앙상블 (Stacking) | ML + DL 모델 조합 |

### 핵심 도전과제
1. **사이트 간 차이**: 학습 3개 사이트 → 검증/테스트는 다른 사이트. 채널 이름, 샘플링 레이트, 장비 모두 다름
2. **클래스 불균형**: 인지 장애 발병은 드문 이벤트 → Group 1이 매우 적을 것
3. **대용량 데이터**: 100GB EDF → 효율적 전처리 파이프라인 필수
4. **결측 채널**: 모든 환자가 모든 채널을 가지고 있지 않음 → 결측 처리 전략 필요
