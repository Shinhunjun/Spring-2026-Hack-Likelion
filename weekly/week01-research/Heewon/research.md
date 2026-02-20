# Week 1 리서치 - 정희원

## 1. 챌린지 이해
- 대회 목표: PSG 데이터를 이용해 cognitive impairment 예측 알고리즘 개발하기 
- 평가 지표: hidden test set에서 가장 높은 점수 받기 
- 제출 방식:팀 등록 + abstract to CinC + 알고리즘 (open source, training code) + 4 page paper to CinC 
- 주요 규칙: abstract 통과될 시 poster/oral로 발표 (poster도 in-person 참여 요구), test 데이터 일부 정보 무 (generalization / 실제 상황 반영)

## 2. 데이터셋 분석
- PSG(수면다원검사)란: 자는데 기록되는 physiological signal / 불면증 진단 등에 사용 
- EDF 파일 포맷:
- 포함된 신호 종류와 의미:
  - EEG: electrical activity of brain / detects alertness, light sleep, deep sleep (different states of the brain) / used to determine brain activity 
  - EOG: eye movement / electrical signal of eye / used to get periods of rapid eye movement (REM) and identify transition between brain states
  - EMG: electrical activity of muscles (chin, lower limb) / used to observe abnormal motor activity & transition between sleep stages
  - ECG: electrical activity of heart / heart rate / reflects autonomic nervous system and cardiovascular stress or dysregulation
  - 호흡 신호: 코와 입을 오가는 공기흐름 측정 / 정상 호흡 판별
- CAISR 주석이란: complete artificial intelligence sleep report; PSG 기록과 분석 자동화 시스템 

## 3. 관련 연구 (1~2편)
### 논문 1: Alzheimer's Disease Detection in EEG Sleep Signals
https://ieeexplore.ieee.org/document/10714019 
- 저자/연도:
- 목표: 수면 EEG 신호 사용해 alzheimer 조기 탐지 딥러닝 방법 개발 / 수면장애와 인지저하의 상관관계를 이용해 증상 이전 단계에서 탐지할 수 있게 
- 데이터: 알츠하이머 vs 건강한 사람들 (대조군)의 EEG data를 비교 / overnight PSG data / age, gender, total sleep, sleep efficiency 등등이 variable이 됨
- 방법:
1. Standardization of hynograms / sleep stages 
2. Remove unncessary artifact annotation (= process of identifying, labeling, marking extraneous electrical signal)
3. Butterworth bandpass filter to smooth passband response and minimize phase distortion 
4. normalization (zero mean and standard dev. of one) and segmentation: segmenting signals so it can be divided by sleep stages (manual work) / reduce variability and enhance stability
5. signal resampling (128hz) and segmentation: to reduce data complexity and 10s segment without overlapping for comparative analysis / create continuity in signal 
6. EEG classification with deep learning: spatial (뇌 영역 간) and temporal (시간 변화) characteristic / alzheimer 관련 패턴 발견하고 대조군과의 차이를 구분하는 분석 역할
- 결과: XCM (best) > SMATE > TapNet > HMM 

### 논문 2: [제목]
- 저자/연도:
- 목표:
- 데이터:
- 방법:
- 결과:

## 4. 방법론 정리
- 사용된 모델: 
1. SMATE - semi supervised model 
- "autoencoder framework to compress MTS data into a condensed embedding space"
- EEG 노이즈 등 불필요한 정보 제거 
- 뇌 영역 간 관계 학습 / 시간별 (뇌 활동) sleep dynamic같은 패턴 학습 
- label이 된 데이터와 안 된 데이터를 동시에 사용해서 훈련
- 3 step regularization (labeled data로 centroid 초기화 / 조정 + unlabeled로 centroid 추가 조정) * a machine learning approach that represents classes, clusters, or data distributions using a single central, representative point (the mean vector or "centroid") in a high-dimensional feature space
2. TapNet - semi supervised model 
- prototype 기반 classification 
- 대표가 되는 prototype 기준으로 sample을 분류 
- LSTM: designed to learn long-term dependencies in sequential data, overcoming the vanishing gradient problem
- good for sparse training labels 
3. XCM model - supervised model 
- parallel information from observed variables and time dimensions 
- 1d & 2d convolutional filters & simplify classification ("employing 1D average pooling before prediction")
4. Hidden Markov Models - unsupervised statistical models 
- learning without labeling 
- "generating labels by minimizing the distances to prototype centroids"
- 주요 피처:
- 우리 프로젝트에 적용 가능한 아이디어: