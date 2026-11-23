# Evaluating When Walking Notifications Work through Causal Analysis of a Micro-Randomized Trial in Python

Individual project, 2026 · [Portfolio page](https://hyeonji-kim-data.github.io/notifications.html)

Does a response mean a notification worked? Using a micro-randomized trial, in which every notification was sent or withheld at random, this project measures whether walking notifications actually increased steps, when, and for whom, and evaluates rules for sending fewer of them.

**Found that users kept responding to about 70% of walking notifications after the step effect had faded, and that a weekday lunch-to-evening rule could remove 44% of them with no detectable loss in steps.**

## Goal

1. Can the published analysis be reproduced, and does the effect last over the study?
2. In which situations, and for which users, do walking notifications increase steps?
3. Does engagement reflect the step effect?
4. Which notifications can be removed, and what would be lost?

## Data

Public [HeartSteps V1 dataset](https://github.com/klasnja/HeartStepsV1) (Klasnja et al., 2019), licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). The raw files are not included here; download them from `data_files/` in that repository into `data/raw/`. The processed table built from them, `data/processed/decision_points.csv`, is included.

| File | Size | Use in this project |
| --- | --- | --- |
| `suggestions.csv` | 8,274 × 86 | Main table: one row per decision point with randomization, context, response, and steps before and after |
| `users.csv` | 37 × 117 | Intake survey: age, gender, activity level, prior fitness app use |
| `jbsteps.csv` | 237,865 × 10 | Minute-level Jawbone steps, summed into daily steps |
| `gfsteps.csv` | 197,524 × 7 | Phone steps; checked and not used (27 of 37 participants, steps missed when the phone is not carried) |

- **Design:** a 6-week micro-randomized trial with 37 participants and five decision points a day (morning commute, lunch, mid-afternoon, evening commute, after dinner). At each available point, a notification was sent with probability 0.6, half walking and half anti-sedentary (0.3 each).
- **Sample:** 6,331 available decision points, excluding travel days.
- **Outcome:** Jawbone steps in the 30 minutes after a decision point, as log(steps + 0.5) as in the paper.
- **Engagement:** a recorded response (good, bad, or snooze). Notifications with no response record (22.5%) are excluded.
- **Changes to the data:** `data/processed/decision_points.csv` (8,274 rows, one per decision point) was built from `suggestions.csv` in notebook 01 by converting times to local time, setting weather placeholder codes to missing, grouping locations into home, work, and other, flagging travel days, and adding log-transformed steps.

## Analysis

| Notebook | Step |
| --- | --- |
| 01 | Check the raw files against the study design and build one row per decision point |
| 02 | Check randomization (send rate 0.56–0.61 in every group) and explore steps, engagement, and missing values |
| 03 | Reproduce the paper's estimates with GEE |
| 04 | Estimate the effect of walking notifications by context and test the differences |
| 05 | Compare engagement with the step effect |
| 06 | Compare the effect across users |
| 07 | Evaluate notification rules |

- **Model:** `log steps after ~ log steps before + notification`, fitted with GEE (statsmodels), standard errors clustered by participant with a small-sample correction. With a constant send probability this matches the paper's estimator.
- **Causal reading:** notifications were randomized at every decision point, so comparing sent with not sent within a situation known before sending estimates the effect in that situation. Responses, recorded after sending, are never used to split data or build rules.
- **Sensitivity analysis:** every analysis is repeated with missing steps set to 0 (main, as in the paper) and left out.

## Results

### 1. Replication (03)

| Estimate | Paper | This project |
| --- | --- | --- |
| Average effect | 0.131 | 0.132 (about +14%, p = 0.050) |
| Effect on day 0 | 0.507 | 0.463 |
| Change per day | −0.018 | −0.016 |
| Walking, home or work vs. elsewhere | 0.377 | 0.378 |

The effect comes mostly from walking notifications (+23.5% vs. +2.9% for anti-sedentary) and declines over the study: +59% on day 0, reaching zero around day 29 under a linear model.

### 2. Context and users (04, 06)

| Situation | Walking effect | Evidence |
| --- | --- | --- |
| Weekday lunch | +75.6% (lunch), +31.5% (weekdays) | Significant in both missing-value versions |
| Morning commute, weekends | −1.1%, +9.0% | No effect in either version |
| Home or work | +41.9%, +65.5% | Weak: depends on missing-value handling |
| New to fitness apps vs. prior users | +39.0% vs. +2.9% | Weak: 14 prior users, difference p = 0.087 |

Tests of differences between situations all had p > 0.1 with 37 participants, so these are candidates to test, not confirmed differences.

### 3. Engagement is not the effect (05)

<img src="figures/engagement_vs_effect_by_week.png" width="650" alt="Engagement stays around 64-74% while the step effect falls after week 1">

| Comparison | Engagement | Step effect |
| --- | --- | --- |
| Week 1 vs. week 6+ | 74.2% vs. 67.9% | +131.4% vs. −4.2% |
| Phone in use vs. not | 84.6% vs. 65.1% | +21.2% vs. +24.0% |
| Lunch vs. mid-afternoon | 68.1% vs. 76.7% | +75.6% vs. +24.1% |
| Walking vs. anti-sedentary | 68.9% vs. 68.1% | +23.5% vs. +2.9% |

Users kept responding after the effect was gone; judged by engagement alone, the notifications would seem to still work.

### 4. Policy evaluation (07)

<img src="figures/policy_evaluation.png" width="650" alt="Effect at no-send times and estimated change in steps for each rule">

A rule differs from the current policy only at its no-send times, so what it gives up is the effect of the notifications the current policy sends there. The overall change is share of no-send times × 0.3 × that effect.

| Rule (evaluated on past data) | Walking notifications removed | Change in 30-min steps, log scale (95% CI) | Change in average 30-min steps (95% CI) |
| --- | --- | --- | --- |
| **Weekdays, no morning commute** | **44%** | **−0.7% (−3.4 to +2.1)** | **+0.8% (−0.8 to +2.4)** |
| First 4 weeks only | 39% | +1.0% (−2.0 to +4.0) | +0.8% (−1.2 to +2.8) |
| New fitness-app users only | 39% | −0.3% (−3.6 to +3.0) | +0.2% (−2.1 to +2.4) |
| Weekday lunch only | 86% | −3.6% (−8.2 to +1.3) | −1.4% (−4.2 to +1.4) |

Compared with the current policy, which sends a notification at random with probability 0.6 at every available decision point. Seven rules were evaluated in notebook 07.

- The notifications skipped by the weekday rule showed no detectable effect (+5.3%, p = 0.626), and the result held with missing steps left out.
- Weekday lunch only removes the most, but may also skip moments that work (+15.3% at its no-send times, p = 0.145).
- Daily steps showed no clear link to the number of walking notifications (+2.4% per notification, p = 0.270).

## Decision implications and limitations

**Proposals**

1. **Timing:** send walking notifications on weekdays from lunch to evening, skipping the morning commute and weekends. The claim is not that people walk more, but that notifications with no detectable effect are removed with no detectable loss in steps.
2. **Frequency:** reduce frequency or change content after the first weeks, when the effect fades.
3. **Measurement:** judge notifications by the step difference against a small random holdout, not by engagement.
4. **Validation:** randomly assign new users to the current policy or the weekday rule, keep a holdout in both groups, and use daily and 30-minute steps as primary outcomes, with snoozing and app dropout as secondary outcomes.

**Limitations**

- 37 participants from a highly educated, university-centered group in the US in 2015–2016.
- Most differences between situations and users were not confirmed by difference tests.
- The rules were built and evaluated on the same data, so skipped effects may look small partly by chance.
- The decline over time was modeled as a straight line, and the effect of reducing notifications for weeks (fatigue, dropout) cannot be answered with this data.

## How to run

1. `pip install -r requirements.txt` (tested on Python X.X).
2. Notebooks 02–05 run on the included `data/processed/decision_points.csv`.
3. Notebooks 01, 06, and 07 also need the raw files: download `suggestions.csv`, `users.csv`, `jbsteps.csv`, and `gfsteps.csv` into `data/raw/`. Running 01 rebuilds `decision_points.csv`.

## Repository structure

```
heartsteps-notification/
├── README.md
├── requirements.txt
├── notebooks/        # 01_data_prep to 07_policy
├── figures/
└── data/
    ├── raw/          # not included; see Data
    └── processed/    # decision_points.csv, created by notebook 01
```

AI tools were used to help with coding and English writing.

## References

Klasnja, P., Smith, S., Seewald, N. J., Lee, A., Hall, K., Luers, B., Hekler, E. B., & Murphy, S. A. (2019). Efficacy of contextually tailored suggestions for physical activity: A micro-randomized optimization trial of HeartSteps. *Annals of Behavioral Medicine, 53*(6), 573–582. https://doi.org/10.1093/abm/kay067

Qian, T., Walton, A. E., Collins, L. M., Klasnja, P., Lanza, S. T., Nahum-Shani, I., Rabbi, M., Russell, M. A., Walton, M. A., Yoo, H., & Murphy, S. A. (2022). The microrandomized trial for developing digital interventions: Experimental design and data analysis considerations. *Psychological Methods, 27*(5), 874–894. https://doi.org/10.1037/met0000283
