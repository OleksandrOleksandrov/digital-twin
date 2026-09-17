# Oleksandr's Digital Twin

An AI-powered digital twin of Oleksandr, built with Next.js, Python (Lambda), and AWS infrastructure.

## Overview

This project is a personal digital twin — an interactive web application that simulates conversations based on the GPT-5.4 nano model. It consists of a Next.js frontend, a Python Lambda backend, and AWS infrastructure managed by Terraform.

## Architecture

| Component | Technology |
|-----------|-----------|
| Frontend | Next.js (App Router) |
| Backend | Python 3.12 (AWS Lambda) |
| API | API Gateway (HTTP API) |
| Infrastructure | Terraform |
| Hosting | S3 + CloudFront |
| AI Model | Amazon Bedrock |
| Memory | S3 (conversation storage) |

## Environments

| Environment | URL | Description |
|-------------|-----|-------------|
| **Production** | https://www.oleksandraitwin.com | Current production deployment |
| **Staging** | https://d27i28hhowm0s1.cloudfront.net | Pre-production deployment |
| **Development** | https://d13aejtk951nms.cloudfront.net | Development deployment |

## Project Structure

```
twin/
├── frontend/          # Next.js application
│   ├── app/
│   │   └── page.tsx   # Main page
│   └── README.md      # Frontend-specific docs
├── backend/           # Python Lambda backend
│   ├── deploy.py      # Lambda deployment package builder
│   ├── lambda_handler.py
│   ├── server.py
│   ├── context.py
│   └── resources.py
├── terraform/         # Infrastructure as Code
│   ├── main.tf        # Main resources (Lambda, API Gateway, CloudFront, S3)
│   └── backend-setup.tf # Terraform state storage (run once)
├── scripts/
│   └── deploy.sh      # Deployment script
└── .github/
    └── workflows/
        └── deploy.yml # GitHub Actions CI/CD
```

## Prerequisites

- [Node.js](https://nodejs.org/) (v20+)
- [Python](https://www.python.org/) (3.12+)
- [Terraform](https://developer.hashicorp.com/terraform/install)
- [Docker](https://www.docker.com/) (for Lambda packaging)
- [UV](https://github.com/astral-sh/uv) (Python package manager)
- AWS CLI configured with appropriate credentials

## Local Development

### Frontend

```bash
cd frontend
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

### Backend

```bash
cd backend
uv run deploy.py
```

## Deployment

### Automated (GitHub Actions)

Push to `main` triggers deployment. You can also manually trigger via the Actions tab:

1. Go to **Actions** → **Deploy Digital Twin**
2. Select environment: `dev`, `test`, or `prod`
3. Click **Run workflow**

### Manual Deployment

```bash
# 1. Deploy infrastructure and frontend
./scripts/deploy.sh <environment> [project_name]
# Example:
./scripts/deploy.sh dev twin
```

The script will:
1. Build the Lambda deployment package
2. Initialize Terraform and apply infrastructure changes
3. Build and deploy the Next.js frontend to S3
4. Invalidate CloudFront cache

### Environment Variables

| Variable | Description |
|----------|-------------|
| `AWS_ACCOUNT_ID` | AWS account ID for deployment |
| `DEFAULT_AWS_REGION` | AWS region (default: `us-east-1`) |
| `AWS_ROLE_ARN` | IAM role ARN for GitHub Actions |
| `OPENAI_API_KEY` | OpenAI API key stored as a GitHub Actions secret |
| `NEXT_PUBLIC_API_URL` | API Gateway URL for frontend |

## Terraform

### Initial Setup (one-time)

```bash
cd terraform
terraform apply -target=aws_s3_bucket.terraform_state -target=aws_dynamodb_table.terraform_locks
```

Then remove `terraform/backend-setup.tf` and run `terraform init` with backend configuration.

### Workspaces

Terraform workspaces correspond to environments:

```bash
cd terraform
terraform workspace new dev
terraform workspace select dev
terraform apply
```

## Features

- **Interactive chat interface** — converse with your digital twin
- **AI-powered responses** — powered by Amazon Bedrock (GPT-5.4 nano)
- **Conversation memory** — stored in S3 for persistence
- **Global CDN** — served via CloudFront for low latency
- **Custom domain support** — optional custom domain with SSL/TLS
- **Serverless** — fully managed AWS infrastructure (Lambda + API Gateway + S3 + CloudFront)

## License

Private project.
