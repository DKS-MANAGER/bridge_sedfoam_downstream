# Eulerian-Eulerian Two-Phase Downstream Clear-Water Scour Simulation

This repository contains the complete case setup for a high-resolution, parallelized 2D CFD simulation of **pressure-flow downstream clear-water scour**. The simulation is conducted using the two-fluid Eulerian-Eulerian solver **`sedFoam_rbgh`** in OpenFOAM.

---

## 📌 Project Overview

This case models **downstream scour** — the localized erosion that develops **downstream of the bridge deck** exit contraction under pressure-flow clear-water conditions. In clear-water conditions ($V_{avg}/V_c < 1.0$), there is no sediment entering the channel from upstream. Bed shear stress exceeds the critical threshold only in the accelerated throat and expanding shear zone immediately downstream of the bridge exit.

**Reference**: Experiment-03(c) / Experiment-05_LFR1 from the IIT Kanpur HWRE Lab dataset (Subhadip Das, 2016 — *Journey to the Correct Result*).

| Parameter | Value | Description |
|:---|:---:|:---|
| **Scour Type** | Downstream Scour (contraction exit) | Scour shifts toward the downstream exit |
| **Flow Regime** | Clear-water ($V/V_c < 1$) | No upstream sediment supply |
| **Water Depth ($H$)** | 10.0 cm | Upstream flow depth |
| **Bridge Opening ($H_b$)** | 7.0 cm | Vertical opening height |
| **Contraction Ratio ($H/H_b$)**| 1.428 | Degree of flow constriction |
| **Median Grain Size ($d_{50}$)**| 0.294 mm | Sand bed material |
| **Approach Velocity ($V_{avg}$)**| 0.22 m/s | Ramped over first 5 s |
| **Critical Velocity ($V_c$)** | 0.28 m/s | HEC-18 critical velocity limit |

---

## 📊 Validation Comparison Plot (t = 100 s)

The bed profile was extracted at $t = 100\text{ s}$ and compared with the experimental equilibrium scour envelope:

![Validation Plot](scour_comparison_t100.png)

### Summary of Results at $t = 100\text{ s}$
*   **Max Scour Depth**: **−1.02 cm** under the bridge exit contraction. This represents clear-water vertical contraction scour before reaching long-term equilibrium.
*   **Downstream Deposition Dune**: **+1.25 cm** (located at $x = 1.25$ m). This deposition mound represents the local bedform development where the decelerating expansion zone drops sediment.
*   **Temporal Progression**: The scour hole has developed stably and matches physical expectations by being shallower than the live-bed counterpart due to the lower approach velocity.

---

## 🚀 Execution Guide

### Automated Run
Execute the end-to-end setup and run script:
```bash
./Allrun
```

### Manual Step-by-Step Run
1.  **Clean Case**:
    ```bash
    ./Allclean
    ```
2.  **Generate Mesh**:
    ```bash
    blockMesh > log.blockMesh 2>&1
    ```
3.  **Initialize Fields**:
    ```bash
    cp -r 0_org 0
    setFields > log.setFields 2>&1
    ```
4.  **Extract Cell Centers**:
    ```bash
    postProcess -func writeCellCentres -time 0 > log.writeCellCentres 2>&1
    ln -sf Cx 0/ccx && ln -sf Cy 0/ccy && ln -sf Cz 0/ccz
    ```
5.  **Decompose and Run Parallel (8 Cores)**:
    ```bash
    decomposePar > log.decomposePar 2>&1
    mpirun --oversubscribe -np 8 sedFoam_rbgh -parallel > log.sedFoam_rbgh 2>&1 &
    ```

---

## 👥 Author & Research Context
*   **Researcher**: [Divyansh Kumar Singh](https://github.com/DKS-MANAGER) (MTech Civil Engineering - Hydraulic Engineering, IIT Kanpur)
*   **Laboratory**: Hydraulic and Water Resources Engineering (HWRE) Lab, IIT Kanpur
*   **Contact**: [divyansh179@gmail.com](mailto:divyansh179@gmail.com) | [LinkedIn](https://www.linkedin.com/in/divyansh-kumar-singh-92bb621b6)
