# Results

This folder contains all simulation outputs, plots, and synthesis reports from the project.

---

## MATLAB Simulation Results

### EEG Signal Processing

| File | Description |
|------|-------------|
| `eeg_raw_vs_filtered.png` | Comparison of raw vs bandpass+notch filtered EEG (C3A2 & C4A1) |
| `eeg_fft_spectrum.png` | FFT spectrum of EEG channels showing dominant frequency bands |
| `eeg_subbands.png` | EEG subband decomposition â€” Delta, Theta, Alpha, Beta, Gamma |
| `eeg_spectrogram.png` | Time-frequency spectrogram of filtered EEG subbands |

### CNN Training Results

| File | Description |
|------|-------------|
| `training_ros.png` | Training accuracy/loss curves with Random Oversampling (99.53% val accuracy) |
| `training_smote.png` | Training accuracy/loss curves with SMOTE (99.62% val accuracy) |
| `confusion_matrix_ros.png` | Confusion matrix â€” Random Oversampling |
| `confusion_matrix_smote.png` | Confusion matrix â€” SMOTE |

### Performance Metrics

| Metric | Random Oversampling | SMOTE |
|--------|:-------------------:|:-----:|
| Accuracy | 99.53% | **99.62%** |
| Balanced Accuracy | 91.23% | **91.58%** |
| Sensitivity | 82.61% | **83.33%** |
| Specificity | **99.85%** | 99.84% |
| Precision | **91.57%** | 86.96% |
| F1 Score | **0.8686** | 0.8511 |

**Conclusion:** SMOTE provides better minority-class (apnea) detection; Random Oversampling gives slightly higher precision. Both exceed 99% overall accuracy.

---

## Verilog Simulation Results

| File | Description |
|------|-------------|
| `sim_relu_neuron.png` | Simulation waveform of ReLU-activated neuron |
| `sim_sigmoid_neuron.png` | Simulation waveform of Sigmoid-activated neuron |
| `sim_fcn_waveform.png` | Full FC network simulation waveform showing inference flow |
| `tcl_accuracy.png` | Tcl console output showing Overall Accuracy: 89% |

---

## FPGA Synthesis Reports

| File | Description |
|------|-------------|
| `timing_summary_neuron.png` | Design timing summary â€” single neuron (WNS: 4.571 ns) |
| `timing_summary_fcn.png` | Design timing summary â€” full FC network (WNS: 4.156 ns) |
| `power_report_neuron.png` | Power breakdown â€” neuron (0.157 W total) |
| `power_report_fcn.png` | Power breakdown â€” full network (0.368 W total) |
| `resource_utilization.png` | LUT/FF/DSP/BRAM utilization for both neuron and full network |

---

## Key Hardware Metrics

| Resource | Neuron Module | Full FC Network | Available |
|----------|:-------------:|:---------------:|:---------:|
| LUT | 51 (0.10%) | 6290 (11.82%) | 53,200 |
| FF | 43 (0.04%) | 6653 (6.25%) | 106,400 |
| DSP | 2 (0.91%) | 160 (72.73%) | 220 |
| BRAM | 0.50 (0.36%) | 15 (10.71%) | 140 |
| Total Power | 0.157 W | 0.368 W | â€” |
| Timing Slack | 4.571 ns | 4.156 ns | â€” |

All timing constraints are met (positive slack on all paths).

---

## Overall System Performance

| System | Accuracy |
|--------|---------|
| MATLAB 1D-CNN (Random Oversampling) | 99.53% |
| MATLAB 1D-CNN (SMOTE) | 99.62% |
| FPGA Hardware (Verilog simulation) | 89.00% |

The ~10% accuracy drop from software to hardware is due to fixed-point quantization effects (16-bit vs 32-bit float). Future work: increase quantization precision or use hardware-aware training.
