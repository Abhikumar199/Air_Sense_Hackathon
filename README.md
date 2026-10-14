# AirSense — Next-Day AQI Forecasting and Health Advisories

AirSense is a hackathon prototype that combines historical Air Quality Index (AQI) data with weather features to estimate next-day AQI for Delhi, Gwalior, and Jabalpur. It pairs the forecast with understandable, rule-based health advisories so users can turn an AQI value into practical precautions.

**Project team:** Team Nexus  
**Event:** AI Synergy Hackathon  
**Status:** Prototype. Forecast quality and live-data integration require further validation before real-world use.

## Problem

Public AQI dashboards often emphasize current or historical measurements. AirSense explores the next step: forecasting the following day's AQI and presenting the result in a way that is easier for everyday users to interpret.

The intended users include commuters, families, people with respiratory sensitivities, and residents of cities affected by air pollution.

## What the prototype does

- Uses historical AQI measurements and weather variables to build a forecasting dataset.
- Creates lag features from recent AQI values and combines them with calendar, city, and weather information.
- Trains an XGBoost regression model to estimate AQI.
- Presents forecast information through a web interface.
- Maps AQI values to CPCB-style categories and rule-based advisories.
- Includes a subscription/notification-oriented interface in the application code.

The repository contains data-preprocessing code, model-training notebooks, prepared Excel datasets, UI code, and a separate website module.

## System workflow

1. **Collect data:** AQI data from CPCB-oriented sources and weather data from Visual Crossing, as described by the project.
2. **Prepare data:** clean numeric fields and handle missing values using preceding observations where possible.
3. **Combine sources:** align AQI and weather records for the city-level forecasting dataset.
4. **Engineer features:** include recent AQI lags, calendar fields, city indicators, and weather variables.
5. **Train model:** fit an XGBoost regressor and evaluate using a chronological train/test split.
6. **Generate forecast:** use the model and saved recent city values to estimate next-day AQI.
7. **Explain output:** show AQI category information and a rule-based advisory in the interface.

## Data included

The `Dataset/` directory contains monthly AQI spreadsheets for Delhi, Gwalior, and Jabalpur for 2025, alongside combined city datasets, weather data, and preprocessing notebooks/scripts. The repository also contains a prepared dataset under `Model_training/` and copies used by the interface.

These files represent the data currently committed to this repository. Coverage and completeness should be checked before training: monthly files, missing months, time alignment, sampling intervals, and weather-data provenance all affect the reliability of the evaluation.

## Model

The project uses **XGBoost regression** for AQI prediction. The training script specifies lag features (`AQI_t-1`, `AQI_t-2`, and `AQI_t-3)), city/calendar indicators, and weather variables such as humidity, precipitation, wind speed, and temperature.

The training code uses the first 80% of rows for training and the final 20% for testing. This is a time-ordered split as long as the dataset has been sorted chronologically before the split. Confirm chronological ordering and evaluate against a simple baseline, such as using the most recent AQI value, before relying on the model.

The script reports **MAE, RMSE, and R²** after training. No validated benchmark values are stated here because performance depends on the exact prepared dataset and a reproducible executed run.

## Technology

- **Language:** Python
- **Data processing:** Pandas, NumPy, OpenPyXL
- **Machine learning:** XGBoost, scikit-learn
- **Model persistence:** Joblib
- **Web interface:** Flask
- **Configuration:** python-dotenv
- **Scheduling:** APScheduler is included in the interface requirements

## Repository structure

```text
Air_Sense_Hackathon/
├── DECISIONS.md
├── Dataset/
│   ├── monthly AQI spreadsheets for Delhi, Gwalior, and Jabalpur
│   ├── combined city datasets and weather data
│   ├── Preprocessing.ipynb
│   └── preprocess.py
├── Model_training/
│   ├── XGBoost.ipynb
│   └── final_dataset.xlsx
├── User_interface/
│   ├── app.py
│   ├── requirements.txt
│   ├── Model_training/
│   │   └── train_model.py
│   ├── templates/
│   └── model/
└── website/
    └── web.py
```

## Setup

Clone the repository:

```bash
git clone https://github.com/Abhikumar199/Air_Sense_Hackathon.git
cd Air_Sense_Hackathon
```

Create and activate a virtual environment, then install the interface dependencies:

```bash
python -m venv .venv
```

Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
pip install -r User_interface/requirements.txt
```

macOS/Linux:

```bash
source .venv/bin/activate
pip install -r User_interface/requirements.txt
```

Check the data paths in `User_interface/Model_training/train_model.py` before running it. The script expects `final_dataset.xlsx` in that directory and writes model artifacts relative to its current implementation. Ensure the expected output directory exists and the application’s artifact paths match the training output before launching the interface.

## Environment variables and API keys

The interface uses `python-dotenv` and the repository includes `User_interface/.env.template`. Copy that template to `.env` in the appropriate application directory and fill in required values locally.

Never commit real API keys, email passwords, or other secrets. Use placeholder values in templates and keep local `.env` files out of version control.

## Run the application

The repository contains both `User_interface/app.py` and `website/web.py`. Inspect the intended entry point and its environment requirements before launching. For the Flask interface, a typical command from the repository root is:

```bash
python User_interface/app.py
```

The app requires its model artifacts and expected data files to be in the paths defined by the code. If the model has not been trained or the artifacts are absent, train the model and verify the saved file locations first.

## Design decisions

See [DECISIONS.md](DECISIONS.md) for the team's documented choices, including using XGBoost with lag/weather features, rule-based AQI advisories, a recent-data window, and a hackathon-sized web prototype.

## Limitations and responsible use

- This is a prototype, not an official AQI forecasting service.
- Current repository data and scripts should be checked for completeness, timestamp alignment, and consistent city coverage.
- A chronological split only represents a valid forecasting test when rows are ordered correctly and feature construction avoids looking into the future.
- Filling missing values from previous observations can be reasonable for a time series, but fallback behavior should be checked at sequence boundaries and for long missing intervals.
- The application contains fallback values for some missing inputs; fallback forecasts should be identified clearly rather than presented as equally reliable predictions.
- Predictions should be compared with persistence and seasonal baselines and evaluated on held-out time periods.
- Rule-based advisories are general informational guidance and are not individual medical advice.
- API-based data collection, robust monitoring, deployment hardening, and broader city coverage are future improvements.

## Future work

1. Evaluate against persistence and seasonal baselines with chronological backtesting.
2. Add forecast uncertainty and clearly show when input data are missing or stale.
3. Automate ingestion with documented source timestamps and data-quality checks.
4. Extend historical coverage and validate performance separately for each city and season.
5. Improve deployment, error handling, and testing for missing model files, API failures, timeouts, and rate limits.

## Acknowledgement

AirSense was developed by **Team Nexus** for the **AI Synergy Hackathon**. The project aims to make air-quality forecasts more understandable by connecting a data-driven estimate with transparent, rule-based advisories.
