---
layout: about
title: About
permalink: /
subtitle: Principal Network Systems & Hardware Architect | Ph.D.

profile: false # Disables the profile image/card

selected_papers: true
social: true

announcements:
  enabled: false
  scrollable: true
  limit: 5

latest_posts:
  enabled: false
---

**Phoenix, Arizona** &nbsp;|&nbsp; [nvha.dtvt@gmail.com](mailto:nvha.dtvt@gmail.com) &nbsp;|&nbsp; *U.S. Permanent Resident*

---

I am a **Principal Network Systems & Hardware Architect (Ph.D.)** with 8+ years of technical R&D leadership. My core value proposition is a seamless **2-in-1 capability**: architecting scale-out network transport protocols and realizing them directly in ultra-high-speed FPGA silicon.

| <i class="fa-solid fa-network-wired"></i> TRACK 1: Network Systems & Protocol Architecture | <i class="fa-solid fa-microchip"></i> TRACK 2: FPGA Acceleration & Hardware Engineering |
| :--- | :--- |
| **Low-Latency Transport:** TCP/NC, Loss-Masking Proxies, AQM | **RTL & Logic Design:** SystemVerilog, Verilog, DSP Pipelines |
| **Programmable Data Planes:** P4-16, INT Telemetry, BMv2 | **Ultra-Wide Datapaths:** 1024-bit / 320 Gbps AXI4-Stream |
| **SDN & Control Planes:** OpenFlow, Ryu Controller, Dynamic Routing | **Offload Engines:** PCIe 3.0x4 DMA, APB Memory-Mapped CSRs |
| **Network Simulation:** ns-3 (C++/Python), Mininet, Scapy | **Silicon Platforms:** Intel Agilex 5/7, Stratix, AMD Xilinx Kintex |

> **Cross-Functional Leadership:** Operating natively across traditional functional boundaries—capable of leading dedicated protocol/software groups, RTL design teams, or cross-functional HW/SW Co-Design divisions to transform complex protocol mathematics into production hardware & software architectures.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/dual_architect.jpg" title="Networking & FPGA Synergy" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption text-center">
Figure 1: The HW/SW Co-Design Engine: Fusing scale-out network transport protocol architecture directly with physical FPGA silicon acceleration. <em>(Illustration generated via Gemini AI)</em>.
</div>

---

### Key Metrics at a Glance

* **8+ Years** of Advanced R&D in Full-Stack Hardware & Network Systems Architecture
* **320 Gbps** Internal FPGA Pipeline Capacity (1024-bit Datapath @ 312.5 MHz)
* **7x SDN Load Balancing** over OSPF & **36% Multicast Transfer Time Reduction** via P4
* **>99% Network Reliability** Sustained Under Severe 50% Link-Loss Conditions
* **40+ Peer-Reviewed Publications** (IEEE / IEICE / Scopus) & **$100,000+** in PI Research Grants

---

### Core Engineering Pillars

#### 1. Network Systems, Protocols & SDN (Primary Focus)
* **High-Speed Transport Protocols:** Loss-masking transport frameworks (TCP/NC), custom ACK accumulation state machines, proactive FEC, duplicate ACK generators, and tail-latency elimination.
* **Programmable Data Planes (P4) & Observability:** P4-16 in-band telemetry (INT) in ns-3 and BMv2 with custom 8-byte control headers (`Success_Check`) for microsecond-precision queue/jitter tracking and localized recirculation port fast-reroute packet recovery.
* **SDN & Control Planes:** Self-healing dynamic routing protocols (SDN-FEDRP on RYU OpenFlow controllers) yielding a 7x load-balancing improvement over standard OSPF, paired with discrete-event network modeling in **ns-3** and **Mininet**.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/HITL/Fig-Case2-OSPF-001-001.jpg" title="SDN-FEDRP on RYU OpenFlow controllers" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 2: Path allocation using the Fairness-Enhanced Dynamic Routing Protocol (SDN-FEDRP).
</div> 

#### 2. Hardware Architecture & FPGA Acceleration
* **Full-Stack RTL & Logic Design:** SystemVerilog/Verilog, high-speed DSP pipelines, 512-bit wide AXI4-Stream interfaces (operating at 375MHz+), APB interfaces, and pre/post-silicon validation.
* **FPGA Datapaths & Offloading:** Ultra-wide 1024-bit packet processing pipelines, host-to-FPGA PCIe 3.0x4 DMA streaming, and custom protocol offload engines.
* **Silicon Platforms & Emulation:** Heterogeneous Hardware-in-the-Loop (HITL) multi-board link emulation, real-time AWGN hardware generators (sweeping SNR from −30 dB to 45 dB), 4×4 MU-MIMO channel emulators, and 2×2 MIMO-OFDM baseband transceivers across **Intel Agilex 5 (DE25)**, **Intel Agilex 7**, **Stratix II/III/IV**, and **AMD Xilinx Kintex UltraScale+ (KU5P)** FPGAs with **TI LMK62E2** SerDes reference clocking.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/HITL/figure1.png" title="Conceptual Block Diagram" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 3: Conceptual system block diagram illustrating Heterogeneous Hardware-in-the-Loop (HITL) multi-board link emulation.
</div>

---

### Research Leadership & Academic Background

I earned my Ph.D. in Computer Science & Systems Engineering from **Kyushu Institute of Technology** (Japan), serving as Principal Investigator (PI) for two competitive JSPS KAKEN research grants focusing on resilient protocol stacks and network reliability. 

As Laboratory Supervisor and Co-founder of the **ICTLAB** at the **University of Science** (Vietnam), I directed 20+ graduate and undergraduate researchers, managed multi-year government and industry R&D grants ($100,000+ funded), and authored **40+ peer-reviewed international publications** earning 3 international Best Paper Awards. 

Additionally, I have designed and delivered specialized graduate-level lectures on SDN architectures, OpenFlow control mechanisms, and Ryu/Mininet network topologies for the **City College of New York (CCNY)**.

---

<img src="/assets/img/skills/cpp_logo.webp" style="height: 32px; vertical-align: middle; margin-right: 16px;"><img src="/assets/img/skills/sv_logo.jpg" style="height: 32px; vertical-align: middle; margin-right: 16px;"><img src="/assets/img/skills/python_logo.webp" style="height: 32px; vertical-align: middle; margin-right: 16px;"><img src="/assets/img/skills/ns-3_logo.png" style="height: 32px; vertical-align: middle; margin-right: 16px;"><img src="/assets/img/skills/packettracer_logo.webp" style="height: 32px; vertical-align: middle; margin-right: 16px;"><img src="/assets/img/skills/wireshark_logo.webp" style="height: 32px; vertical-align: middle; margin-right: 16px;"><img src="/assets/img/skills/Linux_logo.png" style="height: 32px; vertical-align: middle; margin-right: 16px;"><img src="/assets/img/skills/Quartus_logo.png" style="height: 32px; vertical-align: middle; margin-right: 16px;"><img src="/assets/img/skills/vivado_logo.png" style="height: 32px; vertical-align: middle; margin-right: 16px;"><img src="/assets/img/skills/Matlab_logo.png" style="height: 32px; vertical-align: middle; margin-right: 16px;"><img src="/assets/img/skills/P4_logo.webp" style="height: 32px; vertical-align: middle; margin-right: 16px;"><img src="/assets/img/skills/openflow_icon.webp" style="height: 32px; vertical-align: middle; margin-right: 16px;"><img src="/assets/img/skills/ryu_logo.png" style="height: 32px; vertical-align: middle;">

---
