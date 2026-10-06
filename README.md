# NIC Physics-of-Failure Run-to-Failure Dataset

Hourly run-to-failure trajectories for 200 simulated server-class network interface cards (NICs), together with the configuration used to generate them. This repository holds the data and configuration files that accompany the paper:

> M. F. Hossain, N. Jagannath, E. Haque, K. M. Morshed, M. Hassan, A. Jamalipour, and K. Munasinghe, "Physics-of-Failure-Guided Run-to-Failure Predictive Maintenance Dataset Generation and Analysis for Network Interface Cards," in *Proc. International Telecommunication Networks and Applications Conference (ITNAC)*, 2026, to appear.

The repository contains the dataset and its generation configuration. The generator code is not included.

The trajectories come from a physics-of-failure (PoF) model. Environmental and workload profiles set the electrical and thermal state of each card. That state drives four damage mechanisms: electromigration (EM), time-dependent dielectric breakdown (TDDB), thermally activated aging (TH), and solder-joint thermal-cycling fatigue (SF). A card fails when its composite damage index first reaches 1, so each failure time follows from the card's own simulated stress history rather than from a pre-assigned label.

All values are simulated. The dataset contains no measurements from real hardware.

## At a glance

| | |
|---|---|
| Cards | 200: hot 67, cold 67, controlled 66 |
| Sampling | Hourly, for up to 43,800 h (5 years) |
| Rows | 5,704,508 card-hours, 25 columns |
| Outcomes | 134 failures and 66 right-censored cards (33.0%) |
| Damage mechanisms | EM, TDDB, TH, SF |
| Labels | RUL target `RUL_pof_path_h` (blank for censored cards), failure flag `failed_flag`, constant-rate RUL baseline `RUL_const_h` |
| Format | gzip-compressed CSV, one file per card: about 545 MB in total (1.39 GB uncompressed) |
| Version | 1.0.0 (6 October 2026) |
| License | [CC BY 4.0](LICENSE) |

## Contents

```
config/
  generation_config.json    generation configuration
  card_parameters.csv       parameter values drawn for each of the 200 cards
data/
  trajectories/
    hot/                    67 files: card_000.csv.gz, card_003.csv.gz, ...
    cold/                   67 files: card_001.csv.gz, card_004.csv.gz, ...
    controlled/             66 files: card_002.csv.gz, card_005.csv.gz, ...
  card_summary.csv          one row per card: failure or censoring, lifetime, mechanism shares
  dataset_meta.json         dataset counts and the per-environment statistics of Table III
  data_dictionary.csv       definition of every column in the CSV files
SHA256SUMS                  checksums of every file in config/ and data/
CITATION.cff                citation metadata
LICENSE                     CC BY 4.0
```

A plain `git clone` downloads everything (about 0.55 GB). No Git LFS is needed.

## Environments

Cards are assigned round-robin to three thermal environments: `card_id` mod 3 = 0 is hot, 1 is cold and 2 is controlled. Only the ambient-temperature parameters depend on the environment. All other parameters, including the workload, are drawn from the same distributions everywhere, so systematic differences between environments come from the thermal conditions alone.

| | Hot | Cold | Controlled |
|---|---|---|---|
| Setting | Uncooled hot climate | Unheated cold climate | Air-conditioned machine room |
| Site mean ambient T<sub>0</sub> (°C) | N(36, 1.5) | N(8, 3.0) | N(22, 0.5) |
| Yearly amplitude A<sub>y</sub> (°C) | U(4, 7) | U(6, 9) | U(0.2, 0.8) |
| Daily amplitude A<sub>d</sub> (°C) | U(4, 6) | U(4, 6) | U(0.5, 1.5) |
| Hourly noise σ<sub>T</sub> (°C) | 0.7 | 0.7 | 0.25 |

N(μ, σ) is a normal distribution with mean μ and standard deviation σ. U(a, b) is a uniform distribution on [a, b]. Lognormal(μ, σ), used below, means that the logarithm of the value is N(μ, σ).

## Trajectory files

`data/trajectories/<environment>/card_<id>.csv.gz` holds the hourly trajectory of one card. Every file has the same 25 columns:

| Column | Symbol | Unit | Description | Paper |
|---|---|---|---|---|
| `card_id` | i | – | Card identifier, 0–199 | Sec. V |
| `domain` | d<sub>i</sub> | – | Environment: `hot`, `cold` or `controlled` | Sec. III, Table I |
| `hour_index` | t | h | Hours since the start of operation (0-based) | Sec. III |
| `T_ambient_C` | T<sub>amb</sub> | °C | Ambient temperature | Eq. (5) |
| `traffic_load` | L | – | Normalised traffic load in [0, 1] | Eq. (6) |
| `throughput_Gbps` | Q | Gbit/s | Throughput demand | Sec. IV-A |
| `V_core_V` | V | V | Core voltage | Eq. (8) |
| `I_core_A` | I | A | Core current | Eq. (7) |
| `P_core_W` | P | W | Power, P = V·I | Sec. IV-B |
| `theta_C_perW` | θ | °C/W | Junction-to-ambient thermal resistance | Eq. (9) |
| `T_junction_C` | T<sub>j</sub> | °C | Junction temperature | Eq. (10) |
| `deltaT_24h` | ΔT<sup>(24)</sup> | °C | Peak-to-peak junction temperature over the trailing 24 h, current hour included (shorter window during the first 23 h, so 0 at hour 0) | Sec. IV-B |
| `DR_EM` | DR<sub>EM</sub> | h⁻¹ | Electromigration damage rate, S·ζ·a<sub>EM</sub> | Eqs. (11), (16) |
| `DR_TDDB` | DR<sub>TDDB</sub> | h⁻¹ | Dielectric-breakdown damage rate, S·ζ·a<sub>TDDB</sub> | Eqs. (12), (16) |
| `DR_TH` | DR<sub>TH</sub> | h⁻¹ | Thermal-aging damage rate, S·ζ·a<sub>TH</sub> | Eqs. (13), (16) |
| `DR_SF` | DR<sub>SF</sub> | h⁻¹ | Solder-fatigue damage rate, S·ζ·a<sub>SF</sub> | Eqs. (15), (16) |
| `DR_comp` | DR<sub>comp</sub> | h⁻¹ | Composite damage rate, Σ<sub>m</sub> w<sub>m</sub>·DR<sub>m</sub> | Eq. (16) |
| `DI_EM` | DI<sub>EM</sub> | – | Cumulative electromigration damage (unweighted, not capped) | Eq. (1) |
| `DI_TDDB` | DI<sub>TDDB</sub> | – | Cumulative dielectric-breakdown damage (unweighted, not capped) | Eq. (1) |
| `DI_TH` | DI<sub>TH</sub> | – | Cumulative thermal-aging damage (unweighted, not capped) | Eq. (1) |
| `DI_SF` | DI<sub>SF</sub> | – | Cumulative solder-fatigue damage (unweighted, not capped) | Eq. (1) |
| `DI_comp` | DI<sub>comp</sub> | – | Composite damage index, Σ<sub>m</sub> w<sub>m</sub>·DI<sub>m</sub>, capped at 1 | Eqs. (1), (2) |
| `failed_flag` | – | – | 1 on the failure row, otherwise 0 | Sec. IV-E |
| `RUL_pof_path_h` | RUL<sub>path</sub> | h | True remaining useful life (the RUL target); blank for censored cards | Eq. (3) |
| `RUL_const_h` | RUL<sub>const</sub> | h | Constant-rate RUL baseline, (1 − DI<sub>comp</sub>) / max(DR<sub>comp</sub>, 10⁻¹²) | Eq. (4) |

How to read the rows:

- **Failed cards** (134). The trajectory ends at the first hour in which `DI_comp` reaches 1. That last row has `failed_flag = 1`, `DI_comp = 1.0` and `RUL_pof_path_h = 0`. On every earlier row, `RUL_pof_path_h` is the number of hours until that failure row.
- **Right-censored cards** (66). These cards were still working at the end of the 5-year horizon. They have 43,800 rows and `failed_flag` is always 0. `RUL_pof_path_h` is blank, while `RUL_const_h` is still given on every row. Their true RUL at hour t is only known to be at least 43,800 − t hours.
- **Timing.** Each row covers one hour: it holds the operating values of hour t, and its damage indices include the damage accrued during that hour. At `hour_index` 0 each damage index therefore equals its damage rate × 1 h (exactly for `DI_comp`, and to within float32 rounding for the four mechanism columns). The paper writes Eq. (1) with DI = 0 at time 0, whereas here the first row already includes the damage of hour 0. `lifetime_h` in `card_summary.csv` counts the hours of operation up to and including the failure hour; these are the lifetimes reported in Table III.
- **Mechanism columns.** `DR_EM`, `DR_TDDB`, `DR_TH` and `DR_SF`, and the matching `DI_` columns, are unweighted and not capped. The mechanism indices often exceed 1 (`DI_SF` reaches 4.75), because failure depends only on the weighted sum `DI_comp`. `DR_comp` and `DI_comp` apply the card's mechanism weights w<sub>m</sub>, which are listed in `config/card_parameters.csv`.
- **Precision.** Observables, damage rates, damage indices and `RUL_const_h` are single-precision (float32) values, so relations between columns hold to float32 rounding. `RUL_pof_path_h` is a whole number of hours written as a float (for example `2984.0`).

## Card summary

`data/card_summary.csv` has one row per card:

| Column | Description |
|---|---|
| `card_id`, `domain` | Card identifier and environment |
| `failed` | 1 if the card failed within the horizon, 0 if it was right-censored |
| `lifetime_h` | Observed duration in hours, equal to the number of rows in the card's trajectory (43,800 for censored cards). For failed cards the failure row is `hour_index = lifetime_h - 1`. |
| `DI_comp_end` | `DI_comp` at the end of observation (1.0 for failed cards) |
| `share_EM`, `share_TDDB`, `share_TH`, `share_SF` | Each mechanism's share of the weighted composite damage w<sub>m</sub>·DI<sub>m</sub> at the end of observation (Fig. 7 of the paper) |
| `trajectory_file` | Path of the card's trajectory file, relative to the repository root |

## Using the data

Python with pandas, run from the repository root:

```python
import pandas as pd

cards = pd.read_csv("data/card_summary.csv")        # one row per card, in card_id order
params = pd.read_csv("config/card_parameters.csv")  # simulation parameters of each card

# One card
card0 = pd.read_csv(cards.trajectory_file[0])

# All 200 cards in card_id order: about 0.9 GB in memory with float32 columns
not_float = {"card_id", "domain", "hour_index", "failed_flag"}
cols = pd.read_csv(cards.trajectory_file[0], nrows=0).columns
dtypes = {c: "float32" for c in cols if c not in not_float}
df = pd.concat((pd.read_csv(f, dtype=dtypes) for f in cards.trajectory_file),
               ignore_index=True)
```

Points to keep in mind:

- **Split by card, not by row.** Rows from the same card are strongly correlated. Assign whole cards to training and test sets, stratified by environment, for example:

  ```python
  test_ids = cards.groupby("domain").sample(frac=0.2, random_state=0).card_id
  train, test = df[~df.card_id.isin(test_ids)], df[df.card_id.isin(test_ids)]
  ```

- **Use `RUL_pof_path_h` as the RUL target.** It is the true remaining life in hours. The paper calls it an oracle label because it is computed from each card's realised future, which a deployed model cannot see. `RUL_const_h` is a naive baseline, not a target: it extrapolates the current damage rate, and the paper (Sec. VI-E) shows that it is optimistic away from failure, with a median overestimate of about 1,959 h at a true RUL of 20,000 h.
- **Choose inputs deliberately.** The nine observables (`T_ambient_C` to `deltaT_24h`) are the quantities that routine NIC telemetry could report. The damage rates and damage indices are model states that determine the labels; for example, `RUL_const_h` is computed from `DI_comp` and `DR_comp`. They suit physics-informed models and diagnostics. For a telemetry-only task, also keep `failed_flag`, `RUL_const_h`, the outcome columns of `card_summary.csv` and the parameters in `config/card_parameters.csv` (for example `zeta`) out of the model inputs: they encode the outcome, or information that a real NIC does not expose.
- **Handle censoring.** 66 cards are right-censored. Survival models can use `failed` and `lifetime_h` from `card_summary.csv`. For RUL regression, either use failed cards only or treat each censored row as having an RUL of at least 43,800 − t hours.

## Configuration

`config/generation_config.json` records the settings used to generate the data: the experiment design, random seeds, reference stress state, failure threshold, calibration, the distribution of every per-card parameter, noise levels, clipping limits and output conventions. The blocks that implement the model name the paper equation, table or section they follow, and `data/data_dictionary.csv` gives the paper reference for each column that has one.

`config/card_parameters.csv` lists the values actually drawn for each card, one row per card. It covers every per-card parameter, including the mechanism weights w<sub>m</sub> and the variability factor ζ.

Settings stated in the paper:

| Setting | Value |
|---|---|
| Reference stress state | T<sub>ref</sub> = 318.15 K (45 °C junction), I<sub>ref</sub> = 2.5 A, V<sub>ref</sub> = 1.0 V, ΔT<sub>ref</sub> = 10 °C |
| Failure threshold | D<sub>crit</sub> = 1 |
| Calibrated global rate constant | S = 6.477 × 10⁻⁵ h⁻¹ (the paper rounds it to 6.48 × 10⁻⁵ h⁻¹; the exact value is in `generation_config.json`) |
| Calibration | Geometric bisection, so that the median lifetime of the controlled population equals 43,800 h (the 5-year design life) |

Settings that the paper does not list:

| Setting | Value |
|---|---|
| Maximum throughput Q<sub>max</sub> | 25 Gbit/s |
| Noise standard deviations (same for every card) | σ<sub>Q</sub> = 0.3 Gbit/s, σ<sub>I</sub> = 0.04 A, σ<sub>V</sub> = 0.004 V, σ<sub>θ</sub> = 0.05 °C/W |
| Floor on ΔT<sup>(24)</sup> inside a<sub>SF</sub> | 0.1 °C (avoids a zero solder-fatigue rate at hour 0, when ΔT<sup>(24)</sup> = 0) |
| ε in RUL<sub>const</sub> | 10⁻¹² h⁻¹ |
| Calibration bracket | 18 geometric-bisection steps over [10⁻⁷, 10⁻³] h⁻¹ |
| Random seeds | 42 for the per-card parameters and 43 for the hourly noise (NumPy `default_rng`), recorded for provenance; the realised per-card values are in `config/card_parameters.csv` |

<details>
<summary>Columns of <code>config/card_parameters.csv</code></summary>

| Column | Symbol | Unit | Description | Sampling |
|---|---|---|---|---|
| `card_id`, `domain` | i, d<sub>i</sub> | – | Card identifier and environment | – |
| `T0_C` | T<sub>0</sub> | °C | Site mean ambient temperature | See Environments |
| `Ay_C` | A<sub>y</sub> | °C | Yearly ambient amplitude | See Environments |
| `Ad_C` | A<sub>d</sub> | °C | Daily ambient amplitude | See Environments |
| `sigma_T_C` | σ<sub>T</sub> | °C | Ambient noise standard deviation | See Environments |
| `phi_rad` | φ | rad | Diurnal phase | U(0, 2π) |
| `L_base` | L<sub>base</sub> | – | Baseline utilisation | U(0.15, 0.35) |
| `A_day` | A<sub>day</sub> | – | Diurnal load amplitude | U(0.25, 0.45) |
| `A_week` | A<sub>week</sub> | – | Weekly load amplitude | U(0.05, 0.15) |
| `burst_k` | k | – | Shape of the Gamma burst term B | U(1.2, 1.8) |
| `burst_theta` | ϑ | – | Scale of the Gamma burst term B | U(0.02, 0.04) |
| `Q_max_Gbps` | Q<sub>max</sub> | Gbit/s | Maximum throughput | 25 (same for every card) |
| `I_idle_A` | I<sub>idle</sub> | A | Idle current | U(0.9, 1.1) |
| `I_dyn_A` | I<sub>dyn</sub> | A | Load-proportional current | U(2.2, 2.8) |
| `V_nom_V` | V<sub>nom</sub> | V | Nominal core voltage | U(0.98, 1.02) |
| `alpha_V` | α | V | Load-droop coefficient | U(0.02, 0.04) |
| `beta_V` | β | V | Degradation-droop coefficient | U(0.03, 0.06) |
| `theta0_C_perW` | θ<sub>0</sub> | °C/W | Baseline thermal resistance | U(4, 6) |
| `gamma_theta` | γ<sub>θ</sub> | – | Thermal-resistance degradation coefficient | U(0.2, 0.5) |
| `Ea_EM_eV` | E<sub>a,EM</sub> | eV | Electromigration activation energy | U(0.75, 0.90) |
| `n_EM` | n | – | Current-density exponent | U(1.5, 2.0) |
| `Ea_V_eV` | E<sub>a,V</sub> | eV | TDDB activation energy | U(0.60, 0.80) |
| `gamma_tilde` | γ̃ | – | Dimensionless field-acceleration coefficient | U(8, 15) |
| `Ea_TH_eV` | E<sub>a,TH</sub> | eV | Thermal-aging activation energy | U(0.50, 0.70) |
| `Ea_SF_eV` | E<sub>a,SF</sub> | eV | Solder-fatigue activation energy | 0.122 (same for every card) |
| `m_SF` | m | – | Fatigue exponent | U(1.9, 2.5) |
| `w_EM`, `w_TDDB`, `w_TH`, `w_SF` | w<sub>m</sub> | – | Mechanism weights | u<sub>m</sub> ~ U(0.25, 1), w<sub>m</sub> = u<sub>m</sub> / Σ u |
| `zeta` | ζ | – | Unit-to-unit manufacturing variability factor | Lognormal(0, 0.35) |

</details>

## How the data were generated

The paper gives the full derivation. In brief, for card i at hour t:

**Mission profile.** Ambient temperature has a yearly and a daily harmonic plus Gaussian noise, clipped to [−10, 55] °C. The yearly harmonic has the same phase for every card (one shared seasonal clock), while the daily phase φ is drawn per card.

```math
T_{\mathrm{amb}}(t) = T_0 + A_y \sin\left(\frac{2\pi t}{8760}\right) + A_d \sin\left(\frac{2\pi (t \bmod 24)}{24} - \phi\right) + \varepsilon_T
```

The normalised traffic load has daily and weekly harmonics plus a burst term B, clipped to [0, 1]. B is drawn every hour from a Gamma distribution with shape k and scale ϑ. Throughput is Q = max(Q<sub>max</sub>·L + ε<sub>Q</sub>, 0).

```math
\begin{aligned}
L(t) = \mathrm{clip}\Big[\, & L_{\mathrm{base}} + A_{\mathrm{day}}\,\frac{1+\sin\left(2\pi (t \bmod 24)/24-\pi/2\right)}{2} \\
& + A_{\mathrm{week}}\,\frac{1+\sin(2\pi t/168)}{2} + B(t),\ 0,\ 1\Big]
\end{aligned}
```

**Electrical and thermal state.** Voltage and thermal resistance degrade with the composite damage index of the previous hour. This feedback raises the junction temperature as damage builds up.

```math
I = I_{\mathrm{idle}} + I_{\mathrm{dyn}}\,L + \varepsilon_I, \qquad
V = V_{\mathrm{nom}} - \alpha L - \beta\, DI_{\mathrm{comp}}(t-1) + \varepsilon_V, \qquad
P = V I
```

```math
\theta = \theta_0 \left[1 + \gamma_\theta\, DI_{\mathrm{comp}}(t-1)\right] + \varepsilon_\theta, \qquad
T_j = \mathrm{clip}\left(T_{\mathrm{amb}} + \theta P,\ -10,\ 110\right)
```

ΔT<sup>(24)</sup> is the peak-to-peak junction temperature over the trailing 24 hours. It is treated as one thermal cycle per day.

**Damage mechanisms.** Each mechanism's damage rate is proportional to an acceleration factor a<sub>m</sub>, which equals 1 at the reference stress state. With the Arrhenius term

```math
\mathcal{A}(E_a) = \exp\left[\frac{E_a}{k_B}\left(\frac{1}{T_{\mathrm{ref}}} - \frac{1}{T_j + 273.15}\right)\right], \qquad k_B = 8.617 \times 10^{-5}\ \mathrm{eV/K}
```

the four acceleration factors are:

```math
a_{\mathrm{EM}} = \left(\frac{I}{I_{\mathrm{ref}}}\right)^{n} \mathcal{A}(E_{a,\mathrm{EM}}), \qquad
a_{\mathrm{TDDB}} = \exp\left[\tilde{\gamma}\left(\frac{V}{V_{\mathrm{ref}}} - 1\right)\right] \mathcal{A}(E_{a,V})
```

```math
a_{\mathrm{TH}} = \mathcal{A}(E_{a,\mathrm{TH}}), \qquad
a_{\mathrm{SF}} = \left(\frac{\max\left(\Delta T^{(24)},\ 0.1\right)}{\Delta T_{\mathrm{ref}}}\right)^{m} \mathcal{A}(E_{a,\mathrm{SF}})
```

**Damage accumulation and failure.** S is the single calibrated rate constant, and ζ and w<sub>m</sub> are drawn per card. Damage accumulates hour by hour following Miner's rule, with Δt = 1 h:

```math
DR_m = S\,\zeta\,a_m, \qquad DR_{\mathrm{comp}} = \sum_m w_m\, DR_m
```

```math
DI_m(t) = \sum_{s=0}^{t} DR_m(s)\,\Delta t, \qquad DI_{\mathrm{comp}}(t) = \min\left(\sum_m w_m\, DI_m(t),\ 1\right)
```

A card fails in the first hour t<sub>f</sub> in which DI<sub>comp</sub> reaches 1. That is the failure row, with `hour_index` t<sub>f</sub> = `lifetime_h` − 1. The RUL labels are

```math
RUL_{\mathrm{path}}(t) = t_f - t, \qquad
RUL_{\mathrm{const}}(t) = \frac{1 - DI_{\mathrm{comp}}(t)}{\max\left(DR_{\mathrm{comp}}(t),\ 10^{-12}\right)}
```

Here t is the `hour_index` in every equation, including the harmonics of the mission profile. The paper writes Eq. (1) with DI = 0 at time 0, whereas the released damage indices already include the damage of hour 0. `lifetime_h` = t<sub>f</sub> + 1 is the number of hours of operation, including the failure hour.

**Calibration.** S was found by geometric bisection so that the median lifetime of the controlled population equals the 5-year design life, with censored cards counted as infinite. The result is S = 6.477 × 10⁻⁵ h⁻¹, so 1/S = 15,438.5 h is the lifetime of a nominal card (ζ = 1) held at the reference stress state. The paper quotes 15,432 h, which follows from S rounded to 6.48 × 10⁻⁵ h⁻¹. In the released run the controlled median is 43,657 h, 0.3% below the target: the median of 66 cards moves in steps, and the calibration runs did not use exactly the same noise draws as the released run.

## Summary statistics

The statistics below are reported in the paper (Table III and Figs. 5 and 7) and can be recomputed from the released files. The values in the first eight rows are also stored in `data/dataset_meta.json`.

| | Hot | Cold | Controlled |
|---|---|---|---|
| Cards / failed / censored | 67 / 67 / 0 | 67 / 33 / 34 | 66 / 34 / 32 |
| Rows | 768,211 | 2,435,531 | 2,500,766 |
| Median lifetime (h) | 10,445 | > 43,800 | 43,657 |
| Failed-lifetime range (h) | 2,911–26,843 | 14,410–43,754 | 17,413–43,718 |
| Mean / 95th-percentile T<sub>j</sub> (°C) | 49.6 / 58.8 | 20.3 / 33.0 | 34.3 / 39.2 |
| Mean ΔT<sup>(24)</sup> (°C) | 12.1 | 11.7 | 6.0 |
| Weibull shape k | 2.40 | 2.46 | 3.53 |
| Weibull scale λ (h) | 12,955 | 50,895 | 47,904 |
| Damage share EM / TDDB / TH / SF (%) | 29 / 17 / 24 / 30 | 8 / 8 / 15 / 68 | 23 / 21 / 35 / 21 |
| Fleet-mean DI<sub>comp</sub> at 43,800 h | 1.00 | 0.83 | 0.88 |

- Median lifetimes count censored cards as infinite.
- The Weibull parameters are the paper's censoring-aware maximum-likelihood fit to `lifetime_h` and `failed` for each environment, computed with SciPy's Nelder-Mead method at its default tolerances. A fully converged fit agrees to within 0.01%, so other tools may differ in the last digit (cold: k = 2.4636, λ = 50,895.7 h).
- Damage shares are the environment means of the `share_` columns in `card_summary.csv`, over failed and censored cards alike.
- The fleet-mean damage holds failed cards at DI<sub>comp</sub> = 1 after they fail.
- The hot-to-controlled acceleration factor of the median lifetime is 43,657 / 10,445 = 4.18. The paper's Arrhenius comparison value of 3.55 uses two inputs. The first is the mean activation energy of EM, TDDB and TH over all 200 cards, 0.7077 eV from `card_parameters.csv`, which the paper rounds to 0.71 eV. The second is the unrounded mean junction temperatures, 49.556 °C and 34.252 °C.
- The RUL numbers in Sec. VI-E of the paper (+7 h, +1,959 h and about 17,000 h) come from a random sample of 600,000 failed-card rows, binned by true RUL in 2,000-h bins. The sample was drawn with pandas `sample(600000, random_state=1)` from rows ordered by `card_id` and `hour_index`, which is the order the loading example above produces. The three numbers are:
  - the median of `RUL_const_h` − `RUL_pof_path_h` over rows with a true RUL below 1,000 h;
  - the mean of the medians of the two bins centred at 19,000 h and 21,000 h;
  - the largest 90th percentile of any bin with at least 200 rows.

  Using all rows instead of the sample gives 6.3 h, 1,966 h and 17,320 h.

## Checking file integrity

`SHA256SUMS` lists a checksum for every file in `config/` and `data/`. To verify your copy:

```bash
shasum -a 256 -c SHA256SUMS      # macOS
sha256sum -c SHA256SUMS          # Linux, or Windows in Git Bash or WSL
```

## Limitations

The data inherit the assumptions of the model (see Sec. VI-F of the paper):

- The selected PoF models.
- Linear Miner-type damage accumulation.
- An hourly, quasi-static thermal model.
- Daily peak-to-peak thermal cycling instead of rainflow cycle counting.
- A weighted additive combination of mechanisms.

The solder-fatigue parameters follow eutectic SnPb assemblies and would need recalibration for lead-free Sn-Ag-Cu (SAC) solder. Lifetimes are relative to the calibrated 5-year design life and are not vendor-specific predictions.

## Citation

If you use this dataset, please cite the paper:

```bibtex
@inproceedings{hossain2026nicpof,
  author    = {Md Farhad Hossain and Nishant Jagannath and Ershadul Haque and
               Khaled Mahbub Morshed and Mahbub Hassan and Abbas Jamalipour and
               Kumudu Munasinghe},
  title     = {Physics-of-Failure-Guided Run-to-Failure Predictive Maintenance
               Dataset Generation and Analysis for Network Interface Cards},
  booktitle = {Proc. International Telecommunication Networks and Applications
               Conference (ITNAC)},
  year      = {2026},
  note      = {To appear}
}
```

## License

The dataset and configuration files are released under the [Creative Commons Attribution 4.0 International License](LICENSE) (CC BY 4.0). You may share and adapt them for any purpose, provided you give appropriate credit (please cite the paper above), provide a link to the license, and indicate if you made changes.

## Contact

Maintained by Nishant Jagannath ([@Nishantj6](https://github.com/Nishantj6)). Please report problems or questions by opening an issue in this repository.
