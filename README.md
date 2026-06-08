# FlexViT: A Flexible FPGA-based Accelerator for Edge Vision Transformers

FlexViT is a reconfigurable FPGA accelerator for Vision Transformer (ViT) inference on resource-constrained edge devices. Built using the SECDA-TFLite framework, FlexViT adopts a hardware-software co-design approach that maps both fully connected (FC) and convolutional (CONV) layers onto a unified high-throughput GEMM engine via a runtime im2col transformation. 

This repository provides the hardware bitstreams and the custom TensorFlow Lite (TFLite) delegate source code. It is designed to be integrated directly into the SECDA-TFLite build environment for evaluation on the AMD Zynq-7000 SoC (PYNQ-Z2).

---

## Repository Structure

The codebase is organized as follows:

    FlexViT/                   
    ├── bitstreams/   - Accelerator bitstreams                          
    ├── extra_files/  - Configuration files for SECDA-TFLite                      
    ├── models/       - Models needed for experiments                   
    ├── src/          - Source code                                
    └── README.md

---

## Hardware & Software Requirements

* Target Hardware: PYNQ-Z2 board (AMD Zynq-7000 SoC).
* Operating System: PYNQ Linux (Ubuntu base).
* Build System: Bazel.
* Dependencies: SECDA-TFLite framework, TensorFlow Lite.

---

## Integration with SECDA-TFLite

Because FlexViT relies on SECDA-TFLite's hardware-software co-design methodology, this repository is intended to be built within the host framework rather than in isolation. The custom TFLite delegate located in 'src/v9' acts as an extension to the SECDA-TFLite runtime.

When invoked, the SECDA-TFLite framework delegates supported graph operations to our custom driver. The delegate intercepts fully connected (FC) and convolutional (CONV) operations from the INT8 quantized model. For CONV layers, it performs a runtime 'im2col' transformation on the CPU, linearizing them into standard GEMM operations. The host driver then analyzes the layer dimensions to dynamically select between Dense or Mobile execution modes before dispatching the payload to the FlexViT hardware.

**SETUP INSTRUCTIONS:**

**1. Base Setup:** Initialize and set up the base SECDA-TFLite repository by following the official installation instructions in the main SECDA-TFLite repository (https://github.com/gicLAB/SECDA-TFLite). Also, pull the code from this repo.

**2. Copy Files:** Create a folder named 'vit_delegate' inside 'SECDA-TFLite/src/secda_delegates' and copy the 'src/v9' folder from this repo into it.

**3. Configure Files:** In the folder '/extra_files', we provide configuration extensions to add to files in the SECDA-TFLite repository so that you can run the relevant experiments. If these files do not already exist, you can just copy the entire file in instead of extending a pre-existing file. The files are as follows:                     
- launch.json - 'SECDA-TFLite/tensorflow/.vscode/launch.json'                     
- task.json - 'SECDA-TFLite/tensorflow/.vscode/task.json'                  
- vit_9_0.json - 'SECDA-TFLite/hardware_automation/configs/vit_9_0.json'             

**4. Run Experiments:** SECDA-TFLite allows you to run experiments two ways:
  - In order to check correctness in software, you can run experiments in software using VS Code's *run and debug* system. This will tell you whether or not the accelerator design performs correctly in simulation.                 
  - In order to check correctness in hardware, you must use SECDA-TFLite's 'hardware_automation' tool to deploy the bitstream and run the accelerated inference directly on your own FPGA. From here you can:
    - Test correctness using *id_vit_delegate_9* - Test performance using *bm_vit_delegate_9* - Test power using a power meter connected to the board                            

---

## Artifact Evaluation

To reproduce the results presented in our FPL 2026 paper, reviewers should evaluate the provided INT8 quantized models using the deployed hardware bitstream.

**1. Provided Models:** We evaluate five representative Vision Transformer architectures. To ensure exact reproducibility of our quantization and dimensional variance, the pre-converted INT8 .tflite models are provided:
- ViT-T (ImageNet-21k) [src](https://github.com/martinsbruveris/tensorflow-image-models)
- DeiT-T (ImageNet-1k) [src](https://github.com/martinsbruveris/tensorflow-image-models)
- Swin-T (ImageNet-1k) [src](https://github.com/martinsbruveris/tensorflow-image-models)
- MobileViT-S (ImageNet-1k) [src](https://huggingface.co/timm/mobilevit_s.cvnets_in1k)
- EfficientViT-b1 (ImageNet-1k) [src](https://huggingface.co/timm/efficientvit_b1.r224_in1k)

**2. Functional Correctness:** Before benchmarking, verify that the hardware executes the model accurately. Run the model through the hardware using the id_vit_delegate_9 configuration. The inference outputs produced by the hardware accelerator must match the software execution outputs within TFLite's acceptable passing range (cosine similarity > 99%).

**3. Latency Evaluation:** To measure end-to-end and layer-specific speedups, run the models using the bm_vit_delegate_9 configuration.
- CPU Baseline: Execute the inference strictly on the ARM Cortex-A9 CPU with NEON SIMD instructions enabled.
- Hardware Accelerated: Execute using the FlexViT delegate and the VIT_9_0.bit bitstream clocked at 200MHz.
- All latency metrics should be averaged over 100 inference runs to account for system variance.

**4. Energy Evaluation:** Energy per inference (Joules) is measured externally using a Makerfocus USB power meter connected to the PYNQ-Z2 board.

Important Note for CPU Baseline: To ensure a fair comparison and mitigate the static power overhead of the FPGA fabric during CPU-only execution, configure the FPGA with the minimal baseline bitstream (CPU_1_0.bit) rather than the full FlexViT bitstream.

Record the power draw and multiply it by the measured latency (averaged over 100 runs) for both the CPU baseline and the CPU+Accelerator executions.

**5. Resource Utilization:** The FPGA resource utilization (BRAM, DSP, FF, LUT) can be verified by inspecting the Vivado synthesis and implementation reports generated alongside the .bit files.
