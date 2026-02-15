# DevOps Automation Pipeline

**CI pipeline with automated security scanning producing deployable container artifacts**

## 🎯 Overview

Automated CI pipeline that builds Docker images from application source code, scans for vulnerabilities using Trivy, and pushes verified images to Docker Hub. Demonstrates modern DevOps practices with Jenkins, Docker, and security-first automation. The CD workflow is handled separately using ArgoCD in a [companion repository](https://github.com/Ashik-Domain/CD_Pipeline).

## 🏗️ Architecture

```
GitHub (Code Push)
    ↓
Jenkins CI Pipeline (AWS EC2)
    ├─→ Job 1: Build Docker Image
    ├─→ Job 2: Trivy Security Scan (HIGH/CRITICAL)
    └─→ Job 3: Push to Docker Hub
           ↓
    Docker Hub (Container Registry)
           ↓
    [Consumed by CD Pipeline]
```

**Note**: The CD workflow is managed separately using ArgoCD in the [CD Pipeline repository](https://github.com/Ashik-Domain/CD_Pipeline), which deploys applications to AWS EKS following GitOps principles.

## 📸 Pipeline in Action

### Jenkins Dashboard
![Jenkins Jobs Overview](pictures/jobs.png)
*Three chained Jenkins jobs forming the complete CI pipeline*

### Pipeline Execution
![Build Pipeline](pictures/pipeline.png)
*Detailed view of pipeline stages and execution status*
*Automated pipeline execution from code commit to registry push*

### Individual Jobs

**Job 1: Build Docker Image**
![Build Job](pictures/job1.png)

**Job 2: Security Scanning with Trivy**
![Security Scan Job](pictures/job2.png)

**Job 3: Push to Docker Hub**
![Push Job](pictures/job3.png)

### Deployment Result
![Docker Hub Image](pictures/dockerhub-image.png)
*Successfully pushed and tagged image in Docker Hub registry*

## 🔧 Technologies Used

| Category | Tools |
|----------|-------|
| CI/CD | Jenkins (self-hosted on AWS EC2) |
| Containerization | Docker |
| Security Scanning | Trivy |
| GitOps | ArgoCD |
| Container Registry | Docker Hub |
| Orchestration | Kubernetes (AWS EKS) |
| Infrastructure | AWS (EC2, EKS) |

## ✨ Key Features

- ✅ **Automated Builds**: Triggers on GitHub code commits
- ✅ **Security First**: Trivy scans for HIGH and CRITICAL vulnerabilities before deployment
- ✅ **Separated CI/CD**: CI pipeline (build/test/push) separated from CD pipeline (deployment)
- ✅ **Automated CI Workflow**: Three-stage Jenkins pipeline executes automatically
- ✅ **Container Registry Integration**: Automatic push to Docker Hub after successful scan

## 🚀 Pipeline Workflow

### CI Pipeline (This Repository)

1. **Trigger**: Developer pushes code to GitHub
2. **Build**: Jenkins pulls code and builds Docker image from Dockerfile
3. **Scan**: Trivy analyzes image for security vulnerabilities
4. **Gate**: Only images passing security checks proceed
5. **Push**: Verified image pushed to Docker Hub with latest tag

### CD Pipeline (Separate Repository)

The CI pipeline produces deployable container artifacts. The CD workflow is handled separately:

6. **Artifact Storage**: Verified images stored in Docker Hub registry
7. **GitOps Management**: Kubernetes manifests in Git define desired cluster state
8. **ArgoCD Reconciliation**: ArgoCD ensures EKS cluster matches manifest definitions
9. **Image Updates**: When pods are recreated, Kubernetes pulls updated images from Docker Hub

> **Note**: The CD workflow uses ArgoCD in a [separate repository](https://github.com/Ashik-Domain/CD_Pipeline) following GitOps separation of concerns. See the CD repository for details on deployment workflow and Istio service mesh configuration.

## 📊 Jenkins Job Configuration

The pipeline consists of three chained Jenkins jobs:

### Job 1: Build_Docker_Image
- Pulls source code from [Bookinfo application](https://github.com/Ashik-Domain/Bookinfo_app.git)
- Builds Docker image using Dockerfile
- Tags image as `productpage-app`

### Job 2: Test_Docker_Image
- Runs Trivy vulnerability scanner
- Scans for HIGH and CRITICAL severity issues
- Fails pipeline if critical vulnerabilities found

### Job 3: Push_to_DockerHub
- Tags image with Docker Hub username
- Authenticates to Docker Hub
- Pushes verified image to Docker Hub registry
- Produces deployable container artifact for CD pipeline consumption

## 🔐 Security

- **Vulnerability Scanning**: Every image scanned before deployment
- **Severity Filtering**: Only HIGH/CRITICAL issues block deployments
- **GitOps**: No direct cluster access; all changes via Git
- **Least Privilege**: Jenkins service account with minimal AWS permissions

## 📁 Repository Contents

```
.
├── Dockerfile           # Container image definition
├── README.md           # This file
└── pictures/           # Architecture diagrams and screenshots
```

## 🛠️ Local Setup

Want to replicate this pipeline? See [SETUP.md](docs/SETUP.md) for detailed installation instructions including:
- AWS EC2 setup for Jenkins
- Jenkins installation and configuration
- Docker and Trivy installation
- Jenkins job creation steps
- ArgoCD integration

## 🎓 What I Learned

- Designing multi-stage CI/CD pipelines with job chaining
- Integrating security scanning into automated workflows
- Implementing GitOps principles with separate CI/CD repos
- Managing Jenkins on AWS EC2 infrastructure
- Docker image optimization and security best practices

## 🔗 Related Projects

- [CD Pipeline with ArgoCD](https://github.com/Ashik-Domain/CD_Pipeline) - GitOps deployment to EKS
- [Bookinfo Application](https://github.com/Ashik-Domain/Bookinfo_app) - Sample microservices app

## 📝 Technical Details

**Application**: Bookinfo microservices (Product Page component)  
**Base Image**: Python-based application  
**Build Tool**: Docker  
**CI Server**: Jenkins 2.x on Amazon Linux 2  
**Scanner**: Trivy (Aqua Security)  
**Deployment Target**: AWS EKS cluster  

---
