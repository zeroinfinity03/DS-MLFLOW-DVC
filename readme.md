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

## ✅ What Goes Inside the DVC Pipeline

|------------------------|--------------|--------------------------|
| **Task**              | **Inside DVC?** | **Script Name**        |
|------------------------|--------------|--------------------------|
| EDA & exploration     | ❌ No         | Done in notebook         |
| Data ingestion        | ✅ Yes        | data_ingestion.py        |
| Data preprocessing    | ✅ Yes        | data_preprocessing.py    |
| Feature engineering   | ✅ Yes        | FE.py                    |
| Feature selection     | ✅ Yes        | feature_selection.py     |
| Train/Test Split      | ✅ Optional   | split_data.py            |
| Model training        | ✅ Yes        | train_model.py           |
| Evaluation            | ✅ Yes        | evaluate_model.py        |
| Model registry        | ✅ Yes        | register_model.py        |
|------------------------|--------------|--------------------------|

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



## NOTE_2:

✅ When to Set Up MLflow on AWS (Tracking + S3)
	•	You must set up the MLflow tracking server and S3 artifact store before running: 4.dvc_pipeline/7.register_model.py


## 🔹 What Needs to Be Ready Before Running `register_model.py`

To successfully register your model to MLflow (hosted on AWS), the following infrastructure must already be in place:

|-----------------------|------------------------------|------------------------------------------------------------|
| **Component**         | **Where to Set It Up**       | **Purpose**                                                |
|-----------------------|------------------------------|------------------------------------------------------------|
| MLflow Tracking URI   | AWS EC2 (or ECS container)   | Stores experiment metadata, hyperparameters, metrics, etc. |
| Artifact Store        | Amazon S3 Bucket             | Stores model files (e.g., `model.pkl`, logs, checkpoints)  |
| Credentials           | IAM role or AWS access keys  | Grants permission to log to S3 and access tracking server  |
|-----------------------|------------------------------|------------------------------------------------------------|


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
⸻







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








## 🚀 6. Production 

* Add `6.production/` folder

  * We can have here frontend code as well.
  * Contains FastAPI/Flask `app.py`
  * Loads model from MLflow or local registry

---








## 🚀 7. Deployment

* Add `7.deploy/` folder

  * Contains `Dockerfile`, `start.sh`, and CI/CD scripts
  * Used for containerization and pushing to cloud (e.g., AWS/GCP)
  * Now we deploy backend API to AWS (by dockerizing the API). Our ML model is already stored in the model registry, and the model registry is already in AWS, so anyone can access it.

* Step 1: We dockerize the API → store it in Amazon ECR (Elastic Container Registry)

* Step 2: Retrieve the image from ECR and deploy it to an AWS EC2 instance  
  (First we launch the EC2 instance, then pull the image from ECR)

* Step 3: Run the Docker image inside the EC2 instance → API goes live!

## NOTE:
1. We should **not deploy the app (API)** on just one EC2 instance. Instead, use multiple servers (e.g., 2–3 EC2 instances).

2. We pull the same Docker image from ECR to all these instances and run them in parallel.  
   👉 If one instance goes down, the app keeps running on others.

3. How does the frontend know which server to hit?  
   👉 We use a **Load Balancer** — frontend sends requests to the Load Balancer, which routes them to the EC2 instance with the least load.

4. How do we decide how many servers to run?  
   👉 We use **AWS Auto Scaling Group**.  
   Define min & max capacity — AWS adjusts server count based on traffic.

5. What if we update the API code?

   - Rebuild the API  
   - Dockerize it  
   - Push the new Docker image to ECR  
   - Pull this updated image into each EC2 instance

   ❗ If 4 EC2 instances are running, and we redeploy manually, all 4 may restart at once → downtime risk.  
   ✅ To avoid this, we use **Deployment Strategies**:

   a. **Rolling Update** → update 1–2 servers at a time (zero downtime)  
   b. **Blue-Green Deployment** → maintain old & new versions, switch traffic after testing new version

   ✅ Use **AWS CodeDeploy** to automate this!

---

### 🔁 Finally:

> **This is a lot of manual DevOps.**  
> That’s why we use **GitHub Actions for CI/CD pipeline**.  
> Once pipeline stages are defined (build → test → deploy), GitHub Actions automatically handles all steps.