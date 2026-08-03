---
layout: page
title: Reconfigurable FPGA Architecture for Real-Time MIMO Channel Emulation
description: "An FPGA-based, reconfigurable hardware channel emulator modeling indoor MIMO Rician fading, spatial correlation, and Doppler spread using High-Level Synthesis (HLS). <br><br>**Keywords:** MIMO Channel Emulator, FPGA Engineering, IEEE 802.11n TGn, Rician Fading, Spatial Correlation, High-Level Synthesis, Synopsys Synphony, DSP"
img: assets/img/projects/channel_emulator/MIMO_Coefficient_Generator-001-001.jpg
importance: 1
category: work
related_publications: false
---

## 1. Executive Summary

Evaluating high-throughput wireless systems like IEEE 802.11n (Wi-Fi 4), 802.11ac, and LTE under realistic propagation environments requires precise channel emulation. Commercial RF channel emulators are exceptionally expensive and lack the reconfigurability needed for custom research testbeds. Conversely, pure software simulations run far too slowly to support real-time **Hardware-in-the-Loop (HITL)** baseband testing.

This project delivers a **Reconfigurable FPGA-Based MIMO Channel Emulator** designed for low-cost, real-time wireless link evaluation. Operating on a multi-FPGA platform, the system faithfully reproduces complex indoor radio propagation phenomena—including **Additive White Gaussian Noise (AWGN)**, **Doppler power spectra**, **Kronecker antenna spatial correlation**, and **Rician/Rayleigh fading**—matching the IEEE 802.11 TGn standard specification.

Engineered using **High-Level Synthesis (Synopsys Synphony HLS)**, the entire channel coefficient generator fits comfortably within a single target FPGA, consuming under **30% of registers**, **50% of ALMs**, and **39% of onboard DSP blocks**. The emulated channel impulse response, power delay profiles (PDP), and spatial correlation distributions match theoretical mathematical models with high fidelity, offering a scalable, production-ready solution for next-generation physical layer (PHY) verification.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/channel_emulator/MIMO_Coefficient_Generator-001-001.jpg" title="MIMO Channel Fading Coefficient Generator Architecture" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 1: Overall hardware architecture of the MIMO channel coefficient generator detailing AWGN, Doppler filtering, spatial correlation, and rate interpolation blocks.
</div>

---

## 2. Industry Metric & System Parameter Overview

The table below outlines the core hardware specifications, channel model parameters, and target resource budgets engineered into the system:

| Metric / Parameter Category | Industry Setting / Benchmark | Engineering Purpose & Functional Role |
| :--- | :--- | :--- |
| **Wireless Standards** | IEEE 802.11 TGn (2.4 GHz / 5 GHz) | Designed for indoor WLANs; extensible to 802.11ac and LTE systems. |
| **Fading Characterization** | Rician & Rayleigh Fading Models | Decomposes channel matrices into Line-of-Sight (LOS) and Non-Line-of-Sight (NLOS) paths. |
| **Doppler Power Spectrum** | Bell-Shaped Spectrum (Human Motion) | Models indoor human movement (1.2 km/h environmental velocity, Doppler spread ~3 to 6 Hz). |
| **Spatial Correlation Model** | Kronecker MIMO Model | Pre-calculated transmit ($R_{TX}$) and receive ($R_{RX}$) matrices to minimize hardware matrix complexity. |
| **Target FPGA Platform** | Altera/Intel Stratix II EP2S180 | Multi-FPGA development board (Radrix Corp) equipped with 5 FPGAs and 8 AD/DA converters. |
| **Design Toolchain** | Synopsys Synphony HLS & Synplify | High-Level Synthesis workflow enabling rapid prototyping from MATLAB algorithms to RTL. |
| **Interpolation Scheme** | Multi-Stage Rate Conversion | Upsamples Doppler signals from 1750 Hz to 125 kHz before downsampling to 2604 Hz. |
| **Resource Allocation** | < 30% Regs \| 50% ALMs \| 39% DSPs | Single-chip hardware occupancy preserving remaining onboard FPGAs for baseband execution. |

---

## 3. System Architecture & Engineering Innovations

### 3.1 AWGN & Doppler Spectrum Generation Pipeline

Real-time wireless emulation requires continuously generating statistically independent Gaussian noise vectors and shaping them according to physical Doppler fading profiles.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/channel_emulator/AWGN_Doppler_Model-001-001.jpg" title="AWGN and Doppler Generator Hardware Block" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 2: Hardware implementation of the 12-polynomial AWGN generator and multi-stage Doppler spectral filtering pipeline.
</div>

* **12-Uniform Generator Central Limit Architecture:** To achieve high-precision Gaussian noise without massive LUT memory overhead, the AWGN block sums 12 independent uniform random number generators driven by distinct feedback polynomials.
* **Multi-Stage Bell-Shaped Doppler Filtering:** Indoor Doppler effects stem from slow human movement. The system uses a multi-stage Low-Pass Filter (LPF) pipeline to shape the Gaussian output into the standard TGn "bell-shaped" power spectrum.

---

### 3.2 Kronecker Spatial Correlation & Rician Decomposition

MIMO performance relies heavily on antenna spacing and spatial correlation between transmit and receive arrays. Real-time matrix multiplication of complex correlation matrices ($R_{TX}$ and $R_{RX}$) is computationally prohibitive on hardware.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/channel_emulator/Spatial_Correlation_Model-001-001.jpg" title="Spatial Antenna Correlation Engine" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 3: Hardware spatial correlation module utilizing offloaded MATLAB coefficient vectors to execute Kronecker matrix operations.
</div>

* **Pre-Calculated Matrix Offloading:** Complex Angular Spread (AS) and Power Angular Spectrum (PAS) correlation coefficients are pre-computed offline in MATLAB and loaded into hardware register arrays, drastically reducing DSP block utilization.
* **Rician K-Factor Fading Model:** Combines a deterministic Line-of-Sight (LOS) matrix ($H_F$) with a dynamically fading Non-Line-of-Sight (NLOS) Rayleigh matrix ($X_{ij}$) scaled by the target Rician K-factor.

---

### 3.3 Multi-Stage Interpolation & Rate Adjustment

FPGA system clocks typically operate at high frequencies (e.g., 80 MHz), whereas Doppler channel variations occur at low sampling rates (e.g., 1750 Hz).

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/channel_emulator/Sampling_Rate_Adjust-001-001.jpg" title="Sampling Rate Adjustment and Interpolation Flow" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 4: Multi-stage rate conversion pipeline converting low-frequency Doppler fading coefficients to real-time system clock rates.
</div>

* **Interpolation Filtering:** Upsamples the 1750 Hz Doppler output to an intermediate 125,000 Hz rate (8 µs sampling window).
* **Downsampling & Decimation:** Downsamples the stream by a factor of 48 to achieve a final stable coefficient update rate of 2604 Hz, perfectly matching baseband frame processing requirements.

---

## 4. Hardware Synthesis Results & Model Validation

The complete architecture was implemented on a custom multi-FPGA emulation platform equipped with five **Stratix II EP2S180F1508 FPGAs** and eight high-speed AD/DA converters. One FPGA dedicated purely to channel coefficient generation drives four surrounding FPGAs executing baseband PHY layer processing.

### 4.1 FPGA Hardware Resource Utilization

| Hardware Sub-System | ALUTs | Registers | ALMs | DSP Blocks (18x18) |
| :--- | :--- | :--- | :--- | :--- |
| **AWGN Generator** | 363 | 477 (0%) | 306 (0%) | 0 (0%) |
| **Doppler Filter** | 6,180 | 3,023 (2%) | 3,649 (5%) | 0 (0%) |
| **Spatial Correlation** | 1,211 | 1,004 (0%) | 1,487 (2%) | 16 (16.7%) |
| **Rician Fading Engine** | 1,061 | 1,057 (0%) | 1,186 (1%) | 21 (21.9%) |
| **Interpolation Filter** | 36,601 | 37,247 (25%) | 29,552 (41%) | 0 (0%) |
| **Complete Channel Generator** | **45,416** | **42,808 (28%)** | **36,180 (50%)** | **37 (38.5%)** |

---

### 4.2 Mathematical Model Validation

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/channel_emulator/Validation_Plots-001-001.jpg" title="Empirical vs Theoretical Validation Plots" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 5: Comparison of emulated hardware distributions against theoretical IEEE 802.11 TGn models across CDF taps, Doppler power spectrum, Power Delay Profile (PDP), and spatial correlation matrices.
</div>

* **Doppler Spectrum Matching:** The emulated power spectrum aligns cleanly with theoretical bell-shaped curves, dropping sharply by 10 dB at the cut-off frequency $f_d$.
* **Statistical Distribution Accuracy:** Cumulative Distribution Function (CDF) plots of generated taps confirm strict alignment with theoretical Rayleigh/Rician fading distributions.

---

## 5. Conclusion & Future Engineering Directions

This project successfully proves that High-Level Synthesis (HLS) combined with optimized DSP architectures can yield a **flexible, real-time, highly efficient MIMO channel emulator** on hardware.

### Key Engineering Takeaways:
1. **Low Hardware Occupancy:** The entire TGn coefficient generator runs inside a single Stratix II FPGA using less than 28% of registers and 38.5% of DSP blocks.
2. **High Statistical Precision:** Matches theoretical IEEE 802.11 TGn impulse response, PDP, Doppler spread, and spatial correlation distributions.
3. **High-Level Synthesis Acceleration:** Synopsys Synphony HLS reduced prototyping time from algorithm modeling to verified FPGA bitstream execution.

### Future Expansion Roadmap:
* **Migration to Modern UltraScale+/Agilex FPGAs:** Offloading the hardware pipeline onto modern Xilinx Kintex UltraScale+ or Intel Agilex 5 boards for 80 MHz/160 MHz bandwidth 802.11ac/ax emulation.
* **Full-System HITL Integration:** Coupling the channel coefficient generator with physical transceiver RF front-ends for real-time over-the-cable throughput benchmarking.
* **On-the-Fly Dynamic Parameter Tuning:** Implementing PCIe host control interfaces to dynamically update Doppler spread and Rician K-factors in real time without FPGA re-bitstream programming.
