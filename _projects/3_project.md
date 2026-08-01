---
layout: page
title: Scalable & Sustainable Linear WSNs via Erasure Coding TDMA
description: A proactive packet loss recovery framework integrating hop-by-hop Erasure Coding with optimized TDMA scheduling for long-distance linear infrastructure monitoring.
img: assets/img/projects/wsn/figure1.png
importance: 1
category: work
related_publications: false
---

## 1. Executive Summary

Linear Multi-Hop Wireless Sensor Networks (LWSNs) are critical for monitoring long-distance infrastructure—such as power grids, oil pipelines, railway tracks, and bridges—where wired connections are too expensive and long-range single-hop wireless communication is unreliable. However, signals along these long linear paths suffer from severe interference, fading, and packet loss that multiply across every hop. 

This project delivers a commercial-ready, high-reliability networking framework combining **Erasure Coding (EC)** with **Time Division Multiple Access (TDMA) deterministic scheduling**. Instead of relying on traditional retransmissions (ARQ) which waste bandwidth and drain batteries, intermediate sensor nodes mathematically blend multiple data packets into a redundant set of coded packets before forwarding them downstream. As long as a receiving node collects enough coded packets to match the original data count (e.g., receiving *any* 5 out of 8 transmitted packets), it completely reconstructs the original payload.

To make this practical for low-power microcontrollers, a central server uses a high-speed search algorithm powered by precomputed lookup tables to calculate optimal TDMA timeslot schedules in less than **2 milliseconds**. Extensive network simulations on a 30-node topology under extreme channel degradation (50% packet loss rate) prove that this framework achieves **over 99% end-to-end delivery success**, guarantees **10 days of battery life** on a standard 1000 mAh LiFePO4 battery, and enables **multi-year autonomous operation** when paired with micro solar panels.

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
        {% include figure.liquid loading="eager" path="assets/img/projects/wsn/figure1.png" title="Linear Wireless Sensor Network Model" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 1: Dual-gateway linear network topology showing the optimal central link separation boundary and directional hop-by-hop forwarding paths.
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/wsn/figure2.png" title="TDMA Scheduling for Uplink and Downlink" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 2: Deterministic TDMA schedule matrix demonstrating 3-hop spatial slot reuse to prevent collisions and seamless downlink control packet piggybacking.
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

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/wsn/figure3.png" title="Joint Timeslot Alignment and Optimization Flowchart" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 3: Operational workflow of the central server optimization algorithm, showing lookup table initialization, slot alignment, and local search refinement.
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
