# Lab 1 — Amazon SageMaker Console Walkthrough

## Goal

Navigate the Amazon SageMaker console to understand where each ML lifecycle stage lives — Studio, training, models, endpoints, and model registry. **No resources are created. Tour only.**

## Services Used

- **Amazon SageMaker** — Console exploration only

---

## Step 1: Open the SageMaker Console

1. Log in to the **AWS Console**
2. In the search bar at the top, type **SageMaker** → click **Amazon SageMaker**
3. Make sure your region is **ap-southeast-1** (Singapore) — check the top-right dropdown

> 💡 You'll land on the **SageMaker main dashboard**. This is the entry point to all SageMaker features.

### What You See on the Dashboard

The SageMaker dashboard shows:
- A **left navigation pane** with all the major sections
- A **main content area** with quick links and recent activity
- A **Get Started** section with shortcuts to common tasks

---

## Step 2: Explore the Left Navigation Pane

The left sidebar is organized by ML lifecycle stage. Click through each section — **don't create anything, just look.**

### Navigation Pane Structure (2025/2026 UI)

```
📁 SageMaker Console — Left Navigation
│
├── 🏠 Home
│
├── 📊 Studio
│   └── Opens SageMaker Studio (the full IDE)
│
├── 🗂️ Data
│   ├── Data Wrangler — visual data preparation
│   └── Feature Store — reusable ML features
│
├── 🤖 Auto ML (Canvas)
│   └── No-code ML — build models without writing code
│
├── 🧪 Experiments
│   └── Track and compare ML experiment runs
│
├── ⚙️ Jobs
│   ├── Training jobs — where model training runs appear
│   ├── Processing jobs — data processing runs
│   └── Transform jobs — batch inference runs
│
├── 🔄 Pipelines
│   └── ML workflow automation (like CI/CD for ML)
│
├── 📦 Models
│   ├── Model Registry — versioning and approval workflows
│   └── Model groups — organize models into collections
│
├── 🚀 JumpStart
│   └── Pre-trained models catalog (foundation models, etc.)
│
├── 🌐 Deployments
│   ├── Endpoints — live models serving predictions
│   ├── Endpoint configurations — instance type settings
│   └── Inference Recommender — find optimal instance types
│
└── 📓 Notebook instances (legacy)
    └── Standalone Jupyter notebooks
```

---

## Step 3: SageMaker Studio (The IDE)

1. In the left nav, click **Studio**
2. If a domain exists, you'll see the Studio landing page
3. If no domain exists, you'll see a **Set up SageMaker Studio** prompt — **don't create one**, just observe the setup options

> ⚠️ **Don't click "Create domain"** — this creates resources that cost money. We're just looking.

### What Studio Contains (When Set Up)

Studio is the integrated development environment for ML. Inside Studio, the navigation pane shows:

| Studio Section | Purpose |
|---------------|---------|
| **Home** | Overview, getting started, what's new |
| **Running instances** | All currently running compute instances |
| **Data** | Data Wrangler, Feature Store, EMR clusters |
| **Auto ML** | SageMaker Canvas — no-code ML |
| **Experiments** | Track and compare experiment runs |
| **Jobs** | Training jobs, processing jobs, evaluation jobs |
| **Pipelines** | ML workflow automation |
| **Models** | Model Registry — version control for models |
| **JumpStart** | Pre-trained model catalog |
| **Deployments** | Endpoints and Inference Recommender |

> 💡 **Key insight:** Studio is a "one-stop shop" — everything you can do in the SageMaker console, you can also do from inside Studio.

---

## Step 4: JumpStart — Pre-trained Models

1. In the left nav, click **JumpStart**
2. You'll see a **catalog of pre-trained models** organized by:
   - **Task type** — Text Generation, Image Classification, Object Detection, etc.
   - **Provider** — Meta (Llama), Hugging Face, Stability AI, etc.
   - **Framework** — PyTorch, TensorFlow, etc.

### What to Observe

- Browse the model cards — each shows the model name, provider, task type, and a **Deploy** button
- Click on any model card (e.g., **Meta Llama**) to see:
  - Model description and use cases
  - Instance type recommendations
  - Deployment options (real-time endpoint, async, batch)
  - Fine-tuning options (if supported)

> ⚠️ **Don't click Deploy** — just browse. Deploying a JumpStart model creates an endpoint that costs money.

> 💡 **Key insight:** JumpStart is like an "app store for ML models." You can deploy pre-trained models with one click — no training needed. This is similar to Bedrock, but you host the model yourself on SageMaker infrastructure.

---

## Step 5: Jobs — Training Jobs

1. In the left nav, click **Jobs → Training jobs**
2. You'll see a **list of training jobs** (likely empty if this is a fresh account)

### What to Observe

The training jobs page shows:
- **Job name** — unique identifier
- **Status** — InProgress, Completed, Failed, Stopped
- **Creation time** — when the job was launched
- **Training time** — how long it ran
- **Instance type** — what compute was used (e.g., ml.m5.large, ml.p3.2xlarge)

### If You Click on a Training Job (When One Exists)

You would see:
- **Hyperparameters** — the settings used for training
- **Input data configuration** — S3 paths for training/validation data
- **Output data configuration** — S3 path for the model artifact
- **Metrics** — training loss, validation accuracy, etc.
- **Resource configuration** — instance type, count, volume size
- **Billable seconds** — how long you were charged for

> 💡 **Key insight:** SageMaker manages the entire training lifecycle — it provisions compute, pulls your data from S3, runs training, saves the model artifact back to S3, and shuts down the instance. You only pay for the time the job runs.

---

## Step 6: Models — Model Registry

1. In the left nav, click **Models**
2. You'll see two sub-sections:
   - **Model Registry** — versioned model groups
   - **Models** — individual model artifacts

### Model Registry

- This is where you organize models into **model groups** (like a Git repository for models)
- Each group can have multiple **model versions** (like Git commits)
- Each version has a **status**: PendingManualApproval, Approved, Rejected
- This enables **approval workflows** — a data scientist submits a model, a lead reviews and approves it before deployment

### Models (Individual)

- These are model artifacts registered in SageMaker
- Each model points to a `model.tar.gz` file in S3
- A model must be registered here before it can be deployed to an endpoint

> 💡 **Key insight:** The Model Registry is version control for ML models. It's how teams manage the lifecycle from "trained" to "approved" to "deployed."

---

## Step 7: Deployments — Endpoints

1. In the left nav, click **Deployments → Endpoints**
2. You'll see a **list of endpoints** (likely empty)

### What to Observe

The endpoints page shows:
- **Endpoint name** — unique identifier
- **Status** — Creating, InService, Updating, Deleting, Failed
- **Creation time** — when it was deployed
- **Endpoint configuration** — links to the config (instance type, count)

### If You Click on an Endpoint (When One Exists)

You would see:
- **Endpoint configuration** — instance type (e.g., ml.t2.medium), instance count
- **Production variants** — which model version is being served
- **Monitoring** — link to CloudWatch metrics (invocations, latency, errors)
- **Settings** — IAM role, KMS encryption, VPC configuration

### Also Check: Endpoint Configurations

1. Click **Deployments → Endpoint configurations**
2. This is where you define the **blueprint** for an endpoint:
   - Instance type and count
   - Model to serve
   - Data capture settings (for monitoring)

> 💡 **Key insight:** An endpoint is a live, always-on HTTPS API that serves predictions. It costs money while running — that's why cleanup is critical. The pattern is: **Model → Endpoint Configuration → Endpoint**.

---

## Step 8: Pipelines — ML Workflow Automation

1. In the left nav, click **Pipelines**
2. You'll see a list of ML pipelines (likely empty)

### What to Observe

- Pipelines automate the ML workflow: **data prep → training → evaluation → registration → deployment**
- Each pipeline is a **DAG (Directed Acyclic Graph)** of steps
- Think of it as **CI/CD for machine learning** — every time new data arrives, the pipeline retrains and redeploys automatically

> 💡 **Key insight:** In production ML, you don't train models manually. Pipelines automate the entire lifecycle so models stay fresh as new data comes in.

---

## Step 9: Notebook Instances (Legacy)

1. In the left nav, click **Notebook instances**
2. This is the **legacy** way to run Jupyter notebooks on SageMaker

### What to Observe

- Each notebook instance is a **managed EC2 instance** running JupyterLab
- You choose an instance type (e.g., ml.t3.medium)
- It comes pre-installed with Python, Boto3, SageMaker SDK, and common ML libraries
- **Status**: InService (running, costs money) or Stopped (not running, no compute cost)

> 💡 **Key insight:** Notebook instances are being replaced by Studio Spaces, but they're simpler for quick experiments. Always **stop** them when done — they bill per hour while running.

---

## Summary: The ML Lifecycle in SageMaker

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Prepare    │    │    Train    │    │  Register   │    │   Deploy    │
│   (Data)     │ →  │   (Jobs)    │ →  │  (Models)   │ →  │ (Endpoints) │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
  Data Wrangler      Training Jobs     Model Registry      Endpoints
  Feature Store      Experiments       Approval Flow       Inference
  S3                 Hyperparameters   Versioning          Monitoring
```

### Where Each Console Section Fits

| ML Stage | SageMaker Section | What Happens |
|----------|------------------|-------------|
| **Prepare** | Data (Wrangler, Feature Store) | Clean, transform, and store features |
| **Train** | Jobs → Training jobs | Run training on managed compute |
| **Experiment** | Experiments | Track and compare runs |
| **Register** | Models → Model Registry | Version and approve models |
| **Deploy** | Deployments → Endpoints | Serve predictions via HTTPS API |
| **Automate** | Pipelines | CI/CD for the entire lifecycle |
| **Browse** | JumpStart | Use pre-trained models off the shelf |

---

## ⚠️ Clean Up

**Nothing to clean up!** This was a console tour only — no resources were created.

If you accidentally created a Studio domain or notebook instance, delete them now:
1. **Studio domain:** SageMaker → Studio → Domain settings → Delete domain
2. **Notebook instance:** Notebook instances → Select → Actions → Stop, then Delete

---

## 🧠 Next: Amazon Bedrock — GenAI Workshop

Ready to get hands-on with Generative AI? Continue with the official **Building with Amazon Bedrock** workshop:

👉 **[Click here to access the Amazon Bedrock Workshop](https://catalog.workshops.aws/workshops/cdbce152-b193-43df-8099-908ee2d1a6e4/en-US/)**

Covers: Foundation model APIs, chatbots, RAG, image generation, prompt engineering, guardrails, and agentic AI.
