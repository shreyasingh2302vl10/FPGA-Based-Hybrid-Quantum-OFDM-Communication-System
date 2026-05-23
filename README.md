#  Design and FPGA Implementation of a Hybrid RF-Optical Quantum-Secured OFDM Transceiver

Welcome to the official repository of the **Hybrid Quantum-Optical OFDM System** implemented on FPGA. This project focuses on designing a high-speed, low-latency baseband processor that integrates advanced physical layer (PHY) techniques from cutting-edge Visible Light Communication (VLC) and Underwater Optical Wireless Communication (UOWC) paradigms, tailored for quantum-secured hardware infrastructures.

---

##  Project Overview
Multi-carrier modulation like **DCO-OFDM** is highly spectrally efficient but suffers from a severe **PAPR (Peak-to-Average Power Ratio)** problem. High PAPR drives optical transmitters (like Laser Diodes and LEDs) into their non-linear regions, causing severe signal distortion, clipping noise, and bit error rate (BER) degradation in optical and quantum transceiver components.

This project implements a real-time, low-complexity transmitter and receiver chain on FPGA, incorporating joint digital signal processing (DSP) optimization frameworks to curb distortion, mitigate hardware nonlinearity, and minimize FPGA resource utilization.

---

##  Core Architecture & System Blocks

The complete system pipeline—from digital baseband processing to the hybrid switching layer—is designed according to the following hardware architecture:

```text
                                 
                                    ╔════════════════════════╗
                                    ║       Input Data       ║
                                    ╚═══════════╦════════════╝
                                                ║
                                                ▼
                                    ╔════════════════════════╗
                                    ║    QAM / QPSK Mapper   ║ <--- [Paper 1]
                                    ╚═══════════╦════════════╝
                                                ║
                                                ▼
                                    ╔════════════════════════╗
                                    ║   Serial to Parallel   ║ <--- [Paper 1]
                                    ╚═══════════╦════════════╝
                                                ║
                                                ▼
                                    ╔════════════════════════╗
                                    ║          IFFT          ║ <--- [Paper 1 + 2]
                                    ║     (OFDM Symbol)      ║
                                    ╚═══════════╦════════════╝
                                                ║
                                                ▼
                                    ╔════════════════════════╗
                                    ║   Add Cyclic Prefix    ║ <--- [Paper 1 + 2]
                                    ╚═══════════╦════════════╝
                                                ║
                                                ▼
                          ╔═════════════════════╩═════════════════════╗
                          ║      Hybrid Switching Control Layer       ║ <--- [Paper 4]
                          ╚═══════════╦═══════════════════╦═══════════╝
                                      ║                   ║
                               RF Path║                   ║Optical Path [Paper 3]
                                      ║                   ║
                                      ▼                   ▼
                            ╔══════════════════╗ ╔══════════════════╗
                            ║  RF Path (Tx/Rx) ║ ║  Optical LED /PD ║
                            ║    [Paper 2]     ║ ║    [Paper 3]     ║
                            ╚═════════╦════════╝ ╚════════╦═════════╝
                                      ║                   ║
                                      ╚═════════╦═════════╝
                                                ║
                                                ▼
                          ╔═════════════════════╩═════════════════════╗
                          ║         Receiver DSP Engine Layer         ║ <--- [Paper 2]
                          ╚═════════════════════╦═════════════════════╝
                                                ║
                                                ▼
                                    ╔════════════════════════╗
                                    ║       Remove CP        ║ <--- [Paper 1 + 2]
                                    ╚═══════════╦════════════╝
                                                ║
                                                ▼
                                    ╔════════════════════════╗
                                    ║          FFT           ║ <--- [Paper 1 + 2]
                                    ╚═══════════╦════════════╝
                                                ║
                                                ▼
                                    ╔════════════════════════╗
                                    ║      QAM Demapper      ║ <--- [Paper 1]
                                    ╚═══════════╦════════════╝
                                                ║
                                                ▼
                                    ╔════════════════════════╗
                                    ║   NOMA/OFDMA Demux     ║ <--- [Paper 5]
                                    ╚═══════════╦════════════╝
                                                ║
                                                ▼
                                    ╔════════════════════════╗
                                    ║      Output Bits       ║
                                    ╚════════════════════════╝
