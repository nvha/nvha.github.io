---
layout: page
title: FPGA Implementation of Multi-Antenna MIMO-OFDM STBC Transceiver Systems
description: "An Altera FPGA hardware implementation of 2x1, 2x2, and 2x3 MIMO-OFDM transceivers utilizing Alamouti Space-Time Block Coding (STBC), 64-point FFT/IFFT, and 16-path Rayleigh fading emulation. <br><br>**Keywords:** MIMO-OFDM, Space-Time Block Coding (STBC), Alamouti Encoding, FPGA Prototyping, Verilog HDL, Quartus II, DSP Builder, Wireless Communications"
img: assets/img/projects/mimo/MIMO_OFDM_STBC_Hardware_System-001-001.jpg
importance: 1
category: work
related_publications: false
---

## 1. Executive Summary

Combining Multiple-Input Multiple-Output (MIMO) technology with Orthogonal Frequency Division Multiplexing (OFDM) is fundamental to modern high-speed wireless standards, including IEEE 802.11n/ac (Wi-Fi 4/5), 3GPP LTE, and WiMAX. While Space-Time Block Coding (STBC)—specifically the Alamouti scheme—significantly increases diversity gain with minimal processing complexity, most performance evaluations in academic literature rely solely on high-level software simulations.

This project delivers a complete, real-time hardware design and FPGA prototyping implementation of $2 \times 1$, $2 \times 2$, and $2 \times 3$ MIMO-OFDM STBC transceivers. Developed using MATLAB Simulink, Altera DSP Builder, and Quartus II (Verilog HDL), the architecture incorporates 4-QAM (QPSK) digital modulation, 64-point IFFT/FFT core pipelines, cyclic prefix insertion/removal, channel estimation, and a 16-path Rayleigh multipath fading channel emulator.

Synthesized onto an Altera Stratix III 3SL150 FPGA, the hardware implementation uses a 32-bit fixed-point (10.22 format) numerical representation. Bit Error Rate (BER) testing confirms that hardware execution perfectly matches software simulation models. The results demonstrate substantial diversity gains as receive antennas increase, consuming only **38% of ALUTs** and **45% of logic registers** for a complete $2 \times 2$ MIMO-OFDM system.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/mimo/MIMO_OFDM_STBC_Hardware_System-001-001.jpg" title="2x2 MIMO-OFDM STBC Hardware System" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 1: End-to-end block diagram of the 2x2 MIMO-OFDM STBC hardware transceiver system.
</div>

---

## 2. Industry Metric & System Parameter Overview

The table below summarizes the target hardware profiles, modulation settings, and FPGA resource consumption budgets engineered into the system:

| Metric / Parameter Category | Industry Setting / Benchmark | Engineering Purpose & Functional Role |
| :--- | :--- | :--- |
| **Wireless Standards Target** | IEEE 802.11n/ac, 3GPP LTE, WiMAX | Standard-compliant baseband physical layer architecture. |
| **Antenna Configurations** | $2 \times 1$, $2 \times 2$, and $2 \times 3$ MIMO | Evaluates spatial diversity scaling across multiple receiver paths. |
| **Diversity Encoding** | 2-Tx Alamouti STBC | Provides full transmit diversity with simple decoupled linear decoding. |
| **OFDM Subcarrier Setup** | 64-Point FFT/IFFT (52 Data Subcarriers) | Converts frequency-selective channels into flat parallel subchannels. |
| **Digital Modulation** | 4-QAM / QPSK (LUT-based) | Maps binary streams into complex in-phase (I) and quadrature (Q) symbols. |
| **Fixed-Point Precision** | 32-bit Fixed-Point (10.22 Format) | Maintains numerical precision matching floating-point simulation accuracy. |
| **Fading Channel Model** | 16-Path Rayleigh Fading + AWGN | Emulates exponential power delay profiles (1 dB drop per tap). |
| **Target FPGA Platform** | Altera Stratix III 3SL150F1152C2 | High-performance FPGA target using DSP Development Kit hardware. |

---

## 3. System Architecture & Engineering Innovations

### 3.1 Space-Time Block Coding (STBC) Fundamentals

The STBC subsystem employs Alamouti space-time diversity across 2 transmit antennas and $N_R$ receive antennas.

<div class="row justify-content-center">
    <div class="col-sm-6 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/mimo/MIMO_OFDM_STBC_Hardware_System-001-001.jpg" title="System Architecture" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 2: Conceptual block diagram of a MIMO STBC system with 2 transmit antennas and NR receive antennas.
</div>

* **Alamouti Encoding Matrix:** Encodes input symbols $s(T)$ and $s(T+1)$ across two consecutive symbol durations. Antenna 1 transmits $s(T)$ then $-s^*(T+1)$, while Antenna 2 transmits $s(T+1)$ then $s^*(T)$.
* **Decoupled Linear Combining:** At the receiver, incoming signals across $N_R$ antennas are combined using estimated channel response parameters ($h_{ij}$). The orthogonal structure eliminates inter-symbol interference without complex matrix inversions.

---

### 3.2 Complete 2x2 MIMO-OFDM Hardware Transceiver Integration

The baseband transceiver pipeline integrates digital modulation, STBC encoding, pilot insertion, IFFT/FFT cores, multipath fading channels, and channel estimation.

* **Transmitter Subsystem:** Modulates serial binary data via 4-QAM (QPSK) Look-Up Tables (LUTs), applies Alamouti encoding, appends preamble training symbols, and executes 64-point IFFT conversions with Cyclic Prefix (+GI) insertion.
* **Multipath Channel Emulator:** Filters transmitted streams through a 16-path Rayleigh fading channel with a 1 dB exponential Power Delay Profile (PDP) decay per sample, adding complex Additive White Gaussian Noise (AWGN).
* **Receiver Subsystem:** Strips Cyclic Prefixes (-GI), performs 64-point FFT transformations, estimates the $H$ channel matrix from training preambles, decodes STBC symbols, and de-maps 4-QAM constellations back into binary bitstreams.

---

## 4. Performance Results & Benchmark Evaluation

The complete design was synthesized using Altera Quartus II targeting the **Stratix III 3SL150** FPGA. The hardware system was benchmarked for Bit Error Rate (BER) performance and FPGA logic element utilization.

<div class="row justify-content-center">
    <div class="col-sm-6 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/mimo/BER_Performance_Curves-001-001.jpg" title="BER Performance Curves" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 3: Measured Bit Error Rate (BER) curves versus SNR for 2x1, 2x2, and 2x3 MIMO-OFDM STBC configurations in a 16-path fading channel.
</div>

---

### 4.1 BER Diversity Gain Analysis

* **Diversity Scaling:** Increasing receive antennas from 1 ($2 \times 1$) to 3 ($2 \times 3$) yields substantial BER gains. At an SNR of 8 dB, the $2 \times 3$ setup reaches a BER of $5 \times 10^{-5}$, outperforming $2 \times 2$ ($4 \times 10^{-3}$) and $2 \times 1$ ($3 \times 10^{-2}$).
* **Fixed-Point Precision:** The 32-bit (10.22 fixed-point) hardware execution curve matches floating-point Simulink software simulations almost perfectly.

---

### 4.2 FPGA Hardware Resource Utilization

Total resource occupancy on the Altera Stratix III 3SL150 FPGA (Max ALUTs: 113,600, Max Registers: 113,600) across antenna configurations:

| Functional Hardware Module | Max Speed (MHz) | $2 \times 1$ System ALUTs / Regs | $2 \times 2$ System ALUTs / Regs | $2 \times 3$ System ALUTs / Regs |
| :--- | :--- | :--- | :--- | :--- |
| **QPSK Modulation** | 420.0 MHz | 0 (0%) / 20 (<1%) | 0 (0%) / 20 (<1%) | 0 (0%) / 20 (<1%) |
| **STBC Encoder** | 151.8 MHz | 299 (<1%) / 634 (<1%) | 299 (<1%) / 634 (<1%) | 299 (<1%) / 634 (<1%) |
| **64-Point IFFT (2 Units)** | 156.6 MHz | 4,153 (4%) / 7,824 (7%) | 4,153 (4%) / 7,824 (7%) | 4,153 (4%) / 7,824 (7%) |
| **64-Point FFT Engines** | 156.6 MHz | 4,153 (4%) / 7,824 (7%) | 8,306 (8%) / 15,648 (14%) | 12,459 (12%) / 23,472 (21%) |
| **STBC Decoder Engine** | ~132 to 143 MHz | 16,683 (15%) / 9,463 (8%) | 18,069 (16%) / 9,631 (8%) | 19,462 (17%) / 9,719 (8%) |
| **Channel Estimation** | 147.1 MHz | 2,764 (3%) / 5,264 (5%) | 3,530 (3%) / 7,505 (7%) | 4,181 (4%) / 9,520 (8%) |
| **Total Resource Usage** | **> 132 MHz** | **< 33% ALUTs / < 35% Regs** | **< 38% ALUTs / < 45% Regs** | **< 44% ALUTs / < 53% Regs** |

---

## 5. Conclusion & Future Engineering Directions

This project validates a real-time, resource-efficient hardware implementation of $2 \times 1$, $2 \times 2$, and $2 \times 3$ MIMO-OFDM STBC transceivers on FPGA hardware.

### Key Engineering Takeaways:
1. **Efficient Resource Utilization:** A full $2 \times 2$ transceiver plus channel emulator consumes less than 38% of combinational ALUTs on an Altera Stratix III device.
2. **High Diversity Gain:** Adding extra receiver paths yields steep BER improvements under multipath fading, achieving high reliability without increasing transmit power.
3. **Fixed-Point Precision:** Demonstrates that 32-bit (10.22 format) fixed-point arithmetic achieves floating-point quality while maintaining maximum hardware execution speeds above 132 MHz.

### Future Expansion Roadmap:
* **Forward Error Correction (FEC) Integration:** Adding Convolutional Coding, Viterbi decoding, or LDPC engines to further enhance BER performance.
* **Higher-Order Modulation:** Expanding the LUT constellation modules to support 16-QAM and 64-QAM configurations for increased spectral efficiency.
* **Higher Antenna Scaling:** Extending the STBC encoder and channel estimation logic to support $4 \times 4$ MIMO configurations for 802.11ac / 802.11ax specifications.
