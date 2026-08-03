---
layout: page
title: Congestion-Aware Buffer Management for TCP/NC Gateway Tunnels
description: "A cross-layer ingress buffer management and dead-packet suppression framework for TCP/NC protocol gateways, preventing buffer bloat and uncontrolled redundancy growth in lossy, congested networks. <br><br>**Keywords:** TCP/NC Tunnel, Transport Gateway Proxy, Cross-Layer Optimization, Buffer Management, Dead-Packet Suppression, Congestion Control, ns-3, C++"
img: assets/img/projects/tcpnctunnel/ThreeCases-001-001.jpg
importance: 1
category: work
related_publications: false
---

## 1. Executive Summary

Transmission Control Protocol with Network Coding (TCP/NC) dramatically improves reliable data transport over lossy channels by injecting mathematical redundancy (coded packets) into the pipeline. However, resource-constrained end-devices—such as Internet of Things (IoT) sensors or legacy hardware—cannot natively execute TCP/NC stacks due to processing and OS limitations. While deploying an **IP Tunneling Gateway Proxy (TCP/NC Tunnel)** solves this by handling network coding transparently at edge routers, standard gateway implementations suffer from catastrophic performance degradation during network congestion.

When congestion occurs, unmanaged gateways drop packets at the outbound link buffer ($B_{link}$). The underlying TCP/NC engine mistakes these queue drops for wireless channel fading and continuously escalates its coding redundancy factor ($R$). This creates a destructive feedback loop: **more redundancy amplifies congestion, triggering further drops and infinite buffer bloat**. Furthermore, stale end-to-end TCP retransmissions ("dead packets") clog gateway buffers, wasting memory and stalling end-to-end transport pipelines.

This project designs a **Cross-Layer Buffer Management Framework** for TCP/NC Gateway Tunnels. Implemented within a user-space Tunnel Handler operating over network interfaces, the framework introduces two primary engineering mechanisms:

1. **Cross-Layer Ingress Congestion Control:** Evaluates outbound link capacity ($B_{link}$) and TCP window space ($CWND$) before accepting incoming IP packets, preemptively dropping overloaded packets at the ingress TCP sending buffer ($B_{tcp}$) *before* network coding expansion occurs.
2. **4-State Dead-Packet Filtering:** Maintains a real-time Packet Information Table tracking packet arrival timestamps and sequence histories across four operational states to instantly identify and drop duplicate retransmissions ("dead packets").

Extensive discrete-event simulations in **ns-3** across multi-gateway topologies demonstrate that this buffer management strategy eliminates runaway redundancy loops. In heavily congested 12-session scenarios, the proposed gateway architecture maintains high application goodput where standard end-to-end TCP/NC collapses, providing a robust, production-ready proxy solution for edge networks.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/tcpnctunnel/Fig-ProtocolStack_TCPNCtunnel-001-001.jpg" title="TCP/NC Tunnel Gateway Architecture" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 1: Protocol stack and internal handler architecture of the transparent TCP/NC Tunnel Gateway Proxy.
</div>

---

## 2. Industry Metric & System Parameter Overview

The table below summarizes the key deployment settings, queue thresholds, and evaluation parameters engineered into the gateway system:

| Metric / Parameter Category | Industry Setting / Benchmark | Engineering Purpose & Functional Role |
| :--- | :--- | :--- |
| **Gateway Topology** | 3-Gateway Dumbbell Proxy Network | Connects up to 24 end-hosts across lossy intermediate router links (10 ms delay). |
| **Link Bandwidth & Capacity** | 1 Mbps Thin-Pipe Bottleneck | Modeled on bandwidth-constrained IoT backhauls and long-distance satellite links. |
| **Buffer Allocations** | $B_{link} = 100\text{ pkts}$ \| $B_{tcp} = 214\text{ pkts}$ | Outbound hardware link queue ($B_{link}$) vs. ingress software TCP queue ($B_{tcp}$). |
| **Link Loss Conditions** | 0% to 20% Random Loss Rate | Simulates severe wireless channel degradation on the intermediate gateway-to-gateway link. |
| **Ingress Control Check** | $A_L < \lceil R \rceil$ & $A_w \ge \text{Packet Size}$ | Gates incoming packets based on real-time link queue space ($A_L$) and window headroom ($A_w$). |
| **Dead-Packet Tracking Window** | 255-Record Sliding Table | Tracks historical sequence numbers and arrival timestamps ($T_d$) to detect redundant arrivals. |
| **Timeout Threshold Parameters** | $A = 1.0\text{ s}$ (Min RTT) \| $B = 2.0\text{ s}$ | Defines standard TCP retransmission timeout boundaries (RFC 6298) for packet state classification. |
| **Simulation Platform** | ns-3 Discrete-Event Engine | Validated across multi-session workloads (3 to 12 active TCP flows) with Monte Carlo averaging. |

---

## 3. System Architecture & Engineering Innovations

### 3.1 Transparent Tunnel Gateway Pipeline

The TCP/NC Tunnel Gateway operates as an edge proxy between local area networks (LANs) and lossy wide area network (WAN) backhauls. 

* **Zero-Modification End-Host Deployment:** End-devices (IoT sensors, servers) send standard IP/TCP traffic without modified protocol stacks.
* **Transparent Packet Encapsulation:** The gateway's user-space Tunnel Handler intercepts incoming IP packets from local interfaces, encapsulates them into TCP/NC tunnel streams, applies Forward Error Correction (FEC) redundancy, and forwards them across the gateway-to-gateway link.
* **Segmented Loss Isolation:** Packet losses across the middle-mile link are recovered locally between gateways. End-hosts remain unaware of channel drops and avoid premature congestion window reductions.

---

### 3.2 Cross-Layer Ingress Congestion Control

Standard gateways drop packets at the physical link queue ($B_{link}$) after network coding has already amplified the payload size. To prevent runaway queue growth, the Tunnel Handler performs a proactive cross-layer evaluation at the ingress TCP buffer ($B_{tcp}$):

> **Ingress Decision Logic:**  
> When a new packet $p$ arrives from the LAN, the handler inspects the current Redundancy Factor ($R$), available link buffer slots ($A_L$), and open TCP Congestion Window headroom ($A_w$).
> * If link buffer space is insufficient ($A_L < \lceil R \rceil$) AND TCP window headroom is open ($A_w \ge \text{size}(p)$), the handler **drops packet $p$ immediately at ingress**.
> * Dropping $p$ at $B_{tcp}$ signals congestion directly back to the local TCP sender via standard TCP mechanisms, preventing the NC layer from inflating link traffic with useless coded packets.

---

### 3.3 4-State Dead-Packet Filtering Pipeline

Duplicate TCP retransmissions ("dead packets") caused by upstream timeout events clog gateway queues. The handler filters these out by maintaining a 255-record **Packet Information Table** that categorizes arriving duplicate packets into one of four distinct states based on sequence history and elapsed time ($T_d$):

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/tcpnctunnel/ThreeCases-001-001.jpg" title="Packet Information Table State Classification" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 2: Four operational state classifications used by the gateway handler to filter out redundant dead packets.
</div>

* **Case 1 (Active Queue Pending):** The original packet is still waiting in $B_{tcp}$ to be coded. The arriving duplicate is redundant $\rightarrow$ **DROP**.
* **Case 2 (In-Flight / Downstream Loss):** The original packet was sent and acknowledged in this segment but dropped downstream. The duplicate is a valid retransmission $\rightarrow$ **FORWARD**.
* **Case 3 (In-Flight E2E Timeout Stale):** The original packet is actively traversing the network, but the end-to-end TCP sender timed out prematurely ($A \le T_d < B$). The duplicate is unnecessary $\rightarrow$ **DROP**.
* **Case 4 (Hard Packet Loss):** The original packet was lost permanently ($B \le T_d$) and requires full end-to-end recovery $\rightarrow$ **FORWARD**.

---

## 4. Performance Results & Benchmark Evaluation

The proposed buffer-managed TCP/NC Tunnel was benchmarked using **ns-3** against standard End-to-End TCP NewReno (E2E-TCP) and unmanaged End-to-End TCP/NC (E2E-TCP/NC) under varying link loss rates (0% to 20%) and session densities (3 vs. 12 concurrent flows).

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/tcpnctunnel/Result_03session-001-001.jpg" title="Goodput Performance in Low Congestion (3 Sessions)" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 3: System goodput across 3 active sessions: The proposed gateway achieves superior throughput over standard TCP as link loss increases.
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/tcpnctunnel/Result_12session_New20200305-001-001.jpg" title="Goodput Performance in Heavy Congestion (12 Sessions)" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 4: System goodput across 12 active sessions under heavy network congestion, demonstrating immunity to runaway coding redundancy.
</div>

---

### 4.1 Performance Analysis Under Heavy Congestion

| Session Density | Channel Loss Rate | Standard E2E-TCP | Unmanaged E2E-TCP/NC | Proposed TCP/NC Tunnel Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **3 Sessions (Low)** | 0.0 % (Loss-Free) | **0.95 Mbps** | 0.92 Mbps | 0.93 Mbps |
| **3 Sessions (Low)** | 10.0 % (High Loss) | 0.35 Mbps | **0.81 Mbps** | **0.82 Mbps** |
| **12 Sessions (High)** | 0.0 % (Loss-Free) | 0.88 Mbps | 0.12 Mbps *(Redundancy Collapse)* | **0.86 Mbps** |
| **12 Sessions (High)** | 10.0 % (High Loss) | 0.18 Mbps | 0.22 Mbps | **0.58 Mbps** |

* **Prevention of Redundancy Collapse:** Under 12-session congestion at 0% channel loss, unmanaged E2E-TCP/NC suffers severe throughput collapse ($0.12\text{ Mbps}$) because queue drops trigger unnecessary coding redundancy. The proposed gateway maintains **$0.86\text{ Mbps}$** by shedding load at ingress.
* **Loss Fading Resilience:** Under severe 10% link loss with 12 active sessions, the proposed buffer management framework yields **$0.58\text{ Mbps}$** aggregate goodput—outperforming both standard E2E-TCP ($0.18\text{ Mbps}$) and unmanaged E2E-TCP/NC ($0.22\text{ Mbps}$) by over **160%**.

---

## 5. Conclusion & Future Engineering Directions

This project solves the buffer bloat and uncontrolled redundancy challenges inherent to TCP/NC gateway proxies. By combining **cross-layer ingress congestion checks** with **4-state dead-packet filtering**, the architecture provides a scalable, enterprise-grade tunnel solution for resource-constrained edge networks.

### Key Engineering Takeaways:
1. **Targeted Ingress Dropping:** Preemptively dropping packets at the ingress TCP queue ($B_{tcp}$) prevents network coding from amplifying traffic during congestion.
2. **Dead-Packet Suppression:** Timestamped state tracking eliminates stale end-to-end retransmissions, preserving gateway queue capacity.
3. **Seamless Edge Proxy Integration:** Enables legacy IoT devices to achieve high transport performance over lossy wireless links without OS modifications.

### Future Expansion Roadmap:
* **Dynamic RTT Timestamp Integration:** Utilizing TCP Timestamp options ($TSopt$) to adaptively tune timeout parameters ($A$ and $B$) based on real-time RTT variance.
* **Congestion-Aware Redundancy Factor Scaling:** Dynamically throttling the Network Coding redundancy factor ($R$) when ingress congestion thresholds are breached.
* **Kernel-Level Gateway Offloading:** Porting the Tunnel Handler from user space to Linux kernel eBPF/XDP hooks for multi-gigabit throughput execution.
