# OFDM Transceiver Design and Functional Verification using Verilog

## 📖 Overview

This project implements an **Orthogonal Frequency Division Multiplexing (OFDM) Transceiver** in Verilog HDL using Xilinx Vivado. The design integrates QPSK modulation/demodulation, AXI-Stream interfaces, and Xilinx FFT/IFFT IP cores to perform end-to-end OFDM transmission and reception. Functional verification is carried out using simulation waveforms in Vivado.

---

## ✨ Features

- Verilog HDL implementation of an OFDM Transceiver
- QPSK Modulation and Demodulation
- Xilinx FFT/IFFT IP Core Integration
- AXI-Stream Protocol Implementation
- End-to-End Functional Verification
- Vivado Simulation and Waveform Analysis

---

## 🏗️ Transmitter Architecture

The transmitter performs the following operations:

1. Accepts input bit stream.
2. Maps input bits to QPSK symbols.
3. Formats symbols using the AXI-Stream interface.
4. Passes frequency-domain symbols through the Xilinx IFFT IP.
5. Generates time-domain OFDM symbols for transmission.

### Modules

- **QPSK Mapper**
  - Converts binary input bits into QPSK constellation symbols.

- **AXI-Stream Formatter**
  - Implements AXI-Stream handshake signals:
    - `tvalid`
    - `tready`
    - `tlast`
  - Enables reliable data transfer to the IFFT IP.

- **Xilinx IFFT IP**
  - Converts frequency-domain QPSK symbols into time-domain OFDM symbols.

---

## 📡 Receiver Architecture

The receiver performs the reverse OFDM processing:

1. Accepts received OFDM symbols.
2. Uses the Xilinx FFT IP to recover frequency-domain data.
3. Demaps QPSK symbols back into binary bits.
4. Reconstructs the transmitted bit stream.

### Modules

- **Xilinx FFT IP**
  - Converts received time-domain OFDM symbols into frequency-domain symbols.

- **QPSK Demapper**
  - Extracts I/Q components.
  - Reconstructs the original transmitted bit stream.

---

## ✅ Functional Verification

The complete OFDM transceiver was verified using **Vivado Simulation**.

Verification included:

- Correct QPSK Mapping/Demapping
- AXI-Stream Handshake Validation
- FFT/IFFT Functional Verification
- End-to-End Data Recovery
- Waveform Analysis

---

## 🛠️ Tools and Technologies

- Verilog HDL
- Xilinx Vivado
- Xilinx FFT IP Core
- Xilinx IFFT IP Core
- AXI-Stream Interface
- Vivado Simulator

---



> **Note:** The directory structure above is illustrative. Adjust it to match your repository.

---



## 🚀 Future Improvements

- Add Cyclic Prefix (CP) insertion and removal.
- Support higher-order modulation schemes (16-QAM, 64-QAM).
- Implement channel estimation and equalization.
- Introduce pilot subcarriers.
- Add BER performance analysis under AWGN and fading channels.

---


##  Core Architecture & System Blocks

The complete system pipeline—from digital baseband processing to the hybrid switching layer—is designed according to the following hardware architecture:
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/cb6f4bab3b6d8f9479c1797c65960ea23b04bbbe/Latest_Architecture.png)


