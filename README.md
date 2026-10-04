# A New Hampshire Paradox: The Arsenic Extension

An object-oriented, statistically-rigorous companion analysis to the
*A New Hampshire Paradox* capstone project, examining arsenic in New
Hampshire bedrock well water as the exact environmental confounder the
capstone named and deliberately left out of scope.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/<your-username>/<repo-name>/blob/main/A_New_Hampshire_Paradox_The_Arsenic_Extension.ipynb)

---

## Table of Contents

- [What is the New Hampshire Paradox?](#what-is-the-new-hampshire-paradox)
- [Why this project exists](#why-this-project-exists)
- [The dataset](#the-dataset)
- [Architecture](#architecture)
- [What's in the notebook](#whats-in-the-notebook)
- [Running it](#running-it)
- [Repository structure](#repository-structure)
- [What's planned next](#whats-planned-next)
- [Known limitations](#known-limitations)
- [Citations and data sources](#citations-and-data-sources)

---

## What is the New Hampshire Paradox?

New Hampshire looks, by standard economic measures, like one of the
healthiest states in the country: one of the nation's lowest unemployment
rates (around 3.1 percent) and seventh in median household income. Yet
the state also ranks second nationally in bladder cancer incidence, sixth
in breast cancer incidence, and first in brain cancer incidence, figures
that are unexpectedly high given its overall socioeconomic advantages.

That contradiction, a wealthy, low-unemployment state posting cancer
surveillance numbers more typical of a high-risk population, is the
paradox the capstone dashboard investigates. The central question is
whether this reflects genuine elevated disease burden, or a socioeconomic
"masking effect," where higher income drives greater healthcare access
and screening, which inflates *detected* incidence without reflecting a
true difference in underlying disease rates.

**Capstone dashboard:** [A New Hampshire Paradox](https://pritish-ponaka.shinyapps.io/nhparadox/)

## Why this project exists

The capstone dashboard explicitly named arsenic in private well water as
an environmental confounder worth investigating, and just as explicitly
put it out of scope for that project. Arsenic in private well water is an
established, biologically plausible bladder carcinogen specific to this
region (Karagas et al., *Elevated Bladder Cancer in Northern New England:
The Role of Drinking Water and Arsenic*). This notebook is the extension
arm that picks that exact thread back up, on its own terms, using the
same geography and the same underlying public-health question.

**Framing honesty check:** this project predicts arsenic *exceedance*,
the exposure, not bladder cancer diagnosis directly. Individual-level
cancer case data for this population (Karagas et al., 1213 cases / 1418
controls) exists but is restricted human-subjects data, not something
this project can access.

**A second, deliberate choice worth naming:** this dataset has only 373
records, and that was chosen on purpose, not settled for. Most coursework
datasets push toward "more records solves everything." This project
tackles the opposite end of that spectrum: real government
environmental-health data is small and suppressed (small cell counts,
censored location fields, non-detect handling, rare binary flags with a
single positive case) far more often than it is huge and clean, and that
constraint shapes real analysis decisions throughout the notebook.

## The dataset

| | |
|---|---|
| **Name** | Testing data set for independent analysis of New Hampshire arsenic model |
| **Source** | U.S. Geological Survey data release |
| **DOI** | [10.5066/F7XK8CQV](https://doi.org/10.5066/F7XK8CQV) |
| **License** | CC0 (public domain) |
| **Size** | 373 NH bedrock wells, 45 predictor variables |
| **Target** | Binary arsenic-exceedance flag, at a 1, 5, or 10 µg/L threshold |

The three thresholds match the EPA's legal limit for public water systems
(10 µg/L), a more cautious research threshold (5 µg/L), and a near-detection
threshold (1 µg/L), and correspond directly to three published logistic
regression models (Ayotte et al.) with reported accuracies of 54.8
percent, 76.3 percent, and 86.4 percent respectively, used throughout this
notebook as a benchmark.

## Architecture

Five classes, each with one job:

| Class | Responsibility |
|---|---|
| `ArsenicDataset` | fetch, clean, validate, describe |
| `DataQualityAuditor` | missing values, impossible values, outliers, rare flags (all variable-type-aware) |
| `StatisticalScreener` | picks the right significance test per predictor, applies multiple-testing correction |
| `EDAVisualizer` | class balance, distributions, correlation heatmap |
| `BivariateMapVisualizer` | the two NH county maps, using the project's actual color palette |

Every class was built and unit-tested in isolation before being assembled
into the notebook. Inline comments throughout explain *why* a design
decision was made, not just *what* the code does.

## What's in the notebook

| Section | Covers |
|---|---|
| 0. Setup | imports, optional-dependency handling, output directories |
| 1. Helper Functions | county-name normalization, the multiple-testing correction |
| 2. `ArsenicDataset` | why arsenic matters, what the three thresholds mean, fetch/clean/validate logic |
| 3. Data Documentation | every predictor, with an honest HIGH/MEDIUM/LOW confidence level, as both code output and static markdown |
| 4. Dataset Exploration | structure, target/predictor identification |
| 5. Data Quality Assessment | missing values, impossible values, outliers, rare flags, constants, plus anticipated preprocessing and challenges |
| 6. Advanced Statistical Exploration | per-predictor test selection (Welch's t-test vs. Mann-Whitney U, chi-square vs. Fisher's exact), FDR correction, VIF |
| 7. Visualizing the Data | class balance, predictor distributions, correlation heatmap |
| 8. Mapping the Results | a bivariate county choropleth and an explicitly-approximate dot-density well map |
| 9. Project Planning | the prediction problem, target variable, and candidate evaluation metrics, clearly stated |
| 10. What's Next | the planned modeling, suppression-handling, GIS, and website phases, not yet built |

## Running it

**Google Colab:** click the badge at the top of this README, or go to


## What's planned next

This notebook covers data selection, exploration, quality assessment,
statistical screening, visualization, and mapping. The modeling phase has
not been built yet. Section 10 of the notebook lays this out in detail;
summarized here:

- **Baseline models:** a regularized logistic regression (the most direct
  comparison to Ayotte et al.'s own published models) and a tree ensemble
  (random forest or gradient boosting) as a stronger baseline.
- **Suppression-aware modeling**, the part that follows through on this
  project's deliberate small-n choice: a hierarchical model that pools
  information across small groups (counties, or bedrock geology
  categories with very few wells) instead of hiding an unreliable
  small-group estimate outright, plus bootstrapped confidence intervals
  attached to every prediction rather than a single unqualified number.
- **A continuous statewide risk surface**, extending the county-level maps
  into a GIS layer predicting arsenic risk across a continuous grid
  covering all of New Hampshire, the same approach USGS itself used in a
  related 2011 study.
- **An interactive website** pulling the model, its uncertainty, and the
  map together into something explorable, rather than static notebook
  output.

## Known limitations

- **n = 373** is real data, but small relative to deep-learning-scale
  ambitions. This is exactly why the statistical screening section exists:
  to prioritize which predictors are worth including, rather than relying
  on model complexity alone.
- **Several bedrock-geology predictor definitions are LOW confidence**,
  flagged explicitly in the data documentation. Verify against the
  dataset's own `Testing_data.xml` before citing them as authoritative.
- **Location is censored to the county level.** No real well coordinates
  exist in this data, which is why the dot-density map is explicitly
  approximate, not a bug to fix.
- **This predicts arsenic exceedance, not bladder cancer diagnosis.** See
  the framing honesty check above.

## Citations and data sources

- Lombard, M.A., Hayes, Laura, Andy, C.M., Fahnestock, M.F., Bryce, J.G.,
  and Ayotte, J.D., 2017, *Testing data set for independent analysis of
  New Hampshire arsenic model*: U.S. Geological Survey data release,
  [10.5066/F7XK8CQV](https://doi.org/10.5066/F7XK8CQV).
- Karagas, M.R., et al., *Elevated Bladder Cancer in Northern New
  England: The Role of Drinking Water and Arsenic*.
- Ayotte, J.D., et al., published multivariate logistic regression models
  for arsenic occurrence probability in New Hampshire bedrock aquifers
  (1, 5, and 10 µg/L thresholds).
- NH county boundaries: U.S. Census Bureau TIGER/Line cartographic
  boundary files.
- Color palette (`bi_pal_bladder`, `nh_colors`): reused directly from the
  original *A New Hampshire Paradox* Shiny application.