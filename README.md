# Predicting Air Pollutants from Low-Cost Gas Sensors

Two regression projects on the [UCI Air Quality dataset](https://archive.ics.uci.edu/dataset/360/air+quality): hourly readings from a multi-sensor device at road level in an Italian city, alongside a reference-grade analyser. The goal is to predict the accurate reference reading (benzene, then carbon monoxide) from cheap sensor signals and weather, evaluated on a later time period than the one used for training.

## Key findings

- **Benzene is an easy target.** One sensor, `PT08.S2(NMHC)`, follows benzene almost exactly. A decision tree and a random forest reach R² ≈ 1.00 on the test period, and the random forest gives it essentially all of the feature importance. A high score here says little about modelling skill.
- **Carbon monoxide is harder and more informative.** No single sensor is enough (best single sensor R² 0.774, against 0.862 with all features), and weather alone has no skill.
- **Data preparation mattered.** About 330 hours had a reference CO value but no data from any cheap sensor. Imputing them gave every such row the same prediction. Dropping them raised linear regression's R² from 0.77 to 0.86.
- **The simple model holds up.** On a chronological holdout the tuned random forest and linear regression are within 0.004 R² of each other. In time-series cross-validation the forest won 3 of 5 folds and linear regression won the others, so the ranking depends on the period.

## Results (carbon monoxide, chronological test set)

| Model | MAE (mg/m³) | RMSE (mg/m³) | R² |
|---|---|---|---|
| Dummy (median) | 1.000 | 1.301 | -0.001 |
| Linear regression | 0.344 | 0.484 | 0.862 |
| Random forest, original features | 0.368 | 0.538 | 0.829 |
| Decision tree (depth 6) | 0.411 | 0.642 | 0.756 |
| **Random forest, tuned + calendar features** | **0.322** | **0.476** | **0.866** |

## Methods

- Chronological 80/20 train/test split (no random shuffling of time series)
- Dummy baselines, linear regression, decision tree, random forest
- Time-series cross-validation (5 expanding-window folds) and `GridSearchCV` for tuning, on the training period only
- Feature ablation and error analysis (residuals, residuals over time)
- Calendar features (hour encoded with sin/cos, weekend flag)

## How to run

1. Clone the repository and install the dependencies:
   ```bash
   pip install -r requirements.txt
   ```
2. Download `AirQualityUCI.csv` from the [dataset page](https://archive.ics.uci.edu/dataset/360/air+quality) and place it in `data/air_quality/raw/`.
3. Open the notebooks in `notebooks/` (from the project root or from `notebooks/`) and run all cells.

## Project structure

```text
├── data/air_quality/raw/       ← place AirQualityUCI.csv here (not included)
├── notebooks/
│   ├── Benzene_analysis.ipynb
│   └── CarbonMonoxide_analysis.ipynb
├── models/                     ← saved model (created by the CO notebook)
├── requirements.txt
└── README.md
```

## Limitations

- About 13 months of data from one location; the test period is a single 2.5-month window (winter into spring), so seasonal generalisation is untested.
- The test set was also used for model comparison and error analysis, so the final score is not a perfectly untouched holdout.
- The final model's margin over linear regression is small and within the fold-to-fold variation in cross-validation.
- No external validation on another site or year.

## Data

The dataset belongs to its authors; see the UCI page for its licence and citation details.