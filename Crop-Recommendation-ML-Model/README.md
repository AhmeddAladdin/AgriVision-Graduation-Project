# Crop Recommendation ML Model

Flask-based crop recommendation system that predicts the best crops using soil and climate features.

## Highlights
- Predict top-N crops with probabilities
- Web UI and JSON API
- Training pipeline included

## Project Structure
```
.
+- apps/
¦  +- api.py                 # JSON API (Flask)
¦  +- web_app.py             # Web UI (Flask)
¦  +- legacy/                # older versions
¦  +- templates/             # HTML templates
+- artifacts/                # trained model + preprocessors + data splits
+- data/
¦  +- raw/                   # original dataset
+- notebooks/
¦  +- EDA.ipynb
¦  +- model_training.ipynb
¦  +- figures/               # notebook plots
+- scripts/
¦  +- test_model.py           # quick local test
+- src/                       # core ML pipeline
+- Dockerfile
+- requirements.txt
+- README.md
```

## Requirements
- Python 3.9+

Install dependencies:
```bash
pip install -r requirements.txt
```

## Run Web App
```bash
python apps/web_app.py
```
Then open `http://localhost:5000`.

## Run JSON API
```bash
python apps/api.py
```
Default port: `7860`

### Example API Request
```bash
curl -X POST http://localhost:7860/submit_prediction \
  -H "Content-Type: application/json" \
  -d '[{"N":90,"P":42,"K":43,"temperature":20.8,"humidity":82.0,"ph":6.5,"rainfall":120.4}]'
```

Response (top-N crops):
```json
[{"name":"rice","probability":87.12}]
```

## Train The Model
```bash
python src/pipeline/train_pipeline.py
```
This will read the raw dataset from `data/raw/Crop_recommendation.csv` and write artifacts to `artifacts/`.

## Docker
```bash
docker build -t crop-recommendation .
docker run -p 7860:7860 crop-recommendation
```

## Input Features
- `N` nitrogen
- `P` phosphorus
- `K` potassium
- `temperature` (°C)
- `humidity` (%)
- `ph`
- `rainfall` (mm)

## Notes
- `apps/legacy` contains older API/UI versions.
- `docs/api_url.txt` stores the external API URL (if used).
