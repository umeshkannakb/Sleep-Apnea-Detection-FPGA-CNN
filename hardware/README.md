# Hardware Implementation â€” Verilog RTL on Xilinx Zynq-7000

This folder contains the complete Verilog RTL implementation of the feed-forward fully connected neural network for sleep apnea classification on FPGA.

---

## Target Platform

| Item | Specification |
|------|-------------|
| FPGA Board | Xilinx ZedBoard (Zynq-7000 SoC) |
| FPGA Device | XC7Z020-CLG484-1 |
| ARM Processor | Dual-core ARM Cortex-A9 (Processing System) |
| EDA Tool | Xilinx Vivado 2023.1 |
| HDL | Verilog |

---

## Network Configuration

| Parameter | Value |
|-----------|-------|
| Input size | 784 samples (flattened FC input from CNN) |
| Hidden Layer 1 | 30 neurons, ReLU |
| Hidden Layer 2 | 30 neurons, ReLU |
| Hidden Layer 3 | 10 neurons, Sigmoid |
| Hidden Layer 4 | 10 neurons, Sigmoid |
| Output | 2 classes: Normal (0) or Apnea (1) |
| Fixed-point format | 16-bit: [1 sign][2 integer][13 fraction] |

---

## Module Descriptions

### neuron.v
The atomic compute unit. Performs:
1. Multiply-Accumulate: `sum += x[i] * w[i]`  for i = 0..numWeight-1
2. Bias addition: `sum += bias`
3. Activation: ReLU or Sigmoid (selected via `actType` parameter)
4. Overflow saturation: prevents wrapping on overflow/underflow

**Parameters:**
- `layerNo` â€” layer index for weight addressing
- `neuronNo` â€” neuron index within layer
- `numWeight` â€” fan-in (number of weights)
- `dataWidth` â€” 16 (fixed-point word length)
- `actType` â€” "relu" or "sigmoid"

### weight_memory.v
Synchronous dual-port RAM:
- Initialized at synthesis time from `.mif` file via `$readmemb`
- Can also be loaded at runtime via AXI-Lite write path
- Read port: increments address each clock when `ren=1`

### relu.v
Rectified Linear Unit:
```
f(x) = x   if x > 0
f(x) = 0   if x <= 0
```
Includes saturation logic for 16-bit fixed-point range.

### sig_rom.v
Sigmoid activation using ROM lookup table:
```
f(x) = 1 / (1 + e^(-x))
```
Pre-computed values stored for 10-bit input range.
Reduces computation to a single ROM read per activation.

### layer1.v â€” layer4.v
Each layer module:
- Instantiates N neurons in parallel
- Controls neuron-wise traversal via internal FSM
- Serializes outputs for next layer
- Asserts `o_valid` when all outputs are ready

### find_max_out.v
Receives 10 output values from Layer 4.
Implements comparator tree to find maximum value index.
Outputs class ID: `0` (Normal) or `1` (Apnea).

### axi_lite_wrapper.v
AXI-Lite slave interface connecting ARM PS to the neural network PL:
- Accepts weight/bias values from ARM processor
- Controls soft reset and layer/neuron selection
- Triggers inference start
- Returns classification output to ARM via read register
- Asserts interrupt (`intr`) when inference completes

### zynet_top.v
Top-level integration module:
- Connects all layers in pipeline order
- Manages global reset and clock
- Routes AXI signals to wrapper
- Connects findMaxOut to output register

---

## Synthesis Results (Vivado)

### Neuron Module

| Metric | Value |
|--------|-------|
| WNS (Setup slack) | 4.571 ns |
| Total endpoints | 197 |
| Total power | 0.157 W |
| LUT | 51 / 53200 (0.10%) |
| FF | 43 / 106400 (0.04%) |
| DSP | 2 / 220 (0.91%) |
| BRAM | 0.50 / 140 (0.36%) |

### Full FC Network (zyNet)

| Metric | Value |
|--------|-------|
| WNS (Setup slack) | 4.156 ns |
| Total endpoints | 23794 |
| Total power | 0.368 W |
| LUT | 6290 / 53200 (11.82%) |
| FF | 6653 / 106400 (6.25%) |
| DSP | 160 / 220 (72.73%) |
| BRAM | 15 / 140 (10.71%) |
| Overall simulation accuracy | **89%** |

---

## Simulation

### Neuron testbench (`tb/tb_neuron.v`)
- Applies test inputs to a single neuron
- Verifies `outvalid` timing and `out` correctness
- Tests both ReLU and Sigmoid activation modes
- Checks overflow saturation behavior

### Top-level testbench (`tb/tb_top_sim.v`)
- Reads test EEG samples from text file (up to 30 samples)
- Sends data via `send_data` task through AXI-Stream
- Reads output via `read_output` task
- Computes and displays Overall Accuracy via Tcl console
- Classifies each sample as Apnea or Normal

**Running simulation in Vivado:**
```tcl
launch_simulation
run all
# Console shows: "Overall Accuracy: 89.000000"
```

---

## MIF File Format

`.mif` files contain binary representations of quantized weights:

```
// Layer 1 weights for neuron 0
0000000000000001   <- weight[0] in 16-bit binary
1111111111111110   <- weight[1] (negative value in 2's complement)
...
```

Each line is a 16-bit binary string. Read by Verilog using:
```verilog
$readmemb("w_1_0.mif", weight_mem);
```

---

## How to Add Source Files in Vivado

1. File -> Add Sources -> Add or Create Design Sources
2. Add all `.v` files from `rtl/`
3. Add `.v` files from `tb/` as Simulation Sources
4. Add `constraints/zedboard.xdc` as Constraints
5. Set `zynet_top` as top module for synthesis
6. Set `tb_top_sim` as top for simulation

---

## What to Add

- [ ] Add actual trained `.mif` weight files to `mif/` folder
- [ ] Add `zynet_top.v` source code
- [ ] Add all layer modules (`layer1.v` through `layer4.v`)
- [ ] Add testbench source files
- [ ] Add simulation waveform screenshots to `results/`
- [ ] Add `zedboard.xdc` constraint file
