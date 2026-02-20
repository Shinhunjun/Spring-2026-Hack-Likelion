# Week 1 Research - Yujin

## 1. Understanding the Challenge

- **Goal:** To develop automated, robust algorithms that can classify sleep stages and identify sleep-related events (like arousals or apnea) using hidden or "noisy" clinical data.
- **Evaluation Metrics:** Typically a hidden **Challenge Score** that weights the accuracy of sleep stage classification (F1-score/Kappa) alongside the detection of specific sleep events.
- **Submission Method:** You submit your trained model and inference code via a **Docker container** to the PhysioNet evaluation system (Cloud-based).
- **Key Rules:** \* Open-source code only.
- The algorithm must run within a specific time limit (usually 24 hours for the entire test set).
- Entries must be "clinically viable," meaning they shouldn't just overfit to one specific sensor type.

## 2. Dataset Analysis

- **What is PSG (Polysomnography)?** The "gold standard" sleep study. It’s a multi-parametric test that records biophysiological changes that occur during sleep.
- **EDF File Format:** _European Data Format_. It’s the industry standard for storing multichannel biological signals. It consists of a header (metadata) and a data record (the actual waveforms).
- **Signal Types and Meanings:**
- **EEG (Electroencephalogram):** Measures brain waves. Essential for distinguishing between REM, Light Sleep (N1, N2), and Deep Sleep (N3).
- **EOG (Electrooculogram):** Measures eye movements. Crucial for identifying **REM (Rapid Eye Movement)** sleep.
- **EMG (Electromyogram):** Measures muscle tension. Used to detect the "muscle atonia" (paralysis) that happens during REM.
- **ECG (Electrocardiogram):** Measures heart rate. Helps identify arousals or stress responses during sleep apnea events.
- **Respiratory Signals:** Airflow and effort sensors used to detect pauses in breathing (apnea).
- **What is CAISR Annotation?** These are the ground-truth labels provided by clinical experts. They categorize every 30-second window (epoch) of the data into specific sleep stages or event markers.

## 3. Related Research

### Paper 1: [TinySleepNet: An Efficient Deep Learning Model for Sleep Stage Scoring based on Raw Single-Channel EEG](https://pubmed.ncbi.nlm.nih.gov/33018069/)

- **Authors/Year:** Supratak et al. (2017)
- **Goal:** Automatic sleep stage scoring using raw single-channel EEG.
- **Methods:** Uses two CNNs with different filter sizes (to capture both fine-grained and coarse-grained features) followed by a Bidirectional LSTM to learn the transition rules between sleep stages.
- **Results:** Proved that deep learning can outperform human experts in consistency.

### Paper 2: [U-Sleep: resilient high-frequency sleep staging](https://pubmed.ncbi.nlm.nih.gov/33859353/)

- **Authors/Year:** Perslev et al. (2021)
- **Goal:** A fully convolutional neural network for segmentation of sleep stages.
- **Methods:** An Encoder-Decoder (U-Net style) architecture that can handle varying sampling rates and long sequences of data.
- **Results:** Highly scalable and currently one of the state-of-the-art benchmarks for PSG analysis.

## 4. Methodology Summary

- **Models to Consider:** **1D-CNNs** for feature extraction from raw waves, combined with **Transformers** (Attention mechanisms) to handle the temporal sequence of a full night's sleep.
- **Key Features:** \* _Frequency Domain:_ Power Spectral Density (PSD) in Delta (0.5-4Hz), Theta (4-8Hz), Alpha (8-13Hz), and Beta (13-30Hz) bands.
- _Time Domain:_ Statistical moments (skewness, kurtosis) and signal complexity (Hjorth parameters).

- **Ideas for our Project:**
- Data Augmentation: Adding Gaussian noise or slight time-shifting to make the model more robust to different hospital sensors.
- Multi-modal Fusion: Training the model to "weigh" the EEG more heavily for stages, but rely on EMG/EOG specifically for REM detection.
