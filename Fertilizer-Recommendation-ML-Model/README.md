# Fertilizer Recommendation ML Model

Machine learning project for recommending the best fertilizer based on soil, weather, and crop features.

## What This Project Includes
- End-to-end training pipeline (ingestion, preprocessing, model selection, training).
- Inference pipeline with top-N fertilizer recommendations and probabilities.
- Flask API for programmatic prediction.
- Flask web app endpoints for UI-based prediction.
- Docker support for API deployment.

## Updated Project Structure
```text
Fertilizer-Recommednation-ML-Model/
|-- apps/
|   |-- api_app.py                 # Flask API app
|   |-- web_app.py                 # Flask web app (templates expected)
|   `-- __init__.py
|-- artifacts/
|   |-- data.csv
|   |-- train.csv
|   |-- test.csv
|   |-- model.pkl
|   |-- preprocessor.pkl
|   `-- label_encoder.pkl
|-- docs/
|   `-- api_direct_url.txt
|-- notebooks/
|   |-- EDA.ipynb
|   |-- model_training.ipynb
|   `-- data/
|       `-- fertilizer_recommendation_dataset.csv
|-- src/
|   |-- components/
|   |-- pipeline/
|   |-- exception.py
|   |-- logger.py
|   `-- utils.py
|-- tests/
|   `-- test_model.py
|-- Dockerfile
|-- requirements.txt
`-- setup.py
```

## Setup
```bash
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

## Train the Model
```bash
python -m src.pipeline.train_pipeline
```

This regenerates model artifacts under `artifacts/`.

## Run API App
```bash
python -m apps.api_app
```

- Health route: `GET /home_page`
- Prediction route: `POST /submit_prediction`

### Example JSON Body
```json
[
  {
    "Temperature": 25.38,
    "Moisture": 0.398843879,
    "Rainfall": 182.0611454,
    "PH": 7.049864303,
    "Nitrogen": 71.22304264,
    "Phosphorous": 135.1388053,
    "Potassium": 147.4572035,
    "Carbon": 0.800911123,
    "Soil": "Neutral Soil",
    "Crop": "papaya"
  }
]
```

## Run Web App
```bash
python -m apps.web_app
```

Note: `web_app.py` expects `templates/` pages (`index.html`, `home.html`).

## Run Test Script
```bash
python -m tests.test_model
```

## Docker
```bash
docker build -t fertilizer-api .
docker run -p 7860:7860 fertilizer-api
```

## Notes
- Paths/imports were standardized to use `src.*` and root-relative `artifacts/` + `notebooks/` paths.
- Logs are written to `fertilizer_recomm_logs/` at runtime.
