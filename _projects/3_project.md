---
layout: page
title: Scalable & Sustainable Linear WSNs via Erasure Coding TDMA
description: A proactive packet loss recovery framework integrating hop-by-hop Erasure Coding with optimized TDMA scheduling for long-distance linear infrastructure monitoring.
img: assets/img/projects/tdma-nc/Example-001-001.jpg
importance: 1
category: work
related_publications: false
---

## 1. Executive Summary

Linear Multi-Hop Wireless Sensor Networks (LWSNs) are critical for monitoring long-distance infrastructure—such as power grids, oil pipelines, railway tracks, and bridges—where wired connections are too expensive and long-range single-hop wireless communication is unreliable. However, signals along these long linear paths suffer from severe interference, fading, and packet loss that multiply across every hop. 

This project delivers a commercial-ready, high-reliability networking framework combining **Erasure Coding (EC)** with **Time Division Multiple Access (TDMA) deterministic scheduling**. Instead of relying on traditional retransmissions (ARQ) which waste bandwidth and drain batteries, intermediate sensor nodes mathematically blend multiple data packets into a redundant set of coded packets before forwarding them downstream. As long as a receiving node collects enough coded packets to match the original data count (e.g., receiving *any* 5 out of 8 transmitted packets), it completely reconstructs the original payload.

To make this practical for low-power microcontrollers, a central server uses a high-speed search algorithm powered by precomputed lookup tables to calculate optimal TDMA timeslot schedules in less than **2 milliseconds**. Extensive network simulations on a 30-node topology under extreme channel degradation (50% packet loss rate) prove that this framework achieves **over 99% end-to-end delivery success**, guarantees **10 days of battery life** on a standard 1000 mAh LiFePO4 battery, and enables **multi-year autonomous operation** when paired with micro solar panels.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/tdma-nc/Example-001-001.jpg" title="Distribution grid and local event monitoring in areas with developing infrastructure using wireless sensors." class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 1: Distribution grid and local event monitoring in areas with developing infrastructure using wireless sensors.
</div>

---

## 2. Industry Metric & System Parameter Overview

The table below outlines the core hardware parameters, network settings, and operational metrics designed into the system:

| Metric / Parameter Category | Industry Setting / Benchmark | Engineering Purpose & Functional Role |
| :--- | :--- | :--- |
| **Network Scale** | 30 Nodes (15-Hop Dual Paths) | Linear path split into two equal 15-node segments routing to dual edge gateways for load balancing. |
| **Channel Loss Rate** | 10% to 50% Link Loss Rate | Evaluated under both independent random losses and sustained burst fading scenarios. |
| **Traffic Load** | 5 Packets / Node / Minute | Standard telemetric sensing rate for remote infrastructure status and local event reporting. |
| **Delivery Reliability Target** | > 99.0% End-to-End Success | Strict QoS delivery threshold enforced across all nodes even during worst-case channel degradation. |
| **Frame Slot Timing** | 67.87 ms Total Slot Duration | Includes a 21.33 ms active packet transmission window and a 46.54 ms energy-saving guard gap. |
| **Hardware Power States** | Transmit: 62 mA \| Receive: 28 mA <br> Idle: 6.28 mA \| Sleep: 0.03 mA | Realistic radio power profile matching sub-1 GHz Wi-Fi HaLow / Wi-SUN wireless transceivers. |
| **Power Source & Lifespan** | 1000 mAh LiFePO4 Battery | Provides 10 to 12 days of continuous backup power; extends to 10+ years with micro solar harvesting. |
| **Controller Search Speed** | < 2.0 Milliseconds | Real-time central server execution time to compute updated TDMA slot schedules for all 30 nodes. |
| **Memory Footprint** | < 5 KB Buffer Space | Ultra-lightweight memory footprint easily fitting within standard low-power microcontrollers (e.g., ARM Cortex-M3). |

---

## 3. System Architecture & Engineering Innovations

### 3.1 End-to-End Topology & Communication Pipeline

The system architecture routes sensor payloads hop-by-hop toward a central server through dual edge Gateways (GW X and GW Y).

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/tdma-nc/NetworkModel-001-001.jpg" title="Linear Wireless Sensor Network Model" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 2: Dual-gateway linear network topology showing the optimal central link separation boundary and directional hop-by-hop forwarding paths.
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/tdma-nc/TDMA-15Nodes_TDMAexam_downlink_o-001-001.jpg" title="TDMA Scheduling for Uplink and Downlink" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 3: Deterministic TDMA schedule matrix demonstrating 3-hop spatial slot reuse to prevent collisions and seamless downlink control packet piggybacking.
</div>

* **Load-Balanced Dual-Gateway Topology:** The linear path is divided at an optimal "Link Separation" boundary. Nodes on the left side aggregate and forward traffic toward Gateway X, while nodes on the right forward toward Gateway Y. This cuts maximum path delay in half and balances battery drain across the network.
* **3-Hop Spatial Reuse Interference Management:** To eliminate wireless signal collisions without complex conflict graphs, transmissions occurring during the same time window are spaced at least 3 hops apart. This guarantees that two active nodes never interfere with each other's receivers.
* **Bidirectional Traffic Piggybacking:** Essential downlink control commands from the central server (e.g., schedule updates or threshold resets) are merged directly into the coded uplink data blocks. Nodes receive and process control instructions without needing a separate, dedicated downlink cycle that would drain extra power.
* **Proactive Erasure Coding:** Instead of retransmitting individual lost packets, each node combines locally generated data with received upstream packets into a larger set of linearly combined coded packets. A receiving node can reconstruct all original data packets as long as the total number of received coded packets equals or exceeds the original packet count.

---

### 3.2 Low-Power Energy Management & Slot Timing

Battery longevity is a non-negotiable requirement for remote monitoring devices. The network protocol incorporates an aggressive power-saving strategy tied directly to the TDMA slot design:

* **Hardware-Aware Sleep Transitions:** Operating radios in idle mode draws significant current (6.28 mA). The protocol continuously monitors the idle gap between active transmissions. Whenever this gap exceeds the micro-controller's hardware wakeup threshold (5 ms), the radio is immediately forced into ultra-low-power Sleep mode (0.03 mA).
* **Duty Cycle Optimization:** By calculating the exact minimum number of coded packets required to meet the target delivery success rate, the system minimizes active radio transmit and receive times.
* **Long-Term Field Sustainability:** With a standard 1000 mAh LiFePO4 battery, edge nodes handling the heaviest forwarding loads operate for over 10 consecutive days without charging. When integrated with a small 1-watt solar panel, recharging cycles take under 5 days, enabling maintenance-free continuous operation for over a decade.

---

### 3.3 Central Controller Optimization Pipeline

Calculating optimal slot allocations for a multi-hop network typically requires heavy matrix computations that can take hours. This system implements a fast **Hill Climbing (HC)** algorithm combined with precomputed lookup tables to perform dynamic online scheduling in real time.

### Algorithm 1: Joint Timeslot Alignment and Optimization

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/tdma-nc/Algorithm1.png" title="" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

#### Intermediate Node Packet Handling Pipeline
1. **Data Aggregation:** Intermediate nodes store their own sensor readings alongside incoming packet buffers received from upstream neighbors.
2. **On-the-Fly Encoding:** At the assigned TDMA transmission round, the node encodes the combined data buffer into a stream of linear combination packets.
3. **Scheduled Burst Transmission:** Coded packets are transmitted in assigned timeslots. The node immediately enters Sleep mode during all remaining inactive slots in the cycle.
4. **Hop-by-Hop Decoding:** The receiving node collects arriving coded packets. Once enough packets are captured, it runs a fast matrix decoding process to recover all original data bytes, ready to forward to the next hop.

#### Server Scheduling Algorithm
1. **Target Reliability Division:** The central server determines individual path target probabilities based on current loss conditions and traffic volume.
2. **Instant Table Lookup:** The server queries precomputed memory tables to instantly pull initial baseline slot configurations without running complex mathematical formulas.
3. **Slot Alignment (1-Step Hill Climbing):** If the initial schedule length differs from the battery lifetime slot limit, the controller greedily adds or subtracts timeslots where they provide the highest reliability gain or smallest loss.
4. **Local Search Refinement:** A lightweight local search algorithm adjusts slot allocations across active links until the overall end-to-end delivery success rate meets or exceeds the 99% reliability goal.
5. **Separation Balancing:** The controller periodically adjusts the central link separation boundary to ensure both gateways experience equal delay and power consumption.

---

## 4. Performance Results & Benchmark Evaluation

To validate the framework under real-world conditions, extensive Monte Carlo simulations (100 million system cycles) were conducted across a 30-node network (15–15 path topology) under both **Independent Random Loss** and **Bursty Channel Fading (PDA-Gilbert Model)**.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/tdma-nc/Result_M_evaluation_Random-001-001.jpg" title="Successful Delivery Probability in Random Loss Model" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 4: End-to-end packet delivery success probability across varying channel loss rates, comparing legacy Repeating Transmission (RT) against the proposed Erasure Coding (EC) method.
</div>

### 4.1 End-to-End Reliability: Erasure Coding vs. Legacy Retransmission

The benchmark results highlight a dramatic reliability advantage over legacy Repeating Transmission (RT) strategies:

| Channel Loss Rate (PLR) | Channel Model | Legacy Repeating Transmission (RT) | Proposed Erasure Coding (Data Only) | Proposed Erasure Coding (Data + Control Downlink) |
| :--- | :--- | :--- | :--- | :--- |
| **10% Loss (0.1)** | Random Fading | 0.0024 % (Nearly Fails) | **99.99 %** | **99.98 %** |
| **30% Loss (0.3)** | Random Fading | 0.0000 % (Complete Failure) | **99.98 %** | **99.97 %** |
| **50% Loss (0.5)** | Random Fading | 0.0000 % (Complete Failure) | **99.53 %** | **99.50 %** |
| **30% Loss (0.3)** | Burst Fading (Gilbert) | 0.0000 % (Complete Failure) | **99.98 %** | **99.97 %** |
| **50% Loss (0.5)** | Burst Fading (Gilbert) | 0.0000 % (Complete Failure) | **99.51 %** | **99.48 %** |

* **Resilience Under Severe Loss:** When link loss rates reach **50% (0.5)**, legacy retransmission protocols collapse completely (0% success) due to error accumulation across 15 hops. In contrast, the proposed Erasure Coding scheduler maintains **$> 99.5\%$ end-to-end delivery success**.
* **Zero Overhead Downlink Control:** Adding a downlink control packet to the coded payload causes a negligible drop in reliability ($< 0.03\%$), proving that central server signaling can piggyback on sensor uplink data without requiring extra timeslots.

---

### 4.2 Controller Search Speed & Resource Scalability

Deploying centralized scheduling on real-time network controllers requires high computational speed and low memory usage:

* **Sub-2 Millisecond Execution Speed:** The modified Hill-Climbing search algorithm combined with precomputed memory tables computes optimal schedules for a 36-node network in under **1.32 to 2.0 milliseconds**. This enables real-time dynamic re-scheduling when channel conditions suddenly change.
* **Minimal Node Memory Usage:** Intermediate nodes only store packets during active encoding or decoding windows. Total RAM consumption remains under **5 KB (40 to 50 packets max)**, easily fitting within standard low-power microcontrollers (such as ARM Cortex-M3 or ESP32).

---

### 4.3 Energy Distribution & Field Lifespan

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/tdma-nc/Result_Fairness_Probability_5cases-001-001.jpg" title="Per-node Successful Delivery Probability" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 5: Per-node delivery success probability across 15 hops under five non-uniform channel loss distribution scenarios, demonstrating system fairness.
</div>

* **Fairness Across Hops:** Because packets from distant nodes must cross more links, their individual loss risk is naturally higher. However, the slot allocation algorithm distributes redundant coded packets so that even the farthest node (15 hops away) achieves a delivery success rate exceeding **99.87%**.
* **Predictable Battery Drain:** Edge nodes closest to the gateways handle the heaviest relay traffic, consuming up to 955 Joules/day under worst-case 50% loss. A standard **1000 mAh LiFePO4 battery** guarantees **10 to 12 days** of continuous operation without solar input.
* **Multi-Year Autonomous Service:** When paired with a small solar panel, a 2000-cycle battery operating on a 5-day solar recharge cycle yields a practical field lifespan of **10+ years**, making the system maintenance-free for long-distance remote monitoring.

---

## 5. Conclusion & Future Engineering Directions

This project delivers a practical, highly resilient communication architecture for Linear Wireless Sensor Networks. By combining **hop-by-hop Erasure Coding**, **3-hop spatial slot reuse**, and a **sub-millisecond optimization search**, the system solves the long-standing trade-off between wireless link reliability, real-time control, and battery life.

### Key Engineering Takeaways:
1. **Proven High Reliability:** Maintains $> 99\%$ end-to-end delivery across 15 multi-hop links even when individual wireless channels experience 50% packet loss.
2. **Real-Time Controller Speed:** Central server schedule calculation completes in under 2.0 ms, making online dynamic adaptation feasible for large networks.
3. **Ultra-Low Energy Footprint:** Enables 10 days of continuous battery backup and 10+ years of solar-harvested field deployment on standard low-power MCU hardware.

### Future Expansion Roadmap:
* **Tree & Mesh Topology Generalization:** Expanding the dual-gateway linear scheduling model to Y-shaped and tree topologies (e.g., branching pipeline or river network monitoring).
* **Hardware-In-the-Loop (HITL) Deployment:** Implementing the protocol stack on physical sub-1 GHz Wi-Fi HaLow (IEEE 802.11ah) transceiver hardware for real-world field testbed evaluation.
* **AI-Driven Online Channel Estimation:** Integrating lightweight machine learning models at the gateway server to predict channel loss transitions and automatically trigger schedule updates before packet loss occurs.

---

## Appendix: Additional Simulation Results

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/tdma-nc/Result_M_evaluation_Gilbert-001-001.jpg" title="Successful Delivery Probability in PDA-Gilbert Loss Model" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure A1: End-to-end successful delivery probability under the PDA-Gilbert burst loss model.
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/tdma-nc/Result_NetworkDiameter-001-001.jpg" title="Network Diameter vs. Buffer Size" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure A2: Supportable network diameter (hops) versus node buffer size requirement under varying channel loss rates.
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/tdma-nc/Result_TimeConsumption-001-001.jpg" title="Computation Time Consumption" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure A3: Controller computation time consumption across different network topology sizes.
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/tdma-nc/Result_Fairness_Power_5cases-001-001.jpg" title="Per-node Daily Average Power Consumption" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure A4: Per-node daily average power consumption of the proposed method under varying link loss distribution cases.
</div>
