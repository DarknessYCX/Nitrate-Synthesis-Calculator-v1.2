# Nitrate Synthesis Calculator

**A single-file, browser-based batch-sheet calculator for hydrated nitrate precursors in oxide synthesis.**

Design a cation composition by crystal structure and crystallographic site, enter the synthesis scale, and get the mass of each hydrated nitrate — together with the EDTA / citric acid / glycine quantities — for either the EDTA sol–gel route or the nitrate combustion route.

- **Live demo:** https://darknessycx.github.io/Nitrate-Synthesis-Calculator-v1.0/
- **Interface language:** Chinese (the calculator itself); documentation in English.
- **Dependencies:** none. One `index.html`, no build step, no server, works offline from `file://`.

---

## Project Overview

Preparing an oxide powder usually starts with a hand-written recipe: dissolve the nitrates, add the chelating agent and fuel, and adjust the batch to the requested scale. Those recipes are easy to get wrong — hydrate water is forgotten, a cation ratio is not normalized to the site occupancy, or the chelator amount is not scaled with the total metal content.

This calculator formalizes that step. It

1. builds the composition from **crystallographic sites** (A/B for perovskites and spinels, Aₙ₊₁Bₙ for RP phases) instead of a single flat list of ratios,
2. converts a requested **synthesis scale** into total metal moles,
3. resolves each **hydrated nitrate** to a molar mass from its chemical formula,
4. applies **reagent purity** or **solution concentration** to give the quantity to actually weigh or pipette,
5. and adds the **chelating agent / fuel** load for the selected synthesis route.

It is intended for research groups preparing perovskite, spinel and RP-type oxide powders who want a reproducible, checkable batch sheet rather than a spreadsheet copy.

---

## Features

### Composition design

- Three structure families with **independent A-site and B-site normalization** (relative occupancies are normalized per site, not globally).
- Editable **oxidation state** per cation, with an automatic estimate for a single B-site cation once the A-site charge is known.
- **Charge-balance check** against the structure oxygen content; the residual is reported and a warning is raised when it is out of tolerance.
- **Target product formula** field: molar mass is parsed from the formula, the composition is checked element by element against the site model, and common stoichiometries are recognized automatically.
- **"Load composition from product formula"** — paste `La0.6Sr0.4MnO3` and the element rows, sites and ratios are generated; recognized perovskite / spinel / RP formulas also switch the structure automatically.
- Five built-in presets: LaMnO₃, La₀.₆Sr₀.₄MnO₃ (LSM), LaCoO₃, La₀.₆Sr₀.₄Co₀.₂Fe₀.₈O₃ (LSCF), La₂NiO₄ (RP, n = 1).

### Precursors

- Per-row precursor formula, editable — change `·6H2O` to `·4H2O` and the molar mass follows.
- Formula parser supports nested parentheses, decimal subscripts (`La0.6…`), and hydrate notation with `·`, `•` or `*`.
- **Solid reagents:** reagent purity (%) → "weigh this much".
- **Solution reagents:** mass fraction (%) + density (g/mL) → solution mass **and** pipetting volume (e.g. 50 wt% Mn(NO₃)₂).
- Validation: the precursor must contain exactly **one** atom of the target element; duplicate element rows are rejected.

### Synthesis scale

Three interchangeable bases:

| Basis | What you type | Total metal ΣM |
|---|---|---|
| Total metal | mmol or mol of all cations | as entered |
| Product amount | mmol or mol of product formula units | amount × (A + B) |
| Product mass | g or mg of product | (mass / M) × (A + B) |

### Chelating agent / fuel

- Route switch sets the default molar ratios; every value stays editable.
- EDTA: free acid `C10H16N2O8` or disodium salt dihydrate `C10H14N2Na2O8·2H2O`.
- Citric acid: monohydrate `C6H8O7·H2O` or anhydrous `C6H8O7`.
- Glycine: anhydrous `C2H5NO2`.
- **Extra excess (%)** applied to all chelating agent / fuel quantities (not to the nitrates).

### Output and record keeping

- Live batch sheet: per-cation mmol, site, theoretical mass, and mass to weigh (or solution volume).
- Summary: ΣM, A + B structural coefficient, product moles, product molar mass, theoretical product mass, charge-balance status.
- Optional solvent volume → molarity of each cation in the precursor solution.
- **Copy as text**, **export CSV** (type, site, component, formula, site coefficient, mmol, theoretical mass, weighed amount), and **print** (print stylesheet included).
- Inline warnings for every rejected input instead of silently producing a wrong number.
- Inputs and font settings auto-saved in `localStorage`; a reset button restores defaults.
- Font family / font size (10–24 px) dialog for screen readability.

---

## Supported Crystal Structures

| Structure | General formula | A-site total | B-site total | O | Cations (A + B) |
|---|---|---|---|---|---|
| Perovskite | ABO₃ | 1 | 1 | 3 | 2 |
| Spinel | AB₂O₄ | 1 | 2 | 4 | 3 |
| RP (Ruddlesden–Popper) | Aₙ₊₁BₙO₃ₙ₊₁ | n + 1 | n | 3n + 1 | 2n + 1 |

RP phases require an **integer n ≥ 1**; n = 1 (A₂BO₄, e.g. La₂NiO₄) and n = 2 (A₃B₂O₇) are the common cases.

Automatic recognition from the product formula uses exact stoichiometry: A-site total = 1, B-site total = 1, O = 3 (perovskite); A = 1, B = 2, O = 4 (spinel); A = n + 1, B = n, O = 3n + 1 (RP). Anything that does not match exactly leaves the structure unchanged and shows a notice.

---

## Supported Synthesis Routes

| Route | Default molar ratio (metal : additive) | Notes |
|---|---|---|
| EDTA sol–gel | M : EDTA : citric acid = 1 : 1 : 2 | Ratios editable; EDTA free acid / disodium dihydrate selectable |
| Nitrate combustion | M : glycine : citric acid = 1 : 2 : 1 | Ratios editable; glycine is the fuel, nitrate the oxidizer |

Both ratios are **empirical laboratory defaults**, not automatically derived redox stoichiometries. The combustion route in particular is not redox-balanced by the calculator — it reproduces the ratio you specify. Adjust the numbers if your group uses a different fuel-to-oxidizer ratio.

---

## Calculation Principles

### 1. Site-normalized coefficients

For a structure with site capacities *N*<sub>A</sub>, *N*<sub>B</sub> and relative occupancies *r*<sub>i</sub> entered in the table:

```
c_i = N_site(i) × r_i / Σ r_site(i)
```

so the A-site cations sum to *N*<sub>A</sub> and the B-site cations sum to *N*<sub>B</sub>. Summing all cation coefficients gives the structural coefficient per formula unit, `A + B` (2 for perovskite, 3 for spinel, 2n + 1 for RP).

### 2. Scale conversion

```
N = total metal moles (ΣM)
n_product = N / (A + B)
```

Per-cation precursor moles:

```
n_i = N × c_i / (A + B)
```

### 3. Weighing

```
m_theory,i   = n_i × M(precursor formula)
m_weigh,i    = m_theory,i / (purity / 100)            (solid reagent)
m_solution,i = m_theory,i / (mass fraction / 100)     (solution reagent)
V_i          = m_solution,i / density
```

Molar masses are computed from the chemical formula with IUPAC 2021 standard atomic weights; hydrate water is included at the coefficient written in the formula.

### 4. Chelating agent / fuel

```
n_additive = ratio × N × (1 + excess / 100)
m_weigh    = n_additive × M(additive) / (purity / 100)
```

### 5. Product and charge balance

```
m_product(theory) = (N / (A + B)) × M(product formula)
Σ (c_i × z_i) − 2 × O  →  residual
```

The residual is reported in the summary (`平衡` when |residual| < 0.05, otherwise `不平衡` with the numerical residual and a warning). The calculator **never silently rewrites oxidation states or ratios** to force balance.

### 6. Consistency checks

- The parsed product formula must match the site-derived coefficients element by element (tolerance 10⁻⁶); mismatches are listed explicitly.
- Non-stoichiometry written as `+δ` / `−δ` is accepted for the formula, but molar mass is computed from the base oxygen content and a note is displayed.
- Invalid chemistry (missing A-site, unknown element, zero ratios, malformed formula) blocks the calculation and surfaces the reason.

---

## How to Use

1. Open `index.html` (or the Pages link) in a modern browser — no installation needed.
2. Choose the **crystal structure**; for RP phases set **n**.
3. Type the **product formula** (e.g. `La0.6Sr0.4MnO3`) and press **Load composition from product formula**, or add elements one by one and set their **A/B site** and **relative occupancy**.
4. Enter the **oxidation state** of each cation if you want the charge-balance check (a single B-site cation is estimated automatically).
5. Select the **synthesis route** and adjust the chelator / fuel ratios or excess if your recipe differs.
6. Enter the **synthesis scale** using any of the three bases.
7. Read the batch sheet on the right; use **Copy**, **CSV** or **Print** to archive the recipe.

### Worked example (defaults)

La₀.₆Sr₀.₄MnO₃, perovskite, total metal ΣM = 2.000 mmol, EDTA route, reagent purity 99%, precursor purities 99%:

| Component | Site | mmol | Theoretical | Weigh (99%) |
|---|---|---|---|---|
| La(NO₃)₃·6H₂O | A | 0.600 | 259.8 mg | 262.4 mg |
| Sr(NO₃)₂ | A | 0.400 | 84.7 mg | 85.5 mg |
| Mn(NO₃)₂·4H₂O | B | 1.000 | 251.0 mg | 253.5 mg |
| EDTA (free acid) | — | 2.000 | 584.5 mg | 590.4 mg |
| Citric acid·H₂O | — | 4.000 | 840.6 mg | 849.0 mg |

Product: 1.000 mmol, M = 221.33 g/mol, theoretical mass 221.3 mg; charge balance: 平衡 (0.6×3 + 0.4×2 + 1×3.4 = 6 = 3×2).

### Solution precursor example

Selecting **solution** for Mn(NO₃)₂·4H₂O with 50 wt% and ρ = 1.54 g/mL gives 502.0 mg of solution, i.e. 326 µL to pipette — instead of 253.5 mg of the tetrahydrate solid.

---

## Notes and Limitations

- **Stoichiometric bookkeeping only.** No phase equilibria, solubility, viscosity, gelation kinetics, calcination yield or phase-purity prediction.
- **Charge balance is a check, not a solver.** Mixed-valence cations are entered as formal average oxidation states (e.g. Mn³·⁴⁺ in LSM); fractional values are allowed. The tool does not redistribute charge between sites.
- **Default oxidation states are heuristics** (1 for Li/Na/K/Ag, 2 for Mg/Ca/Sr/Ba/Zn/Cd, 3 for most others). Always confirm them for your system.
- **Site assignment defaults** follow a fixed element list (alkali, alkaline earth, Y and lanthanides → A; the rest → B). Override manually when needed.
- **One row per element.** The same element cannot be split between A and B sites or between two rows.
- **Hydrate water is exactly what you type.** Commercial batches differ (e.g. La(NO₃)₃·6H₂O vs. a wet or partly dehydrated bottle); use the assay from the reagent label.
- **Purity defaults to 99%** and is applied to every reagent; replace it with the certificate value.
- **Solution mode** assumes a homogeneous solution of the stated mass fraction and density; temperature effects are ignored.
- **RP phases** accept integer n only; intergrowth or mixed n = 1/2 composites are not modeled.
- **Anion doping, A-site deficiency, δ non-stoichiometry** are not part of the mass balance; a δ-ended formula is accepted for molar mass only, with a note.
- **No upper bound on scale.** Very large batches are arithmetically fine but practically limited by solubility.
- **Local data only.** Inputs live in the browser's `localStorage`; nothing is uploaded. Clearing site data resets the sheet.
- **Safety.** Nitrates are oxidizers and several hydrates are hygroscopic or shock-sensitive when dried with organics; combustion runs release NOₓ/CO₂. This tool only computes masses — it does not replace a risk assessment or standard lab practice.

---

## Repository Structure

```
Nitrate-Synthesis-Calculator-v1.0/
├── index.html    # the complete application (HTML + CSS + JS, no dependencies)
├── README.md
└── LICENSE
```

Deployment: GitHub Pages serves `index.html` from the `main` branch.

---

## Version History

**v1.0.0 — 2026-11-25**

- First public release.
- Structure-aware composition design: perovskite ABO₃, spinel AB₂O₄, RP Aₙ₊₁BₙO₃ₙ₊₁ with independent A/B normalization.
- Oxidation states, automatic B-site charge estimate, charge-balance residual check.
- Product formula parsing, composition consistency check, automatic structure recognition, load-from-formula.
- Hydrated nitrate molar masses from chemical formulas; solid purity and solution (wt% + density → volume) modes.
- EDTA sol–gel (1:1:2) and nitrate combustion (1:2:1) routes with editable ratios and extra excess.
- Three synthesis-scale bases, solvent molarity, copy / CSV / print, inline validation warnings, autosave, font settings.
- Presets: LaMnO₃, LSM, LaCoO₃, LSCF, La₂NiO₄ (RP).

---

## Contributing

Issues and pull requests are welcome, especially:

- additional structure families or site models,
- verified default hydrate forms and assay values,
- additional synthesis routes and fuel systems,
- UI translations.

Please keep the deliverable **one self-contained `index.html`** — no build step, no external runtime dependency — and state the source of any constant you change (atomic weights, hydrate water, default ratios).

---

## License

Released under the **MIT License**. See [LICENSE](LICENSE) for the full text.

---

## Disclaimer

This software is a stoichiometry aid for research use. The authors give no warranty of fitness for a particular synthesis and accept no liability for experimental outcomes. Verify every number against your own reagents and protocol before weighing.
