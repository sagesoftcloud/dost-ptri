# CI/CD & Infrastructure as Code Lab

## DOST PTRI - AWS Fundamentals Training Program

Hands-on labs covering Git/GitHub, AWS CodePipeline, CodeBuild, and CloudFormation — from basic to advanced.

| # | Lab | Difficulty | Time |
|---|-----|:----------:|:----:|
| 1 | [Clone & Push to Your GitHub](./01-github-push-lab.md) | ⭐ | 30 min |
| 2 | [CloudFormation Console Walkthrough](./02-cloudformation-lab.md) | ⭐ | 30 min |
| 3 | [Build Your Own CI/CD Pipeline](./03-build-your-pipeline.md) | ⭐⭐ | 45 min |
| 4 | [CloudFormation Basic: S3 Bucket](./04-cloudformation-basic.md) | ⭐ | 20 min |
| 5 | [CloudFormation Intermediate: VPC Stack](./05-cloudformation-intermediate.md) | ⭐⭐ | 30 min |
| 6 | [CloudFormation Advanced: Serverless API](./06-cloudformation-advanced.md) | ⭐⭐⭐ | 40 min |

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

### ⚠️ Cost

All labs use free-tier eligible services. Clean up resources after each lab to avoid charges.

---

**Start here → [01 - Clone & Push to Your GitHub](./01-github-push-lab.md)**
