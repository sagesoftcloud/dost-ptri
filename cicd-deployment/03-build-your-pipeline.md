# Lab 03 — Build Your Own CI/CD Pipeline from Scratch

## Objective

Create a fully working AWS CodePipeline that automatically builds and tests your code every time you push to GitHub.

**What you'll build:**

```
GitHub (your repo) → CodePipeline → CodeBuild (build & test) → ✅ Success
```

**Time:** ~45 min

---

## Prerequisites

- Completed Lab 01 (you have your own GitHub repo with the sample app)
- AWS Console access with AdministratorAccess
- Region: **ap-southeast-1** (Singapore)

---

## Step 1: Create a GitHub Connection (10 min)

AWS needs permission to access your GitHub repo.

1. Open **AWS Console** → search for **CodePipeline**
2. In the left sidebar, click **Settings** → **Connections**
3. Click **Create connection**
4. Select **GitHub** as the provider
5. Connection name: `dost-ptri-github`
6. Click **Connect to GitHub**
7. A popup appears — click **Authorize AWS Connector for GitHub**
8. Select your GitHub account
9. Choose **Only select repositories** → select `dost-ptri-day6-cicd`
10. Click **Install & Authorize**
11. Back in AWS, click **Connect**
12. Status should show **Available** ✅

> 💡 This connection is reusable — you only create it once.

---

## Step 2: Create a CodeBuild Project (10 min)

This is the "builder" that will compile and test your code.

1. Open **AWS Console** → search for **CodeBuild**
2. Click **Create build project**

### Project configuration

| Field | Value |
|-------|-------|
| Project name | `dost-ptri-day6-build` |
| Description | `Build and test the Day 6 sample app` |

### Source

| Field | Value |
|-------|-------|
| Source provider | **GitHub** |
| Connection | Select `dost-ptri-github` (created in Step 1) |
| Repository | Select `YOUR-USERNAME/dost-ptri-day6-cicd` |
| Branch | `main` |

### Environment

| Field | Value |
|-------|-------|
| Environment image | **Managed image** |
| Operating system | **Amazon Linux** |
| Runtime | **Standard** |
| Image | `aws/codebuild/amazonlinux2-x86_64-standard:5.0` |
| Service role | **New service role** (auto-created) |

### Buildspec

| Field | Value |
|-------|-------|
| Build specifications | **Use a buildspec file** |
| Buildspec name | `buildspec.yml` (default) |

### Artifacts

| Field | Value |
|-------|-------|
| Type | **No artifacts** (for now — pipeline will handle this) |

3. Click **Create build project**

### Test it manually

1. Click **Start build**
2. Watch the **Build logs** in real-time
3. You should see:
   - ✅ Installing dependencies
   - ✅ Running tests (2 passed)
   - ✅ Packaging application
4. Build status: **Succeeded** ✅

> 💡 If the build fails, check the logs — usually a typo in `buildspec.yml` or missing file.

---

## Step 3: Create the CodePipeline (15 min)

Now connect everything into an automated pipeline.

1. Open **AWS Console** → search for **CodePipeline**
2. Click **Create pipeline**

### Pipeline settings

| Field | Value |
|-------|-------|
| Pipeline name | `dost-ptri-day6-pipeline` |
| Pipeline type | **V2** |
| Service role | **New service role** |

3. Click **Next**

### Source stage

| Field | Value |
|-------|-------|
| Source provider | **GitHub (via connection)** |
| Connection | Select `dost-ptri-github` |
| Repository name | `YOUR-USERNAME/dost-ptri-day6-cicd` |
| Branch name | `main` |
| Output artifact format | **CodePipeline default** |
| Trigger | **Push to branch** |

4. Click **Next**

### Build stage

| Field | Value |
|-------|-------|
| Build provider | **AWS CodeBuild** |
| Region | **Asia Pacific (Singapore)** |
| Project name | Select `dost-ptri-day6-build` |
| Build type | **Single build** |

5. Click **Next**

### Deploy stage

6. Click **Skip deploy stage** → Confirm skip

> 💡 We skip deploy for now — in production you'd add CodeDeploy or CloudFormation here.

7. Click **Next** → Review → **Create pipeline**

---

## Step 4: Watch the Pipeline Run (5 min)

After creation, the pipeline **automatically triggers** its first run.

1. Watch the **Source** stage — it pulls your code from GitHub
2. Watch the **Build** stage — it runs CodeBuild
3. Both stages should show ✅ **Succeeded**

### What just happened:

```
Your GitHub repo → CodePipeline detected it → CodeBuild ran buildspec.yml → Tests passed ✅
```

---

## Step 5: Trigger the Pipeline with a Push (5 min)

Now test the automation — push a change and watch the pipeline run automatically.

### Make a code change locally

```bash
cd ~/dost-ptri-day6-cicd
```

Edit `app.py` — add a new endpoint:

```python
@app.route("/version")
def version():
    return jsonify({"version": "1.1.0", "day": "Day 6"})
```

### Push to GitHub

```bash
git add app.py
git commit -m "feat: add version endpoint"
git push
```

### Watch the pipeline

1. Go back to **CodePipeline** in the console
2. Within ~30 seconds, the pipeline starts a new execution
3. Source ✅ → Build ✅

> 🎉 **Congratulations!** You have a working CI/CD pipeline. Every push to `main` now automatically builds and tests your code.

---

## Step 6: (Optional) Make the Build Fail

Let's see what happens when tests fail.

### Break a test intentionally

Edit `app.py` — change the health endpoint:

```python
@app.route("/health")
def health():
    return jsonify({"status": "broken"}), 500
```

### Push

```bash
git add app.py
git commit -m "test: break health endpoint"
git push
```

### Watch the pipeline

- Source ✅ → Build ❌ **Failed**
- Click on the Build stage → **View logs**
- You'll see the test failure: `assert resp.json["status"] == "healthy"` — FAILED

### Fix it

```python
@app.route("/health")
def health():
    return jsonify({"status": "healthy"}), 200
```

```bash
git add app.py
git commit -m "fix: restore health endpoint"
git push
```

Pipeline runs again → ✅ Succeeded

---

## Clean Up

To avoid any charges, delete the resources:

1. **CodePipeline** → Select pipeline → **Delete**
2. **CodeBuild** → Select project → **Delete**
3. **S3** → Delete the `codepipeline-ap-southeast-1-*` artifact bucket (empty it first)
4. **IAM** → Delete the auto-created roles (optional, they cost nothing)
5. **CodePipeline Settings** → Connections → Delete `dost-ptri-github`

---

## ✅ Lab Complete!

You built a real CI/CD pipeline from scratch:

| What | How |
|------|-----|
| Source control | GitHub repo with your code |
| Build & test | CodeBuild reads `buildspec.yml` |
| Automation | CodePipeline triggers on every push |
| Feedback | Build fails → you know immediately |

**This is the foundation of every production deployment pipeline on AWS.**
