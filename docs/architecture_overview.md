# System Architecture Overview

## End-to-End Pipeline

```
+------------------+     +-------------------+     +------------------+
|   EEG Dataset    |---->| MATLAB Preprocessing|---->|  1D-CNN Training |
| (PhysioNet UCDDB)|     | (Filtering, Segment)|     | (MATLAB/Python)  |
+------------------+     +-------------------+     +------------------+
                                                            |
                                                            v
                                                  +-------------------+
                                                  | Weight Extraction |
                                                  | Fixed-point 16-bit|
                                                  | -> .mif / .mem    |
                                                  +-------------------+
                                                            |
                                                            v
+------------------+     +-------------------+     +------------------+
|   ZedBoard FPGA  |<----| Verilog RTL (zyNet)|<----|  Vivado Synthesis|
| Zynq-7000 SoC    |     | FC Network 30-30- |     | Simulation + P&R |
| AXI-Lite PS-PL   |     | 10-10 neurons     |     |                  |
+------------------+     +-------------------+     +------------------+
        |
        v
  Classification Output
  0 = Normal Sleep
  1 = Sleep Apnea
```

---

## Module Hierarchy

```
zynet_top.v  (Top Module)
|
+-- axi_lite_wrapper.v       PS-PL communication interface
|     AXI-Lite registers:
|       - weight/bias loading
|       - soft reset
|       - layer/neuron selection
|       - inference trigger
|       - output readback
|       - interrupt signal to ARM
|
+-- layer1.v  (30 neurons)
|   +-- neuron.v [x30]
|       +-- weight_memory.v   (MIF-initialized ROM)
|       +-- relu.v            (or sig_rom.v)
|
+-- layer2.v  (30 neurons)
|   +-- neuron.v [x30]  ...
|
+-- layer3.v  (10 neurons)
|   +-- neuron.v [x10]  ...
|
+-- layer4.v  (10 neurons)
|   +-- neuron.v [x10]  ...
|
+-- find_max_out.v            Argmax of 10 outputs -> class index
```

---

## Neuron Processing Element

Each neuron implements the computation:

```
output = Activation( sum(x[i] * w[i]) + bias )
```

Where:
- `x[i]` = input from previous layer
- `w[i]` = stored weight from Weight Memory
- `bias` = stored bias value
- `Activation` = ReLU or Sigmoid (ROM-based)

Fixed-point representation:
- Word length: 16 bits
- Format: [1 sign][2 integer][13 fraction]
- Overflow handling: saturation arithmetic (no wrap-around)

---

## AXI-Lite Interface Register Map

| Register | Function |
|----------|---------|
| `S_AXI_WDATA[31:0]` | Weight/bias value to load |
| `layerNumber[31:0]` | Target layer index |
| `neuronNumber[31:0]` | Target neuron index |
| `biasValue[31:0]` | Bias value |
| `weightValue[31:0]` | Weight value |
| `nnOut[31:0]` | Neural network output (read) |
| `S_AXI_RDATA[31:0]` | AXI read data bus |
| `intr` | Interrupt: inference complete |

---

## Dataflow and Pipelining

Each layer operates in pipeline:
1. Layer N starts computation when Layer N-1 asserts `o_valid`
2. Traverser FSM sequences neurons within a layer
3. Output vectors are serialized via shift-register logic
4. `findMaxOut` receives all layer4 outputs and produces class ID

This enables streaming operation â€” multiple EEG windows processed
sequentially without pipeline stalls.

---

## Fixed-Point Quantization

Floating-point model weights extracted from MATLAB are quantized:

```matlab
% MATLAB conversion
fi_weight = fi(float_weight, 1, 16, 13);  % signed, 16-bit, 13 fraction bits
binary_str = bin(fi_weight);              % convert to binary string
% Write to .mif file for $readmemb in Verilog
```

This 16-bit format provides sufficient precision for inference while
fitting efficiently into FPGA BRAM resources.
