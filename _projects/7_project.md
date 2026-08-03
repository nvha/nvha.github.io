---
layout: page
title: In-Band Loss Recovery for Reliable Multicast via P4 Programmable Data Planes
description: "A hardware-transparent, P4-based in-band loss detection and local retransmission framework that accelerates reliable multicast transfers and eliminates end-to-end recovery delays in lossy data center networks. <br><br>**Keywords:** P4 Language, Programmable Data Planes, In-Band Loss Recovery, Reliable Multicast, UFTP, Recirculation Queues, ns-3, C++"
img: assets/img/projects/p4_multicast/Recirculation-001-001.jpg
importance: 1
category: work
related_publications: false
---

## 1. Executive Summary

Modern data centers, cloud infrastructure, and IoT edge networks rely heavily on simultaneous data distribution—such as container image replication, database synchronization, and firmware updates across thousands of servers. While unicast communication wastes network bandwidth and causes severe bottlenecking, standard **Reliable Multicast Protocols** (e.g., UFTP) mitigate traffic duplication. However, these protocols remain acutely vulnerable to packet loss: even low link loss rates ($10^{-3}$ to $10^{-2}$) trigger long-distance end-to-end retransmission loops back to the root source, causing severe completion time inflation and network queue degradation.

This project engineers a **P4-enabled In-Band Hop-by-Hop Loss Detection and Retransmission Framework** deployed directly onto programmable switch data planes. Instead of relying on end-to-end recovery timeouts, intermediate P4 switches detect dropped multicast packets in real time and execute microsecond-scale retransmissions locally from switch memory. The system introduces three core data-plane innovations:

1. **8-Byte L2.5 In-Band Control Header:** Embeds a compact header between L2 and L3 containing localized sequence numbers and a **40-bit sliding bitmap** (`Success_Check`) to convey link-local reception state on reverse-path packets.
2. **P4 Recirculation-Based Local Buffer Engine:** Utilizes switch port recirculation loops to hold unacknowledged packets in hardware queue pipelines without needing host OS modifications or specialized ASIC buffer modifications.
3. **Transparent Switch-to-Switch Loss Recovery:** Executes peer-to-peer retransmissions between adjacent switches, completely isolating end-host applications from physical link degradation.

Evaluated using discrete-event simulations in **ns-3** across a 41-node data center topology delivering a 1 GB file payload across 10 concurrent multicast groups (Scenario 3), the P4-enabled recovery framework maintains completely flat, loss-immune completion times, accelerating file transfer delivery while reducing aggregate core wire volume compared to end-to-end retransmission baseline protocols.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/p4_multicast/Recirculation-001-001.jpg" title="P4 Recirculation Pipeline Buffer Engine" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 1: P4 hardware implementation utilizing egress-to-ingress recirculation port queues for localized packet buffering and retransmission.
</div>

---

## 2. Industry Metric & System Parameter Overview

The table below summarizes the target hardware profiles, header specifications, and performance metrics engineered into the P4 programmable data plane system:

| Metric / Parameter Category | Industry Setting / Benchmark | Engineering Purpose & Functional Role |
| :--- | :--- | :--- |
| **Data Plane Hardware** | P4-Programmable ASIC / Switch | Implements line-rate packet parsing, stateful table matching, and header modification. |
| **Network Fabric Capacity** | 10 Gbps Core / 1 Gbps Access | Modeled on high-throughput data center pod and fabric switch interconnects. |
| **Control Header Overhead** | 8 Bytes total (L2.5 Sandwich) | Features a 1-byte `Type`, 1-byte `Packet_ID`, 1-byte `ACK_ID`, and 5-byte `Success_Check`. |
| **Local Loss Tracking Window** | 40-Bit Reception Bitmap | Tracks status of 40 most recent packets per link to instantly isolate non-consecutive drops. |
| **Buffering Architecture** | Loop-Back Recirculation Queues | Holds active packets in switch pipeline loops until link-local ACKs arrive from downstream. |
| **Multicast Scale (Scenario 3)** | 10 Concurrent Groups (3 Sinks/Group) | High-density concurrent multi-group traffic stress testing multi-tenant data center load. |
| **Target Protocol Engine** | UFTP-Based Reliable Multicast | Handles end-host payload reassembly without requiring protocol or kernel modifications. |
| **Simulation Platform** | ns-3 Discrete-Event Engine | Validated across 100 Monte Carlo iterations with 90% statistical confidence verification. |

---

## 3. System Architecture & Engineering Innovations

### 3.1 8-Byte L2.5 In-Band Control Header

To avoid heavy out-of-band signaling protocols, the system introduces a minimal 8-byte **Control Header** inserted between the Layer 2 Ethernet header and Layer 3 IP header.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/p4_multicast/Sw2SwHeader-001-001.jpg" title="L2.5 Control Header Format" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 2: In-band Control Header layout inserted between Layer 2 and Layer 3 headers for switch-to-switch loss feedback.
</div>

* **Type Field (1 Byte):** Differentiates between standard Data, ACK, Piggybacked Data-ACK, Retransmission, and Border Node Null packets.
* **Link-Local Sequence Tracking (`Packet_ID` & `ACK_ID`):** Maintains independent sequence numbering scoped strictly to individual physical links, avoiding end-to-end sequence space collision.
* **40-Bit Reception Bitmap (`Success_Check`):** Transmits a 5-byte bitmap encoding the exact arrival status of the last 40 packets up to the current `ACK_ID`. A single reverse ACK fully restores loss history even during burst ACK drops.

---

### 3.2 P4 Recirculation-Based Local Buffer Engine

Standard programmable switch ASICs lack arbitrary packet hold timers or deep DRAM storage abstractions. The architecture solves this by leveraging **Port Recirculation Queues**:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/p4_multicast/Illustration_k2opt-001-001.jpg" title="Switch-Level In-Band Loss Detection and Recovery Flow" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 3: Hop-by-hop loss detection timeline showing localized switch retransmissions and recirculation clearing upon link ACK reception.
</div>

* **Hardware Loop-Back Processing:** Transmitted packets pass through the Egress Pipeline and loop back into the Ingress Pipeline via dedicated recirculation ports.
* **Stateful Match-Table Lookups:** While in the recirculation loop, packets are evaluated against an active `loss_list` table populated by reverse ACK bitmaps.
* **Selective Release & Retransmission:** When a downstream switch confirms packet arrival via bitmap, the packet is cleared from recirculation. If marked as lost, the egress pipeline clones and retransmits the frame instantly across the link.

---

### 3.3 Transparent Hop-by-Hop Loss Isolation

Because loss recovery occurs entirely within the physical switch hop, end-host application logic remains completely untouched:

* **Out-of-Order Delivery Tolerance:** Reliable multicast layers (like UFTP) process payloads in section blocks, allowing intermediate P4 switches to forward recovered frames out of order without stalling host TCP or UDP stacks.
* **Zero Host Overhead:** End-host servers do not generate intermediate NACKs or participate in hop-by-hop tracking, freeing server CPU cycles for application execution.

---

## 4. Performance Results & Benchmark Evaluation (Scenario 3 Focus)

The proposed P4-enabled architecture was evaluated in **ns-3** across a 41-node data center topology (10 Gbps core links, 1 Gbps access links) under **Scenario 3 (10 Multicast Groups, 3 Sinks per Group)** delivering a **1 GB file transfer**.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/p4_multicast/Topology-001-001.jpg" title="Data Center Simulation Topology" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 4: 41-node multi-switch data center evaluation topology with 108 unidirectional 10 Gbps links.
</div>

---

### 4.1 Completion Time Immunity Under Multi-Group Load (Case 3)

In Scenario 3, ten concurrent multicast groups compete for link resources, aggravating end-to-end retransmission bottlenecks in standard networks.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/p4_multicast/SimFig_CompTimeCase3-001-001.jpg" title="Transfer Completion Time Comparison (Scenario 3: 10 Multicast Groups)" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 5: Total completion time for Scenario 3 (10 multicast groups delivering 1 GB) as link loss rates scale from 0.0001 to 0.01.
</div>

* **Flat Completion Time Scaling:** Under standard UFTP multicast in Scenario 3, completion time inflates significantly as link loss reaches $1\%$. The P4-enabled scheme maintains completely flat completion times across all loss rates.
* **Multi-Tenant Loss Resilience:** Hop-by-hop retransmission isolates losses locally per link, ensuring that dense concurrent multi-group workloads execute without global tree stalls.

---

### 4.2 Network Resource & Wire Overhead Analysis (Case 3)

Evaluating total transmitted packet counts and byte volumes confirms that link-local ACK overhead remains an efficient trade-off for zero latency degradation.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/p4_multicast/SimFig_TotalPacketCase3-001-001.jpg" title="Total Transferred Packets Comparison (Scenario 3)" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 6: Aggregate packet count transmitted across the fabric for Scenario 3 (with and without switch-to-switch ACK packets).
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/p4_multicast/SimFig_TotalByteCase3-001-001.jpg" title="Total Transferred Data Volume Comparison (Scenario 3)" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 7: Aggregate transmitted wire volume (in bytes) across the fabric for Scenario 3.
</div>

* **Wire Overhead Efficiency:** While small switch-to-switch ACKs increase total packet counts (Figure 6), the aggregate data volume (in total bytes, Figure 7) transmitted by the P4 scheme remains nearly identical to traditional multicast even at high loss rates, completely preventing end-to-end NACK storms across core switches.

---

## 5. Conclusion & Future Engineering Directions

This project demonstrates the power of P4 programmable data planes to solve fundamental multicast scaling bottlenecks. By executing **in-band loss detection** and **recirculation-based local retransmission** directly within switch hardware, the framework eliminates end-to-end recovery delays and guarantees loss-immune performance for multi-recipient data center applications.

### Key Engineering Takeaways:
1. **Loss-Immune Multicast Performance:** Eliminates completion time inflation across multi-group concurrent workloads (Scenario 3).
2. **Hardware-Native Buffering:** Utilizes P4 recirculation queues to achieve stateful packet holding without modifying switch ASIC architectures.
3. **Zero Host Modification:** Operates transparently to endpoints, preserving full compatibility with standard reliable multicast applications.

### Future Expansion Roadmap:
* **Tofino ASIC Deployment:** Porting P4_16 pipeline logic to physical Intel Tofino hardware switches for multi-terabit testbed evaluation.
* **Dynamic Recirculation Queue Sizing:** Incorporating telemetry-driven queue management to dynamically throttle recirculation loop limits during heavy switch port congestion.
* **SmartNIC Offloading:** Offloading the L2.5 header parsing and bitmap tracking to host SmartNICs (e.g., NVIDIA BlueField) for edge-to-edge hardware acceleration.
