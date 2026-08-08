---
layout: page
title: Fairness-Enhanced Dynamic Routing Protocol for SDN Architectures (SDN-FEDRP)
description: "A centralized OpenFlow-based dynamic routing engine that optimizes link utilization, avoids network congestion, and enforces traffic fairness via real-time flow reallocation and DFS topology discovery. <br><br>**Keywords:** Software-Defined Networking (SDN), OpenFlow, RYU Controller, Dynamic Routing, Load Balancing, Traffic Engineering, Mininet, Python"
img: assets/img/projects/sdn_fedrp/Fig-Case2-FEDRP-001-001.jpg
importance: 1
category: work
related_publications: false
---

## 1. Executive Summary

Traditional IP routing protocols like Open Shortest Path First (OSPF) rely on static link metrics (such as link bandwidth or cost) to calculate forwarding paths. Because these algorithms ignore real-time link utilization, multiple data flows frequently pile onto the same "shortest path" while adjacent links remain completely idle. This leads to hot-spot link congestion, elevated queueing delays, packet drops, and degraded Quality of Service (QoS). Even when deployed within Software-Defined Networks (SDN), basic adapted versions of OSPF fail to dynamically reroute active flows once paths are initially established.

This project introduces **Fairness-Enhanced Dynamic Routing Protocol in SDN (SDN-FEDRP)**, a centralized control-plane routing architecture built for OpenFlow-enabled networks. Implemented as an application on the **RYU SDN Controller**, SDN-FEDRP continuously monitors network-wide telemetry and dynamically rebalances active flows across available multi-path topologies. The system introduces three key engineering innovations:

1. **Topology Path Discovery via DFS:** Leverages Depth-First Search (DFS) on the controller to dynamically discover all physically available multi-path routes between source-destination pairs.
2. **Piecewise Link Utilization Costing:** Computes link costs using a steeply escalating non-linear function tied to real-time link utilization ($u = \text{used}/\text{capacity}$). As a link approaches capacity ($u \ge 0.9$), its penalty cost spikes dramatically, steering new and existing flows away from impending bottlenecks.
3. **Variance-Minimizing Flow Reallocation Engine:** Periodically evaluates flow throughputs and executes re-pathing maneuvers when flow volume shifts by more than 10%. By minimizing the throughput variance ($\sigma$) across all network paths, SDN-FEDRP achieves near-zero load variance across parallel links.

Evaluated on the **Mininet** network emulation platform across multi-path switch topologies, SDN-FEDRP completely prevents link congestion during heavy multi-flow bursts, improving network load-balancing fairness by up to **7x** over standard SDN-OSPF while maintaining full compatibility with OpenFlow hardware switches.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/sdn_fedrp/SdnExample2SdnNetwork-001-001.jpg" title="Software-Defined Network (SDN) Architecture" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 1: Centralized Software-Defined Network (SDN) architecture decoupling the control plane from data-plane forwarding devices.
</div>

---

## 2. Industry Metric & System Parameter Overview

The table below outlines the network topology specifications, control-plane thresholds, and performance metrics engineered into the SDN routing architecture:

| Metric / Parameter Category | Industry Setting / Benchmark | Engineering Purpose & Functional Role |
| :--- | :--- | :--- |
| **Control Plane Framework** | RYU SDN Controller (Python APIs) | Logically centralized controller managing data-plane switch flow tables via OpenFlow. |
| **Emulation Platform** | Mininet Network Emulator | Real-time virtual network running native kernel switches, links, and Linux host stacks. |
| **Switch & Data Plane** | OpenFlow v1.3 Virtual Switches | Hardware-abstracted forwarding devices reacting to flow entries installed by RYU. |
| **Topology Scale** | 8-Switch Multi-Path Core | 3 parallel core paths (10 Mbps capacity per link, 10 ms delay) connecting host clusters. |
| **Traffic Workloads** | UDP Flow Generator (1 to 9 Mbps) | Multi-host concurrent flow generation to evaluate bottlenecking and path convergence. |
| **Reallocation Threshold** | $> 10\%$ Throughput Variance | Prevents route flapping by re-pathing flows only when flow bandwidth shifts significantly. |
| **Load Balancing Metric ($\sigma$)** | Path Throughput Variance Ratio | Quantifies network fairness; lower $\sigma$ indicates optimal, even bandwidth distribution. |

---

## 3. System Architecture & Engineering Innovations

### 3.1 Architecture Overview & OpenFlow Control Pipeline

SDN-FEDRP decouples the network control plane from forwarding hardware, enabling centralized path computation and dynamic flow-table programming.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/sdn_fedrp/Fig-Topology-001-001.jpg"  title="Mininet Multi-Path Evaluation Topology" class="img-fluid rounded z-depth-1" responsive=false %}
    </div>
</div>
<div class="caption">
    Figure 2: Multi-path evaluation topology featuring three distinct switch paths (Path 1, Path 2, Path 3) between host edge clusters.
</div>

* **Centralized Telemetry Collection:** The RYU controller regularly polls OpenFlow switch port statistics to calculate exact bandwidth consumption ($f$) and link utilization ($u$) across every link in real time.
* **DFS Path Discovery:** Upon discovering new hosts or topology changes, the controller runs a Depth-First Search (DFS) algorithm to index all valid, loop-free multi-path candidates between ingress and egress switches.

---

### 3.2 Piecewise Non-Linear Link Costing

Standard static metrics fail to penalize heavily loaded links until packet drops occur. SDN-FEDRP implements a **piecewise link-cost function** that scales exponentially as link utilization ($u$) increases:

* **Low Utilization ($u < 33\%$):** Linear low cost; links are treated as preferred candidates.
* **Moderate Utilization ($33\% \le u < 90\%$):** Moderate linear cost scaling to distribute background traffic evenly.
* **Near Capacity ($90\% \le u < 100\%$):** Sharp penalty scaling ($\text{cost factor} \times 70$), making the link highly undesirable for new incoming flows.
* **Overloaded / Congested ($u \ge 100\%$):** Extreme penalty scaling ($\text{cost factor} \ge 500$), forcing the path finder to instantly bypass the link.

---

### 3.3 Dynamic Flow Reallocation & Fairness Engine

When a new flow enters the network or an active flow changes rate, static routing fails to adjust previously established routes. SDN-FEDRP incorporates an active **Flow Reallocation Engine**:

```text
[New Flow Arrival / Telemetry Poll]
         │
         ▼
[Run DFS: Compute All Available Paths]
         │
         ▼
[Calculate Path Costs via Link Utilization Function]
         │
         ▼
[Evaluate Load Balancing Variance (σ) Across Candidate Path Sets]
         │
         ▼
[Select Path Set Minimizing σ] ───► Shift Active Flows (>10% Change) ───► [Update OpenFlow Flow Tables]
```

* **Variance Minimization ($\sigma$):** Computes the variance of average throughput across all core paths. The system selects the global path configuration that yields the lowest variance ($\sigma \approx 0$).
* **Mitigating Packet Reordering:** To avoid TCP packet reordering issues caused by frequent path switching, flows are reallocated **only if their measured throughput fluctuates by more than 10%**.

---

## 4. Performance Results & Benchmark Evaluation

SDN-FEDRP was benchmarked against standard static OSPF and dynamic **SDN-OSPF** across multiple traffic scenarios on Mininet.

| Scenario / Traffic Workload | Evaluation Metric | Legacy Static OSPF | Dynamic SDN-OSPF | Proposed SDN-FEDRP |
| :--- | :--- | :--- | :--- | :--- |
| **Scenario 1: 4 Sequential Flows** | Bottleneck Congestion Event | Severe Loss (Path 2 Overloaded) | Severe Loss (12 Mbps on 10 Mbps Link) | **Zero Congestion (Flows Rerouted)** |
| **Scenario 2: 6 Concurrent Flows** | Load Variance ($\sigma$) | High Variance ($\sigma = 1.63$) | High Variance ($\sigma = 1.63$) | **Optimal Balance ($\sigma \approx 0.00$)** |
| **Scale Benchmark (6 Flows)** | Path Load Distribution | Unbalanced (9 / 8 / 10 Mbps) | Unbalanced (9 / 8 / 10 Mbps) | **Perfect Balance (9 / 9 / 9 Mbps)** |

---

### 4.1 Congestion Avoidance & Flow Rerouting (Scenario 1)

In Scenario 1, four sequential flows were initiated across hosts h1–h4 with varying throughputs (9 Mbps, 4 Mbps, 5 Mbps, and 7 Mbps).

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/sdn_fedrp/Fig-Case1-OSPF-001-001.jpg" title="SDN-OSPF Path Allocation Failure" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 3: SDN-OSPF path allocation resulting in link overload on Path 2 when Flow 4 arrives.
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/sdn_fedrp/Fig-Case1-FEDRP-001-001.jpg" title="SDN-FEDRP Dynamic Flow Reallocation" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 4: SDN-FEDRP dynamically shifts Flow 2 to Path 3 upon Flow 4's arrival, preventing link overload completely.
</div>

* **SDN-OSPF Failure:** When Flow 4 (7 Mbps) arrives on Path 2 (which already carries Flow 2 at 4 Mbps), aggregate demand reaches 12 Mbps on a 10 Mbps physical link, causing heavy packet drops.
* **SDN-FEDRP Success:** Detects the impending bottleneck, calculates path costs, and automatically reroutes Flow 2 to Path 3. All four flows execute cleanly without a single dropped packet.

---

### 4.2 Multi-Flow Load Balancing & Fairness (Scenario 2)

Scenario 2 evaluated six concurrent flows across the three core paths to test total network load-balancing degree ($\sigma$).

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/sdn_fedrp/Fig-Case2-OSPF-001-001.jpg" title="SDN-OSPF Load Imbalance" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 5: SDN-OSPF static load distribution resulting in unbalanced link utilization across the core.
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/sdn_fedrp/Fig-Case2-FEDRP-001-001.jpg" title="SDN-FEDRP Perfectly Balanced Path Allocation" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 6: SDN-FEDRP flow reallocation resulting in perfectly equalized 9 Mbps loads across all three core paths.
</div>

* **Unbalanced SDN-OSPF:** Loads core paths unequally (Path 1: 9 Mbps, Path 2: 8 Mbps, Path 3: 10 Mbps), producing a high load-variance degree ($\sigma = 1.63$).
* **Fair SDN-FEDRP:** Reallocates background flows dynamically, balancing every path to exactly **9 Mbps** ($\sigma \approx 0.00$), yielding a **7x improvement in fairness**.

---

### 4.3 Controller Calculation Overhead

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/sdn_fedrp/time_estimate-001-001.jpg" title="Controller Calculation Time Scaling" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 7: Controller path calculation time versus total active flow count.
</div>

While SDN-FEDRP requires additional CPU computation at the controller compared to static lookup, path computation time remains well under sub-second thresholds for standard flow densities, easily fitting within operational SDN control-plane performance budgets.

---

## 5. Conclusion & Future Engineering Directions

SDN-FEDRP solves the fundamental limitation of static routing in Software-Defined Networks by combining **DFS multi-path discovery**, **utilization-based cost scaling**, and **fairness-driven flow reallocation**.

### Key Engineering Takeaways:
1. **Proactive Congestion Avoidance:** Eliminates hot-spot bottlenecks by dynamically rerouting traffic before links reach physical capacity.
2. **7x Fairness Improvement:** Equalizes bandwidth distribution across parallel core paths ($\sigma \approx 0$).
3. **Seamless OpenFlow Deployment:** Operates purely within the control plane on standard OpenFlow switches without modifying host application stacks.

### Future Expansion Roadmap:
* **Hierarchical Controller Scale-Out:** Distributing the DFS search and flow reallocation algorithms across a cluster of distributed SDN controllers (e.g., ONOS / OpenDaylight).
* **P4 Hardware Acceleration:** Offloading real-time link utilization polling and flow-table matching directly into P4 programmable data-plane ASICs.
* **TCP Reordering-Aware Rerouting:** Integrating micro-burst detection to align flow reallocation timings with TCP quiet periods, guaranteeing zero out-of-order packet delivery.
