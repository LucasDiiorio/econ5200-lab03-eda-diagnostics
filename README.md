
[README.md](https://github.com/user-attachments/files/32783398/README.md)
# Pipeline Health Check — EDA, Corruption & Distribution Shift

## Objective

Audit a multi-country socioeconomic panel for silent data-quality failures and quantify train-versus-inference distribution shift, using exploratory analysis alone to surface defects that would otherwise propagate undetected into downstream modelling.

## Methodology

- **Baseline profiling.** Established a structural baseline of the raw country-year panel: dtype and null audits, per-column range and cardinality checks, duplicate-key scans, and univariate and bivariate distribution review.
- **Domain-constraint validation.** Encoded plausibility bounds for each indicator to isolate values that are statistically unremarkable but physically impossible — negative GDP figures and life-expectancy values recorded in months rather than years.
- **Unit and scale reconciliation.** Identified percentage columns mixing proportion (0–1) and percent (0–100) conventions, and resolved a GDP unit mismatch by cross-checking magnitude against population and internal ratio consistency rather than trusting the declared schema.
- **Duplicate resolution.** De-duplicated on the composite country-year key, separating exact repeats from conflicting records requiring adjudication.
- **Distribution shift measurement.** Computed the Population Stability Index (PSI) per feature across the training and inference partitions using decile binning, with bin edges fixed on the training reference to prevent leakage.
- **Automated-versus-manual benchmarking.** Ran `ydata-profiling` over the same panel and documented the specific defect classes each approach failed to surface.
- **Tooling.** Consolidated the checks into a reusable module, `eda_utils.py`, providing impossible-value detection, PSI computation, and a summary reporting interface.
- **Monitoring surface.** Built an interactive pipeline-health dashboard exposing per-column quality flags, shift metrics, and before-and-after correction views.

## Key Findings

- **[N] planted data-quality defects were recovered and remediated through exploratory analysis alone**, spanning five distinct failure classes: sign violations, unit-of-measure errors, record duplication, mixed percentage scales, and a schema-level GDP unit mismatch.
- **GDP PSI = [PSI] between the training and inference partitions.** [INTERPRETATION SENTENCE — see note below.]
- **Automated profiling and manual EDA fail in complementary ways.** `ydata-profiling` delivered fast, exhaustive coverage of completeness, cardinality, and pairwise correlation, but carried no domain priors and therefore could not flag semantically invalid values — a life expectancy of 840 is a clean, well-behaved number in the absence of a unit assumption. Manual EDA caught every semantic defect but scaled poorly and depended on analyst attention for exhaustive column coverage.
- **Unit-of-measure errors were the highest-risk class.** They neither produce nulls nor register as distributional outliers, so they survive standard validation and corrupt model behaviour silently.
- **Data-quality auditing and drift monitoring are one workflow, not two.** Several defects presented as apparent distribution shift; correcting them materially changed the measured PSI, confirming that shift metrics computed over an unvalidated panel are unreliable.

---

*Deliverables: `eda_utils.py` (reusable validation and PSI utilities), interactive pipeline-health dashboard, and the annotated analysis notebook.*
