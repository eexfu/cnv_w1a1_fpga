## Project Description

With the rapid development of artificial intelligence and deep learning technologies, image recognition is playing an increasingly important role in fields such as security monitoring, autonomous driving, and industrial automation. However, traditional image recognition systems still face certain challenges in terms of processing speed and energy consumption. FPGA (Field-Programmable Gate Array), with its high parallelism and customizability, has become an ideal choice for accelerating neural network computations. This project aims to combine an efficient image input module with an optimized neural network deployed on FPGA to achieve efficient and real-time image recognition.

## Quantize machine learning model

We place the model at the repository 'Neural Network Model', under the name 'Quantizize model_cnv_w1a1'

### Overview

This repository provides an advanced demonstration of using the FINN compiler to build a streaming dataflow FPGA accelerator from a quantized convolutional neural network (CNN) model trained on CIFAR-10. The network in question is a CNV-w1a1 (1-bit weights and 1-bit activations), suitable for hardware acceleration on FPGAs. The tutorial also covers how to integrate pre- and post-processing steps, fine-tune layer specialization, and manipulate folding configurations to optimize performance and resource utilization.

### Key Concepts

1. **QONNX and FINN-ONNX**:
   The workflow starts with a quantized ONNX (QONNX) model, exported directly from Brevitas. This QONNX model is then converted to FINN-ONNX, where quantization is represented as tensor annotations rather than dedicated Quant nodes.
2. **Custom Build Steps**:The FINN builder tool allows you to define custom steps in the compilation flow. For example:

   - **Pre-processing step**: Insert operations that normalize input data (e.g., dividing by 255 for image inputs).
   - **Post-processing step**: Insert operations like TopK to select the top-ranked classification output.
3. **Layer Specialization**:
   By providing a JSON configuration file, it is possible to choose between HLS and RTL implementations for certain layers. This gives you control over the underlying hardware implementation style.
4. **Folding Configuration**:
   Folding determines the parallelization strategy of each layer. By providing a folding configuration JSON, you can set parameters such as `PE` and `SIMD` factors, `mem_mode`, `ram_style` (e.g., block or distributed RAM), `resType` (LUT or DSP-based arithmetic), and other attributes to balance throughput, latency, and resource usage.
5. **Verification Steps**:
   The builder can run various verification steps after each transformation (e.g., QONNX-to-FINN, tidy-up, and streamline) to ensure that functional correctness is preserved. By providing input and expected output data, the verification steps produce output files indicating success or failure.
6. **Additional Builder Arguments**:The FINN compiler supports various arguments:

   - `standalone_thresholds` to separate thresholding from the matrix-vector unit.
   - `board` and `fpga_part` to target specific FPGA platforms.
   - `target_fps` to guide parallelization based on performance goals.
   - `folding_config_file` and `specialize_layers_config_file` to customize layer implementations and folding schemes.

### Workflow Summary

1. **Model Preparation**:
   Start with a trained Brevitas model and export it to QONNX.
   Example: CNV-w1a1 on CIFAR-10.
2. **Pre- and Post-Processing Insertions**:
   Add pre-processing steps (like normalization) and post-processing steps (like TopK for classification results) via custom build steps.
3. **HW Conversion and Specialization**:
   Convert the model from QONNX to FINN-ONNX, streamline it, and then convert layers into hardware-friendly ops. Use JSON configurations to specify HLS or RTL variants.
4. **Folding Configurations**:
   Adjust layer folding factors and memory resource types (BRAM, LUTRAM) to optimize performance or resource utilization.
5. **Verification and Reports**:
   Run verification steps to ensure correctness. Generate resource and performance estimates to guide design space exploration.
6. **Full Flow**:
   When ready, run the complete build flow to generate IP cores, integrate them into a system, and optionally produce a bitfile for deployment on FPGA boards such as Pynq-Z1.

### Getting Started

1. **Requirements**:

   - A working FINN environment (see FINN documentation).
   - Brevitas for quantized model training and export.
   - Netron for visualizing intermediate ONNX models.
   - A compatible FPGA development environment (Vivado) if building the full hardware.
2. **Commands and Scripts**:

   - Use the Jupyter notebooks to run the build steps interactively.
   - Adjust `DataflowBuildConfig` parameters (e.g., `fpga_part`, `target_fps`, `folding_config_file`) to tailor the compilation flow.
3. **Running the Full Flow**:
   Un-comment the full flow commands in the provided notebook cells to generate bitfiles, drivers, and a deployment package for supported FPGA boards.

## Block Design
![Block Design Diagram](/block%20design.png)

## Vivado and Vitis Integration for CNN-based FPGA Accelerator

This section describes the integration of a quantized convolutional neural network (CNN) model, converted using the FINN framework, into a Vivado block design and further fine-tuning of the DMA behavior using Vitis to enable stable data flow and system functionality. The project focuses on hardware acceleration, data management via DMA, and camera data input/output handling.

### Overview

The workflow involves using the FINN toolchain to convert a CNN model into a hardware-optimized device, which is then integrated into a Vivado block design for managing data processing. The model is deployed on an FPGA, and Vitis is used to fine-tune DMA behavior for reliable data transmission between the FPGA and the processing system (PS). Lastly, due to network communication issues between Linux and Windows, shared document-based methods were used to ensure stable camera data input and output.

### Key Components

1. **CNN Model Conversion with FINN**:
   The CNN model, trained and quantized (CNV-w1a1), was converted into a hardware-friendly FINN model using the FINN compiler. The model was optimized for FPGA deployment with 1-bit weights and 1-bit activations, suitable for efficient hardware acceleration.
2. **Vivado Block Design (cnv_w1a1)**:

   - **Model Integration**: The converted FINN model was integrated into a Vivado block design (`cnv_w1a1`) as a custom IP core. This design handles the entire data processing pipeline, from data input to output generation.
   - **Interfacing with PS**: The FPGA design is connected to the Processing System (PS) to manage data flow between the FPGA and the host system. This includes handling input data, performing computation, and managing output.
3. **DMA Data Transfer with Vitis**:

   - **DMA Configuration**: Vitis was used to control the behavior of Direct Memory Access (DMA) for transferring data efficiently between the FPGA and PS. By fine-tuning the DMA settings, we ensured smooth and reliable data transmission, minimizing bottlenecks and optimizing throughput.
   - **Data Flow Management**: The DMA controller is responsible for the correct reception and sending of data, ensuring that streaming data from external devices like cameras is processed efficiently by the FPGA.
4. **Camera Data Input/Output**:

   - **Camera Integration**: To interface the FPGA with a camera, a custom software routine was implemented. However, due to frequent network communication failures between Linux and Windows over Ethernet, we chose a shared document-based solution to handle camera data input and writing.
   - **Shared Document for Camera Data**: As a workaround to the network issues, camera data was written into shared documents (e.g., CSV or log files). These documents were accessed by the system for further processing, ensuring continuous data handling without interruptions.

### Detailed Workflow

1. **Model Preparation**:

   - Start with a trained CNN model (CNV-w1a1) and convert it using the FINN framework into a quantized hardware-friendly model.
   - The model is then exported as an ONNX file, further optimized for hardware deployment.
2. **Vivado Block Design**:

   - Create a Vivado block design that incorporates the FINN-optimized model as a custom IP core.
   - The block design also includes connections to the Processing System (PS) for data handling and computation management.
   - The block design file (`cnv_w1a1`) is then synthesized, generating the necessary bitstream for FPGA deployment.
3. **DMA Data Management with Vitis**:

   - In the Vitis environment, configure the DMA for efficient data transfer between the FPGA and PS.
   - Set up Vitis to fine-tune the DMA's transfer rates and optimize resource usage, ensuring smooth and uninterrupted data flow between the hardware components.
4. **Camera Integration**:

   - Implement a custom code to interface with the camera, reading input frames or data and sending them to the FPGA for processing.
   - Due to network communication issues, utilize a shared document method to store and transfer camera data between systems.

### Getting Started

#### Requirements:

- A working Linux virtual machine with Vivado and Vitis environment.
- A trained CNN model (e.g., CNV-w1a1) ready for quantization.
- FINN toolchain to convert the CNN model into hardware-optimized format.
- Access to an FPGA development board (e.g., PYNQ-Z2 or similar).
- Proper network setup for camera data capture or alternative shared document-based communication.

#### Commands and Scripts:

1. **Model Conversion**:

   - Follow the FINN toolchain instructions to convert the CNN model into a FINN-compatible format.
2. **Vivado Design**:

   - Open Vivado and load the `cnv_w1a1` block design.
   - Synthesize the design and generate the bitstream for FPGA programming.
3. **Vitis Configuration**:

   - Use the Vitis IDE to configure the DMA controller and fine-tune parameters such as transfer rates and buffer sizes.
   - Build the DMA software application to handle data flow between PS and the FPGA.
4. **Camera Data Handling**:

   - Write a script to handle camera input, ensuring data is written to a shared document if network communication fails.
5. **Run Full Flow**:

   - Integrate the bitstream and software into the FPGA system, deploying it onto the board and verifying its operation.

#### Running the Full Flow:

- Run the entire system flow by using the commands in the provided notebooks to:
  - Compile the Vivado project and generate bitstreams.
  - Set up DMA data handling in Vitis.
  - Connect the camera and handle data using the shared document method for stable communication.
