# Docker, MLOps, and Deployment – Comprehensive Guide

This Repo will help Avanindra to understand Docker from fundamentals to production-grade deployment, including logging, debugging, AWS deployment, and operational best practices.

---

# 1. Introduction to Docker

## What is Docker?

Docker is a containerization platform that allows you to package an application along with all its dependencies and run it consistently across different environments.

A container includes:
- Application code
- Runtime
- Libraries
- System tools

### Mental Model

Application + Dependencies + Environment = Docker Container

---

## Key Docker Components

| Component   | Description |
|------------|------------|
| Image      | Blueprint of the application |
| Container  | Running instance of an image |
| Dockerfile | Instructions to build an image |
| Volume     | Persistent storage |
| Network    | Communication layer between containers |

---

# 2. Port Mapping

## Concept

docker run -p <host_port>:<container_port>

- Host port: Port on your machine
- Container port: Port inside the container

Python PORT variable = where Flask listens inside container
Docker -p first number = where your browser enters from your laptop
Docker -p second number = where Docker forwards traffic inside container

## Example

docker run -p 8000:5000 my-app

| Layer              | Port |
|-------------------|------|
| Browser (host)    | 8000 |
| Docker forwards   | 5000 |
| Flask app listens | 5000 |

## Important Rule

The application must run on:

0.0.0.0

Example:

app.run(host="0.0.0.0", port=5000)

---

# 3. Dockerfile

## Basic Example

FROM python:3.11

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

CMD ["python", "app.py"]

---

## Layer Caching Optimization

Docker builds images in layers.

### Incorrect Approach

COPY . .
RUN pip install -r requirements.txt

### Correct Approach

COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .

### Reason

Frequently changing files should be placed later to leverage caching and reduce build time.

Tip: The files which are frequently changed , keep it in downstream. --> Faster Building of Image

---

# 4. Volumes (Persistence)

## Why Volumes Are Required

Containers are ephemeral. When a container is deleted, all internal data is lost.

## Example

docker run -v $(pwd)/data:/app/data my-app

## Use Cases

- Database storage
- Logs
- Model artifacts
- Uploaded files

---

# 5. Docker Compose

Used to manage multiple containers.

## Example

services:
  app:
    build: .
    ports:
      - "8000:8000"

  postgres:
    image: postgres

  redis:
    image: redis

## When to Use

- Multi-service applications
- Local development environments

---

# 6. Logging

## View Logs

docker logs <container_name>

## Follow Logs

docker logs -f <container_name>

## Save Logs to File

docker logs <container_name> > logs.txt

## Best Practices

Applications should log:
- Inputs
- Outputs
- Errors
- System events

---

# 7. Debugging Docker

## Step 1: Check Running Containers

docker ps

## Step 2: Inspect Logs

docker logs <container_name>

## Step 3: Enter Container

docker exec -it <container_name> bash

## Step 4: Inspect Files

ls /app

## Step 5: Check Open Ports

netstat -tuln

## Step 6: Restart Container

docker restart <container_name>

---

# 8. Common Issues

## Port Not Accessible

Possible causes:
- Incorrect port mapping
- Application bound to localhost

Solution:
- Use correct -p mapping
- Ensure 0.0.0.0 binding

---

## File Not Found

Possible causes:
- Incorrect path
- File not copied

Solution:
- Verify working directory
- Use absolute paths

---

## Permission Denied (Linux)

sudo usermod -aG docker ubuntu
newgrp docker

---

# 9. AWS Deployment (Step-by-Step)

## Step 1: Build Docker Image (Local)

docker build -t sentiment-app .

## Step 2: Create ECR Repository

Example:

ml/sentiment-analysis-app

## Step 3: Authenticate Docker with ECR

aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com

## Step 4: Tag Image

docker tag sentiment-app:latest <account-id>.dkr.ecr.<region>.amazonaws.com/ml/sentiment-analysis-app:latest

## Step 5: Push Image to ECR

docker push <account-id>.dkr.ecr.<region>.amazonaws.com/ml/sentiment-analysis-app:latest

## Step 6: Launch EC2 Instance

- Choose Ubuntu
- Configure security group
- Open required ports (e.g., 8000)

## Step 7: Install Docker on EC2

sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker

## Step 8: Authenticate with ECR on EC2

aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com

## Step 9: Pull Image

docker pull <ecr-image-url>

## Step 10: Run Container

docker run -d -p 8000:8000 <image>

## Access Application

http://<EC2-public-ip>:8000

---

# 10. Production Concepts

## Stateless vs Stateful

| Type       | Example |
|-----------|--------|
| Stateless | FastAPI |
| Stateful  | Postgres |

---

## Storage Strategy

| Data Type | Storage |
|----------|--------|
| Application data | Database (Postgres / RDS) |
| Logs | S3 |
| Models | S3 / DVC |

---

# 11. Backup Strategy

## Example Workflow

pg_dump → upload to S3

## Scheduling

Use cron jobs for:
- Database backups
- Log archival

---

# 12. GenAI Progression (Contextual Learning)

LLM → RAG → Tools → Agents

---

# 13. Disk Cleanup

## Check Disk Usage

docker system df

## Remove Unused Resources

docker system prune

## Aggressive Cleanup

docker system prune -a

## Remove Volumes

docker volume prune

## Remove Build Cache

docker builder prune

## Full Cleanup

docker system prune -a --volumes

---

# 14. Final Architecture Flow

Local Development → Docker → ECR → EC2 → Production

---

# 15. Key Takeaways

- Docker ensures consistency across environments
- Volumes provide persistence
- Logs are essential for debugging
- ECR stores container images
- EC2 runs containers in production
- Databases handle durable storage