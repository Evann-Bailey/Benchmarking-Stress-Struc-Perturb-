# Benchmarking-Stress-Struc-Perturb-

> **A 4-part stress-testing framework benchmarking structural AI (GNNs) vs. classical biophysics (PROPKA, Hydride) under geometric and topological degradation.**

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## 📌 Executive Summary

Modern structural bioinformatics relies heavily on two distinct paradigms: **classical/empirical physics tools** (e.g., PROPKA, Hydride) that compute properties via explicit local geometric dependencies, and **structural deep learning models** (e.g., Equivariant Graph Neural Networks / pHoptNN) that map 3D molecular graphs into latent vector representations.

This repository provides a unified experimental framework for stress-testing these tools. By systematically subjecting macromolecular structures (such as Concanavalin A, PDB: `1C57`) to controlled spatial, directional, topological, and fragmentation perturbations, this benchmark isolates:
1. **The exact breaking points** where classical empirical lookup tables and distance thresholds crash.
2. **The latent feature drift** ($\Delta\text{pH}$) in structural GNNs when surface electrostatics and topology are altered.

---

## 🔬 Benchmark Framework (The 4 Stress Tests)

              STRUCTURAL PERTURBATIONS & STRESS BENCHMARK
                                   │
  ┌────────────────┬───────────────┴───────────────┬────────────────┐
  ▼                ▼                               ▼                ▼
Phase 1          Phase 2                         Phase 3          Phase 4[Global Noise] [Directional Anisotropy]       [Global Expansion] [Spatial Fragmentation](Vector vs.     (X/Y/Z Noise vs.              (Uniform Coordinate (Shell Stripping &Empirical Point) Empirical Lookups)             Scaling & Bonds)    Memory Loss)
### 1. Stress-Strain Curve (Isotropic Noise)
* **Perturbation:** Applies random 3D Gaussian noise ($\sigma$) across atomic coordinates.
* **Objective:** Identifies the mathematical breaking point where latent vector representations in Equivariant GNNs filter out random structural noise better than rigid, point-distance empirical lookups.

### 2. Directional Anisotropy ($X, Y, Z$ Single-Axis Noise)
* **Perturbation:** Restricts spatial noise to a single Cartesian axis ($\sigma_x, \sigma_y,$ or $\sigma_z$).
* **Objective:** Isolates Ångström-level sensitivity along specific spatial directions to evaluate how lower-resolution axes (common in Cryo-EM/NMR data) decouple directional hydrogen-bonding lookup tables.

### 3. Coordinate Matrix Expansion (Uniform Topological Scaling)
* **Perturbation:** Uniformly scales the atomic coordinate matrix outward ($1\%, 2\%, 5\%, \dots$).
* **Objective:** Evaluates topological vs. distance rigidity. Preserves overall 3D protein topology while stretching bond lengths out of equilibrium to test hard-coded physical limits against relational graph representations.

### 4. Structural Memory Loss & Shell Fragmentation
* **Perturbation:** Performs layer-by-layer radial residue peeling from the protein exterior toward the hydrophobic core ($0\%$ to $70\%$).
* **Objective:** Measures how the removal of surface titratable charges (Asp, Glu, Lys, Arg) impacts local empirical calculations versus macro-scale ML feature drift ($\Delta\text{pH}$).

---

## 📊 Key Results: Multi-Scale Response to Shell Fragmentation

A. Classical Micro-Scale Breakdown                 B. Deep Learning Macro-Scale Drift┌─────────────────────────────────────────┐       ┌─────────────────────────────────────────┐│  (PROPKA & Hydride Local Errors)        │       │  (pHoptNN Global Predicted Shift)       ││                                         │       │                                         ││  • PROPKA: Exponential $pK_a$ error     │       │  • Phase 1 (0-50%): Rapid drift due to   ││    growth (RMSD > 33 at 70% strip)      │       │    electrostatic depletion.             ││  • Hydride: Near-linear distance error  │       │  • Phase 2 (50-70%): Biphasic asymptote ││    as geometric boundaries are cut.     │       │    as hydrophobic core stabilizes (~10.5) │└─────────────────────────────────────────┘       └─────────────────────────────────────────┘
### Scientific Insights:
* **Micro-Scale Fragility (Classical Physics):** Classical algorithms rely on intact local coordinate networks (e.g., solvent-accessible surface area). Shell stripping causes compounding, exponential errors in PROPKA and linear coordinate errors in Hydride.
* **Macro-Scale Resilience & Biphasic Drift (Deep Learning):** The Equivariant GNN does not crash under structural truncation. Instead, it demonstrates a biphasic drift:
  * **Electrostatic Depletion (0–50%):** Stripping surface polar residues causes rapid alkaline prediction drift.
  * **Core Asymptote (50–70%):** Stripping the remaining uncharged, hydrophobic core produces a flatline ($\Delta\text{pH} \approx +17.9$), demonstrating that the GNN logically recognizes the absence of further titratable charges.

---

## 📁 Repository Structure

```text
Benchmarking-Stress-Struc-Perturb-/
├── README.md                      <-- Master project overview & documentation
├── environment.yml                <-- Conda environment export (stress_strain_env)
├── data/
│   ├── raw/                       <-- Baseline input files (1c57.pdb)
│   └── outputs/                   <-- Calculated prediction tables and CSV logs
├── figures/                       <-- Saved high-resolution benchmark plots (.png)
├── notebooks/                     <-- Modular Jupyter Notebooks for each phase
│   ├── 01_isotropic_noise_stress_strain.ipynb
│   ├── 02_directional_anisotropic_noise.ipynb
│   ├── 03_coordinate_matrix_scaling.ipynb
│   └── 04_shell_fragmentation_memory_loss.ipynb
└── src/                           <-- Core Python utility modules
    ├── __init__.py
    ├── peeling.py                 <-- Residue-level peeling & water-stripping logic
    └── phoptnn_interface.py       <-- Wrapped model execution pipeline
