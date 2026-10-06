# Dimensionless Cross-Field Diffusion in the Souza Tokamak Escape Map

## Overview

This project extends the deterministic Souza tokamak escape map by introducing
a dimensionless model of isotropic cross-field diffusion between successive
map kicks.

The implementation contains three separate components:

1. the deterministic Souza map
2. the existing Souza phenomenological-noise control
3. the new dimensionless cross-field diffusion model

The stochastic extension calculates finite-time Left, Right, Survival, and
Validity-failure probabilities over the selected tokamak edge-region domain.

---

## Repository Contents

The repository contains:

```text
Dimensionless_Cross_Field_Diffusion_Extension_of_the_Souza_Tokamak_Escape_Map.pdf
Souza_Map_Extension_Dimensionless_Diffusion.ipynb
README.md
db_scan_results.csv
global_db_scan_results.csv
Nc_convergence_results.csv
stochastic_map_outputs.npz
```
---

## Requirements

The calculation was developed and run in Google Colab using Python 3.

Required Python packages are:

- NumPy
- Pandas
- Matplotlib
- Numba

For a local Python environment, these can be installed with:

```bash
pip install numpy pandas matplotlib numba
```

No external input data files are required.

---

## Reproducing the Final Run

The main source file is:

```text
Souza_Map_Extension_Dimensionless_Diffusion.ipynb
```

The notebook contains two run modes:

```python
RUN_MODE = "development"
```

and

```python
RUN_MODE = "final"
```

To reproduce the final calculation, ensure that:

```python
RUN_MODE = "final"
```

is selected.

The final numerical settings are:

```text
N_I = 60
N_PSI = 120

RMC_GRID = 64
NC = 32
NMAX = 20000

N_GLOBAL = 1000
RMC_GLOBAL = 64
```

The primary perturbation amplitude is:

```text
PHI_PRIMARY = 8.74e-3
```

The initial-condition domain is:

```text
I_EDGE_MIN = 0.80
I_EDGE_MAX = 0.99

PSI_NORM_MIN = -0.5
PSI_NORM_MAX = 0.5
```

The model-validity limit for the new diffusion model is:

```text
I_VALID_MIN = 0.20
```

The complete diffusion scan is:

```text
DB_SCAN = [
    0,
    1e-6,
    3e-6,
    1e-5,
    3e-5,
    1e-4,
    3e-4,
    1e-3
]
```
In the notebook and numerical output files, the internal variable `Db`
corresponds to the dimensionless diffusion strength $\hat{D}$ used in the report.
---

## Running in Google Colab

Open the notebook in Google Colab and run it from the beginning with:

```python
RUN_MODE = "final"
```

selected.

The first execution may include additional compilation time because several
functions are compiled by Numba.

---

The final calculation is computationally intensive because the stochastic
probability maps require repeated Monte Carlo trajectories at every grid
point. Execution time therefore depends on the available CPU resources. On
the system used for this work, the final run completed within a few minutes.

---

## Random-Number Reproducibility

The master random seed is:

```text
MASTER_SEED = 12345
```

Separate deterministic seed streams are used for:

```text
INITIAL_CONDITION_SEED
GRID_NOISE_SEED
GLOBAL_NOISE_SEED
CONVERGENCE_NOISE_SEED
SOUZA_NOISE_SEED
```

For the stochastic probability grid, each initial condition is assigned a
deterministic random-number stream based on its grid index.

The global stochastic ensemble similarly assigns each sampled initial
condition its own deterministic random-number stream.

These fixed seeds are used so that repeated runs can be reproduced.

---

## Model Outcomes

Each stochastic trajectory is classified as one of four outcomes:

```text
L = Left physical exit
R = Right physical exit
S = Survival until nmax
V = Model-validity failure
```

Physical escape is recorded when:

```text
I > 1
```

after a complete map period.

The wrapped phase determines the exit:

```text
Left:  Psi_w < 0
Right: Psi_w >= 0
```

For the new diffusion model, a trajectory is classified as a validity failure
if:

```text
I < 0.20
```

No clipping, reflection, or reset is applied.

When:

```text
Db == 0
```

the code calls the deterministic escape routine directly.

---

## Ambiguity Definition

For an initial condition, the physical escaped fraction is:

```text
qL + qR
```

When this is non-zero, the conditioned physical-exit probabilities are:

```text
qtilde_L = qL / (qL + qR)
qtilde_R = qR / (qL + qR)
```

The primary ambiguity threshold is:

```text
AMBIGUITY_THRESHOLD = 0.10
MIN_ESCAPED_FRACTION = 0.90
```

An initial condition is included in the ambiguity set when:

```text
qtilde_L >= 0.10
qtilde_R >= 0.10
qL + qR >= 0.90
```

The ambiguity-layer width `w_0.10,0.90` is the 90th percentile of the
periodic distance from ambiguous initial conditions to the deterministic
Left/Right basin boundary `B0`.

---

## Verification Checks

The source code includes five numerical verification checks.

### Verification 1 — Exact zero-diffusion recovery

Checks that the new framework at:

```text
Db = 0
```

matches the original deterministic implementation in:

- one-step map state;
- exit label;
- escape iteration.

A successful final run should report:

```text
PASS
```

### Verification 2 — Isolated free diffusion

Tests the Cartesian diffusion update independently of the Souza kick,
deterministic phase drift, escape boundary, and validity boundary.

The expected relation is:

```text
E[I(u) - I(0)] = 4 * Db * u
```

A successful final run should report:

```text
PASS
```

### Verification 3 — Nc convergence

Runs the stochastic calculation at:

```text
Nc = 8, 16, 32, 64
```

for:

```text
Db = 1e-4
```

The output reports the global Left, Right, Survival, and Validity-failure
probabilities together with the median escape iteration.

The final production calculation uses:

```text
Nc = 32
```

### Verification 4 — Random-number reproducibility

Checks that:

- repeated runs with the same seed give identical outcome arrays;
- repeated runs with the same seed give identical escape-iteration arrays;
- changing the seed changes individual stochastic samples.

A successful final run should report:

```text
PASS
```

### Verification 5 — Probability bookkeeping

Checks at every initial condition that:

```text
qL + qR + qS + qV = 1
```

A successful final run should report:

```text
PASS
```

---

## Generated Figures

The notebook generates the 10 main figures:

```text
figure1_deterministic_Poincare_Section.png
figure2_deterministic_basin_B0.png
figure3_deterministic_vs_souza_noise.png
figure4_right_exit_committor.png
figure5_exit_entropy.png
figure6_ambiguity_layer.png
figure7_global_stochastic_scan.png
figure8_ambiguity_width.png
figure9_local_survival_qS.png
figure10_local_validity_qV.png
```

The representative stochastic maps use:

```text
Db = 1e-6
Db = 1e-4
Db = 1e-3
```

---

## Numerical Output Files

The notebook saves four numerical output files.

### `db_scan_results.csv`

Contains the diffusion-scan summary:

```text
Db
N_ambiguous
ambiguous_fraction
width_0p10_0p90
max_qS
max_qV
max_probability_error
```

### `global_db_scan_results.csv`

Contains:

```text
Db
PL
PR
PS
PV
median_escape_iteration
```

### `Nc_convergence_results.csv`

Contains:

```text
Nc
PL
PR
PS
PV
median_escape_iteration
probability_sum
```

for:

```text
Nc = 8, 16, 32, 64
```

### `stochastic_map_outputs.npz`

Contains the complete numerical arrays underlying the stochastic maps and
diagnostic figures, including:

- `I_grid`;
- `psi_norm_grid`;
- deterministic escape basin;
- deterministic boundary mask;
- distance-to-boundary grid;
- local `qL`;
- local `qR`;
- local `qS`;
- local `qV`;
- escaped fraction;
- conditioned Left- and Right-exit probabilities;
- exit entropy;
- ambiguity masks;
- Figure 3 Souza-noise phase-space data.

The stochastic quantities are stored separately for each value of `Db`.

---

## Existing Souza-Noise Control

The existing phenomenological Souza-noise calculation uses:

```text
P = 1
rho = 1e-3
```

This calculation is included only as a historical/control comparison.

The parameter `rho` is not equivalent to the new dimensionless diffusion
parameter `Db`.

---

## Final Reproduction Checklist

To reproduce the final calculation:

1. install NumPy, Pandas, Matplotlib, and Numba;
2. open `Souza_Map_Extension_Dimensionless_Diffusion.ipynb`;
3. confirm that `RUN_MODE = "final"`;
4. run the full notebook from beginning to end;
5. confirm that Verifications 1, 2, 4, 5 report `PASS`;
6. inspect the `Nc = 8, 16, 32, 64` convergence output;
7. confirm that Figures 1-10 are generated;
8. confirm that the three CSV files and `stochastic_map_outputs.npz` are
   saved successfully.

The accompanying report, source code, figures, and numerical outputs correspond
to the same final parameter set described above.

---

## Reference

The deterministic model and existing phenomenological-noise comparison are
based on:

L. C. Souza, A. C. Mathias, I. L. Caldas, Y. Elskens, and R. L. Viana,
“Fractal and Wada escape basins in the chaotic particle drift motion in
tokamaks with electrostatic fluctuations,” *Chaos*, vol. 33, 083132, 2023.

DOI: `10.1063/5.0147679`