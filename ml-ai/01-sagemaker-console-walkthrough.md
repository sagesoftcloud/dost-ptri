# Lab 1 — Amazon SageMaker AI Console Walkthrough

## Goal

Navigate **SageMaker Studio** to understand where each ML lifecycle stage lives — Models, JumpStart, Jobs, Deployments, Pipelines, and more. Includes a guided tour using a shared domain with individual user profiles.

> 💡 As of December 2024, Amazon SageMaker was renamed to **Amazon SageMaker AI**.

## Services Used

- **Amazon SageMaker AI** — Studio exploration

## Setup Model

| Role | What They Do |
|------|-------------|
| **Admin (1 person)** | Creates the SageMaker Studio domain and adds user profiles for each participant |
| **Participants (all 5)** | Log in to the console and open Studio using their assigned user profile |

> ⚠️ **Only the admin creates the domain.** Participants do NOT create their own domains.

---

## Part A: Admin Setup — Create Domain & User Profiles (10 min)

> 🔒 **Admin only.** The person with AdministratorAccess does this step. Participants wait.

### Step 1: Create the SageMaker Studio Domain

1. Log in to the **AWS Console** as admin
2. Search for **SageMaker** → click **Amazon SageMaker AI**
3. Set region to **ap-southeast-1** (Singapore)
4. Click **Studio** in the left nav
5. Click **Set up SageMaker Studio**
6. Choose **Set up for single user (Quick setup)**
7. Wait for the domain to be created (~2–3 minutes)

> 💡 The domain itself is free. Costs only start when someone launches a JupyterLab Space or app.

### Step 2: Create User Profiles for Each Participant (Skip this part if there's already account)

1. In the SageMaker AI console, click **Domains** (under Admin configurations)
2. Click on your domain → **Add user profile**
3. Create a profile for each participant:

| User Profile Name | For |
|-------------------|-----|
| `participant-1` | Participant 1 |
| `participant-2` | Participant 2 |
| `participant-3` | Participant 3 |
| `participant-4` | Participant 4 |
| `participant-5` | Participant 5 |

4. For each: use the **default execution role** → **Submit**

---

## Part B: All Participants — Open SageMaker Studio (5 min)

### Step 1: Open Studio

1. Log in to the **AWS Console**
2. Search for **SageMaker** → click **Amazon SageMaker AI**
3. Set region to **ap-southeast-1** (Singapore)
4. Click **Studio** → select **your assigned user profile** (e.g., `participant-1`)
5. Click **Open Studio**

### Step 2: You Should See This

You'll land on the **SageMaker Studio Home** page:

- **Top bar:** `SageMaker Studio` with tabs for **JupyterLab** | **RStudio** | **Canvas**
- **Top icons:** Code Editor, MLflow, More Apps (+)
- **Content area:** Home page with **Overview** | **Getting started** | **Sample notebooks** tabs
- Two large cards: **JupyterLab** and **Code Editor** (for creating Spaces)
- Below: **Start your model customization workflow** — Browse, Customize, Deploy

> ⚠️ **Don't click "View JupyterLab spaces" or "View Code Editor spaces"** — creating a Space launches compute that costs money. We're just touring.

---

## Part C: Studio Navigation Tour (15 min)

The **left navigation pane** is your map to the entire ML lifecycle. Click through each section — **don't create anything, just look.**

### Studio Left Navigation (Actual UI)

```
📁 SageMaker Studio — Left Navigation
│
├── 🤝 Partner AI Apps
│
├── 🏠 Home
│       Overview, Getting started, Sample notebooks
│       JupyterLab & Code Editor cards
│       Model customization workflow (Browse → Customize → Deploy)
│
├── 🧠 Models [New]
│       Model Registry — versioning, approval workflows
│       Model groups and model cards
│
├── 🚀 JumpStart
│       Pre-trained model catalog (Llama, Nova, Qwen, GPT-OSS, etc.)
│       One-click fine-tuning and deployment
│
├── 📦 Assets ▾
│       Data Wrangler, Feature Store, and other data assets
│
├── 💻 Compute ▾
│       Running instances, Spaces, and compute resources
│
├── 🧪 Experiments
│       Track and compare ML experiment runs
│
├── ⚙️ Jobs ▾
│       Training jobs, Processing jobs, Transform jobs
│
├── 🔄 Pipelines
│       ML workflow automation (CI/CD for ML)
│
├── 🌐 Deployments ▾
│       Endpoints, Endpoint configurations, Inference Recommender
│
├── ⚙️ More ▾
│       Additional settings and tools
│
└── 📁 Collapse Menu
```

---

### Section 1: Home

This is the landing page you see when Studio opens.

**What to observe:**
- **Overview tab** — Quick-start cards for JupyterLab and Code Editor Spaces
- **Getting started tab** — Guided tour and documentation links
- **Sample notebooks tab** — Pre-built example notebooks you can explore
- **Model customization workflow** — Three-step flow: **Browse** (find models) → **Customize** (fine-tune) → **Deploy** (serve predictions)

> 💡 The Home page is designed to get you started fast — either launch a notebook or start a model workflow.

---

### Section 2: Models [New]

1. Click **Models** in the left nav

**What to observe:**
- This is the **Model Registry** — version control for ML models
- Organize models into **model groups** (like a Git repo for models)
- Each group has multiple **versions** (like Git commits)
- Each version has a **status**: PendingManualApproval → Approved → Rejected
- Enables **approval workflows** — data scientist submits, lead reviews before deployment

> 💡 The "New" badge means this section was recently redesigned. It now includes model cards with metadata, lineage, and deployment history.

---

### Section 3: JumpStart

1. Click **JumpStart** in the left nav

**What to observe:**
- A **catalog of pre-trained models** from Amazon, Meta, Hugging Face, and more
- The banner at the top highlights: fine-tuning of **Amazon Nova**, **Llama**, **Qwen**, and **GPT-OSS** models using Reinforcement Learning, Supervised Fine-tuning, and Direct Preference Optimization
- Models organized by **task type** (Text Generation, Image Classification, etc.)
- Click any model card to see: description, instance recommendations, deployment options

> ⚠️ **Don't click Deploy or Fine-tune** — this creates resources that cost money.

> 💡 JumpStart is like an "app store for ML models." Similar to Bedrock, but here **you host the model yourself** on SageMaker infrastructure.

---

### Section 4: Assets

1. Click **Assets** (expandable ▾) in the left nav

**What to observe:**
- **Data Wrangler** — visual data preparation tool
- **Feature Store** — store and reuse ML features across teams
- Other data assets and artifacts

---

### Section 5: Compute

1. Click **Compute** (expandable ▾) in the left nav

**What to observe:**
- **Spaces** — JupyterLab and Code Editor instances (this is where compute runs)
- Running instances and their status
- This is where costs come from — running Spaces bill per hour

> ⚠️ If you see any running Spaces, **do not stop or delete them** unless they're yours.

---

### Section 6: Experiments

1. Click **Experiments** in the left nav

**What to observe:**
- Track and compare ML experiment runs
- Each experiment logs metrics, parameters, and artifacts
- Useful for comparing different model configurations

---

### Section 7: Jobs

1. Click **Jobs** (expandable ▾) in the left nav

**Sub-sections:**
- **Training jobs** — where model training runs appear
- **Processing jobs** — data processing runs
- **Transform jobs** — batch inference runs

| Field | What It Means |
|-------|--------------|
| **Job name** | Unique identifier |
| **Status** | InProgress, Completed, Failed, Stopped |
| **Training time** | How long it ran |
| **Instance type** | Compute used (e.g., ml.m5.large) |
| **Billable seconds** | What you're charged for |

> 💡 SageMaker AI manages the entire training lifecycle — provisions compute, pulls data from S3, trains, saves the model back to S3, and shuts down. You only pay while the job runs.

---

### Section 8: Pipelines

1. Click **Pipelines** in the left nav

**What to observe:**
- Pipelines automate: **data prep → training → evaluation → registration → deployment**
- Each pipeline is a **DAG** (Directed Acyclic Graph) of steps
- Think of it as **CI/CD for machine learning**

> 💡 In production, you don't train models manually. Pipelines automate the lifecycle so models stay fresh as new data arrives.

---

### Section 9: Deployments

1. Click **Deployments** (expandable ▾) in the left nav

**Sub-sections:**
- **Endpoints** — live models serving predictions
- **Endpoint configurations** — instance type and scaling settings
- **Inference Recommender** — find optimal instance types for your model

| Field | What It Means |
|-------|--------------|
| **Endpoint name** | Unique identifier |
| **Status** | Creating, InService, Updating, Deleting, Failed |
| **Endpoint configuration** | Instance type, count |
| **Production variants** | Which model version is being served |

> 💡 An endpoint is a live HTTPS API serving predictions. It costs money while running. Pattern: **Model → Endpoint Config → Endpoint**.

---

## Summary: The ML Lifecycle in SageMaker Studio

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Prepare    │    │    Train    │    │  Register   │    │   Deploy    │
│  (Assets)    │ →  │   (Jobs)    │ →  │  (Models)   │ →  │(Deployments)│
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
  Data Wrangler      Training Jobs     Model Registry      Endpoints
  Feature Store      Experiments       Approval Flow       Inference
  S3                 Hyperparameters   Versioning          Monitoring
```

| ML Stage | Studio Section | What Happens |
|----------|---------------|-------------|
| **Prepare** | Assets (Wrangler, Feature Store) | Clean, transform, store features |
| **Train** | Jobs → Training jobs | Run training on managed compute |
| **Experiment** | Experiments | Track and compare runs |
| **Register** | Models | Version and approve models |
| **Deploy** | Deployments → Endpoints | Serve predictions via HTTPS API |
| **Automate** | Pipelines | CI/CD for the entire lifecycle |
| **Browse** | JumpStart | Pre-trained models off the shelf |
| **Code** | Home → JupyterLab / Code Editor | Write and run ML code |

---

## ⚠️ Clean Up (Admin Only)

After the walkthrough, the **admin** should clean up:

### 1. Delete User Profiles

1. Go to **SageMaker AI console** (not Studio) → **Domains**
2. Click your domain → select each user profile → **Delete**
3. Repeat for all 5 participant profiles

### 2. Delete the Domain

1. After all user profiles are deleted, click **Delete domain**
2. Confirm deletion

> ⚠️ If anyone accidentally launched a Space inside Studio, stop/delete it first before deleting the user profile.

---

## 🧠 Next: Amazon Bedrock — GenAI Workshop

Continue with the official **Building with Amazon Bedrock** hands-on workshop:

👉 **[Click here to access the Amazon Bedrock Workshop](https://catalog.workshops.aws/workshops/cdbce152-b193-43df-8099-908ee2d1a6e4/en-US/)**

Covers: Foundation model APIs, chatbots, RAG, image generation, prompt engineering, guardrails, and agentic AI.
