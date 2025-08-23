# kbo_npb_project

This repository supports the CS8674 summer research project on building a predictive framework for cross-league player transitions among **Korea Baseball Organization (KBO)** and **Nippon Professional Baseball (NPB)**, and **Major League Baseball (MLB)** modeling plate-appearance outcomes before and after a switch.

## Introduction

**Goal**: To build a structured dataset and apply machine learning models to understand performance after transitions from one league to another. This involves:

- Curating comprehensive datasets from KBO and NPB
- Building a player database (biographical, statistical, and transition data)
- Applying ML algorithmsto assess predictive power
- Validating predictions on past player transitions

**Scope**: We focus on the 4 core tables: MNK_people, MNK_batting, MNK_pitching, MNK_fielding (plus supporting tables, namely, MNK_team and MNK_league).

This work is conducted by **Myungkeun Park** under the supervision of **Prof. Eric Gerber**.

---

## Datasets Preparation

### Overview

- Scrapers were written to collect player & season stats and saved as CSVs (see `Source Files/`).
- Data was cleaned and aligned to a common (Lahman-style) schema. The following were produced:
  - **Original**: source CSVs (per league).
  - **Aligned (KBO + NPB)**: datasets containing only columns shared by **KBO** and **NPB**.
  - **MNK**: cross-league datasets containing columns shared by **MLB**, **NPB**, and **KBO**.

* For detailed schema, please refer to ER diagrams in each directory

### Sources

- \*\*KBO (official): https://www.koreabaseball.com/Default.aspx
- \*\*NPB (official): https://npb.jp/eng/
- \*\*NPB stats: http://npbstats.com/eng/
- \*\*MLB (Lahman): https://sabr.org/lahman-database/

### Cross-League Players Matching

- Primary match key: FirstName + LastName + DOB
  -\*Manual adjustments applied for translation/romanization differences across leagues
- A unified `playerID` assigned in `MNK_people` and used to link all stats tables

### Deduplication & Integrity Rules

- Deduplicate on (`playerID`, `yearID`, `teamID`, `lgID`, `stint`)
- No drops after merging. All source rows preserved; conflicts resolved or flagged
- Confirm no (playerID, season) appears twice with conflicting `lgID` for the same `stint`
- Enforce one record per composite key; remove exact duplicates; resolve soft conflicts e.g. teamID variants

### Reshaping

- A **tall/long** version was derived from the merged dataset

### Access

- Completed CSVs are included in this repo and are also available via **PyPI**:  
  `nk-datasets` --> https://pypi.org/project/nk-datasets/
- **Note:** **MLB/Lahman data is unavailable** here. Please self-download from: https://sabr.org/lahman-database/

### Repository Structure

```

kbo_npb_project/
├── Source_Files/ # Raw datasets and web scraper files (if available) for KBO, NPB, and
│ ├── KBO/
│ ├── NPB/
│ ├── MLB/
├── KBO+NPB/ # Combined KBO and NPB datasets
│ ├── raw/
│ ├── kbo_batting.csv
│ ├── kbo_fielding.csv
│ ├── kbo_people.csv
│ ├── kbo_pitching.csv
│ ├── npb_batting.csv
│ ├── npb_fielding.csv
│ ├── npb_people.csv
│ ├── npb_pitching.csv
│ ├── ERD_initial.drawio
│ └── ERD_initial.pdf
├── KBO+NPB+MLB/ # Combined KBO, NPB, and MLB datasets
│ ├── raw/
│ ├── kbo_batting.csv
│ ├── kbo_fielding.csv
│ ├── kbo_people.csv
│ ├── kbo_pitching.csv
│ ├── npb_batting.csv
│ ├── npb_fielding.csv
│ ├── npb_people.csv
│ ├── npb_pitching.csv
│ ├── mlb_batting.csv
│ ├── mlb_pitching.csv
│ ├── mlb_fielding.csv
│ ├── mlb_people.csv
│ ├── ERD_cleaned.drawio
│ └── ERD_cleaned.pdf
├── MNK_merged/ # Merged datasets of KBO, NPB and MLB: a master people, batting, pitching and batting
│ ├── raw/
│ ├── ERD_final.drawio
│ ├── ERD_final.pdf
│ ├── merged_player_records.csv
│ ├── mnk_batting.csv
│ ├── mnk_fielding.csv
│ ├── mnk_people.csv
│ ├── mnk_league.csv
│ ├── mnk_team.csv
│ └── mnk_pitching.csv
├── MNK_merged_tall/ # (Tall-version) Merged datasets of KBO, NPB and MLB: a master people, batting, pitching and batting
│ ├── raw/
│ ├── ERD_tall.drawio
│ ├── ERD_tall.pdf
│ ├── mnk_league.csv
│ ├── mnk_team.csv
│ └── mnk_playerstats.csv
├── vignettes/ # Data exploration using datasets above
└── README.md # This file

* Note: the scrappers for KBO are mistakenly overridden. Will push them once I can access to my laptop
```

## Data Prep and MNLR (Multinomial Logistic Regression)

### What we used

- Tables: `MNK_batting`, `MNK_people` (merged player records), `MNK_fielding`
- Goal: Predict plate-appearance outcomes (HR, BB, SO, Other) for the first season after a league switch

### Minimal prep

- Primary position (POS): fill in this order: stint -> season -> career (from `MNK_fielding`)
  If still missing, set DH where DH rules apply (AL >= 1973, NL >= 2022, PL >= 1975, all KBO), else PH
- Features (X):
  - Numeric: `age` (year − birthYear), `age2`, `height`, league-specific `weight` (league-specific weights: `mlb_weight` / `npb_weight` / `kbo_weight`)
  - Categorical: `bats_norm` (L/R/S), `lgID`, position groups (`C`, `DPH` for DH/PH, `IF`, `OF`)
- Outcomes (Y): season totals per (`playerID`,`yearID`,`lgID`):
  - Ensure `Other = PA − (HR+BB+SO)` and `HR+BB+SO+Other = PA`
  - If `PA < HR+BB+SO` or `PA` missing -> set `PA = HR+BB+SO`, `Other = 0`

### Train / target split

- Targets (post-move): seasons where a player appears in year T and T−1 and the `lgID` changed from T−1 -> T (consecutive years)
- Training set: all non-target seasons
- Evaluation set: the target (post-move) seasons only

### Fitting MNLR

- We made a long dataset with 4 rows per observation (HR/BB/SO/Other) and use
  `sample_weight = count`. This is equivalent to repeating labels many times but more efficient
- Fit `LogisticRegression(multi_class="multinomial", solver="lbfgs", penalty="l2", C=1e6, max_iter=5000)` using the long X/y and `sample_weight`.

### Evaluation

- Converted predicted probabilities to expected counts by `proba * PA`
- Plotted Actual vs Predicted scatter diagrams. I also added a 45° line to separate over- and under-predictions: above the baseline means under-prediction; below the baseline means over-prediction
- Summarized above vs below counts by switch lane: **MLB -> NPB, MLB -> KBO, NPB -> MLB, NPB -> KBO, KBO -> MLB, KBO -> NPB**

### Observed pattern

- **Strikeouts (SO):** generally **under-predicted** across transitions.
- **HR / BB / Other:** often **over-predicted**.

## Future Works

### Data + Pipeline

- **Incremental scraping**: Add checkpoints e.g. max `yearID` per league so scrapers fetch **only new seasons**
- **Reflect any changes to existing data**: Consider hashing rows to detect updates so we can only reprocess changed data

### Modeling (MNLR & beyond)

- **MNLR tuning.** Penalization (`C`), class weights, solver convergence, and leakage-safe standardization (fit on train only).
- **Alternatives.** Try GLMnet-style multinomial, gradient boosting (XGBoost/LightGBM), and shallow **NN** with softmax.

### Extensions

- **Pitching & fielding.** Mirror the framework for `MNK_pitching` (e.g., PA-level outcomes vs batters faced) and `MNK_fielding`.
- **Panel for next-season.** Publish a ready-to-fit panel with t (features) → t+1 (outcomes) alignment for movers and non-movers.

### Data + Pipeline

- **Incremental scraping**: Add checkpoints e.g. max `yearID` per league so scrapers fetch only new seasons whichmakes re-runs fast and avoids re-downloading old data

- **Detect and refresh changes**: Consider hashing rows to only reprocess any changed rows

### Modeling (MNLR & beyond)

- **Tune the MNLR model**: Adjust the regularization strength, try class weights if one outcome dominates, and make sure any scaling (e.g. z-scores) is fit on the training data only, then applied to the test data

- **Other models exploration**:

  - Gradient boosting (e.g. XGBoost)
  - A neural network

- **Check calibration**: Compare predicted vs. actual counts and use simple calibration steps if predictions are consistently high or low for a class

## Contact

For questions please reach out via GitHub or contact:
**Myungkeun Park** | Northeastern University | frankmkpark (GitHub)
