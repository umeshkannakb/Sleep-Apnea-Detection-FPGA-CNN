# MIF Files â€” Quantized Weight and Bias Initialization

This folder contains Memory Initialization Files (`.mif`) for preloading the FPGA BRAM with trained neural network weights and biases.

---

## File Naming Convention

```
<layer_name>_weights.mif    <- Weights for a specific layer
<layer_name>_bias.mif       <- Bias for a specific neuron
```

Example:
```
layer1_neuron0_weights.mif
layer1_neuron0_bias.mif
layer2_neuron5_weights.mif
...
```

---

## Format

Each `.mif` file is a plain text file where each line contains one 16-bit binary value:

```
// layer1_neuron0_weights.mif
0000000101000011
1111110001001010
0000001000000000
...
```

- Each line = one weight value
- 16-bit two's complement fixed-point
- Format: 1 sign bit + 2 integer bits + 13 fraction bits
- Total weights per neuron = `numWeight` (e.g., 784 for first layer)

---

## How Files Are Generated (MATLAB)

```matlab
w_fp = fi(layer_weight, 1, 16, 13);   % Fixed-point quantization
for i = 1:length(w_fp)
    fprintf(fid, '%s\n', bin(w_fp(i)));
end
```

---

## How Files Are Used (Verilog)

```verilog
// In weight_memory.v
reg [15:0] mem [0:numWeight-1];
initial begin
    $readmemb("w_1_0.mif", mem);
end
```

---

## Important Notes

- Place `.mif` files in the **Vivado project working directory** so `$readmemb` can find them
- Or specify the full path in the `$readmemb` call
- Files can also be loaded at runtime via AXI-Lite for dynamic weight updates
- The `pretrained` compile switch enables MIF-based initialization:
  ```verilog
  `define pretrained
  ```

---

## Sample Files

`w_1_15.mif` and `b_1_15.mif` are included as sample files from training.
Replace with your own trained weight files after running `extract_weights.m`.
