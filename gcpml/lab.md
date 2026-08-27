# Enterprise MLOps End-to-End on Google Cloud Platform
### Session Explainer (Markdown)

---

## Overview

This is a **3-hour live teaching session** that walks learners through the complete enterprise machine learning lifecycle — end to end — using **Google Cloud Platform (GCP)**.

It covers everything from setting up a cloud environment and deploying a model, through production monitoring, retraining, governance, automation, and finally tying it all back to business value (ROI).

The session is built as **one single connected workflow**, not disconnected demos — learners follow one business problem, one model, and one dataset all the way through: setup → deployment → monitoring → retraining → governance → cost/ROI → automation.

**Type:** Teaching session
**Duration:** 3 hours
**Teaching style:** ~20% concept explanation, ~80% guided hands-on implementation

---

## Prerequisites

Learners should already know:
- Basic supervised ML and MLOps concepts
- Python fundamentals
- REST APIs and Docker basics
- How to evaluate a model (common ML metrics)
- Basic cloud computing concepts

---

## Outcomes — What Learners Will Be Able to Do

By the end, learners can:

1. Set up a GCP environment — account, project, billing, budgets, alerts
2. Prepare and validate a lightweight dataset and model for deployment
3. Build and containerize a **FastAPI** inference service
4. Deploy that service on **Cloud Run** and generate predictions
5. Store prediction outputs in **Cloud Storage**
6. Monitor infrastructure, data quality, drift, and model performance
7. Track experiments and model versions (**MLflow**)
8. Retrain a production model on new data and compare it against the current one
9. Deploy updated model versions and **roll back** if needed
10. Apply **IAM**, governance, and approval practices
11. Evaluate cloud costs, break-even points, and business **ROI**
12. Automate scheduled predictions using **Cloud Scheduler**
13. Explain how all these GCP services fit together into one enterprise MLOps architecture

---

## Learner Timetable (High Level)

| Section | Topic |
|---|---|
| 1 | GCP Environment Setup and ML Deployment |
| 2 | Production Predictions, Monitoring, and Experiment Tracking |
| 3 | Retraining, Model Lifecycle, and Governance |
| 4 | Cost, ROI, Automation, and Enterprise Architecture |
| 5 | Conclusion and Q&A |

*(One 10-minute break is scheduled at an appropriate point by the instructor.)*

---

## Instructor Timetable (Detailed)

### Section 1 — GCP Environment Setup and ML Deployment *(50 min)*

**A. GCP Account, Billing, and Environment Setup**
- Introduce Google Cloud's role in enterprise MLOps
- Demonstrate: account creation, payment setup, project creation, billing account linking, Cloud Billing dashboard, budgets, billing alerts, project/service management, resource usage monitoring, enabling/disabling services
- Key principle taught: **be cost-aware before provisioning anything**

**B. Data and Model Preparation**
- Introduce the chosen business problem: objective, prediction task, historical dataset, prediction dataset, target variable, pre-trained model, expected output
- Discuss: dataset schema, data validation, data quality, model artifact management, storage organization
- Demonstrate: uploading historical data, prediction data, and the model artifact to **Cloud Storage**, plus basic schema validation

**C. Inference Service and Deployment**
- Introduce: FastAPI, REST APIs, Docker, Artifact Registry, Cloud Run
- Explain the deployment chain:
  **Model → FastAPI → Docker → Artifact Registry → Cloud Run**
- Demonstrate: building the FastAPI app (with request validation, model loading, `/predict`, `/health`, `/metadata` endpoints), writing the Dockerfile, building the container, pushing the image to Artifact Registry, deploying to Cloud Run, and exposing an HTTPS endpoint

**D. Production Prediction**
- Demonstrate: uploading the prediction CSV to Cloud Storage, sending requests to the deployed Cloud Run API, generating predictions, writing outputs back to Cloud Storage, reviewing results and model metadata
- Concept introduced: **online inference vs. batch prediction**

---

### Section 2 — Production Predictions, Monitoring, and Experiment Tracking *(45 min)*

**A. Production Monitoring**
- Introduce Cloud Logging and Cloud Monitoring
- Operational metrics covered: CPU utilization, memory utilization, request count, request latency, error rate, HTTP status codes, service availability
- Demonstrate: Cloud Run logs, Cloud Monitoring metrics, an operational dashboard, monitoring alerts
- Key distinction taught: **infrastructure monitoring vs. ML monitoring**

**B. ML Monitoring and Drift Detection**
- Introduce: data quality monitoring, feature distribution, prediction distribution, data drift, concept drift, model performance, business KPIs
- Key principle: production data must always be compared against the **training baseline**
- Demonstrate using **prepared healthy and drifted datasets**: feature distribution comparison, prediction distribution comparison, drift detection, model performance evaluation, investigation thresholds
- Vertex AI Model Monitoring introduced **conceptually only** — as a managed alternative to what's being demonstrated manually

**C. Experiment Tracking**
- Introduce experiment tracking and model lineage
- Discuss: experiment runs, parameters, metrics, artifacts, model versions, dataset versions, deployment history
- Demonstrate: experiment tracking with **MLflow**, and connect the tracked records to real production model decisions

---

### Section 3 — Retraining, Model Lifecycle, and Governance *(45 min)*

**A. Production Retraining**
- Explain retraining triggers: data drift, concept drift, declining performance, new production data, business process changes
- Retraining workflow presented:
  **Historical Data → Production Data → Validation → Preparation → Retraining → Evaluation → Model Comparison → Deployment Decision**
- Demonstrate: combining historical + new production data, schema validation, data quality checks, retraining the lightweight model, evaluating it, and comparing it against the current production model

**B. Model Versioning, Deployment Update, and Rollback**
- Discuss: production model vs. candidate model, model versions, **champion–challenger** comparison, deployment approval
- Demonstrate: creating the updated model artifact, building an updated container image, deploying a new **Cloud Run revision**, validating predictions, reviewing revision history, and **rolling back** to the previous revision
- Concepts introduced (briefly): blue-green, canary, and rolling deployments — noting which can be done directly here vs. which need extra orchestration tooling

**C. Governance and IAM**
- Introduce governance across the ML lifecycle: data governance, model governance, operational governance, security governance, business governance
- Introduce **Google Cloud IAM**: users, groups, service accounts, roles, permissions, least privilege, separation of responsibilities
- Demonstrate: reviewing IAM permissions, service accounts, deployment history, model versions, audit logs, and approval checkpoints

---

### Section 4 — Cost, ROI, Automation, and Enterprise Architecture *(40 min)*

**A. Cost and Resource Management**
- Review resources used across the lifecycle: Cloud Run, Cloud Storage, Artifact Registry, Cloud Monitoring, Cloud Logging
- Use **Cloud Billing** to discuss: current usage, compute costs, storage costs, monitoring/logging costs, retraining costs, operational expenditure
- Demonstrate: practical cost estimation and optimization — resource sizing, autoscaling, storage management, cleaning up unused resources

**B. Business Impact and ROI**
- Key point: **model performance alone does not prove business value**
- Discuss: revenue improvement, cost reduction, productivity improvement, risk reduction, operational efficiency, business KPIs
- Demonstrate: operational cost estimation, expected business benefit estimation, **break-even analysis**, **ROI calculation** — all tied back to the specific business problem chosen

**C. Production Automation**
- Demonstrate one complete scheduled workflow:
  **Cloud Scheduler → Cloud Run → Prediction Generation → Cloud Storage**
- Demonstrate: creating the Cloud Scheduler job, scheduling execution, verifying the Cloud Run run, reviewing logs, validating outputs in Cloud Storage
- Introduced conceptually only: Cloud Workflows, Pub/Sub, Eventarc, CI/CD pipelines, Infrastructure as Code, Vertex AI Pipelines

**D. End-to-End MLOps Architecture**
- Present the full lifecycle as one diagram:
  **Business Problem → Historical Data → Model → Cloud Storage → FastAPI → Docker → Artifact Registry → Cloud Run → Predictions → Monitoring → Drift Detection → Experiment Tracking → Retraining → Model Versioning → Governance → Automation → Business Evaluation**
- Explain how this architecture delivers ML operations that are reliable, maintainable, governed, and cost-aware

---

### Section 5 — Conclusion and Q&A *(10 min)*

- Recap the entire lifecycle covered: cloud/billing setup, data & model prep, deployment, production inference, prediction storage, monitoring, drift detection, experiment tracking, retraining, versioning, deployment updates, rollback, governance/IAM, cost management, ROI, automation
- Core message: **enterprise MLOps is a continuous lifecycle** — technical performance, operational reliability, governance, cost efficiency, automation, and business value all have to be considered *together*, not in isolation
- Open Q&A

---

## Notes to Instructor (Facilitation Guidance)

- Pick **one** business problem and use it consistently for the entire session
- Keep the balance at ~20% concept / ~80% hands-on
- The business problem must fit a **lightweight** supervised learning workflow
- Focus on demonstrating the *full lifecycle*, not deep-diving into any single GCP service
- Complete account/billing/budget setup **before** provisioning any cloud resources
- Use **pre-prepared** historical and prediction datasets so the 3-hour window is respected
- Actually **perform** the retraining live — don't just present the retraining architecture as slides
- Use real monitoring findings as the justification for retraining decisions
- Demonstrate rollback using **actual Cloud Run revisions**
- Use IAM/governance examples to show separation of responsibilities and least privilege in action
- Use **live Cloud Billing data** during the cost section — don't treat costing as purely theoretical
- Demonstrate **one** complete automation workflow (Scheduler + Cloud Run); everything else (Pub/Sub, Eventarc, CI/CD, IaC, Vertex Pipelines) stays conceptual only
- Keep the whole implementation cost-efficient, and clean up all temporary resources after the session

---

## Mandatory Technical Constraints

The instructor may choose any suitable lightweight supervised learning business problem, but must stay within these limits:

### 1. Cloud Configuration
- Region: **asia-south1 (Mumbai)**
- Python: **3.11**
- One individual GCP project **per learner**
- **CPU-only** resources

**Must be used hands-on:** Cloud Storage, Artifact Registry, Cloud Run, Cloud Logging, Cloud Monitoring, Cloud Billing, IAM, Cloud Scheduler

**Conceptual only (not hands-on):** Vertex AI, Vertex AI Model Monitoring, Vertex AI Experiments, Vertex AI Pipelines, Cloud Workflows, Pub/Sub, Eventarc, Cloud Build/CI-CD, Infrastructure as Code

### 2. Dataset Constraints

**Historical training dataset:**
- Structured tabular data, CSV (UTF-8)
- Max **25 MB**
- Max **5,000** records
- Max **15** input features
- One target column

**Prediction dataset:**
- CSV format
- **1,000** records
- Same schema as historical data, **target column excluded**
- Max **5 MB**

### 3. Model Constraints
- Must be a **lightweight** supervised model
- **Supported:** Logistic Regression, Decision Tree only
- Must be **pre-trained before the session**
- Format: `.joblib` or `.pkl`
- Max model size: **5 MB**
- **CPU inference only**
- **Not allowed:** deep learning, GPU workloads, distributed training, Random Forest, XGBoost, LLMs

### 4. Compute and Deployment Constraints
- Cloud Run: **1 vCPU**, max **2 GB RAM**
- HTTPS endpoint, automatic scaling
- **One** inference service per learner

### 5. Storage Constraints

Cloud Storage bucket structure:
```
training-data/
prediction-data/
models/
predictions/
logs/
artifacts/
```

Prediction outputs must include: prediction results, timestamp, model version, input dataset identifier

### 6. Retraining Constraints
- Max **25 MB** retraining dataset, max **5,000** records, max **15** features
- Logistic Regression or Decision Tree only
- **CPU-only** training
- Target training time: **5–10 minutes**
- Retrained model **must be compared** against the current production model before any deployment decision

### 7. Cost Constraints
- Learners configure: billing account, budget, billing alerts
- Session must demonstrate cost estimation for: compute, storage, monitoring/logging, retraining, prediction workloads
- All temporary resources must be **removed after the session**

### 8. Automation Constraint
- Exactly **one** complete scheduled workflow must be implemented:
  **Cloud Scheduler → Cloud Run → Prediction Generation → Cloud Storage**

---

## Lab Resource Guidelines

- Each learner works in their **own individual GCP environment**

**Provide before the live session, where possible:**
- Prepared historical dataset
- Prepared prediction dataset
- Pre-trained Logistic Regression or Decision Tree model
- FastAPI application files
- Dockerfile
- Requirements file
- MLflow setup / prepared MLflow environment
- Drift-detection scripts or notebooks
- Prediction output template

**Resource limits to enforce:**
- Historical dataset ≤ 25 MB; max 5,000 records, 15 features
- Prediction dataset: 1,000 records, ≤ 5 MB
- Model artifact ≤ 5 MB
- CPU-only compute throughout
- Cloud Run limited to 1 vCPU / 2 GB RAM

**Operational tips:**
- Use pre-prepared **drifted** and **non-drifted** datasets for the monitoring demo
- Use pre-prepared production data so retraining is actually feasible within the time window
- One Cloud Run service + one Cloud Storage bucket per learner
- Set up billing alerts **before** any resource-intensive activity
- Clean up temporary revisions, container images, logs, and unused artifacts after the session
