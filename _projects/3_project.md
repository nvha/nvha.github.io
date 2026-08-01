---
layout: page
title: Scalable & Sustainable Linear WSNs via Erasure Coding TDMA
description: A proactive packet loss recovery framework integrating hop-by-hop Erasure Coding with optimized TDMA scheduling for long-distance linear infrastructure monitoring.
img: assets/img/projects/wsn/figure1.png
importance: 1
category: work
related_publications: false
---

## 1. Abstract

Linear multi-hop wireless sensor networks (LWSNs) are vital for monitoring critical long-distance infrastructure such as power distribution grids, oil pipelines, and bridges where wired interconnects and single-hop wireless links are cost-prohibitive. However, these networks suffer severe packet loss due to signal attenuation, fading, channel interference, and multi-hop accumulation. This project presents a proactive, highly reliable packet loss recovery architecture that integrates hop-by-hop **Erasure Coding (EC)** with **Time Division Multiple Access (TDMA) scheduling**.

The framework features a general $N_l\text{--}N_r$ dual-gateway model supporting concurrent sensor data uplink and control packet downlink within a unified TDMA cycle. By encoding $d$ original data packets into $c$ linear combination packets using Galois Field $\mathbb{GF}(2^8)$ arithmetic, intermediate nodes allow downstream receivers to fully reconstruct original messages as long as any $d$ out of $c$ coded packets arrive successfully. To satisfy strict energy constraints and real-time controller updates, a heuristic search algorithm based on **Hill Climbing (HC)** and precomputed lookup tables finds near-optimal timeslot allocations in microsecond scale. Extensive simulations on a 30-node topology under severe channel loss ($PLR = 0.5$) demonstrate end-to-end delivery success exceeding **0.99**. Power consumption models confirm an operational lifetime of **10 days** on a standard 1000 mAh LiFePO4 battery, achieving continuous multi-year operation when paired with micro solar harvesting units.

---

## 2. System Model & Parameter Inventory

The experimental linear sensor network model and scheduling parameters span four key functional domains:

| Parameter Category | Notation / Variable | Operational Definition & Specification |
| :--- | :--- | :--- |
| **Topology & Nodes** | $N, N_l, N_r$ | Total sensor nodes ($N = 30$), partitioned into left ($N_l = 15$) and right ($N_r = 15$) paths via an optimal link separation. |
| **Link & Channel** | $j, q_j$ | Link index ($0 \le j \le N$) and link packet loss rate ($0 \le q_j < 1$, evaluated up to $q_j = 0.5$). |
| **Traffic Load** | $d_i, d^{(j,t)}, c^{(j,t)}$ | Generated data packets at Node $i$ per cycle ($d_i = 5$), data packets forwarded on link $j$ at round $t$, and transmitted coded packets ($c^{(j,t)} \ge d^{(j,t)}$). |
| **TDMA Cycle** | $T, \mathcal{T}, t$ | Total timeslots per TDMA cycle ($T$), total system cycle duration ($\mathcal{T} = 60\text{ s}$), and transmission round index $t$. |
| **Slot Timing** | $\tau_{\text{slot}}, \tau_{\text{pkt}}, \tau_{\text{gap}}$ | Slot duration ($\tau_{\text{slot}} = 67.87\text{ ms}$), active Tx/Rx time ($\tau_{\text{pkt}} = 21.33\text{ ms}$), and gap duration ($\tau_{\text{gap}} = 46.54\text{ ms}$). |
| **Hardware State** | $P_{tx}, P_{rx}, P_{idle}, P_{sleep}$ | Power consumption states: Transmit ($62\text{ mA}$), Receive ($28\text{ mA}$), Idle ($6.28\text{ mA}$), and Sleep ($0.03\text{ mA}$) at $1\text{ V}$ nominal. |
| **MCU & Battery** | $P_{mcu}, \beta$ | Baseline MCU power draw ($3.88\text{ mA}$) and battery capacity ($1000\text{ mAh}$, $3.2\text{ V}$ LiFePO4 chemistry). |
| **Target Quality** | $M_{\text{target}}$ | Target end-to-end delivery success probability threshold ($M_{\text{target}} \ge 0.99$). |

---

## 3. System Architecture

### 3.1 End-to-End Pipeline Setup

The LWSN architecture routes sensor payload hop-by-hop toward central server S via dual edge Gateways (GW X and GW Y).

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/wsn/figure1.png" title="Linear Wireless Sensor Network Model" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 1: Linear Wireless Sensor Network model illustrating dual gateways (GW X and GW Y), link separation boundary, and hop-by-hop directional forwarding paths.
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/wsn/figure2.png" title="TDMA Scheduling for Uplink and Downlink" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 2: TDMA-based timeslot allocation matrix for concurrent data uplink and control downlink, maintaining a 3-hop spatial separation rule to eliminate signal collisions.
</div>

* **Topology Partitioning ($\text{Path}_l \text{ and } \text{Path}_r$):** The linear path is split at a central "Link separation" point into two oppositely directed routes. Path $\text{Path}_l$ comprises nodes $\{0, \ldots, N_l-1\}$ forwarding leftward to GW X, while $\text{Path}_r$ comprises nodes $\{N_l, \ldots, N-1\}$ forwarding rightward to GW Y.
* **3-Hop Spatial Reuse Constraint:** To avoid co-channel signal collisions under two-hop interference models, adjacent nodes in the same forwarding path cannot transmit simultaneously. Active links within the same transmission round $t$ are scheduled with a minimum 3-hop physical separation ($\mathbf{I}_l^{(t)}$ and $\mathbf{I}_r^{(t)}$).
* **Integrated Bidirectional Flow:** Downlink control packets (e.g., configuration updates sent at $t = -1$) are merged into systematic coded blocks at edge nodes and transmitted alongside uplink sensor payloads without degrading uplink throughput.
* **Erasure Coding Execution:** At each intermediate hop, received data packets are combined with locally generated sensor data. Node $j$ generates a set of $c^{(j,t)}$ coded packets $\mathbf{C}^{(j,t)}$ using coefficients $\delta_{n,m}$ derived from a Galois Field $\mathbb{GF}(2^8)$ Vandermonde matrix:

$$c^{(j,t)}_n = \begin{cases} \text{d}^{(j,t)}_n, & n \in \{0, 1, \ldots, d^{(j,t)} - 1\} \\ \sum_{m=1}^{d^{(j,t)}} \delta_{n,m} \times \text{d}^{(j,t)}_m, & n \in \{d^{(j,t)}, \ldots, c^{(j,t)} - 1\} \end{cases}$$

---

### 3.2 Lifetime, Energy & Slot Dimensioning Model

To guarantee target system operation time ($LT_{\text{target}}$) while accommodating burst-loss channel characteristics, the available timeslot capacity $T_{\text{limit}}$ within cycle $\mathcal{T}$ is strictly bounded by node energy constraints.

The timeslot duration $\tau_{\text{slot}} = \tau_{\text{pkt}} + \tau_{\text{gap}}$ is configured based on the minimum hardware threshold $\tau_{\text{th}}$ required to enter low-power Sleep mode. The node average power consumption $E(\tau_{\text{max}})$ over $\tau_{\text{max}} = \frac{\mathcal{T}}{T}$ is modeled across two distinct radio power regimes:

$$\text{Case A (Sleep Mode, } \tau_{\text{max}} - \tau_{\text{pkt}} \ge \tau_{\text{th}}\text{): } E_{\text{sleep}}(\tau_{\text{max}}) = \frac{\tau_{\text{pkt}}}{3}(P_{tx} + P_{rx} - 2P_{sleep}) + P_{sleep}\tau_{\text{max}}$$

$$\text{Case B (Idle Mode, } 0 < \tau_{\text{gap}} < \tau_{\text{th}}\text{): } E_{\text{idle}}(\tau_{\text{max}}, \tau_{\text{gap}}) = \frac{\tau_{\text{pkt}}}{3}(P_{tx} + P_{rx} - 2P_{sleep}) + \frac{2\tau_{\text{gap}}}{3}(P_{idle} - P_{sleep}) + P_{sleep}\tau_{\text{max}}$$

Defining hardware constants $A = P_{sleep} + P_{mcu}$, $B = \frac{\tau_{\text{pkt}}}{3}(P_{tx} + P_{rx} - 2P_{sleep})$, and $C = \frac{2}{3}(P_{idle} - P_{sleep})$, the overall network lifetime $LT$ is constrained by:

$$LT_1(\tau_{\text{max}}) = \frac{\beta}{A + \frac{B}{\tau_{\text{max}}}} \quad \text{and} \quad LT_2(\tau_{\text{max}}, \tau_{\text{gap}}) = \frac{\beta}{A + \frac{B + C \cdot \tau_{\text{gap}}}{\tau_{\text{max}}}}$$

By selecting $\tau_{\text{max}_{\min}} = \max(\tau_{\text{burst}}, \tau_{\text{life}})$ to balance burst-loss tolerance and lifetime targets, the maximum allowable frame dimension in timeslots is fixed at:

$$T_{\text{limit}} = \left\lfloor \frac{\mathcal{T}}{\tau_{\max_{\min}}} \right\rfloor$$

---

### 3.3 Detailed Stage-by-Stage Processing Flow

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/wsn/figure3.png" title="Joint Timeslot Alignment and Optimization Flowchart" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 3: Detailed functional flowchart of the Joint Timeslot Alignment and Optimization search algorithm (Algorithm 1) executing on the central server controller.
</div>

#### Transmit Node & Scheduling Pipeline
1. **Sensor Ingress & Buffer Aggregation:** Sensor Node $i$ periodically samples local telemetry ($d_i$ packets per cycle $\mathcal{T}$) and stores payloads alongside forwarded packets received from upstream neighbors.
2. **Galois Field Encoding ($\mathbb{GF}(2^8)$):** At assigned transmission round $t$, the node retrieves $d^{(j,t)}$ buffered packets and applies Vandermonde matrix multiplication over $\mathbb{GF}(2^8)$ to synthesize $c^{(j,t)}$ coded packets.
3. **TDMA Slot Transmission:** Coded packets are transmitted over link $j$ within assigned timeslots $\tau_{\text{slot}}$. During unassigned or silence slots, radio interfaces transition to low-power Sleep mode to minimize energy consumption.
4. **Hop-by-Hop Decoding & Egress:** The receiving node evaluates incoming coded packets. As long as $k \ge d^{(j,t)}$ valid packets arrive, Gaussian elimination decodes all original data packets, which are forwarded toward the gateway.

#### Central Controller Optimization Pipeline (Algorithm 1)
1. **Stage 1 - Target Probability Derivation:** The central controller computes path-specific target probabilities $M_l^{\text{target}}$ and $M_r^{\text{target}}$ weighted by traffic load $D_l$ and $D_r$:

$$M_l^{\text{target}} = \left( M_{\text{target}} \right)^{\frac{D_l}{D_l + D_r}}$$

2. **Stage 2 - Base Configuration Initialization:** The controller queries a precomputed $c$-table using current worst-case link loss parameters $q^{(t)}$ and block size $d^{(t)}$ to generate initial baseline slot allocations $\mathbf{C}_l^{\text{base}} = \{c^{(t)}\}$.
3. **Stage 3 - Timeslot Alignment (1-HC):** If the initial frame length $T(\mathbf{C}_l^{\text{base}})$ deviates from $T_{\text{limit}}$, a modified 1-step Hill Climbing algorithm iteratively increments slots (+1-HC with max reliability gain) or decrements slots (-1-HC with min reliability loss) until $T(\mathbf{C}_*^{\text{base}}) = T_{\text{limit}}$.
4. **Stage 4 - Local Search Optimization ($k$-HC):** Starting from $\mathbf{C}_*^{\text{base}}$, a $k$-step Hill Climbing search evaluates candidate slot configurations using a Move-Set table and an $M^{(t)}$-lookup table to maximize delivery probability $M(\mathbf{C}_l)$:

$$\text{Find } \mathbf{C}_l \quad \text{s.t.} \quad M(\mathbf{C}_l) = \prod_{t=0}^{N_l-1} M^{(t)} \ge M_l^{\text{target}} \quad \text{with } T(\mathbf{C}_l) = T_{\text{limit}}$$

5. **Stage 5 - Link Separation Selection:** The link separation index $K$ is dynamically adjusted around the midpoint ($K = \lceil \frac{N}{2} \rceil$) to balance path delays ($T_l \approx T_r$) and equalize energy consumption across edge nodes.
