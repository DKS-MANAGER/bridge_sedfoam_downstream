# Eulerian-Eulerian Two-Phase Downstream Clear-Water Scour Simulation

This repository contains the complete case setup for a high-resolution, parallelized 2D CFD simulation of **pressure-flow downstream scour** under a bridge deck. The simulation is conducted using the two-fluid Eulerian-Eulerian solver **`sedFoam_rbgh`** in OpenFOAM.

---

## 📌 Project Overview
The objective of this simulation is to resolve the detailed fluid-sediment mechanics, sediment suspension, contraction scour, and bed morphology under pressure-flow conditions. When high river discharge causes the water level to exceed the lower chord (soffit) of a bridge deck, it creates a vertical flow contraction. The resulting localized velocity acceleration and turbulence generation drive massive scour.

This case models **Experiment-03(c)** from the reference paper, characterized by a clear-water scour regime ($V_a / V_c = 0.84 < 1.0$, where approach flow velocity is less than the critical sediment entrainment velocity). Scouring is localized under the bridge and shifts downstream of the deck due to decelerating flow expansion.

---

## ⚙️ Dependencies
*   **OpenFOAM**: Version `2412` (OpenCFD / www.openfoam.com).
*   **Solver**: `sedFoam_rbgh` (two-fluid Eulerian solver for sediment-water mixtures).
*   **Utilities**:
    *   `ParaView` (v5.12+ recommended) for 3D/2D visualization.
    *   `Python 3` (with `numpy` and `struct`) for post-processing binary data.

---

## 📐 Geometry & Mesh
*   **Domain Dimensions**: Length $x \in [0, 3.5]\text{ m}$, Height $y \in [-0.10, 0.10]\text{ m}$.
*   **Erodible Bed Depth**: $10.0\text{ cm}$ (initialized from $y = -0.10\text{ m}$ to $y = 0.0\text{ m}$).
*   **Bridge Deck Location**: Blocked out from $x \in [1.0, 1.15]\text{ m}$, $y \in [0.070, 0.10]\text{ m}$.
*   **Bridge Opening chord ($H_b$)**: $7.0\text{ cm}$ (giving an opening ratio $H_b/Y = 0.70$).

### Schematic Diagram of Domain Layout
```
                                 x = 1.0 m       x = 1.15 m
       +----------------------------+               +----------------------------+ y = 0.10 m
       |                            |  Bridge Deck  |                            |
       |       Water column         |  [BLOCKED]    |                            |
       |       (Phase B)            +---------------+                            | y = 0.070 m (Soffit)
       |                                                                         |
       |                                                                         |
=======+=========================================================================+ y = 0.0 m (Bed Interface)
       :                                                                         :
       :                       Erodible Sand Bed (Phase A)                       :
       +-------------------------------------------------------------------------+ y = -0.10 m (Flume Floor)
      x = 0.0 m                                                                 x = 3.5 m
```
*   **Mesh Configuration**:
    *   **Grid Count**: Exactly $198,000\text{ cells}$ ($1000 \times 198 \times 1$).
    *   **Refinement**: Highly dense grid mapping along the sediment-water interface ($y = 0.0\text{ m}$) and in the downstream expansion zone ($x \in [1.15, 3.0]\text{ m}$) to capture downstream scour profiles.
    *   **Generation Tool**: Structured multi-block mesh generated via `blockMesh`.

---

## 🧪 Physics & Solver Setup
The case is solved using the Eulerian-Eulerian two-phase equations, treating water as Phase B (fluid) and sediment as Phase A (solid dispersed grains).

*   **Governing Model**: Frictional kinetic theory combined with dense-fluid rheology.
*   **Turbulence Model**: `twophasekOmega` (Wilcox 2006 $k-\omega$ modified for two-phase density-stratified mixtures).
*   **Friction Rheology**: $\mu(I)$ rheology model (`FrictionModel MuI`).
*   **Particle Pressure Model**: `JohnsonJackson` (`ppModel JohnsonJackson`) configured with:
    *   `Fr = 5e-2` (repulsion scale coefficient).
    *   `eta1 = 5` (exponential packing stiffness).
    *   `alphaMax = 0.635` (packing limit cap).
    *   `packingLimiter = yes` (numerical flux limiter to prevent touching the singularity).
*   **Threshold Coupling**: Frictional singularity threshold `alphaMaxG` in `granularRheologyProperties` is set to `0.625`, providing a strict threshold hierarchy `alphaMinFriction (0.57) < alphaMaxG (0.625) < alphaMax (0.635)` to prevent negative denominators in the $\mu(I)$ rheology model.
*   **Time Integration**: Courant number limits set to `maxCo 0.5` and `maxAlphaCo 0.5` with `relaxPa 5e-6` for numerical stability.

---

## 🎛️ Boundary & Initial Conditions

### Boundary Patches
1.  **`inlet`**: Left boundary ($x = 0$).
2.  **`outlet`**: Right boundary ($x = 3.5\text{ m}$).
3.  **`bottom`**: Solid flume floor ($y = -0.10\text{ m}$).
4.  **`top`**: Upper water boundary ($y = 0.10\text{ m}$, excluding bridge blocked block).
5.  **`bridge`**: Bridge deck boundaries (entry, soffit ceiling, exit).

### Fields Summary
| Field | inlet | outlet | bottom / bridge |
|---|---|---|---|
| **`alpha.a`** | `codedFixedValue` (tanh bed profile) | `zeroGradient` | `zeroGradient` |
| **`alpha.b`** | `codedFixedValue` ($1.0 - \alpha_a$) | `zeroGradient` | `zeroGradient` |
| **`U.b`** | `codedFixedValue` ($1/7^\text{th}$ log-law ramped over $5\text{ s}$; max $0.263\text{ m/s}$) | `inletOutlet` | `noSlip` |
| **`U.a`** | `fixedValue uniform (0 0 0)` | `zeroGradient` | `noSlip` |
| **`p_rbgh`** | `zeroGradient` | `fixedValue 0` | `fixedFluxPressure` |
| **`k.b`** | `codedFixedValue` ($2.0\times 10^{-4}$ water, $10^{-8}$ bed) | `zeroGradient` | `kqRWallFunction` |
| **`omega.b`** | `codedFixedValue` ($3.66$ water, $10^{-8}$ bed) | `zeroGradient` | `omegaWallFunction` |
| **`Theta`** | `codedFixedValue` ($10^{-8}$ water, $10^{-4}$ bed) | `zeroGradient` | `zeroGradient` |

---

## 📂 Directory Structure
```
bridge_sedfoam_downstream/
├── 0_org/                   # Boundary and initial condition templates
│   ├── alpha.a              # Sediment volume fraction
│   ├── alpha.b              # Water volume fraction
│   ├── U.a                  # Sediment velocity
│   ├── U.b                  # Water velocity
│   └── [k.b, omega.b, pa...]
├── constant/
│   ├── g                    # Gravitational acceleration
│   ├── ppProperties         # Johnson-Jackson particle pressure properties
│   ├── granularRheologyProperties # Frictional rheology properties
│   └── transportProperties  # Phase densities (rhoa=2650, rhob=1000) and diameters
├── system/
│   ├── blockMeshDict        # Mesh layout definitions
│   ├── controlDict          # Adjustable timestep controls and write intervals
│   ├── decomposeParDict     # Domain decomposition settings (8 cores)
│   └── fvSchemes / fvSolution # Discretization schemes and linear solvers
├── Allclean                 # Utility script to clean existing simulation files
└── Allrun                   # Automated meshing, setup, and parallel run script
```

---

## 🚀 Execution Guide

### Automated Run
Execute the all-in-one setup and parallel run command:
```bash
./Allrun
```

### Manual Step-by-Step Run

1.  **Clean Case Directories**:
    ```bash
    ./Allclean
    ```

2.  **Generate Structured Mesh**:
    ```bash
    blockMesh > log.blockMesh 2>&1
    ```

3.  **Initialize Boundary Fields**:
    ```bash
    cp -r 0_org 0
    setFields > log.setFields 2>&1
    ```

4.  **Extract Cell Centers**:
    ```bash
    postProcess -func writeCellCentres -time 0 > log.writeCellCentres 2>&1
    ln -sf Cx 0/ccx
    ln -sf Cy 0/ccy
    ln -sf Cz 0/ccz
    ```

5.  **Decompose Domain (8 Cores)**:
    ```bash
    decomposePar > log.decomposePar 2>&1
    ```

6.  **Run Solver in Parallel**:
    ```bash
    mpirun --oversubscribe -np 8 sedFoam_rbgh -parallel > log.sedFoam_rbgh 2>&1 &
    ```

---

## 📊 Post-Processing
*   **Reconstruction**: Reconstruct parallel time fields to single-directory format:
    ```bash
    reconstructPar -time [TIME_DIR]
    ```
*   **Bed Profile Extraction**: Identify the bed elevation contour line (where $\alpha_a = 0.30$) using the parsed `0/ccx`, `0/ccy`, and `[TIME_DIR]/alpha.a` coordinates.
*   **Visualization**: Load the case in ParaView by creating an empty `.foam` file:
    ```bash
    touch fo.foam && paraview fo.foam
    ```

---

## 👥 Author & Research Context
*   **Researcher**: [Divyansh Kumar Singh](https://github.com/DKS-MANAGER) (MTech Civil Engineering - Hydraulic Engineering, IIT Kanpur)
*   **Laboratory**: Hydraulic and Water Resources Engineering (HWRE) Lab, IIT Kanpur
*   **Contact**: [divyansh179@gmail.com](mailto:divyansh179@gmail.com) | [LinkedIn](https://www.linkedin.com/in/divyansh-kumar-singh-92bb621b6)
