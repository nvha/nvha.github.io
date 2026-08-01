---
layout: page
title: High-Speed HITL Link Emulation
description: Multi-vendor FPGA link emulation platform bridging Intel Agilex 5 and AMD Kintex UltraScale+ with a 320 Gbps processing datapath.
img: assets/img/projects/HITL/figure2.png
importance: 1
category: work
related_publications: false
---

## 1. Abstract

Heterogeneous multi-vendor field-programmable gate array (FPGA) environments present significant physical and logical alignment challenges for high-throughput, low-latency inter-chip communication. This project presents the design and implementation of a multi-vendor Hardware-in-the-Loop (HITL) link emulation platform bridging an **Intel Agilex 5 FPGA** and an **AMD/Xilinx Kintex UltraScale+ FPGA**. 

The physical link utilizes an external Texas Instruments LMK62E2 high-performance oscillator delivering a 156.25 MHz low-jitter reference clock to establish lock and phase synchronization across heterogeneous serializer/deserializer (SerDes) architectures. The data plane features an ultra-wide 1024-bit parallel packet processing pipeline operating on a dual-clock domain (156.25 MHz / 312.5 MHz), yielding an internal pipeline processing bandwidth up to **320 Gbps**. Real-time frame ingress is driven from a host source over Gigabit Ethernet, while egress traffic is streamed via a four-lane PCI Express (PCIe 3.0 x4) Direct Memory Access (DMA) engine into a host sink.

---

## 2. Hardware Component Inventory

The experimental HITL system consists of eight key hardware entities spanning control, processing, conversion, and physical transmission domains:

| Component Category | Device / Part Model | Vendor / Model ID | Primary Functional Role & Interconnect |
| :--- | :--- | :--- | :--- |
| **Data Source Host** | Workstation PC1 | Standard x86 Host | Generates and transmits raw data packets via Gigabit Ethernet to the transmitter FPGA. |
| **Transmitter Platform** | Terasic DE25 Board | Intel Agilex 5 FPGA | Executes ingress packet parsing, 1024-bit header processing, and network coding offload. |
| **High-Speed Breakout** | Terasic HSMC-XTS | Terasic | Adapts high-density HSMC transceiver lines to coaxial SMA ports (supports 6.25 Gbps per channel). |
| **Clock Reference Source** | LMK62E2-156M25EVM | Texas Instruments | Generates a differential 156.25 MHz low-jitter reference clock for SerDes PLL locking. |
| **Coaxial to SFP+ Adapter** | TE0422-03 Module | Trenz Electronic | Plug-in module adapting 4x SMA differential coax cables (2x TX, 2x RX) to an SFP+ footprint. |
| **Form-Factor Adapter** | WADQS-28-MEL | 10GTek | Passive port adapter converting SFP+ plug-in modules to a QSFP28 port interface. |
| **Receiver Platform** | RK-XCKU5P-F V1.2 | OpenSourceSDRLab (Kintex UltraScale+) | Receives high-speed serial streams via QSFP28 port and bridges payload to host memory via PCIe DMA. |
| **Data Sink Host** | Workstation PC2 | Standard x86 Host | Captures, logs, and evaluates output frame integrity over a PCIe 3.0 x4 bus link. |

---

## 3. System Architecture

### 3.1 End-to-End Pipeline Setup

The complete system topology is modeled as a formal end-to-end data processing chain across physical and logical boundaries.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/HITL/figure1.png" title="Conceptual Block Diagram" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 1: Conceptual system block diagram illustrating end-to-end data pipelines, physical interconnects, and external SerDes clock distribution.
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/HITL/figure2.png" title="Physical Laboratory Hardware Setup" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 2: Physical laboratory hardware-in-the-loop setup demonstrating inter-board coaxial differential cabling and form-factor converters.
</div>

* **Ingress Stage ($\text{PC1} \rightarrow \text{DE25}$):** Test payloads are framed and transmitted across a Gigabit Ethernet network interface. The onboard Ethernet MAC on the Intel Agilex 5 converts byte-oriented network streams into internal wide-bus words.
* **Transmitter Processing:** Incoming packets are parallelized into an ultra-wide $W = 1024\text{-bit}$ internal datapath. The pipeline performs single-cycle header parsing, field insertion/modification, and network coding (NC) linear combination encoding.
* **SerDes Physical Layer:** Wide parallel words are serialized by the Agilex 5 transceivers. SerDes phase locking across vendors is maintained via the common TI LMK62E2 reference clock ($f_{\text{ref}} = 156.25\text{ MHz}$) over RG405 SMA coaxial lines. High-speed differential data lanes (SS405, 2x TX, 2x RX) transport multi-gigabit serial data into the Trenz TE0422-03 module.
* **Form-Factor Conversion:** The Trenz TE0422-03 module outputs an SFP+ optical/electrical format, which is passively adapted to a QSFP28 form factor via the 10GTek WADQS-28-MEL converter before entering the AMD/Xilinx Kintex UltraScale+ transceiver RX pins.
* **Egress Stage ($\text{Kintex} \rightarrow \text{PC2}$):** The Kintex UltraScale+ FPGA performs deserialization, packet validation, and Network Coding decoding. High-throughput scatter-gather DMA engines transfer completed frames across a PCIe 3.0 x4 interface into PC2 host memory.

---

### 3.2 Throughput & Datapath Capacity Model

The internal datapath throughput capacity $C_{\text{pipeline}}$ is defined as a function of parallel bus width $W_{\text{bus}}$ and core clock frequency $f_{\text{core}}$:

$$C_{\text{pipeline}} = W_{\text{bus}} \times f_{\text{core}}$$

Given $W_{\text{bus}} = 1024\text{ bits}$:

* At $f_{\text{core}} = 156.25\text{ MHz}$:
  $$C_{\text{pipeline}} = 1024 \times 156.25 \times 10^6\text{ bits/s} = 160\text{ Gbps}$$

* At $f_{\text{core}} = 312.50\text{ MHz}$:
  $$C_{\text{pipeline}} = 1024 \times 312.50 \times 10^6\text{ bits/s} = 320\text{ Gbps}$$

This 320 Gbps internal capacity guarantees zero-drop line-rate processing for multi-gigabit physical link speeds, providing adequate headroom for hardware-assisted Network Coding overhead and real-time packet filtering operations.

---

### 3.3 Detailed Stage-by-Stage Processing Flow

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/HITL/figure3.png" title="Detailed Functional Flowchart" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 3: Detailed functional flowchart of the end-to-end hardware-in-the-loop (HITL) datapath across TX (Agilex 5) and RX (Kintex UltraScale+) processing pipelines.
</div>

#### Transmitter Pipeline: Terasic DE25 (Intel Agilex 5)
1. **Stage 1 - Host Ingress & DMA:** Workstation PC1 transmits raw Ethernet frames to the Hard Processor System (HPS). The HPS transfers packet buffers into the HPS DMA engine, streaming word-aligned payload directly to the FPGA core logic fabric.
2. **Stage 2 - Ethernet Ingress Deparser:** Extracts Ethernet MAC layer fields, strips Destination/Source MAC addresses and EtherType headers, and evaluates the Frame Check Sequence (CRC check) in parallel.
3. **Stage 3 - IP Ingress Deparser:** Identifies IP protocol version, header length, and 5-tuple addressing parameters. Strips the IP header and verifies IP header checksum integrity.
4. **Stage 4 - Filtering & Pipeline Sync:** Applies real-time rule match logic (e.g., 5-tuple packet filtering). Holds packet payload within a delay pipeline buffer while CRC and checksum verification flags settle.
5. **Stage 5 - Network Coding Offload:** Encodes valid payload blocks using linear combination coefficients and appends a structured Network Coding (NC) metadata header.
6. **Stage 6 - Packet Re-Encapsulation:** Re-encapsulates the encoded payload by prepending an updated IP header with recomputed checksum and re-attaching standard Ethernet MAC framing.
7. **Stage 7 - High-Speed SerDes Egress:** Parallelizes data across the 1024-bit wide datapath into the Intel Agilex 5 transceiver block for multi-gigabit serialization through the HSMC-XTS breakout board.

#### Receiver Pipeline: OpenSourceSDRLab KU5P (AMD/Xilinx Kintex UltraScale+)
1. **Stage 1 - High-Speed SerDes Ingress:** Receives multi-gigabit serial stream through the QSFP28 interface into Kintex UltraScale+ transceivers, deserializing into a 1024-bit parallel bus.
2. **Stage 2 - Ethernet Deparser:** Extracts and strips Ethernet encapsulation while performing parallel CRC validation.
3. **Stage 3 - IP Deparser:** Extracts and strips IP headers while evaluating IP header checksum integrity.
4. **Stage 4 - NC Header Deparsing:** Parses NC generation vectors and coding coefficients from the header and strips the NC header.
5. **Stage 5 - Network Coding Offload:** Executes Gaussian elimination decoding logic to reconstruct original raw packet payload blocks.
6. **Stage 6 - Packet Reconstruction:** Re-attaches valid IP headers with recalculated checksums and reconstructs standard Ethernet framing.
7. **Stage 7 - Host Egress & PCIe DMA:** Streams reconstructed packets directly to PC2 host memory via a high-performance PCIe 3.0 x4 scatter-gather DMA engine.
