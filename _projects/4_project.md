---
layout: page
title: Bi-Directional Loss-Tolerant Transport Architecture via Network Coding (TCP/NCwBLT)
description: "A high-performance transport-layer framework combining Transport FEC with SACK Bitmaps, Receiver Tail Loss Probes (TLP), and Virtual DupACK Synthesis for severe bi-directional lossy networks. <br><br>**Keywords:** Transport Layer FEC, TCP Optimization, SACK Bitmaps, Receiver TLP, RACK Signal Emulation, Network Protocols, ns-3, C++"
img: assets/img/projects/tcpnc/Fig-ProtocolStack-001-001.jpg
importance: 1
category: work
related_publications: false
---

## 1. Executive Summary

Standard Transmission Control Protocol (TCP) assumes that all packet drops stem from network congestion, mistakenly throttling its Congestion Window (CWND) and severely reducing throughput when operating over lossy wireless or constrained links. While combining Network Coding (NC) with TCP inserts mathematical redundancy to recover lost data packets without retransmission, legacy TCP/NC architectures break down under **bi-directional packet loss**—specifically when return Acknowledgments (ACKs) are lost in bursts. Lost ACKs corrupt channel loss estimation, cause excessive bandwidth waste through unnecessary redundancy, or stall the pipeline into catastrophic TCP Timeouts.

This project engineers **TCP/NC with Bi-directional Loss Tolerance (TCP/NCwBLT)**, a enterprise-grade transport layer protocol designed for high reliability across severe, bursty, and lossy communication channels. Implemented as an intermediary shim layer between TCP and IP, TCP/NCwBLT introduces three major architectural innovations:

1. **Accumulative Loss Tracking (32-bit PLS):** Piggybacks a 32-bit arrival history bitmap onto reverse ACKs to maintain complete loss awareness at the sender even when multiple consecutive ACKs are dropped.
2. **Timer-Driven ACK Retransmission:** Executes lightweight 200 ms receiver-side ACK retransmissions during return-path blackout periods, preventing sender pipeline stalls.
3. **Proxy Duplicate ACK Generator:** Synthesizes local duplicate ACKs at the sender's NC layer based on bitmap feedback, instantly triggering Fast Retransmit and Fast Recovery without waiting for delayed reverse-path ACKs.

Extensive discrete-event simulations in **ns-3** across both 1 Mbps constrained links and 100 Mbps high-speed paths demonstrate that TCP/NCwBLT substantially outperforms standard TCP variants (SACK, Westwood+) and previous state-of-the-art TCP/NC frameworks. Under severe 10% data loss combined with 20% burst ACK loss, TCP/NCwBLT maintains steady application goodput, improves network bandwidth efficiency by **15%** (0.67 vs 0.58 goodput-to-throughput ratio), and significantly accelerates total file transfer completion times.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/tcpnc/Fig-ProtocolStack-001-001.jpg" title="Network Coding Shim Layer Insertion" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 1: Transparent integration of the Network Coding (NC) shim layer between standard TCP and IP protocol layers.
</div>

---

## 2. Industry Metric & System Parameter Overview

The table below outlines the target deployment parameters, protocol overheads, and operational performance metrics engineered into the architecture:

| Metric / Parameter Category | Industry Setting / Benchmark | Engineering Purpose & Functional Role |
| :--- | :--- | :--- |
| **Network Environments** | 1 Mbps Constrained & 100 Mbps High-Speed | Evaluated across lossy narrow-band bottleneck links (10 ms delay) and high-bandwidth pipes. |
| **Channel Loss Profile** | 5% to 20% Data & ACK Loss Rates | Tested under both independent random packet drops and sustained Gilbert-Elliot burst fading. |
| **Protocol Architecture** | Transparent Shim Layer (TCP-IP) | Operates seamlessly between standard TCP applications and IP routing without kernel API changes. |
| **Header Overhead** | 52 Bytes total (4 Bytes added) | Integrates a minimal 32-bit Packet Loss Sequence (PLS) bitmap into standard TCP/NC ACK headers. |
| **ACK Retransmit Timer** | 200 ms Receiver Timeout | Prevents sender window stalls and catastrophic TCP Timeouts during reverse-link blackout bursts. |
| **Congestion Control Cores** | TCP NewReno / TCP Westwood+ | Evaluated with standard loss-based (NewReno) and high-speed bandwidth-estimation (Westwood+) cores. |
| **Bandwidth Efficiency** | 0.67 GP/TP Ratio (vs 0.58 Baseline) | Maximizes usable payload throughput per byte transmitted by dynamically optimizing coding redundancy. |
| **Simulation Platform** | ns-3 Discrete-Event Engine | Validated via 50-run Monte Carlo iterations with 99% statistical confidence interval verification. |

---

## 3. System Architecture & Engineering Innovations

### 3.1 Architecture Pipeline & Protocol Layering

The system introduces a specialized Network Coding (NC) shim layer positioned transparently between the standard TCP transport layer and the IP network layer.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/tcpnc/Fig-Topology-001-001.jpg" title="Lossy Backbone Evaluation Topology" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 2: End-to-end simulation topology featuring a lossy backbone network with independent bi-directional packet degradation.
</div>

* **Transparent Protocol Insertion:** Upper-layer application software and standard TCP congestion control logic operate without modification. The NC layer inspects outgoing TCP segments, generates redundant mathematical packet combinations, and strips headers upon receipt at the destination.
* **Sliding-Window Linear Combination:** Sender-side encoding combines sliding windows of original payload packets into redundant coded streams using low-overhead Galois Field matrix operations, enabling real-time micro-controller execution with minimal CPU overhead.
* **Early Loss Recovery:** As long as the receiver captures a sufficient count of coded packets matching the original segment count, it decodes the payload instantly—masking channel loss completely from the TCP layer and preventing unwarranted CWND reductions.

---

### 3.2 Bi-Directional Loss Tolerance Mechanics

When return-path ACK packets are lost in bursts, standard TCP/NC algorithms miscalculate channel conditions and overestimate data loss, injecting excessive redundancy that clogs the link. TCP/NCwBLT resolves this using three synchronized mechanisms:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/tcpnc/Illustration_LossSequence-001-001.jpg" title="Packet Loss Sequence (PLS) Updating" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 3: Real-time update of the 32-bit Packet Loss Sequence (PLS) bitmap at the receiver, recovering lost context after multi-ACK drops.
</div>

* **Accumulative Loss Tracking (PLS Bitmap):** The receiver embedding a 32-bit Packet Loss Sequence (PLS) bitmap into every outgoing NC-ACK header. This bitmap records the exact arrival or drop status of the last 32 packet combinations. Even if several ACKs drop in succession, the arrival of a single subsequent ACK fully informs the sender of all historical channel events.
* **Timer-Driven ACK Retransmission:** During severe reverse-link fading bursts, the receiver runs a dedicated 200 ms timer. If no new outgoing ACK is generated before the timer expires, the node retransmits the last valid ACK, preventing sender pipeline stalls and avoiding catastrophic TCP Timeout resets.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/tcpnc/Illustration_FakeAck-001-001.jpg" title="Duplicate ACK Generator Operation" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 4: Comparison showing how synthesized proxy duplicate ACKs at the sender immediately trigger Fast Retransmit and Fast Recovery without delay.
</div>

* **Proxy Duplicate ACK Generator:** When a arriving ACK contains a PLS bitmap indicating lost packets, the sender's NC layer locally synthesizes and feeds proxy duplicate ACKs upward into the TCP stack. This tricks the local TCP core into immediately launching Fast Retransmit and Fast Recovery cycles, maintaining high pipeline utilization without waiting for delayed reverse-path feedback.

---

### 3.3 High-Speed Link & Congestion Control Adaptability

Standard TCP engines (like NewReno) recover window sizes too slowly on high-bandwidth lossy pipes (e.g., 100 Mbps). TCP/NCwBLT features a modular architecture that easily integrates with bandwidth-estimating TCP cores:

* **TCP Westwood+ Core Integration:** By pairing TCP/NCwBLT with TCP Westwood+, the system measures actual packet arrival rates to estimate link capacity rather than relying purely on loss events, enabling rapid throughput recovery on high-speed paths.
* **Precise Redundancy Control:** By separating ACK losses from actual data losses using the PLS field, the sender avoids inflating redundancy ratios unnecessarily, preserving valuable link bandwidth for real payload traffic.

---

## 4. Performance Results & Benchmark Evaluation

The proposed architecture was evaluated through Monte Carlo simulations (50 runs per scenario) on **ns-3** under both **Random Loss** and **Gilbert-Elliot Burst Loss** conditions across forward and reverse directions.

| Channel Loss Profile | Data Loss ($r_d$) | ACK Loss ($r_a$) | Legacy TCP SACK | Legacy TCP Westwood+ | Previous TCP/NC Variant | Proposed TCP/NCwBLT |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Random Loss** | 5.0 % | 10.0 % | ~0.15 Mbps | ~0.18 Mbps | 0.62 Mbps | **0.84 Mbps** |
| **Random Loss** | 10.0 % | 20.0 % | ~0.02 Mbps | ~0.04 Mbps | 0.41 Mbps | **0.78 Mbps** |
| **Burst Loss ($L=2$)** | 5.0 % | 10.0 % | ~0.10 Mbps | ~0.12 Mbps | 0.55 Mbps | **0.81 Mbps** |
| **Burst Loss ($L=4$)** | 10.0 % | 20.0 % | ~0.00 Mbps | ~0.01 Mbps | 0.28 Mbps | **0.69 Mbps** |

---

### 4.1 End-to-End Goodput Resilience

Under severe bi-directional burst loss conditions (10% forward loss, 20% reverse ACK loss, burst length L=4), standard TCP variants collapse due to recurring timeouts. 

* **Maintain High Throughput:** Legacy TCP SACK and Westwood+ stall completely (<0.01 Mbps). Legacy TCP/NC degrades sharply to 0.28 Mbps due to corrupted loss estimates causing excessive redundant transmissions. TCP/NCwBLT maintains a strong, stable goodput of **0.69 Mbps**.
* **ACK Loss Immunity:** Thanks to the 32-bit PLS bitmap and 200 ms ACK retransmission timer, reverse-link ACK drops cause virtually zero throughput penalty on the forward data path.

---

### 4.2 Bandwidth Efficiency & CWND Stability

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/tcpnc/ThroughputVsGoodput.png" title="Bandwidth Efficiency (Goodput vs Throughput)" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 5: Ratio of useful application goodput relative to total physical throughput across varying burst loss conditions.
</div>

* **Optimized Redundancy Ratio:** Previous TCP/NC schemes inject unnecessary redundancy packets under ACK loss, achieving a Goodput-to-Throughput (GP/TP) efficiency ratio of only **0.58**. TCP/NCwBLT achieves a high GP/TP ratio of **0.67**, preventing network link bloat.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/tcpnc/Result_CWND_r005_l4.png" title="Congestion Window Dynamics Over Time" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 6: Congestion Window (CWND) trace showing rapid recovery and shorter total completion time for a 100 MB file transfer.
</div>

* **Rapid CWND Recovery:** The Proxy Duplicate ACK Generator maintains smooth congestion window scaling. In a 100 MB file transfer benchmark, TCP/NCwBLT completed the payload delivery significantly faster than legacy frameworks while preventing pipeline stalls.

---

### 4.3 High-Speed 100 Mbps Benchmark Performance

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/tcpnc/Result_Gilbert_l2_Dataloss_100Mbps.png" title="100 Mbps High-Speed Performance Comparison" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 7: Protocol goodput performance operating over 100 Mbps high-speed lossy links across varying loss rates.
</div>

When deployed on high-speed 100 Mbps links, standard loss-based TCP variants fail to utilize available bandwidth once loss exceeds 0.1%. Combining TCP/NCwBLT with a **TCP Westwood+** core delivers superior throughput scalability, maintaining high link utilization even under severe multi-megabit loss conditions.

---

## 5. Conclusion & Future Engineering Directions

This project successfully addresses the critical vulnerability of Network Coding transport protocols under bi-directional and bursty ACK loss conditions. By introducing **accumulative PLS bitmap tracking**, **timer-based ACK retransmissions**, and a **proxy duplicate ACK generator**, TCP/NCwBLT guarantees robust, high-efficiency data delivery across unstable network links.

### Key Engineering Takeaways:
1. **Bi-Directional Loss Tolerance:** Maintains optimal end-to-end goodput across lossy channels even when return ACK links experience up to 20% burst loss.
2. **High Bandwidth Efficiency:** Eliminates over-coding overhead, achieving a superior 0.67 goodput-to-throughput ratio compared to legacy TCP/NC implementations.
3. **Modular Transport Stack Integration:** Integrates seamlessly with standard TCP congestion engines (NewReno, Westwood+) without requiring application-level modifications.

### Future Expansion Roadmap:
* **Linux eBPF / XDP Kernel Implementation:** Porting the NC shim layer into native Linux kernel hooks (eBPF/XDP) for real-world high-throughput server deployment.
* **Multipath & Reordering Optimization:** Extending the PLS tracking logic to handle out-of-order packet arrival dynamics over multi-path wireless routers.
* **Adaptive AI Redundancy Tuning:** Incorporating lightweight online machine learning at the sender to proactively forecast loss burstiness and dynamically adjust Galois Field coding rates.
