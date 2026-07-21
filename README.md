# Benchmarking-Stress-Struc-Perturb-

> **A 4-part stress-testing framework benchmarking structural AI (GNNs) vs. classical biophysics (PROPKA, Hydride) under geometric and topological degradation.**

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## Summary

Modern structural bioinformatics relies heavily on two distinct paradigms: **classical/empirical physics tools** (e.g., PROPKA, Hydride) that compute properties via explicit local geometric dependencies, and **structural deep learning models** (e.g., Equivariant Graph Neural Networks / pHoptNN) that map 3D molecular graphs into latent vector representations.

This repository provides a unified experimental framework for stress-testing these tools. By systematically subjecting macromolecular structures (such as Concanavalin A, PDB: `1C57`) to controlled spatial, directional, topological, and fragmentation perturbations, this benchmark isolates:
1. **The exact breaking points** where classical empirical lookup tables and distance thresholds crash.
2. **The latent feature drift** ($\Delta\text{pH}$) in structural GNNs when surface electrostatics and topology are altered.

---

## Benchmark Framework (The 4 Stress Tests)

> **Core Objective:** Evaluate how classical biophysics and machine learning models handle progressive structural degradation across four distinct physical axes.

* **Phase 1: Stress-Strain Curve (Isotropic Noise)**
  * **Perturbation:** Random 3D Gaussian noise ($\sigma$) applied universally across atomic coordinates.
  * **Focus:** Identifies the mathematical breaking point where latent vector spaces in Equivariant GNNs filter out noise better than rigid point-distance empirical lookups.

* **Phase 2: Directional Anisotropy ($X, Y, Z$ Single-Axis Noise)**
  * **Perturbation:** Spatial noise restricted to a single Cartesian axis ($\sigma_x, \sigma_y,$ or $\sigma_z$).
  * **Focus:** Isolates Ångström-level sensitivity along individual axes to evaluate how asymmetric resolution (common in Cryo-EM and NMR) decouples empirical lookup tables.

* **Phase 3: Coordinate Matrix Expansion (Uniform Topological Scaling)**
  * **Perturbation:** Uniform outward scaling of the coordinate matrix ($1\%, 2\%, 5\%, \dots$).
  * **Focus:** Tests topological vs. distance rigidity. Preserves 3D protein topology while stretching bond lengths out of equilibrium to test hard-coded physical bounds against relational graphs.

* **Phase 4: Structural Memory Loss & Shell Fragmentation**
  * **Perturbation:** Layer-by-layer radial residue peeling from the protein exterior toward the hydrophobic core ($0\%$ to $70\%$).
  * **Focus:** Measures how the loss of surface titratable charges (Asp, Glu, Lys, Arg) impacts local empirical calculations versus macro-scale ML feature drift ($\Delta\text{pH}$).

---

## Key Results: Multi-Scale Response to Shell Fragmentation

### Comparative Model Trajectories

| Analytical Scale | Paradigm & Tool | Failure Mode / Behavior | Physical Explanation |
| :--- | :--- | :--- | :--- |
| **Micro-Scale** | **PROPKA** *(Empirical)* | **Exponential $pK_a$ Error** | Surface stripping destroys solvent-accessible surface area (SASA) and hydrogen-bonding coordinate networks, causing compounding errors (RMSD > 33 at 70% stripping). |
| **Micro-Scale** | **Hydride** *(Geometric)* | **Linear Coordinate Error** | Distance-based hydrogen placement experiences localized error at the boundary layer of the cut, remaining near-linear as geometry is stripped away. |
| **Macro-Scale** | **pHoptNN** *(Deep Learning GNN)* | **Biphasic Feature Drift** | The GNN does not crash under structural truncation. Instead, it exhibits a smooth, two-phase prediction drift ($\Delta\text{pH}$) mapping surface charge removal. |

### The Two Phases of ML Drift (Panel B)

1. **Electrostatic Depletion Phase (0% – 50% Stripping):**
   * *Behavior:* Rapid shift in predicted pH ($\Delta\text{pH} \approx +17.5$).
   * *Mechanism:* Message-passing layers lose key polar and acidic surface residues (Asp, Glu, Lys, Arg), biasing internal node embeddings toward an uncharged environment.

2. **Core Asymptote Phase (50% – 70% Stripping):**
   * *Behavior:* Prediction curve stabilizes into a flat plateau ($\sim 10.5$).
   * *Mechanism:* Stripping the non-polar, hydrophobic core (Leu, Ile, Val) removes no further polar ch

---

---

## 🚀 Quick Start & Installation

### 1. Environment Setup
Clone the repository and recreate the exact Conda environment:

```bash
git clone [https://github.com/YOUR_USERNAME/Benchmarking-Stress-Struc-Perturb-.git](https://github.com/YOUR_USERNAME/Benchmarking-Stress-Struc-Perturb-.git)
cd Benchmarking-Stress-Struc-Perturb-

# Create and activate environment from environment.yml
conda env create -f environment.yml
conda activate stress_strain_env

## Repository Structure

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

---
    ├── __init__.py


    ├── peeling.py                 <-- Residue-level peeling & water-stripping logic
    └── phoptnn_interface.py       <-- Wrapped model execution pipeline
