# Oleksandr's Digital Twin

An AI-powered digital twin — an interactive web application that simulates conversations as Oleksandr Oleksandrov, built with Next.js, Python (Lambda), and AWS infrastructure.

## Overview

This project is a personal digital twin: an interactive chat interface that represents Oleksandr Oleksandrov on his professional website. The AI twin answers questions about his background, skills, and experience by drawing from a curated knowledge base (CV, summary notes, style guidelines) and generating responses via Amazon Bedrock.

The system consists of:

- **Next.js frontend** — chat UI with session management
- **Python Lambda backend** — FastAPI app serving chat endpoints via Mangum
- **Amazon Bedrock** — AI model for generating responses
- **S3** — conversation memory storage
- **Terraform** — infrastructure provisioning

## Architecture

```
User → CloudFront → S3 (Next.js static site)
                 ↘ API Gateway (HTTP API) → Lambda (Python/FastAPI) → Amazon Bedrock
                                                              ↘ S3 (conversation history)
```

| Component | Technology |
|-----------|-----------|
| Frontend | Next.js 16 (App Router) + React 19 + Tailwind CSS v4 |
| Backend | Python 3.12 (AWS Lambda) via FastAPI + Mangum |
| API | API Gateway v2 (HTTP API) |
| AI Model | Amazon Bedrock (`us.amazon.nova-micro-v1:0`) |
| Memory | S3 (per-session conversation JSON) |
| Infrastructure | Terraform (workspaces: dev, test, prod) |
| Hosting | S3 + CloudFront |
| Custom Domain | Route 53 + ACM (optional) |
| Lambda Runtime | Python 3.12, arm64, SnapStart enabled |

## Environments

| Environment | URL | Description |
|-------------|-----|-------------|
| **Production** | https://www.oleksandraitwin.com | Live deployment with custom domain |
| **Staging** | https://d27i28hhowm0s1.cloudfront.net | Pre-production deployment |
| **Development** | https://d13aejtk951nms.cloudfront.net | Development deployment |

## Project Structure

```
twin/
├── frontend/              # Next.js application
│   ├── app/
│   │   ├── page.tsx       # Main page (chat interface)
│   │   ├── layout.tsx     # Root layout (fonts, metadata)
│   │   └── globals.css    # Global styles (Tailwind + animations)
│   ├── components/
│   │   └── twin.tsx       # Chat component (message handling, API calls)
│   ├── public/            # Static assets (avatars, icons)
│   ├── package.json
│   ├── next.config.ts     # Static export config
│   └── tsconfig.json
├── backend/               # Python Lambda backend
│   ├── lambda_handler.py  # Mangum entrypoint
│   ├── server.py          # FastAPI app (chat, health, conversation endpoints)
│   ├── context.py         # System prompt builder (from knowledge base)
│   ├── resources.py       # Knowledge base loader (PDF, text, JSON)
│   ├── deploy.py          # Lambda deployment package builder (Docker)
│   ├── data/              # Knowledge base files
│   │   ├── facts.json     # Personal info (name, role, education, etc.)
│   │   ├── summary.txt    # Role summary
│   │   ├── style.txt      # Communication style guidelines
│   │   └── Oleksandrov_CV_2026.pdf  # LinkedIn profile (PDF)
│   ├── pyproject.toml     # Python dependencies
│   ├── requirements-lambda.txt  # Lambda-specific dependencies
│   └── .python-version    # Python 3.12
├── terraform/             # Infrastructure as Code
│   ├── main.tf            # All AWS resources (Lambda, API GW, CloudFront, S3, IAM, Route53, ACM)
│   ├── variables.tf       # Input variables
│   ├── outputs.tf         # Output values
│   ├── versions.tf        # Terraform & provider versions
│   ├── prod.tfvars        # Production variables (custom domain)
│   └── terraform.tfvars   # Default/dev variables
├── scripts/
│   ├── deploy.sh          # Full deployment script (infra + frontend)
│   └── destroy.sh         # Environment destruction script
├── .github/
│   └── workflows/
│       ├── deploy.yml      # CI/CD: auto-deploy on push to main + manual trigger
│       └── destroy.yml     # CI/CD: manual destroy with confirmation
└── README.md
```

## Knowledge Base

The digital twin's responses are grounded in data files in `backend/data/`:

| File | Format | Purpose |
|------|--------|---------|
| `facts.json` | JSON | Personal details: name, role, location, education, specialties, LinkedIn URL |
| `summary.txt` | Text | Role description — "chatbot acting as a Digital Twin" |
| `style.txt` | Text | Communication style: professional but approachable, practical, concise |
| `Oleksandrov_CV_2026.pdf` | PDF | Full LinkedIn profile (parsed at runtime via `pypdf`) |

These files are loaded by `backend/resources.py`, and assembled into a system prompt by `backend/context.py`. The prompt instructs the model to faithfully represent Oleksandr, avoid hallucination, and maintain professionalism.

## Prerequisites

- **Node.js** (v20+)
- **Python** (3.12+) — managed with [uv](https://github.com/astral-sh/uv)
- **Terraform** (>= 1.0)
- **Docker** (for Lambda packaging)
- **AWS CLI** configured with appropriate credentials
- **UV** (Python package manager)

## Local Development

### Frontend

```bash
cd frontend
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

### Backend (local)

```bash
cd backend
uv run python server.py
```

The FastAPI server runs on `http://localhost:8000`. The `/chat` endpoint accepts requests with `{"message": "...", "session_id": "..."}` and returns `{"response": "...", "session_id": "..."}`.

### Building Lambda Package

```bash
cd backend
uv run deploy.py
```

This creates `lambda-deployment.zip` using the AWS Lambda Python 3.12 Docker image (arm64) with all production dependencies.

## API Endpoints

The backend exposes these endpoints via API Gateway:

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/` | Root — returns API info, model, and storage status |
| `GET` | `/health` | Health check |
| `POST` | `/chat` | Send a message — returns AI response with session ID |
| `GET` | `/conversation/{session_id}` | Retrieve conversation history |

**Chat request:**
```json
{
  "message": "What is your background?",
  "session_id": "optional-session-id"
}
```

**Chat response:**
```json
{
  "response": "...",
  "session_id": "session-id"
}
```

## Deployment

### Automated (GitHub Actions)

Push to `main` triggers deployment to `dev`. You can also manually trigger via the Actions tab:

1. Go to **Actions** → **Deploy Digital Twin**
2. Select environment: `dev`, `test`, or `prod`
3. Click **Run workflow**

The workflow:
1. Configures AWS credentials via OIDC
2. Builds the Lambda deployment package
3. Applies Terraform infrastructure
4. Verifies API Gateway routes (minimum 3) and health endpoint
5. Builds and deploys the Next.js frontend to S3
6. Invalidates CloudFront cache

### Manual Deployment

```bash
./scripts/deploy.sh <environment> [project_name]
# Example:
./scripts/deploy.sh dev twin
```

The script will:
1. Build the Lambda deployment package
2. Initialize Terraform and apply infrastructure changes
3. Verify API Gateway routes and health endpoint
4. Build and deploy the Next.js frontend to S3
5. Invalidate CloudFront cache

### Destroy Environment

```bash
./scripts/destroy.sh <environment>
# Or via GitHub Actions:
# Go to Actions → Destroy Environment → select environment → type confirmation
```

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `AWS_ACCOUNT_ID` | AWS account ID for deployment | — |
| `DEFAULT_AWS_REGION` | AWS region | `us-east-1` |
| `AWS_ROLE_ARN` | IAM role ARN for GitHub Actions | — |
| `NEXT_PUBLIC_API_URL` | API Gateway URL for frontend | `http://localhost:8000` |

## Terraform

### Initial Setup (one-time)

If using remote state, configure the S3 backend and DynamoDB lock table before running `terraform init`.

### Workspaces

Terraform workspaces correspond to environments:

```bash
cd terraform
terraform workspace new dev
terraform workspace select dev
terraform apply
```

### Variables

| Variable | Type | Default | Description |
|----------|------|---------|-------------|
| `project_name` | string | — | Name prefix for all resources |
| `environment` | string | — | Environment: `dev`, `test`, or `prod` |
| `bedrock_model_id` | string | `us.amazon.nova-micro-v1:0` | Bedrock model ID |
| `lambda_timeout` | number | `60` | Lambda timeout in seconds |
| `api_throttle_burst_limit` | number | `10` | API Gateway throttle burst limit |
| `api_throttle_rate_limit` | number | `5` | API Gateway throttle rate limit |
| `use_custom_domain` | bool | `false` | Enable custom domain |
| `root_domain` | string | `""` | Apex domain (e.g., `oleksandraitwin.com`) |

### Outputs

| Output | Description |
|--------|-------------|
| `api_gateway_url` | API Gateway endpoint URL |
| `api_gateway_id` | API Gateway ID |
| `cloudfront_url` | CloudFront distribution URL |
| `s3_frontend_bucket` | Frontend S3 bucket name |
| `s3_memory_bucket` | Memory S3 bucket name |
| `lambda_function_name` | Lambda function name |
| `lambda_snapstart_alias` | SnapStart alias name |
| `lambda_published_version` | Published Lambda version |
| `custom_domain_url` | Custom domain URL (if configured) |

## Features

- **Interactive chat interface** — converse with the digital twin in a polished UI
- **AI-powered responses** — powered by Amazon Bedrock (Nova Micro)
- **Knowledge-grounded** — responses based on real data (CV, notes, style guide), no hallucination
- **Conversation memory** — per-session history stored in S3
- **Session management** — automatic session ID generation; pass your own for continuity
- **Global CDN** — served via CloudFront for low latency
- **Custom domain support** — optional Route 53 + ACM with SSL/TLS
- **Serverless** — fully managed AWS infrastructure (Lambda + API Gateway + S3 + CloudFront)
- **SnapStart** — Lambda SnapStart for faster cold starts
- **arm64** — Lambda functions run on ARM64 architecture
- **Rate limiting** — API Gateway throttling configured per environment
- **CI/CD** — GitHub Actions workflows for deploy and destroy with environment isolation

## CI/CD Workflows

### Deploy (`deploy.yml`)

- **Trigger**: Push to `main` (auto-deploy to dev) or manual dispatch (any environment)
- Uses OIDC (`AWS_ROLE_ARN`) for AWS authentication
- Runs `scripts/deploy.sh` with the target environment

### Destroy (`destroy.yml`)

- **Trigger**: Manual dispatch only
- Requires confirmation input matching the environment name
- Runs `scripts/destroy.sh` with the target environment

## Configuration Files

| File | Purpose |
|------|---------|
| `frontend/package.json` | Node.js dependencies (Next.js 16, React 19, Tailwind CSS 4, Lucide icons) |
| `frontend/next.config.ts` | Next.js config (static export for S3 hosting) |
| `frontend/eslint.config.mjs` | ESLint config (Next.js core vitals + TypeScript) |
| `frontend/tsconfig.json` | TypeScript config (strict, path aliases `@/*`) |
| `backend/pyproject.toml` | Python dependencies (FastAPI, Boto3, Mangum, pypdf, etc.) |
| `backend/requirements-lambda.txt` | Lambda runtime dependencies (excludes uvicorn) |
| `backend/.python-version` | Python 3.12 |
| `terraform/versions.tf` | Terraform >= 1.0, AWS provider ~> 6.10 |
| `.gitignore` | Ignores build artifacts, node_modules, Terraform state, env files |

## License

Private project.
