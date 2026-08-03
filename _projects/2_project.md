---
layout: page
title: Scalable AWGN Hardware Emulator
description: "High-throughput, resource-optimized AWGN hardware emulator reaching 400 MHz on Intel Agilex FPGAs with ±0.1 dB accuracy. <br><br>**Keywords:** HW/SW Co-Design, Hardware Emulation, FPGA Design, RTL Design, DSP Algorithms, AXI, APB, SystemVerilog, C/C++, Python"
img: assets/img/projects/awgn_emulator/HPS-to-FPGA.jpg
importance: 1
category: work
related_publications: false
---

## 1. Executive Summary

This project presents a high-throughput, resource-optimized **Additive White Gaussian Noise (AWGN) Hardware Emulation System** designed for satellite (DVB-S2X) and wideband wireless communications testbeds. Integrated into a multi-rate SoC architecture featuring an **AXI4 Crossbar** and **Framer/Deframer pipelines**, the design injects frame-synchronized, mathematically precise Gaussian noise across multiple high-speed user data streams simultaneously.

Driven by an embedded **Linux OS running on the Agilex Hard Processor System (HPS)**, the system features a dynamic C/C++ HW/SW co-design platform. Operating at a **400 MHz core clock** on an Intel Agilex FPGA, the shared-resource datapath supports variable sample rates from **375 MSPS up to 6 GSPS** across arbitrary $N$-channel configurations ($N \le 64$) while maintaining a lightweight FPGA footprint.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/awgn_emulator/APB.jpg" title="Agilex FPGA / HPS SoC Architecture" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 1: Top-level Agilex FPGA/HPS SoC architecture diagram illustrating the interaction between the Linux kernel drivers, Framer/Deframer pipelines, and the shared multi-lane AWGN engine.
</div>

### Key Highlights

* **SoC Integration & Multi-Stream Support:** Interfaces with an AXI4 Crossbar and Framer/Deframer pipeline to perform frame-based real-time noise injection across multiple concurrent user streams.
* **Shared-Resource Architecture:** Decouples physical AWGN hardware engines from active user channel counts using AXI4-Stream `tkeep` sample masking and multiplexed adder trees, drastically reducing DSP and logic usage.
* **Linux-HPS HW/SW Co-Design:** Interfaced via an APB slave bus mapped into user space (`/dev/uioX`) on Linux HPS. Controlled by two dynamic C applications for configuration, active headroom auto-scaling, real-time power tracking, and live terminal histogram plotting.
* **High-Throughput Datapath:** Architected 256-bit and 512-bit AXI4-Stream datapaths reaching a maximum operating frequency ($F_{\max}$) of **400 MHz** on Intel Agilex, with a fixed pipeline latency of 57 clock cycles.
* **Dynamic Precision:** Real-time Exponential Moving Average (EMA) power estimation dynamically measures input signal power ($P_{\text{signal}}$) and scales injected noise to enforce target $E_s/N_0$ settings with **$\pm 0.1\text{ dB}$ precision**.

---

## 2. System Architecture & Dynamic HW/SW Co-Design

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/awgn_emulator/DesignBlock_AWGN_Adder-001-001.jpg" title="System Architecture" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 2: System architecture block diagram illustrating the datapath, adder trees, and multi-channel resource distribution.
</div>

### 2.1 Linux HPS & C Driver Architecture

The control plane is hosted on an embedded Linux distribution running on the ARM Cortex-A53 Hard Processor System (HPS). The FPGA register map is mapped into user space via Linux UIO drivers (`/dev/uioX`) through a 32-bit APB slave interface.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/awgn_emulator/HPS-to-FPGA.jpg" title="HW/SW Co-Design Interface" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 3: HW/SW co-design architecture detailing Linux HPS user-space C drivers communicating with FPGA memory-mapped APB registers through the UIO framework.
</div>

The control system consists of two dynamic C driver applications:

1. **Configuration Engine (`awgn_config.c`):**
   * Translates double-precision system inputs ($F_s = 375\text{ MHz}$, Bandwidth $B = 20\text{ MHz}$, target $E_s/N_0$) into Q2.30 initial power hex values and Q5.11 software scale factors.
   * Generates and programs 5 independent 64-bit seed pairs per AWGN core (50 register writes total for an 8-channel setup).
   * Performs automated read-back verification on APB registers prior to releasing soft reset and enabling the core datapath.

2. **Telemetry & Auto-Headroom Engine (`awgn_monitor.c`):**
   * Computes total expected peak amplitude ($2.0\sqrt{P_{\text{total}}}$) in real time and automatically packs 4-bit shift values into register `0x04C0` (`ADDR_NUM_SHIFT_BIT_BASE`) to prevent fixed-point pipeline clipping.
   * Periodically reads online signal (`0x0D40`) and total (`0x0E40`) power accumulators, displaying decibel-accurate SNR and $E_s/N_0$ calculations across all active channels.
   * Controls hardware bin selection (`0x0F40`), auto-zooms scale modes (Modes 0–15) based on visual peak levels, reads 64-bit bin values (`0x0F44`/`0x0F48`), and renders a live ASCII bell curve in the terminal.

---

### 2.2 Taus258 Combined LFSR Generator

To eliminate statistical correlations common in single LFSRs, the PRNG implements the Taus258 algorithm running five independent 64-bit LFSR components in parallel ($\approx 2^{258}-1$ period, or $2.45 \times 10^{61}$ years):

$$u = \text{LFSR}_1 \oplus \text{LFSR}_2 \oplus \text{LFSR}_3 \oplus \text{LFSR}_4 \oplus \text{LFSR}_5$$

```c
// Taus258 Reference Generator Snippet
unsigned long long z1, z2, z3, z4, z5;

double lfsr258() {
    unsigned long long b;
    b = (((z1 << 1) ^ z1) >> 53);   z1 = (((z1 & 18446744073709551614ULL) << 10) ^ b);
    b = (((z2 << 24) ^ z2) >> 50);  z2 = (((z2 & 18446744073709551104ULL) << 5) ^ b);
    b = (((z3 << 3) ^ z3) >> 23);   z3 = (((z3 & 18446744073709547520ULL) << 29) ^ b);
    b = (((z4 << 5) ^ z4) >> 24);   z4 = (((z4 & 18446744073709420544ULL) << 23) ^ b);
    b = (((z5 << 3) ^ z5) >> 33);   z5 = (((z5 & 18446744073701163008ULL) << 8) ^ b);
    return (z1 ^ z2 ^ z3 ^ z4 ^ z5);
}
```

---

### 2.3 Box-Muller Transformation Engine

Uniform random variables $u_0$ (48-bit) and $u_1$ (16-bit) are transformed into normalized Gaussian variables ($awgn_0, awgn_1$):

$$awgn_0 = f \cdot \cos(\theta), \quad awgn_1 = f \cdot \sin(\theta)$$

Where amplitude $f = \sqrt{-2 \ln(u_0)}$ and phase $\theta = 2\pi u_1$.

* **Amplitude ($f$):** Normalizes $u_0 = 2^{-k}(1+m)$, split into an Exponent LUT ($2k\ln 2$) and Mantissa LUT ($-2\ln(1+m)$), followed by a Digit-Recurrence Square Root module.
* **Phase ($\theta$):** Uses a Quarter-Wave Sine/Cosine LUT mapping $[0, 65535] \rightarrow [0, 2\pi]$.
* **Pipeline Synchronization:** Aligns the 20-cycle amplitude calculation delay with the 3-cycle phase path via shift registers before final multiplication to Q6.12 (18-bit fixed-point).

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/awgn_emulator/DesignBlock_BoxMuller-001-001.jpg" title="Box-Muller Transformation" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 4: Pipeline-aligned Box-Muller transformation engine architecture.
</div>

---

### 2.4 Real-Time Power Estimation & Scaling

To maintain a target $E_s/N_0$ dynamically, input power $P_{\text{inst}}[n] = I[n]^2 + Q[n]^2$ is computed using dedicated 19x18 DSP blocks and filtered via an Exponential Moving Average (EMA) pipeline:

$$P_{\text{avg}}[n] = \alpha \cdot P_{\text{inst}}[n] + (1 - \alpha) \cdot P_{\text{avg}}[n-1], \quad \alpha = 2^{-16}$$

The gain scaling factor $K_{\text{noise}}$ is derived as:

$$K_{\text{noise}} = \sqrt{P_{\text{signal}}} \cdot \frac{1}{\sqrt{2}} \cdot \frac{1}{\sqrt{10^{\text{SNR}_{\text{dB}}/10}}}$$

$$\text{SNR}_{\text{dB}} = \left(\frac{E_s}{N_0}\right)_{\text{dB}} - 10 \log_{10}\left(\frac{F_s}{B}\right)$$

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/awgn_emulator/DesignBlock_PowerEst-001-001.jpg" title="Real-Time Power Estimation" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 5: Online Exponential Moving Average (EMA) power estimation and noise scaling module.
</div>

---

## 3. Register Map & Control Plane (APB Slave)

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/awgn_emulator/DesignBlock_AWGN_Top-001-001.jpg" title="APB Register Top Block" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 6: APB control plane integration and top-level module register mapping.
</div>

| Address | Bit Range | Access | Register Name | Description |
| :--- | :--- | :--- | :--- | :--- |
| `0x000` | `[0]` | R/W | **System Enable** | `0`: Disable, `1`: Enable noise injection |
| `0x000` | `[1]` | R/W | **System Reset** | `0`: Active Reset, `1`: Normal Operation |
| `0x000` | `[31]` | R/W | **Power Est. Enable** | `0`: Static Mode, `1`: Adaptive EMA Mode |
| `0x040` | `[31:0]` | R/W | **Init Power Ch 0** | Seed power parameter (Q2.30 format) |
| `0x140` | `[15:0]` | R/W | **SW Data / SNR Ch 0** | Noise gain scaling factor (Q5.11 format) |
| `0x240` | `[31:0]` | R/W | **PRNG Seed Low 0** | 32 LSBs of Taus258 LFSR seed |
| `0x244` | `[31:0]` | R/W | **PRNG Seed High 0** | 32 MSBs of Taus258 LFSR seed |
| `0x4C0` | `[31:0]` | R/W | **Num Shift Bits** | Packed 4-bit headroom shift settings for channels 0–7 |
| `0x0D40` | `[31:0]` | RO | **P_Signal Mon Ch 0** | Measured signal power accumulator (Q2.30 format) |
| `0x0E40` | `[31:0]` | RO | **P_Total Mon Ch 0** | Measured total power accumulator |
| `0x0F40` | `[5:0]` | R/W | **Hist Bin Index** | Active bin index selection (0–63) |
| `0x0F40` | `[11:8]` | R/W | **Hist Scale Mode** | Scale selector (Modes 0–15) for bell-curve viewing |
| `0x0F44` | `[31:0]` | RO | **Hist Data Low** | 32 LSBs of selected 64-bit histogram bin count |
| `0x0F48` | `[31:0]` | RO | **Hist Data High** | 32 MSBs of selected 64-bit histogram bin count |

---

## 4. Hardware Resource Utilization (Intel Agilex)

All configurations achieve a maximum operating frequency ($F_{\max}$) of **400 MHz**:

| Hierarchy Configuration | ALUTs | Dedicated Registers | Memory Bits | DSP Blocks | Maximum Frequency |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **256-bit Core (8 AWGN Lanes)** | 13,544 | 27,973 | 423,424 | 42 | **400 MHz** |
| **512-bit Core (16 AWGN Lanes)** | 23,431 | 51,411 | 846,848 | 82 | **400 MHz** |
| **256-bit Core + Full Monitoring Suite** | 21,463 | 38,309 | 423,424 | 76 | **400 MHz** |
| **512-bit Core + Full Monitoring Suite** | 32,481 | 65,471 | 846,848 | 148 | **400 MHz** |

---

## 5. Verification & Live Terminal Diagnostics

### 5.1 Fixed-Point RTL vs. Floating-Point Software Model

Validated SystemVerilog RTL (16-bit Q2.14 fixed-point) against a double-precision floating-point Python model across $N = 10,000,000$ samples:
* **Time-Domain Waveform:** Cycle-by-cycle tracking without cumulative phase drift.
* **Variance Match:** Hardware variance measured **1.00072** vs. floating-point reference **1.00026**.
* **Quantization Error:** Mean Absolute Error (MAE) of **0.00051**; maximum absolute error bounded at **0.02946**.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/awgn_emulator/Result_Sim_WaveOverlay-001-001.jpg" title="Waveform Overlay" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 7: Time-domain waveform overlay of the first 1000 samples.
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/awgn_emulator/Result_Sim_Dist-001-001.jpg" title="Gaussian Distribution" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 8: Measured hardware Gaussian probability density function (PDF).
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/awgn_emulator/Result_Sim_Err-001-001.jpg" title="Absolute Error Distribution" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 9: Quantization and calculation error distribution relative to theoretical model.
</div>

---

### 5.2 Dynamic Headroom Auto-Scaling (2.0-Sigma Crest Rule)

The runtime monitor scans power levels and programs bit-shift scaling in real time to prevent internal pipeline clipping:

```text
--- Auto-Adjusting AWGN Datapath Limits (2.0-Sigma Rule) ---
 CH |  P_Sig   | SwData | P_Total  | 2.0-Sigma Peak | Shift | Max Range
-----------------------------------------------------------------------
 Visual Peak:   1.73, Hist Mode: 7
  0 |   0.2600 |  0.968 |   0.7474 |        1.7291 |   0   | +/-   2.0
  1 |   0.2600 |  0.544 |   0.4141 |        1.2870 |   0   | +/-   2.0
  2 |   0.2600 |  0.306 |   0.3087 |        1.1112 |   0   | +/-   2.0
  3 |   0.2600 |  0.172 |   0.2754 |        1.0496 |   0   | +/-   2.0
  4 |   0.2600 |  0.097 |   0.2648 |        1.0292 |   0   | +/-   2.0
  5 |   0.2600 |  0.055 |   0.2615 |        1.0228 |   0   | +/-   2.0
  6 |   0.2600 |  0.031 |   0.2605 |        1.0207 |   0   | +/-   2.0
  7 |   0.2600 |  0.017 |   0.2601 |        1.0200 |   0   | +/-   2.0
==================================================================================
```

---

### 5.3 Real-Time $E_s/N_0$ Telemetry Table

The monitor polls hardware power registers across 8 active channels in real time, validating configured versus measured performance:

```text
==================================================================================
                        AWGN CORE LIVE EsNo MONITOR
                        Running Time: 00:04:01
                        Active Hardware Mode: 7
==================================================================================
 CH | P_Signal (Dec) |  P_Total (Dec) |  P_Noise (Dec) | SNR (dB) | Es/No (dB)
------------------------------------------------------------------------------
  0 |       0.259972 |       0.746871 |       0.486900 |   -2.725 |     10.005
  1 |       0.259972 |       0.414127 |       0.154155 |    2.270 |     15.000
  2 |       0.259972 |       0.308752 |       0.048780 |    7.267 |     19.997
  3 |       0.259972 |       0.275439 |       0.015468 |   12.255 |     24.985
  4 |       0.259972 |       0.264841 |       0.004869 |   17.275 |     30.005
  5 |       0.259972 |       0.261525 |       0.001554 |   22.236 |     34.966
  6 |       0.259972 |       0.260460 |       0.000488 |   27.265 |     39.995
  7 |       0.259972 |       0.260121 |       0.000149 |   32.407 |     45.137
==============================================================================
```

---

### 5.4 Live Terminal Histogram (64-Bin PDF Plot)

Extracted directly from the FPGA histogram hardware block, the software plots a symmetrical Gaussian distribution curve:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/awgn_emulator/hist.jpg" title="Absolute Error Distribution" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 10: Live terminal output displaying the real-time 64-bin Gaussian PDF histogram monitored via the embedded Linux C driver.
</div>
---

## 6. Conclusion

The fully scalable AWGN hardware emulator provides a robust, high-throughput, and resource-efficient solution for real-time wireless link emulation. By combining Taus258 PRNG, Box-Muller transformation, dynamic EMA power estimation, frame-based multi-user support via Framer/Crossbar integration, and APB host control on Linux HPS, the system achieves $\pm 0.1\text{ dB}$ noise injection accuracy at $400\text{ MHz}$ on Intel Agilex FPGAs.
