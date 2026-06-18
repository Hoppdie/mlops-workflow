# MLOps Workflow — Architecture Guide

## System Overview

This repo implements a production-grade MLOps pipeline covering the full model lifecycle: experiment tracking → deployment → monitoring → automated retraining.

## Component Breakdown

### 1. Data Versioning (DVC)

DVC tracks dataset versions alongside code, ensuring full reproducibility across runs.

    dvc init
    dvc add data/raw/dataset.csv
    dvc push
    git add data/raw/dataset.csv.dvc .gitignore
    git commit -m 'data: add raw dataset v1'

### 2. Experiment Tracking (MLflow)

Every training run logs parameters, metrics, and model artifacts:

    import mlflow
    with mlflow.start_run(run_name="xgboost-baseline"):
        mlflow.log_params({"n_estimators": 300, "max_depth": 6})
        model.fit(X_train, y_train)
        mlflow.log_metrics({"auc": auc_score, "f1": f1})
        mlflow.sklearn.log_model(model, "model")

Access the UI: `mlflow ui --port 5000`

### 3. Model Registry

| Stage | Meaning |
|---|---|
| None | Logged, not yet evaluated |
| Staging | Passed offline eval, ready for shadow testing |
| Production | Live, serving real traffic |
| Archived | Replaced by newer version |

### 4. CI/CD (GitHub Actions)

On every push to `main`:
1. Lint + unit tests
2. Training smoke test on small data subset
3. Model evaluation against held-out test set
4. If metrics pass threshold → promote to Staging
5. Build and push Docker image

### 5. Model Serving (FastAPI + Docker)

    docker build -t mlops-workflow .
    docker run -p 8000:8000 mlops-workflow

The API loads the Production model from the registry on startup — registry-driven, no manual file management.

### 6. Monitoring & Drift Detection

Evidently generates drift reports comparing recent traffic against training distribution.
Drift alerts trigger a retraining pipeline via scheduled GitHub Actions.

## Local Development

    git clone https://github.com/Hoppdie/mlops-workflow.git
    cd mlops-workflow
    pip install -r requirements.txt
    docker-compose up

## Directory Structure

    mlops-workflow/
    ├── data/               # DVC-tracked datasets
    ├── src/
    │   ├── train.py        # MLflow training script
    │   ├── evaluate.py     # threshold-based evaluation
    │   └── serve.py        # FastAPI app
    ├── tests/
    ├── .github/workflows/
    ├── Dockerfile
    └── docs/
        └── ARCHITECTURE.md

## Contact

Adhiyan Anbazhagan · adhiyan2005@gmail.com