# Madrid Air Quality: NO₂ Forecasting and Early Alerts

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/santihm/TimeSeries-MadridAirQuality-NO2/blob/main/madrid_no2_forecasting.ipynb)

Group project for **Machine Learning in Time Series and Data Streams** (Master's in Applied Artificial Intelligence, UC3M), by Santiago Herron Mulet, Javier Ramírez Zarzoso and Javier Aceituno Monja.

> The notebook is written in Spanish. This README summarizes it in English.

## What it does

Forecasts daily **NO₂** concentrations in Madrid from 25 years of public data (January 2001 to August 2026) and checks whether the forecasts are useful in practice, as an early warning of high-pollution days and months.

Three series are built from the city's monitoring network: a **general** indicator, a **green-zone** group (3 suburban stations next to Casa de Campo, El Pardo and Parque Juan Carlos I) and an **urban** group (9 traffic stations).

## Method

- **Honest evaluation.** Chronological split (train until the end of 2024, test from 2025), backtesting for hyperparameters, and every model compared against a naive baseline on the same dates. Metrics: MAE and RMSE.
- **Exploratory analysis.** Trend, annual and weekly seasonality, stationarity tests and ACF/PACF.
- **Classical models.** AR, MA, ARMA, ARIMA, SARIMA and SARIMAX (statsmodels).
- **Machine learning.** Ridge and Random Forest (skforecast), with lag and hyperparameter search.
- **Deep learning.** LSTM (Keras with the PyTorch backend).
- **Calendar and weather.** Weekday and holiday features, and rain and wind from the AEMET Madrid-Retiro station as exogenous variables.
- **Horizons.** 1 day, 1 week and 1 month ahead.

## Results

Test MAE on the general series (µg/m³), best model of each family:

| Model | 1 day | 1 week | 1 month |
|---|---|---|---|
| Naive | 5.87 | 8.75 | 9.60 |
| SARIMAX | 5.19 | 6.88 | 7.66 |
| **Ridge (differenced + calendar)** | **5.16** | **6.85** | **7.17** |
| Random Forest (calendar) | 5.49 | 7.48 | 8.49 |
| LSTM (calendar) | 7.42 | 7.72 | 7.89 |

- **A simple model won.** Ridge with differencing and calendar features is the most reliable model, best in 7 of the 9 combinations of series and horizon. It cuts the error versus the naive baseline by 12% at 1 day, 22% at 1 week and 25% at 1 month. Random Forest and LSTM did not beat it, and the Ridge search takes under a minute against more than half an hour for a single LSTM configuration.
- **Early alert at 1 day.** With the WHO threshold (25 µg/m³), Ridge detects the same high days as the naive baseline (62%) with fewer false alarms (56 down to 46). The 50 µg/m³ limit was almost never exceeded in the test period, so it cannot be evaluated well.
- **Alert at 1 month.** Redefining the monthly horizon as the mean of the next 30 days was more useful. Against the 2030 EU limit (20 µg/m³), detection rises from 79% to 84% and the error of the monthly mean drops by about 20%.
- **Green vs urban zones.** The urban series is 1.6 to 2.4 times the green one every year. During the 2020 lockdown the gap fell from 19.8 to 8.7 µg/m³ and went back to 17.1 afterwards, which points to traffic as the main difference. The analysis is descriptive and makes no causal claim.
- **Weather.** Rain and wind from the previous day add little at 1 day (0.5% to 2.4% for SARIMAX and Ridge, up to 5.6% for Random Forest) and hurt the LSTM.

**Limitations.** The test period has very few high-pollution episodes, there is a data gap in December 2024, and the series are averages of several stations, which can hide local episodes.

## How to run

The notebook is designed for **Google Colab**.

1. Click the Colab badge above to open the notebook.
2. Upload the contents of the `data/` folder to your Google Drive in a folder named `AireMadrid` (`My Drive/AireMadrid`).
3. Run all cells and authorize access to Drive when asked. The notebook installs `skforecast` and `holidays` by itself.

The libraries used are listed in `requirements.txt`.

## Data

- **Air quality:** daily measurements, one file per year (`aire2001.csv` to `aire2025.csv`), and the station catalogue (`estaciones_control.csv`). Source: Ayuntamiento de Madrid, open data portal ([datos.madrid.es](https://datos.madrid.es)), *Calidad del aire*.
- **Weather:** daily precipitation and wind speed from the Madrid-Retiro station (3195). Source: AEMET (Agencia Estatal de Meteorología). Files: `retiro_precipitacion_sol_2001_2026.csv` and `retiro_viento_2001_2026.csv`.

The data belong to their original publishers and are included here for reproducibility.
