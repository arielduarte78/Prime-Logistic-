# PRIME LOGISTICS | Stochastic Risk & Topology Engine

## Quantification of Supply Chain Fragility through Hierarchical Cascade Modeling

---

## 1. Executive Summary (The "Why")

In industrial logistics, most planning systems fail because they assume ideal, static conditions. **PRIME LOGISTICS** is not simply a routing system; it is an **operational stress engine** that quantifies hidden risk within your supply infrastructure.

Using a **Dual Convergence Monte Carlo** simulation architecture, the engine generates thousands of "degradation scenarios" to identify not only network failure points, but also how delivery costs and times are affected under systemic pressure.

**Primary Purpose:** Enable organizations to measure and understand the true resilience of their logistics networks before a crisis occurs.

---

## 2. Current Capabilities (The "What")

### A. Multidimensional Topology Construction (Block 1)

Transforms raw operational data (coordinates, capacities, cost profiles) into a **balanced five-matrix model**:

| Matrix/Variable | Description |
|----------------|-------------|
| **Adjacency Matrix (A)** | Represents the physical connectivity of the network (which nodes are linked). |
| **Cost Matrix (C)** | Transport costs per arc, based on real distances (Haversine formula) and rates. |
| **Time Matrix (T)** | Transit times per arc, using stochastic speed profiles. |
| **Capacity Matrix (K)** | Physical flow limits (volume/weight) per connection. |
| **Demand Vector (D)** | Load balance between origin and destination nodes. |

### B. Stochastic Cascade Engine (Block 2)

Implements a **hierarchical risk propagation model** that simulates the reality of volatile markets:

1.  **Systemic Level (Macro):** Major disruptive events ("Black Swans") that activate a **Stress Index (S)**.
2.  **Tactical Level (Network):** Failures in specific nodes or connections, whose probability of occurrence depends on the systemic state.
3.  **Operational Level (Micro):** Daily noise and variability (e.g., traffic, local weather) whose intensity scales according to the environmental degradation level.

---

## 3. Mathematical Foundation (The "Logic")

### The Risk Intensity Model

The system's core is an **Aggregate Stress Index (S)**, which dynamically modifies network parameters in each simulation iteration:

```
S = Σ (wₖ · I{Sₖ active})
```

**Where:**
*   `wₖ` = Structural severity of event *k*.
*   `I{Sₖ active}` = Indicator function (1 if systemic event *k* is active, 0 otherwise).

Based on `S`, key network metrics (like mean transit time) are **recalculated in real-time**:

```
μ_effective = μ_base · (1 + α · S)
σ_effective = σ_base · (1 + β · S)
```

**Where:**
*   `μ_effective` = Adjusted mean (e.g., average delivery time).
*   `σ_effective` = Adjusted standard deviation (operational uncertainty).
*   `α`, `β` = Stress sensitivity factors.

### Fundamental Design Condition

The system ensures **`β > α`**. This means that, in a crisis, **uncertainty (`σ`)** grows faster than the **average delay (`μ`)**.

**Interpretation:** Mathematically models the **loss of predictability and operational control** characteristic of logistics crisis situations, where times not only worsen but become unpredictable.

---

## 4. System Architecture

The system is structured in **three modular blocks** that operate sequentially:

```
┌─────────────────┐    ┌───────────────────┐    ┌─────────────────────┐
│   BLOCK 1       │    │    BLOCK 2        │    │    BLOCK 3         │
│   Multidimensional │───▶│   Stochastic     │───▶│   Optimization     │
│   Topology      │    │   Risk Engine     │    │   (Future)         │
└─────────────────┘    └───────────────────┘    └─────────────────────┘
       │                         │                         │
       ▼                         ▼                         ▼
   Static Model            Cascade Scenario           Risk Mitigation
   Generation              Simulation                 Planning
   (A, C, T, K, D)
```

**Data Flow:** **Block 1** transforms raw logistics data into a structured network model. This model feeds **Block 2**, where it is subjected to thousands of simulated perturbations. The results of these simulations (distributions of costs, times, and critical failure points) are designed to inform the strategies of **Block 3**.

---

## 5. Implementation Rigor (Tech Specs)

### Sparse Matrix Handling
- **Technology:** Implemented with `scipy.sparse`.
- **Formats:** Strategic use of **CSR** (efficient operations) and **LIL** (flexible construction) formats.
- **Objective:** Allows modeling of large-scale networks (> 10,000 nodes) with O(n) memory efficiency, avoiding the O(n²) bottleneck of dense matrices.

### Dual Stochastic Convergence
The engine does not use a fixed number of iterations. Instead, it implements an **adaptive stopping criterion** based on the statistical stability of the simulation:
```
(Δσ² / σ²) < ε
```
**Where:**
- `Δσ²` = Change in the variance of the observed metric between simulation batches.
- `σ²` = Current variance of the metric.
- `ε` = Predefined tolerance threshold (e.g., 0.001).

**Advantage:** Ensures results are statistically representative without unnecessary computation cycles.

### Integrity Shielding
- **Mechanism:** The system includes internal resilience layers to handle degraded network states.
- **Function:** Automatically manages *fallback* between metrics when encountering extreme values (infinities) or total graph disconnections. Prioritizes maintaining connectivity (`A`) over capacity (`K`) in critical failure scenarios.

---

## 6. Project Status

| Block | Name | Status | Notes |
|--------|---------|---------|-------|
| **1** | Multidimensional Topology |  **Complete** | Stable generation of the 5-matrix model (A, C, T, K, D). |
| **2** | Stochastic Risk Engine |  **Complete** | Cascade simulation with operational dual convergence. |
| **3** | Optimization & Mitigation |  **In Conceptual Design** | Defining the "Minimum Regret Objective Function". |

---

## 7. Intellectual Property Notice

⚠️ **PRIVATE CODE | PUBLIC DOCUMENTATION**

- **Private IP:** The risk engine architecture, matrix mutation algorithms, and simulation core are **private Intellectual Property (IP)**.
- **Repository Purpose:** This space exposes the technical logic, mathematical foundations, and system capabilities for **audit, academic collaboration, and proof-of-concept demonstrations**. It does not contain the complete executable source code.

---

## 8. Contact

**Ariel Duarte** - Industrial Engineering Student   
- **LinkedIn:** www.linkedin.com/in/arielduarte-j*  
- **Email:** Arielduartejesus@gmail.com  

