# Heart Disease Prediction: CI-CD

<p align="center">
  <strong>From clinical data to a retrained model, automatically on every push.</strong>
</p>

<p align="center">
  A reproducible MLOps pipeline for heart disease prediction, powered by MLflow, Docker and GitHub Actions.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white" alt="Python" />
  <img src="https://img.shields.io/badge/pandas-150458?style=flat&logo=pandas&logoColor=white" alt="pandas" />
  <img src="https://img.shields.io/badge/scikit--learn-F7931E?style=flat&logo=scikitlearn&logoColor=white" alt="scikit-learn" />
  <img src="https://img.shields.io/badge/MLflow-0194E2?style=flat&logo=mlflow&logoColor=white" alt="MLflow" />
  <img src="https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white" alt="Docker" />
  <img src="https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat&logo=githubactions&logoColor=white" alt="GitHub Actions" />
</p>

<p align="center">
  <a href="#features">Features</a> •
  <a href="#machine-learning">Machine Learning</a> •
  <a href="#architecture">Architecture</a> •
</p>

---

## What is Heart Disease Prediction: CI-CD?

**Heart Disease Prediction: CI-CD** is a machine learning pipeline that trains a **Random Forest classifier** to predict the presence of heart disease from clinical measurements, and keeps the whole process **reproducible, tracked and automated**.

Every push to `main` triggers retraining through GitHub Actions. Each run is recorded in MLflow, and the trained model is packaged as a **Docker image and published to Docker Hub**, ready to serve predictions.

```text
Push → Retrain → MLflow Tracking → Trained Model → Docker Image → Docker Hub
```

> **Disclaimer:** This is a learning and portfolio project by Dicoding Indonesia Bootcamp. It is not a medical device and must not be used for diagnosis.

---

## Features

| Feature                 | Description                                                         |
| ----------------------- | ------------------------------------------------------------------- |
| MLflow Project          | Package the training code as a reusable, reproducible project       |
| Experiment Tracking     | Log parameters, metrics and the model with MLflow autologging       |
| Automated Retraining    | Retrain the model through GitHub Actions on every push to `main`    |
| Model Serving Image     | Build an MLflow model-serving Docker image from the trained model   |
| Docker Hub Publishing   | Push the image automatically after each successful training run     |
| Artifact Storage        | Upload MLflow run artifacts from every CI run                       |
| Pinned Environment      | Lock Python and library versions for consistent results             |
| Model Export            | Save the trained model as `random_forest_model.pkl`                 |

---

## Machine Learning

This project uses a **Random Forest** classifier for **binary classification** on tabular clinical data.

### Dataset

| Property        | Value                                                        |
| --------------- | ------------------------------------------------------------ |
| File            | `heartdisease_preprocessing.csv` (cleaned and encoded)       |
| Samples         | 296 patients                                                 |
| Features        | 13 clinical features                                         |
| Target          | `target`: 1 = disease present (160), 0 = absent (136)        |

```text
age · sex · chest_pain_type · resting_blood_pressure · cholesterol
fasting_blood_sugar · resting_electrocardiogram · max_heart_rate_achieved
exercise_induced_angina · st_depression · st_slope
num_major_vessels · thalassemia
```

### Model Architecture

```text
Input (13 clinical features)
      │
      ▼
Train / Test Split (80% / 20%)
      │
      ▼
Random Forest (100 trees)
      │
      ▼
Binary Prediction (0 / 1)
```

### Training Configuration

| Parameter          | Value                                   |
| ------------------ | --------------------------------------- |
| Algorithm          | `RandomForestClassifier`                |
| Trees              | 100 (`n_estimators`)                    |
| Train / Test Split | 80% / 20%                               |
| Random Seed        | 2026                                    |
| Tracking           | MLflow autolog + manual accuracy metric |

The trained model is logged to MLflow and also exported with `joblib`.

### Results

<!-- TODO: re-run with the pinned environment and update these numbers if they differ -->

| Metric               | Score  |
| -------------------- | ------ |
| Accuracy             | 83.33% |
| Precision (disease)  | 91.31% |
| Recall (disease)     | 75.00% |

---

## Architecture

```text
┌──────────────────────────────┐
│     Push to `main` branch    │
└──────────────┬───────────────┘
               ▼
┌──────────────────────────────┐
│   GitHub Actions (CI)        │
│   Python 3.10 · install deps │
└──────────────┬───────────────┘
               ▼
┌──────────────────────────────┐
│   mlflow run MLProject       │
│   modelling.py               │
└──────────────┬───────────────┘
        ┌──────┴──────┐
        ▼             ▼
┌──────────────┐  ┌────────────────────┐
│    MLflow    │  │ Upload `mlruns/`   │
│ Params·Metric│  │ as CI artifact     │
│    ·Model    │  └────────────────────┘
└──────┬───────┘
       ▼
┌──────────────────────────────┐
│ mlflow models build-docker   │
│ (model-serving image)        │
└──────────────┬───────────────┘
               ▼
┌──────────────────────────────┐
│ docker push → Docker Hub     │
└──────────────────────────────┘
```

### CI/CD Workflow

The workflow `MLflow CI Pipeline` runs on every push to `main` and performs these steps:

1. Check out the repository and set up Python 3.10
2. Install the pinned dependencies from `MLProject/requirements.txt`
3. Point MLflow tracking to a local `mlruns/` folder
4. Run the MLflow project (`mlflow run MLProject --env-manager=local`) to retrain the model
5. Read the run ID written by `modelling.py` and upload `mlruns/` as a CI artifact
6. Log in to Docker Hub, build a serving image from the logged model with `mlflow models build-docker`, and push it

> Docker Hub publishing requires two repository secrets: `DOCKER_USERNAME` and `DOCKER_PASSWORD` (an access token is recommended).

### Repository Structure

```text
Workflow-CI/
├── .github/workflows/          # CI workflow (GitHub Actions)
├── MLProject/
│   ├── MLproject               # MLflow Project definition
│   ├── python_env.yaml         # Python version and dependency spec
│   ├── requirements.txt        # Pinned dependencies
│   ├── modelling.py            # Training script
│   ├── heartdisease_preprocessing.csv
│   └── Dockerfile              # Optional image for training locally
├── .gitignore
└── README.md
```

---

<p align="center">
  <strong>Workflow-CI</strong><br>
  Train once. Retrain automatically.
</p>
