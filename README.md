# FlexViT: A Flexible FPGA-based Accelerator for Edge Vision Transformers

FlexViT is a reconfigurable FPGA accelerator for Vision Transformer (ViT) inference on resource-constrained edge devices. Built using the SECDA-TFLite framework, FlexViT adopts a hardware-software co-design approach that maps both fully connected (FC) and convolutional (CONV) layers onto a unified high-throughput GEMM engine via a runtime im2col transformation. 

This repository contains the software drivers, custom TensorFlow Lite (TFLite) delegate source code, and pre-compiled bitstreams for evaluation on the AMD Zynq-7000 SoC (PYNQ-Z2).

---

## Repository Structure

The codebase is organized as follows:

FlexViT/
├── bitstreams/                   
├── src/                          
└── README.md

---

## Hardware & Software Requirements

* Target Hardware: PYNQ-Z2 board (AMD Zynq-7000 SoC).
* Operating System: [TODO: Insert PYNQ Linux version].
* Build System: Bazel.
* Dependencies: TensorFlow Lite framework.

---

## Getting Started

### 1. Installation & Setup
* TODO: Add instructions for setting up the environment on the PYNQ-Z2.
* TODO: Add instructions for fetching necessary Git submodules or external dependencies.

### 2. Building the Project
We use Bazel for building the TFLite delegate.
* TODO: Add specific bazel build commands required to compile src/.

---

## Integration with SECDA-TFLite

* TODO: Detail how the delegate integrates with the SECDA-TFLite toolkit.
* TODO: Explain the dynamic dual-mode dataflow (Dense vs. Mobile mode switching) implementation within the delegate.

---

## Artifact Evaluation

* TODO: Add instructions for reproducing the latency, energy, and resource utilization experiments for FPL 2026.