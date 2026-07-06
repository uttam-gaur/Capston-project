# GitHub Actions CI/CD Capstone Project 

A complete end-to-end CI/CD pipeline built using GitHub Actions, Docker, Python Flask, and Pytest.

This project demonstrates how multiple GitHub Actions workflows can work together to automate application testing, Docker image creation, container registry publishing, deployment verification, and scheduled health monitoring.

## Project Overview

The goal of this project is to build a production-style CI/CD pipeline using reusable GitHub Actions workflows.

The pipeline automatically:

* Runs tests when a Pull Request is opened or updated
* Validates code before merging into the main branch
* Builds and pushes Docker images after code is merged into `main`
* Uses GitHub Secrets for Docker Hub authentication
* Uses a GitHub Environment named `production`
* Runs the Docker container during the deployment verification stage
* Verifies the application using the `/health` endpoint
* Performs an automated health check every 12 hours
* Generates workflow summaries using `$GITHUB_STEP_SUMMARY`

---

## Tech Stack

* Python
* Flask
* Pytest
* Docker
* GitHub Actions
* Docker Hub
* YAML

---

## Project Structure

```text
.
├── .github/
│   └── workflows/
│       ├── reusable-build-test.yml
│       ├── reusable-docker.yml
│       ├── pr-pipeline.yml
│       ├── main-pipeline.yml
│       └── health-check.yml
│
├── app.py
├── test_app.py
├── requirements.txt
├── Dockerfile
└── README.md
```

---

##  CI/CD Pipeline Architecture

### Pull Request Pipeline

```text
Feature Branch
      │
      ▼
Pull Request to Main
      │
      ▼
Reusable Build & Test Workflow
      │
      ▼
Install Dependencies
      │
      ▼
Run Pytest
      │
      ▼
PR Checks Passed
```

Docker images are not built or pushed during the Pull Request pipeline.

### Main Branch Pipeline

```text
PR Merge / Push to Main
          │
          ▼
     Build & Test
          │
          ▼
   Docker Image Build
          │
          ▼
    Push to Docker Hub
          │
          ▼
 Production Environment
          │
          ▼
 Pull Production Image
          │
          ▼
 Run Docker Container
          │
          ▼
 Verify /health Endpoint
          │
          ▼
 Deployment Summary
```

### Scheduled Health Check

```text
Every 12 Hours
      │
      ▼
Pull Latest Docker Image
      │
      ▼
Start Container
      │
      ▼
Wait for Application Startup
      │
      ▼
Call /health Endpoint
      │
      ▼
Generate Health Report
      │
      ▼
Stop and Remove Container
```

---

## GitHub Actions Workflows

### 1. Reusable Build and Test

File:

```text
.github/workflows/reusable-build-test.yml
```

This reusable workflow is triggered using `workflow_call`.

It performs the following tasks:

* Checks out the repository
* Sets up the requested Python version
* Installs application dependencies
* Runs Pytest when `run_tests` is enabled
* Returns the test result as a workflow output

---

### 2. Reusable Docker Build and Push

File:

```text
.github/workflows/reusable-docker.yml
```

This reusable workflow:

* Checks out the repository
* Authenticates with Docker Hub
* Builds the application Docker image
* Creates image tags
* Pushes the image to Docker Hub
* Returns the complete image path as a workflow output

---

### 3. Pull Request Pipeline

File:

```text
.github/workflows/pr-pipeline.yml
```

Trigger:

```text
Pull Request → main
```

Supported PR events:

* `opened`
* `synchronize`

The pipeline runs application tests and generates a PR validation summary.

No Docker image is pushed during PR validation.

---

### 4. Main CI/CD Pipeline

File:

```text
.github/workflows/main-pipeline.yml
```

Trigger:

```text
Push / Merge → main
```

Pipeline sequence:

```text
Build & Test
      ↓
Docker Build & Push
      ↓
Production Deployment Verification
```

The deployment job pulls the latest image, starts the container, and verifies the Flask health endpoint.

---

### 5. Scheduled Health Check

File:

```text
.github/workflows/health-check.yml
```

The health check runs automatically every 12 hours.

Cron schedule:

```text
0 */12 * * *
```

The workflow also supports manual execution through `workflow_dispatch`.

---

##  Required GitHub Secrets

The following repository secrets are required:

| Secret            | Purpose                 |
| ----------------- | ----------------------- |
| `DOCKER_USERNAME` | Docker Hub username     |
| `DOCKER_TOKEN`    | Docker Hub access token |

Add them from:

```text
Repository
→ Settings
→ Secrets and variables
→ Actions
→ New repository secret
```

Never store Docker Hub credentials directly inside workflow files.

---

##  Production Environment

This project uses a GitHub Environment named:

```text
production
```

Create it from:

```text
Repository
→ Settings
→ Environments
→ New environment
→ production
```

Environment protection rules and required reviewers can be configured when supported by the repository and account settings.

---

## Run the Application Locally

### Install dependencies

```bash
pip install -r requirements.txt
```

### Run tests

```bash
pytest -v
```

### Start the Flask application

```bash
python app.py
```

Test the health endpoint:

```bash
curl http://localhost:5000/health
```

Expected response:

```json
{
  "status": "healthy"
}
```

---

##  Run with Docker

Build the image:

```bash
docker build -t github-actions-capstone .
```

Run the container:

```bash
docker run -d \
  --name capstone-app \
  -p 5001:5000 \
  github-actions-capstone
```

Verify the application:

```bash
curl http://localhost:5001/health
```

Stop and remove the container:

```bash
docker stop capstone-app
docker rm capstone-app
```

---

##  Workflow Execution Flow

| Event               | Workflow      | Result                                         |
| ------------------- | ------------- | ---------------------------------------------- |
| Pull Request opened | PR Pipeline   | Build and Test                                 |
| PR updated          | PR Pipeline   | Build and Test                                 |
| Merge to `main`     | Main Pipeline | Test, Build, Push, and Deployment Verification |
| Every 12 hours      | Health Check  | Container Health Validation                    |
| Manual trigger      | Health Check  | On-demand Health Validation                    |

---

##  Future Improvements

The next improvements planned for this project are:

* Add Trivy vulnerability scanning
* Fail builds when critical vulnerabilities are detected
* Upload security scan reports as workflow artifacts
* Add Slack notifications for failed pipelines
* Add development, staging, and production environments
* Deploy to a persistent cloud target such as a virtual machine or container orchestration platform
* Add automatic rollback for failed deployments
* Use Terraform for infrastructure provisioning
* Add Kubernetes and Helm-based deployments
* Add monitoring and observability
* Use OIDC authentication instead of long-lived cloud credentials

---

##  Key Learnings

Through this project, I practiced:

* Creating reusable GitHub Actions workflows
* Using `workflow_call`
* Passing inputs and secrets to reusable workflows
* Managing workflow outputs
* Creating job dependencies with `needs`
* Working with Pull Request events
* Creating branch-based CI/CD pipelines
* Building and pushing Docker images automatically
* Managing credentials using GitHub Secrets
* Using GitHub Environments
* Running scheduled workflows with cron
* Using `workflow_dispatch`
* Performing container health checks
* Creating workflow reports with `$GITHUB_STEP_SUMMARY`

---

## Conclusion

This project demonstrates a complete GitHub Actions-based CI/CD workflow for a containerized Flask application.

The pipeline validates code changes through Pull Requests, runs automated tests, builds and publishes Docker images after changes reach the main branch, performs deployment verification, and continuously checks container health on a schedule.

This capstone project brings together reusable workflows, Docker automation, secrets management, deployment environments, scheduled workflows, and health monitoring into one end-to-end DevOps project.

---

##Acknowledgements

Built as part of the **90DaysOfDevOps** learning journey with the TrainWithShubham community.

#90DaysOfDevOps #DevOpsKaJosh #TrainWithShubham
