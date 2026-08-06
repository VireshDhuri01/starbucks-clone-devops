# ☕ Starbucks Clone - DevOps Project

## 📌 Overview

This project demonstrates an end-to-end DevOps implementation for deploying a **Starbucks Clone** application on **Amazon EKS** using a fully automated CI/CD pipeline.

## 🚀 Tech Stack

- GitHub
- Jenkins
- SonarQube
- Trivy
- Docker
- AWS ECR
- AWS EC2
- Amazon EKS
- Kubernetes
- Helm
- Argo CD
- Prometheus
- Grafana

## ⚙️ CI/CD Workflow

```
GitHub
   ↓
Jenkins
   ↓
SonarQube Analysis
   ↓
Trivy Security Scan
   ↓
Docker Build & Push on AWS ECR
   ↓
Update Kubernetes Manifest
   ↓
Argo CD Sync
   ↓
Amazon EKS Deployment
```
## 🔐 Security Groups

Allow:

- 22 (SSH)
- 80 (HTTP)
- 8080 (Jenkins)
- 9000 (SonarQube)
- 3000 (Grafana)
- 9090 (Prometheus)
- 443 (EKS API)
- Application - (Check service.yml)

## ✨ Features


## 📸 Screenshots

- Jenkins Pipeline
- Argo CD Dashboard
- Starbucks Clone Application

## 👨‍💻 Author

**Viresh Dhuri**
