# Lab 1 — Amazon SageMaker AI Console Walkthrough

## Goal

Navigate the **Amazon SageMaker AI** dashboard to understand where each ML lifecycle stage lives — Studio, JumpStart, Training Jobs, Models, Endpoints, and Pipelines. Includes a guided Studio tour using a shared domain with individual user profiles.

> 💡 As of December 2024, Amazon SageMaker was renamed to **Amazon SageMaker AI**. The console URL still uses `sagemaker`.

## Services Used

- **Amazon SageMaker AI** — Console + Studio exploration

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
   - This creates a domain with a default execution role
7. Wait for the domain to be created (~2–3 minutes)

> 💡 The domain itself is free. Costs only start when someone launches a notebook or app inside Studio.

### Step 2: Create User Profiles for Each Participant

1. In the SageMaker AI console, click **Studio** → **Domain settings**
2. Under **User profiles**, click **Add user profile**
3. Create a profile for each participant:

| User Profile Name | For |
|-------------------|-----|
| `participant-1` | Participant 1 |
| `participant-2` | Participant 2 |
| `participant-3` | Participant 3 |
| `participant-4` | Participant 4 |
| `participant-5` | Participant 5 |

4. For each profile:
   - **Execution role:** Use the default role created during domain setup
   - Leave all other settings as default
   - Click **Submit**

> 💡 Each user profile is isolated — participants can browse Studio independently without affecting each other.

### Step 3: Share Access

Each participant logs in to the **same AWS account** via the AWS Console. When they navigate to SageMaker AI → Studio, they select their assigned user profile to enter Studio.

---

## Part B: All Participants — Open the SageMaker AI Dashboard (5 min)

### Step 1: Open the SageMaker AI Console

1. Log in to the **AWS Console**
2. In the search bar, type **SageMaker** → click **Amazon SageMaker AI**
3. Set your region to **ap-southeast-1** (Singapore) — top-right dropdown

> 💡 You'll land on the **SageMaker AI dashboard**. The left navigation pane organizes everything by ML lifecycle stage.

### Step 2: Explore the Left Navigation Pane

**Click through each section — don't create anything, just look.**

```
📁 Amazon SageMaker AI — Left Navigation
│
├── 🏠 Home
│       Dashboard overview, quick links, recent activity
│
├── 📊 Studio
│       Integrated IDE for ML (notebooks, experiments, pipelines)
│
├── 🚀 JumpStart
│       Pre-trained model catalog — deploy or fine-tune with one click
│
├── ⚙️ Jobs
│   ├── Training jobs — model training runs
│   ├── Processing jobs — data processing runs
│   └── Transform jobs — batch inference runs
│
├── 📦 Models
│   ├── Model Registry — versioning and approval workflows
│   └── Model groups — organize models into collections
│
├── 🌐 Deployments
│   ├── Endpoints — live models serving predictions
│   ├── Endpoint configurations — instance type settings
│   └── Inference Recommender — find optimal instance types
│
├── 🔄 Pipelines
│       ML workflow automation (CI/CD for ML)
│
├── 🗂️ Data
│   ├── Data Wrangler — visual data preparation
│   └── Feature Store — reusable ML features
│
├── 🤖 Auto ML (Canvas)
│       No-code ML — build models without writing code
│
├── 🧪 Experiments
│       Track and compare ML experiment runs
│
└── 📓 Notebook instances (legacy)
        Standalone Jupyter notebooks
```

---

## Part C: Studio Tour — All Participants (10 min)

### Step 1: Open Studio with Your User Profile

1. Click **Studio** in the left nav
2. You'll see the domain with user profiles listed
3. Select **your assigned user profile** (e.g., `participant-1`)
4. Click **Open Studio**
5. Studio opens in a new tab — this is the integrated IDE

> ⚠️ **Just browse the interface.** Do NOT launch any Spaces, notebooks, or apps — that spins up compute and costs money.

### Step 2: Explore the Studio Navigation

Inside Studio, the left nav shows:

| Studio Section | Purpose |
|---------------|---------|
| **Home** | Overview, getting started, what's new |
| **Running instances** | All currently running compute (should be empty) |
| **Data** | Data Wrangler, Feature Store, EMR clusters |
| **Auto ML** | SageMaker Canvas — no-code ML |
| **Experiments** | Track and compare experiment runs |
| **Jobs** | Training, processing, evaluation jobs |
| **Pipelines** | ML workflow automation |
| **Models** | Model Registry — version control for models |
| **JumpStart** | Pre-trained model catalog |
| **Deployments** | Endpoints and Inference Recommender |

> 💡 Studio is a "one-stop shop" — everything in the SageMaker AI console is also accessible from inside Studio.

---

## Part D: Console Walkthrough — Key Sections (15 min)

Go back to the **SageMaker AI console** (not Studio) and explore each section below.

### JumpStart — Pre-trained Models

1. Click **JumpStart** in the left nav
2. Browse the **catalog of pre-trained models**
3. Models organized by **task type** (Text Generation, Image Classification, etc.)
4. Click any model card to see: description, instance recommendations, deployment options

> ⚠️ **Don't click Deploy** — deploying creates an endpoint that costs money.

> 💡 JumpStart is like an "app store for ML models." Similar to Bedrock, but here **you host the model yourself** on SageMaker infrastructure.

### Jobs — Training Jobs

1. Click **Jobs → Training jobs**
2. List of training jobs (likely empty on a fresh account)

| Field | What It Means |
|-------|--------------|
| **Job name** | Unique identifier |
| **Status** | InProgress, Completed, Failed, Stopped |
| **Training time** | How long it ran |
| **Instance type** | Compute used (e.g., ml.m5.large) |
| **Billable seconds** | What you're charged for |

> 💡 SageMaker AI manages the entire training lifecycle — provisions compute, pulls data from S3, trains, saves the model back to S3, and shuts down. You only pay while the job runs.

### Models — Model Registry

1. Click **Models** in the left nav

**Model Registry**
- Organize models into **model groups** (like a Git repo for models)
- Each group has multiple **versions** (like Git commits)
- Each version has a **status**: PendingManualApproval → Approved → Rejected
- Enables **approval workflows** — data scientist submits, lead reviews before deployment

> 💡 Model Registry = version control for ML models. Lifecycle: "trained" → "approved" → "deployed."

### Deployments — Endpoints

1. Click **Deployments → Endpoints**

| Field | What It Means |
|-------|--------------|
| **Endpoint name** | Unique identifier |
| **Status** | Creating, InService, Updating, Deleting, Failed |
| **Endpoint configuration** | Instance type, count |
| **Production variants** | Which model version is being served |

2. Also check **Deployments → Endpoint configurations** — the blueprint for an endpoint

> 💡 An endpoint is a live HTTPS API serving predictions. It costs money while running. Pattern: **Model → Endpoint Config → Endpoint**.

### Pipelines — ML Workflow Automation

1. Click **Pipelines** in the left nav
2. Pipelines automate: **data prep → training → evaluation → registration → deployment**
3. Each pipeline is a **DAG** (Directed Acyclic Graph) of steps

> 💡 Think of it as **CI/CD for machine learning** — models stay fresh as new data arrives.

---

## Summary: The ML Lifecycle in SageMaker AI

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Prepare    │    │    Train    │    │  Register   │    │   Deploy    │
│   (Data)     │ →  │   (Jobs)    │ →  │  (Models)   │ →  │ (Endpoints) │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
  Data Wrangler      Training Jobs     Model Registry      Endpoints
  Feature Store      Experiments       Approval Flow       Inference
  S3                 Hyperparameters   Versioning          Monitoring
```

| ML Stage | Console Section | What Happens |
|----------|----------------|-------------|
| **Prepare** | Data (Wrangler, Feature Store) | Clean, transform, store features |
| **Train** | Jobs → Training jobs | Run training on managed compute |
| **Experiment** | Experiments | Track and compare runs |
| **Register** | Models → Model Registry | Version and approve models |
| **Deploy** | Deployments → Endpoints | Serve predictions via HTTPS API |
| **Automate** | Pipelines | CI/CD for the entire lifecycle |
| **Browse** | JumpStart | Pre-trained models off the shelf |

---

## ⚠️ Clean Up (Admin Only)

After the walkthrough, the **admin** should clean up:

### 1. Delete User Profiles

1. SageMaker AI → Studio → **Domain settings**
2. Under **User profiles**, select each profile → **Delete**
3. Repeat for all 5 participant profiles

### 2. Delete the Domain

1. After all user profiles are deleted, click **Delete domain**
2. Confirm deletion

> ⚠️ If anyone accidentally launched a Space or app inside Studio, stop/delete it first before deleting the user profile.

---

## 🧠 Next: Amazon Bedrock — GenAI Workshop

Continue with the official **Building with Amazon Bedrock** hands-on workshop:

👉 **[Click here to access the Amazon Bedrock Workshop](https://catalog.workshops.aws/workshops/cdbce152-b193-43df-8099-908ee2d1a6e4/en-US/)**

Covers: Foundation model APIs, chatbots, RAG, image generation, prompt engineering, guardrails, and agentic AI.
