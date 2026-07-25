#  Design and FPGA Implementation of a Hybrid RF-Optical Quantum-Secured OFDM Transceiver

Welcome to the official repository of the **Hybrid Quantum-Optical OFDM System** implemented on FPGA. This project focuses on designing a high-speed, low-latency baseband processor that integrates advanced physical layer (PHY) techniques from cutting-edge Visible Light Communication (VLC) and Underwater Optical Wireless Communication (UOWC) paradigms, tailored for quantum-secured hardware infrastructures.

---

##  Project Overview
Multi-carrier modulation like **DCO-OFDM** is highly spectrally efficient but suffers from a severe **PAPR (Peak-to-Average Power Ratio)** problem. High PAPR drives optical transmitters (like Laser Diodes and LEDs) into their non-linear regions, causing severe signal distortion, clipping noise, and bit error rate (BER) degradation in optical and quantum transceiver components.

This project implements a real-time, low-complexity transmitter and receiver chain on FPGA, incorporating joint digital signal processing (DSP) optimization frameworks to curb distortion, mitigate hardware nonlinearity, and minimize FPGA resource utilization.

---

##  Core Architecture & System Blocks

The complete system pipeline—from digital baseband processing to the hybrid switching layer—is designed according to the following hardware architecture:
![image](https://github.com/shreyasingh2302vl10/FPGA-Based-Hybrid-Quantum-OFDM-Communication-System/blob/cb6f4bab3b6d8f9479c1797c65960ea23b04bbbe/Latest_Architecture.png)


