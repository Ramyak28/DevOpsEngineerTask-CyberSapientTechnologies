# DevOpsEngineerTask-CyberSapientTechnologies
This repository contains tasks, infrastructure templates, and automation scripts created for the DevOps Engineer Task at CyberSapient Technologies.


# Architecture/infra diagram
![image](https://github.com/user-attachments/assets/08050b66-3047-4a66-b269-4bb0dddc9ae8)

# 1. Developer → CI/CD Pipeline
## Developer Pushes Code
The flow begins with a developer committing and pushing their application code (and any Terraform infra manifests) into a Source Code Repository (e.g., GitHub, GitLab).

## CI Server Polls or Is Triggered
A CI Server (running on an EC2 instance in this diagram) detects the new commit. It checks out the latest code and any updated deployment/Terraform files.

## Static Code Analysis

Branching off first is a Code Analyzer step, which runs linters, style checks, and security‐lint rules against your source to catch bugs, coding‐standard violations, or potential injection flaws early.

## Dependency Scanning (SCA / OWASP Scan)

Next, a Dependency Scanner (or OWASP scanner) inspects your project’s libraries (e.g., NPM modules, Maven/JARs) to flag known vulnerabilities.

## Infrastructure Provisioning via Terraform

In parallel (or sequence, depending on your pipeline), the CI server invokes Terraform to ensure your cloud infra (VPCs, EKS cluster, load balancer, S3 backends, etc.) is created or updated to the desired state.

## Container Image Build & Security Scan

The application is built into one or more Docker images.

A Container Security Scanner (e.g., Trivy, Clair) inspects the resulting images for OS‐package or library vulnerabilities.

## Publish to Container Registry

Once they pass all checks, the CI server pushes the signed, scanned images to your Container Registry (e.g., Amazon ECR, Docker Hub).

# 2. Container Registry → Kubernetes Cluster
![image](https://github.com/user-attachments/assets/eef38efa-ed84-4e78-8a7e-c82601a09ce7)

## Kubernetes Pulls Images

Inside your Kubernetes Cluster (presumably running in EKS), your Deployment objects reference images in the registry. When you apply your Deployment manifests, each Pod pulls its container image from the registry.

## Web Frontend Service & Pod

The Frontend Deployment spins up one or more Frontend Pods behind a Frontend Service.

This Service exposes your UI tier internally (ClusterIP or LoadBalancer, depending on setup).

## API Backend Service & Pod

Similarly, the API Deployment creates API Pods behind an API Service.

These Pods expose endpoints like /api, /healthz, /ready, and /started for your backend logic, readiness and liveness checks.

## Data Store (MongoDB) Pod & Persistent Volume

Your API Pods connect to a MongoDB Service, which load‑balances to one or more MongoDB Pods that mount a Persistent Volume for durable data storage.

# 3. External Traffic & Monitoring
## External Load Balancer & Public DNS

To get traffic in from the Internet, you’ve provisioned an External Load Balancer (e.g., AWS ALB).

A Public DNS record (Route 53) points your user‑friendly domain (e.g. app.example.com) at that Load Balancer.

User Browser → Load Balancer → Frontend/API

End users open their browser and hit your DNS name.

The request goes to the Load Balancer, which routes HTTP(S) traffic to the Frontend Service in the cluster (and can also route API calls to the API Service).
![image](https://github.com/user-attachments/assets/11ca56c6-8b60-4520-8893-bd78418e989d)


# Monitoring with Prometheus & Grafana

Your cluster is instrumented with Prometheus, scraping metrics from Kubernetes nodes, Pods, and your application endpoints (e.g., /metrics).

Grafana sits on top of Prometheus, giving you dashboards and alert rules—so you can watch CPU/memory, request latency, error rates, and fire alerts when thresholds are breached.
![image](https://github.com/user-attachments/assets/1d1ad790-1f78-4e9a-acb0-cdbad457d38a)


# Putting It All Together
Code changes flow into your CI/CD pipeline, get tested (static, dependency, container), and push artifacts to a registry.

Terraform ensures your cloud infra is up and your EKS cluster exists.

Kubernetes deploys your front end, API, and database, pulling the hardened images.

Users connect via a Load Balancer and Public DNS, and your whole system is monitored by Prometheus + Grafana for health and performance.


