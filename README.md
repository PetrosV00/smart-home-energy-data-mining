# Smart Home Energy Data Mining

Data mining on almost four years of per-minute electricity measurements from a single household: data cleaning, feature engineering, classification, regression, clustering, association rule mining and time-series forecasting, all in Python.

> Group project for the *Data Mining* course, Department of Informatics and Telematics, Harokopio University of Athens (2025–2026).

## Dataset

[Individual Household Electric Power Consumption](https://archive.ics.uci.edu/dataset/235/individual+household+electric+power+consumption) (UCI Machine Learning Repository): 2M+ measurements taken every minute between 2006 and 2010, including global active/reactive power, voltage, current intensity and three sub-meters (kitchen, laundry, water heater/air conditioning).

The data is messy: it contains gaps (whole missing days), irregular timestamps and strong daily/weekly/yearly seasonality. Daily weather data (min/max temperature, humidity, cloud cover) for Paris from [Open-Meteo](https://open-meteo.com/) was added as an external source.

The dataset is **not included** in this repository (see [How to run](#how-to-run)).

## What was done

**1. Preprocessing and feature engineering**
- Missing values handled with time-based interpolation (limited to 30 consecutive minutes, so that entire missing days are not invented); the remaining gaps were dropped (18 days, about 1.25% of the data).
- Resampling from per-minute to daily data, and conversion of measurements to kWh.
- New features: `Daily_total_power`, `Peak_hour_power` (18:00–22:00), `Nighttime_usage` (00:00–06:00), day of week, month, season, `is_workDay`/weekend flag.
- Weather features shifted by one day so that they act as forecasts for the next day.
- Standardization (Z-score) instead of Min-Max scaling, which is sensitive to outliers.

**2. Classification** – predict whether a day has high or low consumption (Decision Tree, entropy criterion).

**3. Regression** – predict next-day consumption in kWh (Linear Regression vs. Random Forest, evaluated with cross-validation).

**4. Clustering** – group days with similar consumption profiles (K-Means, k = 2–6, evaluated with Silhouette and Davies–Bouldin).

**5. Association rules** – find patterns linked to high daily consumption (Apriori, `min_support = 0.1`, `lift >= 1.2`).

**6. Time-series forecasting** – daily consumption forecast with Prophet (yearly and weekly seasonality, weather regressors, 80/20 chronological split).

## Results

| Task | Model | Main results |
|---|---|---|
| Classification | Decision Tree | Accuracy 0.827, F1 0.82 / 0.83 (low / high), ROC-AUC 0.828 |
| Regression | Linear Regression | Cross-validated RMSE ≈ 7.4 kWh (normalized RMSE 0.737) |
| Regression | Random Forest | Cross-validated RMSE ≈ 6.7 kWh (normalized RMSE 0.679) |
| Clustering | K-Means (daily power + workday flag) | Silhouette 0.41–0.51, Davies–Bouldin 0.72–0.88 for k = 2–6 |
| Association rules | Apriori | Top rules reach lift ≈ 2.1 (e.g. weekend + high sub-meter 3 usage → high daily consumption) |
| Forecasting | Prophet | RMSE 7.09 kWh, MAPE 34.4% (days near zero consumption filtered out) |

### Limitations

Day-ahead regression and forecasting stayed far from an accurate prediction. Our analysis points to three causes: unpredictable human behaviour (guests, extra laundry, absences), the lack of per-appliance state history (e.g. whether the water heater is already hot), and noisy day-to-day fluctuations in the data. Clustering is most informative on weekdays vs. weekends, while unusual days (high consumption on weekdays, low on weekends) keep appearing.

## Repository structure

```
.
├── Notebooks/
│   ├── preprocessing.ipynb        # cleaning, resampling, feature engineering, scaling
│   └── models.ipynb               # classification, regression, clustering, association rules, forecasting
├── CSV-pickles/
│   ├── df_final.pkl               # preprocessed (scaled) daily dataset
│   ├── df_denormanized.pkl        # same dataset in the original scale
│   └── weather_conditions.csv     # daily weather data (Open-Meteo)
├── .gitignore
└── README.md
```

## How to run

The notebooks were developed and run in [Google Colab](https://colab.research.google.com/), so no local setup is needed:

| Notebook | Open in Colab |
|---|---|
| `preprocessing.ipynb` | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/PetrosV00/smart-home-energy-data-mining/blob/main/Notebooks/preprocessing.ipynb) |
| `models.ipynb` | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/PetrosV00/smart-home-energy-data-mining/blob/main/Notebooks/models.ipynb) |

1. `preprocessing.ipynb` needs the raw dataset, which is too large for GitHub. Download it from the [UCI repository](https://archive.ics.uci.edu/dataset/235/individual+household+electric+power+consumption) and unzip it. Run the first code cell of the notebook (it clones the repository), then use the Colab file browser to upload `household_power_consumption.txt` into the cloned folder `smart-home-energy-data-mining/Notebooks/`. The daily weather data is in `CSV-pickles/weather_conditions.csv`.
2. `models.ipynb` can be run directly: it loads the preprocessed dataset from `CSV-pickles/df_final.pkl`, so the preprocessing does not have to be repeated.
3. The first code cell of each notebook clones this repository, so that the files in `CSV-pickles/` are available in the Colab session.

Main libraries: pandas, NumPy, scikit-learn, mlxtend, Prophet, matplotlib, seaborn. Colab provides most of them; if an import fails, install the package in the first cell with `!pip install <package>`.

To run locally instead, clone the repository and install the libraries above (`pip install pandas numpy scikit-learn mlxtend prophet matplotlib seaborn jupyter`).

## Team

Group project by:
- [Nikos Kaparos](https://github.com/nikos-kaparos)
- [Petros Vougioukas](https://github.com/PetrosV00) – **My contribution:** data preprocessing (cleaning, resampling, feature engineering, scaling) and the implementation, evaluation and interpretation of the classification, regression, clustering and association rule models. The time-series forecasting part (Prophet) was done by my teammate.
