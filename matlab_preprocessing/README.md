# MATLAB Preprocessing Pipeline

This folder contains all MATLAB scripts for EEG signal preprocessing, CNN training, and weight extraction for FPGA deployment.

---

## Script Descriptions

| Script | Purpose |
|--------|---------|
| `preprocess_eeg.m` | Load EDF, extract C3-A2 and C4-A1 channels, apply notch + bandpass filter |
| `extract_channels.m` | Isolate specific EEG channels and save as .mat files |
| `segment_eeg.m` | Create overlapping 11s windows (2816 samples), generate per-window labels |
| `normalize_zscore.m` | Z-score normalization: X_norm = (X - mean) / std |
| `handle_imbalance.m` | Apply Random Oversampling or SMOTE to training set |
| `train_1dcnn.m` | Define and train 1D-CNN architecture, save model as global_model.mat |
| `extract_weights.m` | Load trained model, quantize weights to 16-bit fixed-point, save as .mif files |

---

## Prerequisites

- MATLAB R2023a or later
- Deep Learning Toolbox
- Fixed-Point Designer Toolbox
- Signal Processing Toolbox
- MATLAB add-on: `edfread` (for EDF file loading)

---

## Run Order

```matlab
% Step 1: Load and preprocess EEG
preprocess_eeg        % generates *_C3A2.mat, *_C4A1.mat, *_label.mat

% Step 2: Segment and partition
segment_eeg           % generates *_eeg_train.mat, *_eeg_valid.mat, *_eeg_test.mat

% Step 3: Handle class imbalance
handle_imbalance      % generates balanced training data

% Step 4: Train CNN
train_1dcnn           % generates global_model.mat

% Step 5: Extract weights for hardware
extract_weights       % generates .mif files in ../hardware/mif/
```

---

## 1D-CNN Architecture (MATLAB)

```matlab
layers = [
    sequenceInputLayer(2816)
    batchNormalizationLayer
    convolution1dLayer(100, 3, 'Stride', 2)
    reluLayer
    maxPooling1dLayer(2, 'Stride', 2)
    convolution1dLayer(10, 50, 'Stride', 1)
    maxPooling1dLayer(2, 'Stride', 2)
    reluLayer
    convolution1dLayer(30, 30, 'Stride', 1)
    maxPooling1dLayer(2, 'Stride', 2)
    reluLayer
    batchNormalizationLayer
    fullyConnectedLayer(2)
    softmaxLayer
    classificationLayer
];
```

**Training options:**
```matlab
options = trainingOptions('adam', ...
    'MaxEpochs', 10, ...
    'MiniBatchSize', 64, ...
    'InitialLearnRate', 0.001, ...
    'ValidationFrequency', 50, ...
    'Plots', 'training-progress');
```

---

## Weight Extraction for FPGA

```matlab
% Load trained model
load('global_model.mat', 'trainedNet');

for i = 1:numel(trainedNet.Layers)
    layer = trainedNet.Layers(i);
    if isprop(layer, 'Weights') && ~isempty(layer.Weights)
        % Quantize to 16-bit fixed-point (1 sign, 2 integer, 13 fraction)
        w_fp = fi(layer.Weights, 1, 16, 13);
        w_bin = bin(w_fp);
        % Write to .mif file
        writeMIF(w_bin, sprintf('../hardware/mif/%s_weights.mif', layer.Name));
    end
end
```

**Fixed-point format:** `fi(value, 1, 16, 13)` â€” signed, 16 bits total, 13 fraction bits

---

## EEG Filtering Parameters

| Filter | Type | Parameters |
|--------|------|-----------|
| Notch filter | IIR Notch | Center: 50 Hz, Q=35 (remove powerline noise) |
| Bandpass filter | Butterworth 4th order | 0.5 Hz - 45 Hz |

**EEG frequency subbands extracted:**
- Delta: 0.5-4 Hz
- Theta: 4-8 Hz
- Alpha: 8-13 Hz
- Beta: 13-30 Hz
- Gamma: 30-45 Hz
