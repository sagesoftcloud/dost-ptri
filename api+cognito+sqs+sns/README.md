# Day 4 — API Layer, Authentication & Messaging Labs

## DOST PTRI - AWS Fundamentals Training Program

> ⚠️ **For training purposes only.**

Four hands-on labs covering API Gateway, Cognito authentication, messaging services, and a React integration app — all via the AWS Console (ClickOps) + local development.

| # | Lab | Services Used | Time |
|---|-----|--------------|:----:|
| 1 | [REST API with Cognito Auth](./01-api-gateway-cognito-auth.md) | API Gateway, Lambda, Cognito | 30 min |
| 2 | [Order Queue with SQS](./02-sqs-order-queue.md) | SQS, Lambda (×2) | 20 min |
| 3 | [SNS Notification Fan-Out](./03-sns-fanout-notifications.md) | SNS, SQS (×2), Lambda | 25 min |
| 4 | [React Integration App](./04-react-integration-app.md) | All of the above + React | 30 min |

### Prerequisites

- AWS account with AdministratorAccess
- Region: **ap-southeast-1** (Singapore) or **us-east-1** (N. Virginia)
- Completed Day 3 Lambda labs (basic Lambda knowledge)
- **Lab 4 only:** Node.js 18+ installed locally

### Naming Convention

All resources: `dost-ptri-day4-[yourname]-<resource>`

### ⚠️ Cost

All services used have generous free tiers. These labs will cost **$0** if within free tier limits.

---

**Start here → [01 - REST API with Cognito Auth](./01-api-gateway-cognito-auth.md)**
