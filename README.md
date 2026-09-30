<h1 align="center">🌱 AgriVision</h1>
<h3 align="center">An End-to-End Smart Agriculture Platform — Recommendation Systems, Computer Vision & Edge Deployment</h3>

<p align="center">
<i>Graduation Project — Faculty of Engineering, Mansoura University (2021–2026)</i>
</p>

> **Note:** This repository is a **case-study writeup** of AgriVision. The full source code lives in a private repository (university/team policy), so this repo documents the problem, approach, architecture, and results with real metrics, diagrams, and media instead of source code.

---

## 📌 Overview

AgriVision is a smart agriculture platform built by a 4-person graduation team, combining **machine learning recommendation systems**, **computer vision**, and **robotics/edge deployment** into one integrated pipeline to support farmers with data-driven crop management decisions.

The platform is made up of four collaborating components:

| Component | Description | Owner |
|---|---|---|
| 🌾 Crop Recommendation | ML model recommending the optimal crop for a given soil/environment profile | **Ahmed AlaaEldin** |
| 🧪 Fertilizer Recommendation | ML model recommending the optimal fertilizer type based on soil composition | **Ahmed AlaaEldin** |
| 🌿 Plant Growth Stage Detection | Computer vision model detecting crop growth stage (tomato & cotton) | **Ahmed AlaaEldin** |
| 🤖 Autonomous Navigation | 3-model CV pipeline for obstacle detection, depth estimation & terrain segmentation | **Ahmed AlaaEldin** |
| 💧 Irrigation System | Smart irrigation model | **Ahmed Orabi** |
| 🐛 Pest & Disease Detection | CV models for pest and disease identification | **Ahmed Orabi** |

This README focuses on **my contributions**: the two recommendation models, the Plant Growth Stage Detection model, the Autonomous Navigation pipeline design, and the edge/cloud deployment of the platform's CV and ML models.

---

## 🌾 1. Crop Recommendation Model

**Goal:** Recommend the most suitable crop to plant given a set of soil and environmental readings.

- **Dataset:** [Crop Recommendation Dataset (Kaggle)](https://www.kaggle.com/datasets/atharvaingle/crop-recommendation-dataset) — soil nutrients (N, P, K), temperature, humidity, pH, and rainfall, labeled with the recommended crop.
- **Approach:**
  1. Exploratory data analysis of nutrient/climate distributions per crop class.
  2. Preprocessing and feature scaling.
  3. Trained and compared multiple classifiers.
  4. Selected the best-performing model based on validation accuracy.
- **Best Model:** `RandomForestClassifier`
- **Result:** **99% accuracy** on the test set.

[Crop Recommendation Results](assets/Crop_model_results.png)

---

## 🧪 2. Fertilizer Recommendation Model

**Goal:** Recommend the optimal fertilizer type based on soil composition and crop type.

- **Dataset:** [Fertilizer Recommendation Dataset (Kaggle)](https://www.kaggle.com/datasets/nishchalchandel/fertilizer-recommendation) — soil nutrient levels, crop type, and soil type, labeled with the recommended fertilizer.
- **Approach:**
  1. Preprocessing of categorical (crop type, soil type) and numerical (N, P, K) features.
  2. Trained and compared multiple classifiers.
  3. Selected the best-performing model based on validation accuracy.
- **Best Model:** `GradientBoostingClassifier`
- **Result:** **99% accuracy** on the test set.

[Fertilizer Recommendation Confusion Matrix](assets/Fertilizer_model_results.png)

Both models were deployed to production on an **AWS EC2** instance, serving predictions in real time within the AgriVision platform.

---

## 🌿 3. Plant Growth Stage Detection (Computer Vision)

**Goal:** Detect the current growth stage of tomato and cotton crops from field images, to support timely agricultural decisions (irrigation, fertilization, harvesting).

- **Model:** YOLOv8 (object detection)
- **Classes:** 7 growth stages across tomato and cotton crops
- **Result:** **mAP50 ≈ 0.83** on the validation set

[Growth Stage Detection Metrics](assets/growth_stage_metrics.jpeg)

---

## 🤖 4. Autonomous Navigation Pipeline

**Goal:** Enable a field robot to navigate autonomously between crop rows — detecting obstacles, estimating depth, and understanding terrain in real time.

A 3-model pipeline combining:

- **YOLOv8** — real-time obstacle detection
- **Depth Anything V2 (Small)** — monocular depth estimation
- **SegFormer-B0** — terrain/path segmentation

These three outputs are combined to give the robot a real-time understanding of what's ahead, how far it is, and where safe terrain is — enabling it to navigate the field autonomously.

[AgriVision Field Robot](assets/robot.jpeg)

---

## ⚙️ Edge & Cloud Deployment

| Model(s) | Deployment Target | Why |
|---|---|---|
| Crop & Fertilizer Recommendation | **AWS EC2** | Centralized, always-on inference for the platform's web/app users |
| Growth Stage, Pest Detection, Disease Detection (CV models) | **Raspberry Pi 5** | Local, low-latency, fully offline edge inference in the field — no dependency on internet connectivity |

Running the CV models locally on a Raspberry Pi 5 was a deliberate choice to keep the field hardware functional even with poor or no connectivity, which is a common constraint in rural agricultural settings.

---

## 🛠️ Tech Stack

`Python` · `Scikit-learn` · `YOLOv8` · `Depth Anything V2` · `SegFormer` · `Pandas` · `NumPy` · `joblib` · `AWS EC2` · `Raspberry Pi 5` · `Flask`

---

## 👥 Team

| Role | Name |
|---|---|
| ML Lead — Crop & Fertilizer Recommendation, CV (Growth Stage, Autonomous Navigation), Deployment | **Ahmed AlaaEldin** |
| Team Member | Ahmed Basem |
| Team Member | Ahmed Orabi |
| Team Member | Mohamed Atef |
| Academic Supervisor | Assoc. Prof. Doaa Adel |

**Institution:** Faculty of Engineering, Mansoura University — Communications and Electronics Engineering (2021–2026)

---

## 👤 Author

**Ahmed AlaaEldin**
[LinkedIn](https://www.linkedin.com/in/ahmed-alaaeldin-1395292a6/) · [GitHub](https://github.com/AhmeddAladdin) · [Kaggle](https://www.kaggle.com/ahmedaldaly)
