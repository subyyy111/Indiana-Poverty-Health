# Poverty as a Signal for County-Level Health Outcomes in Indiana

## Overview

**The question:** Do county-level economic and structural conditions — poverty rate chief among them — predict health outcomes across Indiana, and *how much* of the variation in outcomes like diagnosed diabetes, fair/poor self-rated health, and frequent mental distress can they actually explain?

**Why it matters:** If economic disadvantage is a measurable driver of population health, that's a lever policymakers can pull. A state health department or population-health organization could use it to see where economic conditions are most tightly linked to poor health, and therefore where economic support might matter most as a health intervention.

---

## Data

The analysis joins two public datasets at the county level across all 92 Indiana counties.

**Predictors — U.S. Census American Community Survey (ACS), 2023.** The ACS supplies the economic and structural side of the model: county poverty rate, elderly share of population, and a labor force ratio. These describe the *conditions* a county's residents live under.

**Outcomes — CDC PLACES, 2023.** PLACES supplies the health side: county-level prevalence of three adult health outcomes. These are the things being predicted.

The two sources were merged on 5-digit county FIPS codes. ACS arrives keyed by county name, so a FIPS crosswalk was used to attach codes before the join, and both keys were normalized to zero-padded 5-digit strings to ensure a clean match. Note on PLACES: its county values are not direct survey counts. CDC estimates them with a model (multilevel regression and poststratification) built from BRFSS survey responses, and that model uses individual age and county poverty (share of adults below 150% of the federal poverty level, from the ACS) as inputs. So the two datasets are not fully independent: part of the poverty and age relationship measured here is built into the outcome estimates. See Limitations.

---

## Methods

**Predictors.** I used three county-level predictors, anchored on **poverty rate** as the primary signal of economic disadvantage. I added **elderly share** because age structure shapes county health independently of income — older populations carry more chronic disease — so it gives a more nuanced read than poverty alone. I included a **labor force ratio** (civilian labor force 16+ divided by population 18–64) as a proxy for economic engagement. It is not the official participation rate, which divides by everyone 16+, so its values run higher (73% to 93%). Two equally poor counties tell different stories if one has high workforce participation and the other doesn't, the latter suggesting deeper structural barriers.

**Outcomes.** I modeled three health outcomes rather than one, chosen to span different relationships to poverty. **Fair or poor self-rated health** is a broad, validated summary measure of overall health. **Diagnosed diabetes** is a concrete chronic condition with a well-established link to poverty in the public-health literature. **Frequent mental distress** was included as an exploratory outcome — less directly tied to material conditions, and the one I was most curious about.

**Validation.** With only 92 counties, a single train/test split would leave roughly 18 counties in the test set, making the result highly sensitive to which counties happened to land there. I used **5-fold cross-validation** instead: the data is divided into five parts, and the model trains five times, each time holding out a different fifth as the test set. Even then, one 5-fold split is sensitive to how counties are assigned to folds (single folds ranged from below 0 to above 0.7), so I shuffled the folds and **repeated the whole procedure 50 times** with a fixed seed, reporting the mean R² across all 250 folds.

**What drives each outcome.** R² from all three predictors together doesn't say which one matters, so I also checked each predictor's R² on its own and compared standardized coefficients (each variable rescaled to standard deviations).

**Crude vs. age-adjusted prevalence.** I used crude prevalence rather than age-adjusted. Age-adjustment would strip age out of the outcome — which would waste the elderly-share predictor I deliberately included to let age do explanatory work. Crude outcomes keep age in the picture, so elderly share earns its place in the model.

**A path I dropped.** I originally planned to break poverty into bands — from deep poverty to comfortable — by share of each county's population. On inspection, those columns were empty in the source table, so I dropped the band scheme and kept the single poverty rate as the economic signal. The simpler predictor was also the more defensible one given the sample size.

---

## Results

The three predictors together explain a **moderate share of county-to-county variation (cross-validated R² 0.38 to 0.50)**. With 92 counties the estimates are noisy: R² varies a lot from fold to fold, so the averages matter more than any single number.

| Outcome | CV R² (mean) | Std across folds | Strongest predictor |
|---|---|---|---|
| Diagnosed diabetes | 0.50 | 0.23 | Elderly share |
| Frequent mental distress | 0.42 | 0.26 | Poverty rate |
| Fair or poor self-rated health | 0.38 | 0.34 | Poverty and labor force ratio |

**The main finding is that the outcomes are driven by different things.**

| R² from one predictor alone | Diabetes | Mental distress | Self-rated health |
|---|---|---|---|
| Poverty rate | 0.25 | **0.50** | 0.38 |
| Elderly share | **0.38** | 0.00 | 0.14 |
| Labor force ratio | 0.30 | 0.38 | **0.40** |
| All three (in-sample) | 0.61 | 0.55 | 0.54 |

**Frequent mental distress is the outcome most tied to poverty.** Poverty alone explains about half the variation, and it has the largest standardized coefficient. Age plays no role.

**Diagnosed diabetes is mostly about age.** It has the highest R², but elderly share is the strongest predictor, and poverty adds little once age and the labor force ratio are included. That is expected with crude prevalence, which keeps age in the outcome: older counties have more diabetes.

**Fair or poor self-rated health** sits in between, with poverty, age, and the labor force ratio all contributing.

**What the map added.** The regression measures *how much*; the choropleth shows *where*. The lowest rates of fair/poor health are in the wealthy suburban counties around Indianapolis (Hamilton County is lowest at 12.8%), which also have the lowest poverty rates. Marion County itself is near the state average (21.0%). The highest rate is Crawford County in southern Indiana (28.8%), which also has the highest poverty rate. Two exceptions stand out: Monroe and Tippecanoe counties (Indiana University and Purdue) have high poverty rates but good self-rated health, likely because college students count as low-income in Census poverty data.

![Fair or poor self-rated health by county, Indiana 2023](srh_choropleth.png)

---

## Limitations

**The outcome data is partly modeled from poverty and age.** CDC PLACES estimates come from a model that uses county poverty and individual age as inputs. Some of the relationship between ACS poverty and PLACES outcomes is therefore built in, and the R² values likely overstate how much poverty explains real-world health differences. The results describe how these county estimates line up with poverty, not an independent test of poverty's effect.

**Small sample.** 92 counties is a small dataset for regression, and cross-validated R² varies widely from fold to fold. Repeating the cross-validation gives a stable average, but individual results are noisy.

**Association, not causation.** County-level relationships do not show that poverty causes these outcomes, and they don't describe individuals.

**Data reflects the system that produces it.** Diagnosed diabetes requires a clinical diagnosis, so it may be undercounted where access to care is poorest.

**Denominator approximations.** The labor force ratio divides the labor force 16+ by the population 18–64, so it is a proxy rather than the official participation rate. The poverty rate counts off-campus college students, which inflates poverty in college counties.

With more time, I would extend the analysis to all U.S. counties — adding state-level context and a far larger sample would let the poverty–health relationship be tested across more varied economic settings than a single state allows.

---

## Implications

These are directions for a closer look, not conclusions from this analysis alone.

**Mental health is where economic conditions and health line up most closely.** If a health department wanted to pair economic support with a health program, mental health services in high-poverty counties are the clearest match in this data.

**Diabetes planning should start from age structure.** Diabetes prevalence follows the age of the population more than poverty, so older counties are where demand for diabetes care will be highest.

**Better data would make the poverty question testable.** Because PLACES estimates are partly modeled from poverty, a real test of poverty's effect would need direct survey data or claims data, such as Medicaid claims, rather than modeled estimates.

---

## Stack

Python · pandas · scikit-learn (LinearRegression, repeated 5-fold cross-validation) · geopandas · matplotlib · Census ACS & CDC PLACES data · county-level FIPS merge
