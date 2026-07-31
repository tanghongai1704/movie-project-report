---
title: "Integrate Application on EC2"
date: 2026-07-30
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

The repository does not provision EC2 infrastructure. The GitHub Actions workflow assumes an existing target host with SSH accessibility prepared to run Docker Compose.

## 1. Prepare EC2 Host

The platform owner must provide:

- `<EC2_INSTANCE_ID>`.
- An IAM instance profile.
- An application directory on the host.
- Inbound security group rules for `<APPLICATION_PORT>` or reverse proxy access.
- Adequate disk storage capacity.
- DNS names and TLS certificates for public endpoints.
- Git repository access credentials.
- Docker Engine and Docker Compose v2 installation.

AMI selection, instance type, VPC, subnets, security group rules, storage volumes, DNS names, and TLS configurations are not defined within this repository.

Deployment evidence confirms that the `movie-recommendation-server` instance is `Running`, uses the `t3.micro` instance type, and has both public and private IPv4 addresses. This records the current environment but does not replace Infrastructure as Code.

![EC2 instance hosting the movie recommendation application](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.3-integrate-ec2-application/ec2-instance-summary.jpg)

*The EC2 Console confirms the application host state, instance type, network addresses, hostname, and VPC.*

![Successful SSH connection to the EC2 host](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.3-integrate-ec2-application/ec2-ssh-session.jpg)

*The SSH session confirms access to Ubuntu 24.04.4 LTS on EC2. The host reports a required restart and pending updates, so maintenance should be completed before treating it as production-ready.*

## 2. Configure Application Environment

Place the `.env` file directly on the EC2 host in accordance with approved secrets management procedures. Do not commit `.env` files or create secrets directly within GitHub Actions scripts.

On EC2, prefer using an IAM instance profile so the AWS SDK acquires credentials via the default provider chain. Do not copy static, permanent access keys onto the server.

![GitHub deploy key configured for EC2](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.3-integrate-ec2-application/github-ec2-deploy-key.jpg)

*The repository uses the read-only `EC2 Deploy` key so the host can retrieve source code. Its private key must remain only on the host or in an approved secret store.*

## 3. Deployment Workflow

Upon pushing to the `main` branch, GitHub Actions executes:

1. Frontend static asset build.
2. Backend dependency installation and `compileall` validation.
3. SSH connection to the target EC2 host.
4. Navigation to `EC2_APP_DIR`.
5. Git pull from `main`.
6. Docker Compose deployment.

<!-- IMAGE-5.4.3-01: GitHub Actions deployment logs with hosts, users, and secrets redacted. -->

{{% notice warning %}}
The workflow executes `docker compose pull`, but `docker-compose.yml` uses local build context instead of remote container image registries. If the host does not rebuild local images, `docker compose up -d` may continue running older images.
{{% /notice %}}

## 4. Runtime Integration

Upon startup, backend services perform:

1. boto3 session initialization.
2. STS caller identity verification.
3. DynamoDB key schema verification for five tables.
4. S3 bucket existence and permission checks.
5. Non-blocking SageMaker endpoint description health checks.
6. Repository, service, and provider initialization.

If the SageMaker endpoint is unavailable, guest APIs continue functioning normally, while personalized cache misses return controlled, graceful error responses.

## 5. Launch Application Containers

In the application directory on EC2, once code, `.env`, Docker, and IAM roles are ready:

```bash
docker compose config --quiet
docker compose up --build -d
docker compose ps
docker compose logs backend --tail 100
```

## 6. Verify Application Services

```bash
curl -f "http://127.0.0.1:<BACKEND_PORT>/health"

curl -f \
  "http://127.0.0.1:<BACKEND_PORT>/api/v1/movies?limit=1"
```

Expected results:

- Backend container status is healthy.
- Frontend responds with HTML content.
- `/health` returns `{"status":"ok"}`.
- `/movies` returns a JSON array, or a controlled `503` if data resources are misconfigured.
- Startup logs expose zero credentials.

<!-- IMAGE-5.4.3-02: Docker services status and EC2 health check with IP addresses and hostnames redacted. -->

## 7. Distinguishing EC2 Application Deployment from EC2 Retraining

`ml/deploy/ec2_bootstrap.sh` configures a systemd timer for model retraining, not web application deployment. This script template requires modification before use:

- Default subdirectory paths do not match the `ml` submodule directory structure.
- Event path prefix `events/` does not match canonical configuration `datasets/exports/`.

Do not deploy this bootstrap script template to production without reviewing and updating these path definitions.

**Reference Sources:** `.github/workflows/deploy.yml`, `docker-compose.yml`, `backend/app/aws/infrastructure.py`, and `ml/deploy/ec2_bootstrap.sh`.
