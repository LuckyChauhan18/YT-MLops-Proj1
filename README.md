This is  readme file 
# 🚗 Vehicle Insurance Purchase Prediction — MLOps End-to-End (FastAPI • Docker • AWS • CI/CD)

[![Python](https://img.shields.io/badge/Python-3.10-blue.svg)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/API-FastAPI-009688.svg)](https://fastapi.tiangolo.com/)
[![MongoDB Atlas](https://img.shields.io/badge/DB-MongoDB%20Atlas-47A248.svg)](https://www.mongodb.com/atlas)
[![Docker](https://img.shields.io/badge/Container-Docker-2496ED.svg)](https://www.docker.com/)
[![GitHub Actions](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-2088FF.svg)](https://github.com/features/actions)
[![AWS](https://img.shields.io/badge/Cloud-AWS-FF9900.svg)](https://aws.amazon.com/)
[![scikit-learn](https://img.shields.io/badge/ML-scikit--learn-F7931E.svg)](https://scikit-learn.org/)

> A production-grade MLOps project that takes data from MongoDB Atlas → validates/transforms it → trains and evaluates models → versions artifacts on S3 → ships a Dockerized FastAPI service to EC2 through GitHub Actions, ECR, and a self-hosted runner.

---

## ✨ What makes this project interesting

* **Real MLOps workflow**: ingestion → validation → transformation → training → evaluation → model registry (S3) → pusher → prediction API.
* **Cloud-native CI/CD**: GitHub Actions builds & pushes images to **Amazon ECR**, then a **self-hosted runner on EC2** deploys the container automatically.
* **Configuration-first**: schema-driven validation (`config/schema.yaml`) and environment-driven secrets.
* **Clean packaging**: `setup.py` + `pyproject.toml` enable local package imports (`src/...`).
* **Observability & reliability**: centralized **logging** and custom **exceptions**.
* **Secure & reproducible**: Dockerfile + `.dockerignore`, versioned infra steps, and environment-agnostic bootstrapping.

---




---

## 🧱 Tech Stack

* **Language**: Python 3.10
* **Core**: FastAPI, scikit-learn, pandas, numpy, imbalanced-learn
* **Storage**: MongoDB Atlas (ingestion), Amazon S3 (model registry)
* **Packaging**: `setup.py`, `pyproject.toml`
* **Orchestration**: GitHub Actions, EC2 self-hosted runner
* **Containerization**: Docker, Amazon ECR
* **Infra/Cloud**: AWS (IAM, S3, ECR, EC2, Security Groups)

---

## 📁 Repository Structure (key parts)

```
.
├── src/
│   ├── components/                 # ingestion, validation, transformation, trainer, etc.
│   ├── configuration/              # mongo_db_connections.py, aws_connection.py
│   ├── data_access/                # DB adapters (proj1_data)
│   ├── entity/                     # config & artifact dataclasses, estimators, s3_estimator.py
│   ├── exception.py                # custom exception
│   ├── logger.py                   # logging configuration
│   └── aws_storage/                # S3 push/pull helpers
├── notebook/
│   ├── mongoDB_demo.ipynb          # push dataset to MongoDB
│   └── EDA_Feature_Engg.ipynb
├── config/
│   └── schema.yaml                 # data validation schema
├── templates/                      # FastAPI templates (if any)
├── static/                         # static assets (if any)
├── app.py                          # FastAPI entrypoint
├── requirements.txt
├── setup.py
├── pyproject.toml
├── template.py                     # project scaffolder
├── Dockerfile
├── .dockerignore
└── .github/workflows/aws.yaml      # CI/CD pipeline
```

---

## ⚡ Quickstart (local)

1. **Generate template (optional)**

```bash
python template.py
```

2. **Create & activate env, install deps**

```bash
conda create -n vehicle python=3.10 -y
conda activate vehicle
pip install -r requirements.txt
pip list  # verify local packages are visible
```

3. **MongoDB Atlas (once)**

* Create **Project** → **M0 Cluster** (free) → **DB User** (username/password)
* Network Access → add IP: `0.0.0.0/0`
* Get **Connection String** (Python 3.6+)

4. **Set Mongo URL**

```bash
# Bash
export MONGODB_URL="mongodb+srv://<user>:<pass>@<cluster>/?retryWrites=true&w=majority"

# PowerShell
$env:MONGODB_URL = "mongodb+srv://<user>:<pass>@<cluster>/?retryWrites=true&w=majority"
```

5. **Load dataset to Mongo (notebook)**

* Put dataset in `notebook/`
* Run `notebook/mongoDB_demo.ipynb` to push documents
* Validate in Atlas → Database → Browse Collections

6. **Run pipelines locally (example)**

```bash
python demo.py    # exercises logger/exception & ingestion pipeline wiring
uvicorn app:app --host 0.0.0.0 --port 5000
```

---

## 🧪 Data & Modeling Workflow

* **Constants & Entities**: configure `constants/__init__.py`, `entity.config_entity.py`, `entity.artifact_entity.py`
* **DB connectivity**: `configuration.mongo_db_connections.py` + `data_access/proj1_data.py`
* **Validation**: define full dataset schema in `config/schema.yaml`
* **Transformation**: feature engineering in component + `entity/estimator.py`
* **Trainer**: scikit-learn models (SVM, Logistic Regression, Random Forest etc.)
* **Evaluation**: production gate via `MODEL_EVALUATION_CHANGED_THRESHOLD_SCORE = 0.02`
* **Registry**: S3 bucket `my-model-mlopsproj` with key prefix `model-registry`

---

## ☁️ AWS Setup (one-time)

### 1) IAM & Env

* Create IAM user (e.g., `firstproj`) → attach **AdministratorAccess**
* Create **Access Keys** (CLI) and export:

```bash
# Bash
export AWS_ACCESS_KEY_ID="..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_DEFAULT_REGION="us-east-1"

# PowerShell
$env:AWS_ACCESS_KEY_ID="..."
$env:AWS_SECRET_ACCESS_KEY="..."
$env:AWS_DEFAULT_REGION="us-east-1"
```

Also add the same in `constants/__init__.py` where required.

### 2) S3 Model Registry

* Create bucket: **`my-model-mlopsproj`** (Region: `us-east-1`)
* Uncheck “Block all public access” (acknowledge)
* `MODEL_PUSHER_S3_KEY = "model-registry"`

### 3) ECR, EC2, Runner

* **ECR**: create repo `vehicleproj`, copy the URI.
* **EC2**: launch **Ubuntu** (`t2.medium`, 30 GiB gp3).
  Allow **HTTP(80)** and **HTTPS(443)** in the Security Group; you’ll add app port later.
* **Install Docker** on EC2:

  ```bash
  curl -fsSL https://get.docker.com -o get-docker.sh
  sudo sh get-docker.sh
  sudo usermod -aG docker ubuntu
  newgrp docker
  docker --version
  ```
* **Self-hosted runner**:
  Repo → Settings → Actions → Runners → *New self-hosted runner* (Linux).
  Run the **Download** & **Configure** commands on EC2, then `./run.sh`.
  Runner should show as **idle** in GitHub.

---

## 🔁 CI/CD

### Secrets (GitHub → Settings → Secrets and variables → Actions)

```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_DEFAULT_REGION  (us-east-1)
ECR_REPO            (vehicleproj)
```

### Pipeline

`.github/workflows/aws.yaml`

* **CI**: Builds Docker image → pushes to **ECR**
* **CD**: On the **EC2 self-hosted runner**, pulls & runs the image

> The app listens on **5000** inside the container.
> Expose it publicly by mapping **5080:5000** (or 80:5000) and opening that port.

**Security Group inbound rule (to test quickly):**

* Type: **Custom TCP**
* Port: **5080** *(or any port you map)*
* Source: **0.0.0.0/0** *(restrict to your IP in production)*
* Save.

Then open: `http://<EC2-Public-IP>:5080`

---

## 🐳 Docker (local dev)

```bash
docker build -t vehicle-app:latest .
docker run -d -p 5080:5000 \
  -e MONGODB_URL="$MONGODB_URL" \
  -e AWS_ACCESS_KEY_ID="$AWS_ACCESS_KEY_ID" \
  -e AWS_SECRET_ACCESS_KEY="$AWS_SECRET_ACCESS_KEY" \
  -e AWS_DEFAULT_REGION="$AWS_DEFAULT_REGION" \
  vehicle-app:latest
```

---

## 🔐 Environment Variables

| Variable                                                           | Purpose                       |
| ------------------------------------------------------------------ | ----------------------------- |
| `MONGODB_URL`                                                      | Atlas connection string       |
| `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_DEFAULT_REGION` | S3/ECR access                 |
| (optional) project-specific                                        | e.g., model thresholds, flags |

---

## 📌 Endpoints (example)

* `GET /` – health/info
* `POST /predict` – JSON payload → model inference
* `GET /training` – (optional) trigger training pipeline
* `GET /version` – active model metadata (from S3)

---

## 🧰 Troubleshooting

* **ECR push error “repository … does not exist”**
  Make sure the repo name in **`ECR_REPO`** secret exactly matches the ECR repository (e.g., `vehicleproj`).

* **Port not reachable**
  Confirm `docker run -p 5080:5000 ...` and add inbound rule for **5080/TCP** in the EC2 Security Group.

* **EC2 costs**
  `t2.medium` is **not free tier**. Stop the instance when not in use (EBS storage may still incur small charges).
  For demos, `t3.micro` is often enough for the API (but not for heavy training).

* **Runner stopped**
  If you pressed `Ctrl+C` in the runner shell, start again with `./run.sh`.

---

## 🧾 Project Setup — Step-by-Step (condensed)

1. Template, env, requirements
2. MongoDB Atlas M0 → user → network `0.0.0.0/0` → **MONGODB_URL**
3. Push dataset via `notebook/mongoDB_demo.ipynb`
4. Implement components: **ingestion → validation → transformation → trainer → evaluation**
5. S3 model registry: bucket **`my-model-mlopsproj`**, key **`model-registry`**
6. Dockerize (`Dockerfile`, `.dockerignore`)
7. GitHub Actions + Secrets + ECR repo
8. EC2 (Ubuntu), install Docker, register self-hosted runner
9. Open inbound port **5080** → browse `http://<EC2-IP>:5080`
10. (Optional) Train via `/training`

---

## 🔍 Notes for Reviewers

* Clean separation of **config**, **entities**, **components**, and **storage** layers.
* Reproducible pipelines; **schema-based** checks reduce data drift surprises.
* Real **cloud delivery** with container images, a private registry (ECR), and **push-button deploys**.

---

## 📄 License

This project is for educational and demonstration purposes. Add your preferred license.

---

## 🙌 Acknowledgements

* scikit-learn, FastAPI communities
* MongoDB Atlas free tier
* AWS Free Tier services used where possible (note: EC2 `t2.medium` is billable)

---

**Pro tip:** In production, lock down security groups to your office IP, rotate IAM keys, store secrets in a vault (e.g., AWS Secrets Manager), and enable logging/metrics for the API and training jobs.
