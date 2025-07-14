# kbo_npb_project

This repository supports a CS8674 summer research course focused on building a predictive framework for evaluating player transitions from the **Korea Baseball Organization (KBO)** and **Nippon Professional Baseball (NPB)** leagues into **Major League Baseball (MLB)**.

## Project Overview

The goal of this project is to build a structured dataset and apply machine learning models to understand which KBO or NPB players are most likely to succeed when transitioning to MLB. This involves:

- Curating comprehensive datasets from KBO and NPB
- Building a player database (biographical, statistical, and transition data)
- Applying ML algorithms (e.g., neural networks, Bayesian models) to assess predictive power
- Validating predictions on past player transitions

This work is conducted by **Myungkeun Park** under the supervision of **Prof. Eric Gerber**.

---

## Current Repository Structure (to be updated)

```
kbo_npb_project/
├── KBO/                 # Raw datasets and web scraper files for KBO
│   ├── data/
│   └── scrapper/
├── KBO+NPB/             # Combined KBO and NPB datasets
│   ├── rawfiles/
│   ├── kbo_batting.csv
│   ├── kbo_fielding.csv
│   ├── kbo_people.csv
│   ├── kbo_pitching.csv
│   ├── npb_batting.csv
│   ├── npb_fielding.csv
│   ├── npb_people.csv
│   ├── npb_pitching.csv
│   ├── ERD_initial.drawio
│   └── ERD_initial.pdf
├── KBO+NPB+MLB/             # Combined KBO, NPB, and MLB datasets
│   ├── code/
│   ├── kbo_batting.csv
│   ├── kbo_fielding.csv
│   ├── kbo_people.csv
│   ├── kbo_pitching.csv
│   ├── npb_batting.csv
│   ├── npb_fielding.csv
│   ├── npb_people.csv
│   ├── npb_pitching.csv
│   ├── mlb_batting.csv
│   ├── mlb_pitching.csv
│   ├── mlb_fielding.csv
│   ├── mlb_people.csv
│   ├── ERD_future_DRAFT.drawio
│   ├── ERD_future_DRAFT.pdf
│   ├── ERD_cleaned.drawio
│   └── ERD_cleaned.pdf
├── MLB/                         # Raw datasets from Lahman Baseball Database: https://sabr.org/lahman-database/
│   ├── data/
│   └── scrapper/
├── NPB/                         # Raw datasets and web scraper files for NPB
│   ├── data/
│   └── scrapper/
├── vignettes/                   # Data exploration using datasets above
└── README.md                    # This file
```

---
## Contact

For questions please reach out via GitHub or contact:
**Myungkeun Park** | Northeastern University | frankmkpark (GitHub)
