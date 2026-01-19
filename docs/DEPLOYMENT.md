# Deployment

## Local

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn api.main:app --host 0.0.0.0 --port 8080 --reload
```

## Environment variables

```bash
export FAISS_BASE="faiss_index"
export UPLOAD_BASE="data"
export FAISS_INDEX_NAME="index"
```

Configure provider credentials through `utils/model_loader.py` (for example `OPENAI_API_KEY`, `GEMINI_API_KEY`, `GROQ_API_KEY`, etc.).

## ECS deployment (high level)

1) Pull the latest code and ensure workflow files are up to date.
2) Create AWS account and IAM user with required permissions.
3) Store AWS credentials in GitHub secrets: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`.
4) Store model API keys in AWS Secrets Manager.
5) Create ECR repository and ECS cluster.
6) Attach the policies in `incline_policy.json` to the IAM user or ECS task role.
7) Update EC2 security group inbound rules for port 8080.
8) Deploy service and locate the task public IP to access the app.

Example access:
`http://<public_ip>:8080`

## Logs

Use CloudWatch Logs:
- CloudWatch -> Log Groups -> Task ID -> log stream

## Operations checklist

- Persist `FAISS_BASE` and `UPLOAD_BASE` on mounted volumes.
- Set proper CORS settings for production.
- Validate LLM provider quotas and timeouts.
- Rotate API keys and secrets regularly.
