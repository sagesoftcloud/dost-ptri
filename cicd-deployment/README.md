# CI/CD & Infrastructure as Code Lab

## DOST PTRI - AWS Fundamentals Training Program

A sample CI/CD project demonstrating AWS CodePipeline, CodeBuild, CodeDeploy, and CloudFormation — used as a hands-on reference for Day 6.

| # | Activity | Services/Tools | Time |
|---|----------|---------------|:----:|
| 1 | [Clone & Push to Your GitHub](./01-github-push-lab.md) | Git, GitHub | 30 min |
| 2 | [CloudFormation Console Walkthrough](./02-cloudformation-lab.md) | CloudFormation, S3 | 30 min |

### Prerequisites

- AWS account with AdministratorAccess
- Region: **ap-southeast-1** (Singapore)
- GitHub account (free) — https://github.com/signup
- Git installed on your machine

### Project Structure

```
cicd-deployment/
├── app.py                  # Flask API (the application)
├── test_app.py             # Pytest tests (run during build)
├── requirements.txt        # Python dependencies
├── buildspec.yml           # CodeBuild instructions
├── appspec.yml             # CodeDeploy instructions
├── pipeline-template.yaml  # CloudFormation — defines the CI/CD pipeline
├── lab-s3-versioning.yaml  # CloudFormation — lab exercise template
└── scripts/                # CodeDeploy lifecycle hooks
    ├── stop_server.sh
    ├── install_dependencies.sh
    ├── start_server.sh
    └── validate_service.sh
```

### How the Pipeline Works

```
GitHub Push → CodePipeline → CodeBuild → CodeDeploy → EC2
                                │              │
                          buildspec.yml    appspec.yml
                          (build & test)   (deploy steps)
```

### ⚠️ Cost

CloudFormation and S3 (within free tier) cost **$0**. The pipeline template is for reference only — do not deploy it unless instructed.

---

**Start here → [01 - Clone & Push to Your GitHub](./01-github-push-lab.md)**
