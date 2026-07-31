---
title: "Integrate the Application on EC2"
date: 2026-07-30
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

The repository does not create EC2 resources. The GitHub Actions workflow assumes that an existing host is available, reachable over SSH, and prepared to run Docker Compose.

## 1. Create an EC2 Instance in the AWS Console

1. Open **Amazon EC2** → **Instances** → **Launch instances**.
2. Enter the name `movie-recommendation-server` and add the tag `Environment=<ENVIRONMENT>`.
3. Select the **Ubuntu Server 24.04 LTS, 64-bit (x86)** AMI or an organization-approved Linux AMI.
4. Select an appropriate instance type. The current workshop environment uses `t3.micro`; increase the capacity if Docker builds or actual traffic exceed its resources.
5. Under **Key pair (login)**, select an existing key pair or create a new one. Store the private key once in a secure location; AWS does not allow the private key to be downloaded again.
6. Under **Network settings**, select the correct VPC and subnet. Enable auto-assign public IPv4 only when the workshop requires direct Internet access.
7. Select or create a security group dedicated to the application host; configure its inbound rules as described in section 2 below.
8. Under **Configure storage**, select a `gp3` volume large enough for the OS, source code, Docker images, containers, and logs. Enable EBS encryption.
9. Open **Advanced details** and attach the backend IAM instance profile; do not place access keys in user data.
10. Review **Summary** → **Launch instance** → wait until the instance state is `Running` and both status checks pass.

## 2. Configure the Security Group and Inbound Rules

1. Open **EC2** → **Security Groups** → select the security group attached to the instance.
2. Select the **Inbound rules** tab → **Edit inbound rules** → **Add rule**.
3. Add only the rules that are actually required, as shown in the table below.
4. Select **Save rules** and verify access from an authorized client.

| Purpose | Type/Protocol | Port | Recommended source |
|---|---|---:|---|
| Linux administration | SSH / TCP | 22 | `<ADMIN_PUBLIC_IP>/32`, a VPN CIDR, or a bastion security group; do not use `0.0.0.0/0` |
| Web without TLS | HTTP / TCP | 80 | `0.0.0.0/0` and `::/0` only when the website must be public |
| Web with TLS | HTTPS / TCP | 443 | `0.0.0.0/0` and `::/0` when the website must be public |
| Direct application port | Custom TCP | `<APPLICATION_PORT>` | The load balancer/reverse proxy security group or an approved test CIDR |
| Internal backend | Custom TCP | `<BACKEND_PORT>` | Do not create a public rule when the frontend/reverse proxy runs on the same host |

![Inbound rules of the EC2 security group](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.3-integrate-ec2-application/ec2-security-group-inbound-rules.png)

*The `launch-wizard-1` security group has three inbound TCP rules for SSH port `22`, frontend port `5173`, and backend port `8000`.*

## 3. Prepare the EC2 Host

The platform owner must provide:

- `<EC2_INSTANCE_ID>`.
- IAM instance profile.
- Application directory on the host.
- An inbound rule for `<APPLICATION_PORT>` or a reverse proxy.
- Disk capacity.
- DNS and TLS if the application is public.
- Git access.
- Docker Engine and Docker Compose v2.

The repository does not currently specify the AMI, instance type, VPC, subnet, security group, disk, DNS, or TLS configuration.

The deployment environment screenshot confirms that the `movie-recommendation-server` instance is `Running`, uses the `t3.micro` instance type, and has both public and private IPv4 addresses. This is evidence of the current environment, not a substitute for Infrastructure as Code.

![EC2 instance details for the movie recommendation application](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.3-integrate-ec2-application/ec2-instance-summary.jpg)

*The EC2 Console confirms the application host's status, instance type, network addresses, hostname, and VPC.*

![Successful SSH connection to the EC2 host](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.3-integrate-ec2-application/ec2-ssh-session.jpg)

*The SSH session confirms access to Ubuntu 24.04.4 LTS on EC2.*

## 4. Configure the Application

Place `.env` directly on EC2 according to the approved secret-management process. Do not commit `.env` or create this file in GitHub Actions.

On EC2, prefer an instance profile so the AWS SDK obtains credentials through the default provider chain. Do not copy permanent access keys to the server.

![GitHub deploy key for EC2](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.3-integrate-ec2-application/github-ec2-deploy-key.jpg)

*The repository has configured the `EC2 Deploy` deploy key in read-only mode so the host can retrieve the source code.*

## 5. Deployment Workflow

When a commit is pushed to the `main` branch, GitHub Actions:

1. Builds the frontend.
2. Installs backend dependencies and runs `compileall`.
3. Connects to EC2 over SSH.
4. Changes to `EC2_APP_DIR`.
5. Pulls source from `main`.
6. Runs Docker Compose.

![Successful GitHub Actions workflow build](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.3-integrate-ec2-application/github-actions-build-success.png)

## 6. Runtime Integration

When the backend starts, it:

1. Creates a boto3 session.
2. Verifies the STS identity.
3. Describes the key schemas of the DynamoDB tables.
4. Checks the S3 bucket.
5. Describes the SageMaker endpoint through a health check that does not block the entire application.
6. Initializes repositories, services, and the provider.

When the endpoint is unavailable, the guest API can continue to run; a personalized cache miss returns a controlled error.

## 7. Start the Application

From the application directory on EC2, after the code, `.env`, Docker, and IAM role are ready:

```bash
docker compose config --quiet
docker compose up --build -d
docker compose ps
docker compose logs backend --tail 100
```

## 8. Verify the Services

```bash
curl -f "http://127.0.0.1:<BACKEND_PORT>/health"

curl -f \
  "http://127.0.0.1:<BACKEND_PORT>/api/v1/movies?limit=1"
```

Expected results:

- The backend container is healthy.
- The frontend returns HTML.
- `/health` returns `{"status":"ok"}`.
- `/movies` returns a JSON array, or a controlled `503` error if a data resource is configured incorrectly.
- Startup logs do not expose credentials.

![Swagger UI for the Movie Recommendation API running on EC2](/images/5-Workshop/5.4-Recommendation-pipeline/5.4.3-integrate-ec2-application/ec2-fastapi-swagger-ui.png)

## 9. Distinguish the EC2 Application from EC2 Retraining

`ml/deploy/ec2_bootstrap.sh` configures a systemd timer for retraining, not web deployment. This template currently requires two fixes:

- Its default subdirectory does not match the `ml` submodule path.
- Its `events/` event prefix does not match the canonical `datasets/exports/` configuration.
