## 🧪 Full Production-Ready Data Science Project: README (Explains Flow + Logic)

---

## ✅ Environment Setup

### 1. Initialize project with uv

```bash
uv init
```

### 2. Install core ML/data libraries

```bash
uv add pandas polars matplotlib seaborn scipy pydantic pingouin statsmodels scikit-learn mlflow dvc fastapi requests numpy torch tensorflow prophet transformers jupyter uvicorn python-multipart joblib docker boto3
```

### 3. Sync lockfile

```bash
uv sync
```

### 4. Run any script

```bash
uv run main.py  # Replace with train.py, evaluate.py, etc.
```

---






## 📂 Folder Structure (This Project)

```
FULL_DS_PROJECT/
├── .venv/
├── 1.data/                         ← Raw & cleaned data
├── 2.notebooks/                   ← Jupyter notebooks
│   ├── eda.ipynb
│   ├── cleaning.ipynb
│   └── fe.ipynb
├── 3.experiments/                 ← MLflow experiments
├── 4.dvc_pipeine/                 ← DVC pipeline scripts
│   ├── 1.get_data.py
│   ├── 2.prep_clean_data.py
│   ├── 3.FE.py
│   ├── 4.split_data.py
│   ├── 5.train_model.py
│   ├── 6.evaluate_model.py
│   ├── 7.register_model.py
│   ├── dvc.yaml
│   └── params.yaml
├── 5.testing/                     ← Model readiness checks
│   ├── loading.py
│   └── performance.py
├── 6.production/                  ← FastAPI app for inference
├── 7.deploy/                      ← Docker, CI/CD configs
├── main.py                        ← Entry script (optional)
├── pyproject.toml                 ← Python dependencies
├── .python-version                ← Python version pinning
└── readme.md                      ← Project flow + doc
```

---









## 🔁 Industry-Grade Lifecycle of a Data Science Project

### 1. Notebooks (EDA, Cleaning, Feature Engineering)

* Performed in `2.notebooks/`
* Use Jupyter to understand patterns, distributions, and generate features
* Cleaning and feature ideas are prototyped here before converting to scripts

### 2. Experimentation with MLflow

* Use `mlflow` to try different models, track hyperparameters and scores
* Helps decide best model + parameter combo to move to production

### 3. Store final parameters

* Once best run is found, store its parameters in `params.yaml`
* This will be used in DVC pipeline

---

## ⚙️ 4. DVC Pipeline: Automate Data → Model Flow

### ✅ Why use `dvc.yaml` and `params.yaml`?

| File        | Purpose                          | Used By      |
| ----------- | -------------------------------- | ------------ |
| dvc.yaml    | Defines step-by-step pipeline    | DVC engine   |
| params.yaml | Stores tunable parameters/config | Your scripts |

### ✅ What DVC Automates

| Goal                            | DVC Handles? |
| ------------------------------- | ------------ |
| Auto-run only changed steps     | ✅ Yes        |
| Version control for data/models | ✅ Yes        |
| Reproducibility                 | ✅ Yes        |
| Avoid retraining needlessly     | ✅ Yes        |
| Re-run stages only if needed    | ✅ Yes        |

### ✅ What Goes Inside the Pipeline

| Task                | Inside DVC? | Notes                |
| ------------------- | ----------- | -------------------- |
| EDA & exploration   | ❌ No        | Done in notebook     |
| Cleaning logic      | ✅ Yes       | `prep_clean_data.py` |
| Feature engineering | ✅ Yes       | `FE.py`              |
| Train/test split    | ✅ Optional  | `split_data.py`      |
| Model training      | ✅ Yes       | `train_model.py`     |
| Evaluation          | ✅ Yes       | `evaluate_model.py`  |
| Model registry      | ✅ Yes       | `register_model.py`  |

## NOTE_1:
✅ In This Step (4.dvc_pipeline/7.register_model.py)
	•	The model is pushed to the MLflow model registry, which is hosted on AWS
	•	All model files (artifacts like model.pkl) are saved to Amazon S3
	•	This makes the model available for remote access during deployment
	•	Example logic in register_model.py:

```bash
import mlflow
import mlflow.sklearn

mlflow.set_tracking_uri("https://<your-mlflow-server-on-aws>")
mlflow.set_experiment("credit-risk")

with mlflow.start_run():
    mlflow.log_artifact("model.pkl")
    mlflow.sklearn.log_model(model, "model")
    mlflow.register_model("runs:/<run_id>/model", "CreditRiskModel")

```
✅ By the time deployment begins, the ML model is already registered on AWS and ready to be accessed by the API.

⸻


## NOTE_2:

✅ When to Set Up MLflow on AWS (Tracking + S3)
	•	You must set up the MLflow tracking server and S3 artifact store before running:

4.dvc_pipeline/7.register_model.py


⸻

🔹 What needs to be ready beforehand?

Component	Where to set it up	Purpose
MLflow tracking URI	On AWS EC2 (or ECS)	Stores experiment runs, metrics, metadata
Artifact store	Amazon S3 bucket	Stores model files (model.pkl, logs, etc.)
Credentials	IAM role or access keys	Allows MLflow to upload to S3


⸻

✅ Why before?

Inside register_model.py you set:
mlflow.set_tracking_uri("https://<your-mlflow-server-on-aws>")

→ So if that MLflow server isn’t already running,
→ the script will fail to log or register anything.

⸻


### ✅ What if new data comes in (after 6 months)?

```bash
# If same structure (just new rows):
# ✅ Update CSV → dvc repro

# If structure changed (new cols, formats):
# ❗ Update cleaning or FE script → dvc repro
```

| Situation                      | What You Do                          |
| ------------------------------ | ------------------------------------ |
| Just new rows                  | ✅ Replace data → `dvc repro`         |
| New column structure or format | ❗ Edit script → `dvc repro`          |
| Only parameter change          | ✅ Update `params.yaml` → `dvc repro` |

### ✅ DVC’s Strength in One Line:

🧠 You write logic ONCE → ⚙️ DVC reruns when inputs change

---

## 🧪 5. Testing: Model Readiness Before Deployment

We do **2 stages of testing** — in this order:

### ✅ 1. Model Loading Test (Most Important)

```text
Script: 5.testing/loading.py
→ Check if model loads from registry
→ Check if it gives any prediction at all
→ If it fails here, skip further testing
```

### ✅ 2. Performance Testing (Choose ONE)

```text
Script: 5.testing/performance.py
→ Option A: Check if accuracy ≥ required threshold (e.g., 85%)
→ Option B: Compare v18 with v17 → if better, promote
```

| Type             | What it checks                    |
| ---------------- | --------------------------------- |
| Load Test        | Can we load model from registry?  |
| Prediction Test  | Does it give valid predictions?   |
| Performance Test | Either threshold or version-based |

---

## 🚀 6. Production & Deployment

* Add `6.production/` folder

  * We can have here frontend code as well.
  * Contains FastAPI/Flask `app.py`
  * Loads model from MLflow or local registry

* Add `7.deploy/` folder

  * Contains `Dockerfile`, `start.sh`, and CI/CD scripts
  * Used for containerization and pushing to cloud (e.g., AWS/GCP)
  * Now we deploy backend api in AWS(by dockerizing the api), our ML Model is already stored in the model registry, and model registry is already in the AWS, so anyone can access it.


---
