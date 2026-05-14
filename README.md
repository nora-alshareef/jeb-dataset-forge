# JEB Dataset Forge

A pipeline for expanding the **SimJEB** structural simulation dataset through
geometric synthesis — reads, simulates, and augments OptiStruct FEM files to
grow the dataset for machine learning surrogate models.

---

## Source Dataset — SimJEB

**SimJEB: Simulated Jet Engine Bracket Dataset**
Whalen E., Beyene A., Mueller C. — MIT, 2021
Harvard Dataverse · DOI: [10.7910/DVN/XFUWJG](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/XFUWJG)
License: **Open Data Commons Attribution License (ODC-By)**

SimJEB is a public collection of **381 bracket designs** cleaned, meshed,
and simulated from the original submissions to the
[GE Jet Engine Bracket Challenge](https://grabcad.com/challenges/ge-jet-engine-bracket-challenge)
— an open competition hosted by GrabCAD in 2013 with over 700 hand-designed
CAD entries from 320 designers across 56 countries.

Each of the 381 designs ships with five file types:

| File | Format | Contents |
|------|--------|----------|
| CAD model | `.stp` | Clean solid geometry |
| FEM model | `.fem` | OptiStruct finite element model |
| Solid mesh | `.vtk` | Tetrahedral volume mesh |
| Surface mesh | `.obj` | Triangular surface mesh |
| Simulation results | `.csv` | Displacement + von Mises stress per load case |

> Access requires a free Harvard Dataverse account.

---

## Why We Need Synthesis

381 samples is not enough to train a surrogate model that generalises to new,
unseen bracket geometries. This repository addresses that gap by applying
controlled geometric transformations to each existing design, producing
up to **10× more samples** while preserving the boundary condition topology
(bolt holes, pin interface, SPCs) so every synthetic variant is a valid
simulation input.

---

## Notebooks

| # | Notebook | Description | Status |
|---|----------|-------------|--------|
| 01 | `01_cad_to_fem.ipynb` | Convert new CAD geometries to OptiStruct FEM | 🔜 Coming soon |
| 02 | `02_fem_simulation_synthesis.ipynb` | Read, simulate, visualise, and synthesise SimJEB FEM files | ✅ Ready |

---

## Notebook 02 — Pipeline Overview
SimJEB .fem files (381 designs)
│
▼
Section 1: Read & Parse
├── GRID nodes, CTETRA elements
├── RBE2 (bolt-hole rigid connections → fixed BCs)
├── RBE3 (pin interface → load application region)
└── SPC (additional single-point constraints)
│
▼
Section 2: Visualise
├── Surface mesh render (3D + 4-view orthographic)
└── Side-by-side comparison of variants
│
▼
Section 3: Synthesise + Simulate
├── 6 geometric transformation strategies
├── FEA under 4 load cases per variant
├── Von Mises stress + displacement extraction
└── Export to master CSV

---

## Material & Load Cases

All brackets are Ti-6Al-4V titanium alloy per the original competition spec:

| Property | Value |
|----------|-------|
| Young's modulus (E) | 113.8 GPa |
| Poisson's ratio (ν) | 0.342 |
| Density (ρ) | 4.47 g/cm³ |

Four structural load cases from the GE bracket challenge:

| Load case | Type | Magnitude |
|-----------|------|-----------|
| Vertical | Force | 8,000 lbf (↑ Z) |
| Horizontal | Force | 8,500 lbf (→ X) |
| Diagonal | Force | 42,300 lbf at 42° from vertical |
| Torsion | Moment | 5,000 lb·in about −Z |

---

## Synthesis Strategies

| # | Strategy | What changes | Parameter |
|---|----------|-------------|-----------|
| 1 | Uniform scaling | Overall size up or down from centroid | ±5% |
| 2 | Axis scaling | X, Y, or Z stretched independently | ±3% per axis |
| 3 | Node perturbation | Gaussian noise — simulates manufacturing tolerance | 0.1–0.2% of bbox |
| 4 | Taper | Cross-section linearly varies along one axis | 3% |
| 5 | Shear | Load-path angle shifted in XY plane | 2% |
| 6 | Combined | Two transforms stacked (e.g. scale + noise) | — |

---

## Output CSV Schema

Each row = one bracket (original or synthetic).

**Geometry**

`id` · `volume` · `mass_g` · `mass_kg` · `num_vertices` · `num_faces` · `num_tets`

**Per load case** — prefix: `ver` · `hor` · `dia` · `tor`

`max_{case}_xdisp` · `max_{case}_ydisp` · `max_{case}_zdisp` · `max_{case}_magdisp` · `max_{case}_stress`

---

## Setup

Runs on **Google Colab** (L4 GPU recommended for large meshes).

```bash
pip install numpy==1.26.4 scipy pyNastran==1.4.1 scikit-fem==11.0.0 matplotlib
```

Set your paths in the notebook:

```python
INPUT_DIR  = '/content/drive/MyDrive/content/fem'        # SimJEB .fem files
OUTPUT_DIR = '/content/drive/MyDrive/content/fem/augmented'
```

---

## Citation

If you use SimJEB data in your work, please cite:

```bibtex
@article{whalen2021simjeb,
  title   = {SimJEB: Simulated Jet Engine Bracket Dataset},
  author  = {Whalen, Eamon and Beyene, Azariah and Mueller, Caitlin},
  journal = {Computer Graphics Forum},
  volume  = {40},
  number  = {5},
  year    = {2021},
  doi     = {10.1111/cgf.14353}
}
```

---

## License

**Code** in this repository: MIT License.
**SimJEB data**: Open Data Commons Attribution License (ODC-By) —
you must credit Whalen et al. (2021) when using or redistributing the data.
