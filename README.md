# Sleep Apnea Detection on FPGA using Hybrid 1D-CNN

> Biosignal-based sleep apnea detection using EEG signals â€” 1D Convolutional Neural Network trained in MATLAB, implemented in Verilog HDL on Xilinx Zynq-7000 FPGA (ZedBoard) for real-time edge-intelligent healthcare.

**Final Year B.E. Project | Electronics and Communication Engineering**  
**College of Engineering Guindy, Anna University, Chennai â€” November 2025**

**Team:**
- Harini S (2022105006)
- Umesh Kanna K B (2022105010)
- Dheepikha V (2022105507)
- Tarini R (2022105533)

**Supervisor:** Dr. S. Kirubaveni, Associate Professor, ECE Dept., CEG Anna University

---

## Abstract

Sleep apnea is a critical health condition characterized by repeated breathing interruptions during sleep, potentially leading to cardiovascular disease, diabetes, and cognitive impairment. This project develops an intelligent edge-deployable system that detects sleep apnea events by analyzing EEG biosignals using a 1D Convolutional Neural Network (1D-CNN). The trained model is quantized and implemented in Verilog HDL on a Xilinx Zynq-7000 FPGA, enabling real-time, low-power, and low-latency inference without cloud dependency.

---

## System Overview

```
EEG Signal (EDF)
      |
      v
[MATLAB Preprocessing]
  - Artifact removal (Notch filter @ 50 Hz)
  - Bandpass filter (0.5-45 Hz)
  - Subband extraction (Delta/Theta/Alpha/Beta/Gamma)
  - Segmentation: 11s windows, 10s overlap, 128 Hz -> 2816 samples/segment
  - Z-score normalization
  - Class imbalance handling: Random Oversampling + SMOTE
      |
      v
[1D-CNN Training (MATLAB/Python)]
  - 15-layer architecture
  - Binary classification: Apnea vs Normal
  - 65,185 trainable parameters
  - Adam optimizer, lr=0.001, batch=64
  - Accuracy: 99.62% (SMOTE)
      |
      v
[Weight Extraction]
  - Fixed-point quantization: 16-bit (1 sign + 2 integer + 13 fraction)
  - Exported as .mif / .mem files for BRAM initialization
      |
      v
[Verilog Hardware Implementation on Zynq-7000 ZedBoard]
  - Feed-forward fully connected network: 30-30-10-10 neurons
  - Modules: neuron, weight_memory, ReLU, Sig_ROM, layers, findMaxOut, AXI-Lite wrapper
  - AXI-Lite interface for PS-PL communication
  - Overall hardware accuracy: 89%
  - Total power: 0.368 W
```

---

## Key Results

| Metric | Random Oversampling | SMOTE |
|--------|:-------------------:|:-----:|
| Accuracy | 99.53% | 99.62% |
| Balanced Accuracy | 91.23% | 91.58% |
| Sensitivity | 82.61% | 83.33% |
| Specificity | 99.85% | 99.84% |
| Precision | 91.57% | 86.96% |
| F1 Score | 0.8686 | 0.8511 |

**FPGA Hardware Results (Zynq-7000 ZedBoard):**

| Metric | Neuron Module | Full FC Network |
|--------|:-------------:|:---------------:|
| WNS (Timing Slack) | 4.571 ns | 4.156 ns |
| Total Power | 0.157 W | 0.368 W |
| LUT Utilization | 51 / 53200 (0.10%) | 6290 / 53200 (11.82%) |
| FF Utilization | 43 / 106400 (0.04%) | 6653 / 106400 (6.25%) |
| DSP Utilization | 2 / 220 (0.91%) | 160 / 220 (72.73%) |
| BRAM | 0.50 / 140 (0.36%) | 15 / 140 (10.71%) |
| Overall Simulation Accuracy | â€” | **89%** |

---

## Repository Structure

```
Sleep-Apnea-Detection-FPGA-CNN/
|
+-- README.md                        <- This file
+-- .gitignore                       <- Ignore MATLAB/Vivado generated artifacts
+-- LICENSE                          <- MIT License
|
+-- docs/
|   +-- project_report.pdf           <- Full B.E. project report
|   +-- architecture_overview.md     <- System architecture description
|
+-- matlab_preprocessing/
|   +-- README.md                    <- MATLAB pipeline documentation
|   +-- preprocess_eeg.m             <- EEG artifact removal and bandpass filtering
|   +-- extract_channels.m           <- C3-A2 and C4-A1 channel extraction
|   +-- segment_eeg.m                <- Sliding window segmentation (2816 samples)
|   +-- normalize_zscore.m           <- Z-score normalization per segment
|   +-- handle_imbalance.m           <- Random oversampling + SMOTE implementation
|   +-- train_1dcnn.m                <- 1D-CNN training script
|   +-- extract_weights.m            <- Weight/bias extraction and MIF generation
|
+-- dataset/
|   +-- README.md                    <- Dataset description and download links
|
+-- hardware/
|   +-- README.md                    <- Hardware implementation documentation
|   +-- rtl/
|   |   +-- neuron.v                 <- Neuron PE: MAC + activation
|   |   +-- weight_memory.v          <- Synchronous weight RAM (MIF initialized)
|   |   +-- relu.v                   <- ReLU activation module
|   |   +-- sig_rom.v                <- Sigmoid activation via ROM lookup
|   |   +-- layer1.v                 <- Hidden layer 1 (30 neurons)
|   |   +-- layer2.v                 <- Hidden layer 2 (30 neurons)
|   |   +-- layer3.v                 <- Hidden layer 3 (10 neurons)
|   |   +-- layer4.v                 <- Hidden layer 4 (10 neurons)
|   |   +-- find_max_out.v           <- Argmax: final class decision
|   |   +-- axi_lite_wrapper.v       <- AXI-Lite PS-PL interface
|   |   +-- zynet_top.v              <- Top module integrating all submodules
|   +-- tb/
|   |   +-- tb_neuron.v              <- Neuron testbench
|   |   +-- tb_top_sim.v             <- Full network testbench
|   +-- constraints/
|   |   +-- zedboard.xdc             <- ZedBoard pin constraints
|   +-- mif/
|       +-- README.md                <- MIF file format description
|       +-- w_1_15.mif               <- Layer 1 weights (sample)
|       +-- b_1_15.mif               <- Layer 1 biases (sample)
|
+-- results/
    +-- README.md                    <- Results summary
    +-- confusion_matrix_ros.png     <- Confusion matrix - Random Oversampling
    +-- confusion_matrix_smote.png   <- Confusion matrix - SMOTE
    +-- eeg_raw_vs_filtered.png      <- Raw vs filtered EEG (C3A2 & C4A1)
    +-- eeg_subbands.png             <- EEG subband decomposition
    +-- eeg_spectrogram.png          <- Spectrogram of filtered subbands
    +-- training_ros.png             <- CNN training curves - Random Oversampling
    +-- training_smote.png           <- CNN training curves - SMOTE
    +-- sim_relu_neuron.png          <- ReLU neuron simulation waveform
    +-- sim_sigmoid_neuron.png       <- Sigmoid neuron simulation waveform
    +-- sim_fcn_waveform.png         <- Full FC network simulation waveform
    +-- timing_summary.png           <- Design timing summary
    +-- power_report.png             <- FPGA power report
    +-- resource_utilization.png     <- LUT/FF/DSP/BRAM utilization
```

---

## 1D-CNN Architecture

| Layer | Type | Filter/Units | Output Dim | Activation |
|-------|------|-------------|-----------|-----------|
| 1 | Input | â€” | 2816 x 1 x 1 | â€” |
| 2 | Batch Normalization | â€” | 2816 x 1 x 1 | â€” |
| 3 | Conv1D | [100x1], 3 filters, stride 2 | 1359 x 1 x 3 | â€” |
| 4 | ReLU | â€” | 1359 x 1 x 3 | ReLU |
| 5 | MaxPool | [2x1], stride 2 | 679 x 1 x 3 | â€” |
| 6 | Conv1D | [10x1], 50 filters, stride 1 | 670 x 1 x 50 | â€” |
| 7 | MaxPool | [2x1], stride 2 | 335 x 1 x 50 | â€” |
| 8 | ReLU | â€” | 335 x 1 x 50 | ReLU |
| 9 | Conv1D | [30x1], 30 filters, stride 1 | 306 x 1 x 30 | â€” |
| 10 | MaxPool | [2x1], stride 2 | 305 x 1 x 30 | â€” |
| 11 | ReLU | â€” | 305 x 1 x 30 | ReLU |
| 12 | Batch Normalization | â€” | 305 x 1 x 30 | â€” |
| 13 | Fully Connected | 2 neurons | 2 x 1 | â€” |
| 14 | Softmax | â€” | 2 x 1 | Softmax |
| 15 | Classification | â€” | â€” | Cross-entropy loss |

**Total trainable parameters: 65,185**  
**Optimizer:** Adam | **Learning rate:** 0.001 | **Batch size:** 64

---

## Hardware Architecture (Verilog)

The FPGA implementation mirrors the fully connected layers of the trained CNN as a pipelined feed-forward network:

```
Input (784 samples)
       |
  [Layer 1 - 30 neurons] -- Weight_Memory -- MAC -- ReLU/Sigmoid
       |
  [Layer 2 - 30 neurons] -- Weight_Memory -- MAC -- ReLU/Sigmoid
       |
  [Layer 3 - 10 neurons] -- Weight_Memory -- MAC -- ReLU/Sigmoid
       |
  [Layer 4 - 10 neurons] -- Weight_Memory -- MAC -- ReLU/Sigmoid
       |
  [findMaxOut] --> Class: 0 (Normal) or 1 (Apnea)
       |
  [AXI-Lite Wrapper] <--> Zynq PS (ARM)
```

**Key modules:**

| Module | Description |
|--------|-------------|
| `neuron.v` | MAC unit: computes wx+b with 16-bit fixed-point arithmetic, applies activation |
| `weight_memory.v` | Synchronous ROM, initialized from `.mif` file via `$readmemb` |
| `relu.v` | ReLU: passes positive values, clips negatives; includes saturation logic |
| `sig_rom.v` | Sigmoid via pre-computed ROM lookup table (10-bit input index) |
| `layer1-4.v` | Groups parallel neuron instances per layer; FSM-controlled serialization |
| `find_max_out.v` | Argmax comparator tree; determines final predicted class |
| `axi_lite_wrapper.v` | AXI-Lite interface for weight loading, reset, inference trigger from ARM PS |
| `zynet_top.v` | Top-level: integrates all submodules, controls dataflow and pipelining |

**Fixed-point format:** 16 bits â€” 1 sign bit + 2 integer bits + 13 fraction bits

---

## Dataset

**Sources:**
- [St. Vincent's University Hospital / UCD Sleep Apnea Database (PhysioNet)](https://physionet.org/content/ucddb/1.0.0/)
- Cleveland Family Study Sleep Dataset (private)

**EEG Channels used:** C3-A2 and C4-A1 (128 Hz sampling rate)

**Preprocessing pipeline:**
- Notch filter at 50 Hz (powerline artifact removal)
- Bandpass filter: 0.5â€“45 Hz
- Segmentation: 11s windows with 10s overlap â†’ 2816 samples per segment
- Labels: 0 = Normal, 1 = Apnea (from `timing_apnea.mat`)
- Split: 80% train / 10% validation / 10% test

**Class imbalance handling:**
- Random Oversampling (duplicates minority apnea samples)
- SMOTE (synthesizes new minority samples by k-NN interpolation)

---

## How to Run

### MATLAB Preprocessing and Training

```matlab
% 1. Preprocess EEG data
run('matlab_preprocessing/preprocess_eeg.m')

% 2. Train 1D-CNN
run('matlab_preprocessing/train_1dcnn.m')

% 3. Extract weights to MIF files
run('matlab_preprocessing/extract_weights.m')
```

### Verilog Simulation (Vivado)

```
1. Open Xilinx Vivado
2. Create Project -> Add all .v files from hardware/rtl/
3. Add hardware/tb/tb_top_sim.v as simulation source
4. Set hardware/mif/ as working directory (for $readmemb)
5. Run Behavioral Simulation
6. View waveforms in Vivado Simulator
```

### FPGA Synthesis and Programming

```
1. Add hardware/constraints/zedboard.xdc
2. Run Synthesis -> Implementation -> Generate Bitstream
3. Open Hardware Manager -> Program ZedBoard
4. Use AXI-Lite to send test data and read classification output
```

---

## Tools and Technologies

| Tool | Purpose |
|------|---------|
| MATLAB R2023b | EEG preprocessing, 1D-CNN training, weight extraction |
| Python / TensorFlow | Alternative CNN training pipeline |
| Verilog HDL | RTL hardware implementation |
| Xilinx Vivado 2023.1 | Synthesis, simulation, FPGA bitstream |
| Xilinx Zynq-7000 (ZedBoard) | Target FPGA platform |
| MATLAB Fixed-Point Designer | 16-bit quantization of weights and biases |

---

## EEG Frequency Bands

| Band | Frequency Range | Significance in Sleep |
|------|-----------------|-----------------------|
| Delta | 0.5-4 Hz | Deep slow-wave sleep; dominant during apnea-related arousals |
| Theta | 4-8 Hz | Light sleep, drowsiness |
| Alpha | 8-13 Hz | Relaxed wakefulness, eyes closed |
| Beta | 13-30 Hz | Active thinking, alertness |
| Gamma | 30-45 Hz | High cognitive processing |

---

## Future Work

1. **Hybrid CNN-LSTM:** Combine 1D-CNN with LSTM layers to capture longer temporal EEG dependencies
2. **Multi-modal input:** Integrate ECG, SpO2, and airflow signals alongside EEG for higher diagnostic accuracy
3. **Model compression:** Apply weight pruning and further quantization for wearable deployment
4. **On-device learning:** Adaptive FPGA framework for incremental patient-specific model updates
5. **Clinical validation:** Expand testing to larger, more diverse patient cohorts for regulatory compliance

---

## References

Key references from the literature review:

- Alam et al. (2024) - Energy-efficient FPGA CNN+LSTM for sleep apnea, 97.63% accuracy, 0.42W â€” *IEEE Access*
- Saha et al. (2024) - Stapneanet: stage-adaptive EEG apnea detection, 94.68% accuracy
- Hassan et al. (2023) - SABiNN binarized neural network on FPGA, 92.34% accuracy, 78% less energy
- Hassan et al. (2022) - Multi-sensor FPGA sleep apnea, 90.45% accuracy â€” *IEEE MeMeA*
- John et al. (2021) - 1D-CNN for IoT sleep apnea detection â€” *IEEE ISCAS*

Full reference list in `docs/project_report.pdf`.

---

## License

MIT License â€” see [LICENSE](LICENSE)

---

## Citation

If you use this work, please cite:

```
Harini S, Umesh Kanna K B, Dheepikha V, Tarini R.
"Biosignal-Based Sleep Apnea Detection on FPGA Using a Hybrid CNN Architecture
for Edge-Intelligent Healthcare."
B.E. Project Report, Dept. of ECE, College of Engineering Guindy,
Anna University, Chennai, November 2025.
Supervisor: Dr. S. Kirubaveni.
```
