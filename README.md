# MedPharma Frontend

## Production-Style React Application | Docker · Nginx · Kubernetes · AWS EKS · GitOps

> **Modern React frontend engineered for a cloud-native healthcare platform and delivered through an automated, security-focused CI/CD and GitOps pipeline on AWS EKS.**

The MedPharma frontend is the user-facing application layer of the MedPharma platform.

Rather than treating the frontend as a standalone React application, this project demonstrates how a modern web application can be:

* developed with React
* tested automatically
* statically analyzed
* containerized with Docker
* served through Nginx
* security-scanned
* cryptographically signed
* published to Amazon ECR
* promoted through GitOps
* deployed to Amazon EKS through ArgoCD

The result is a **reproducible application delivery workflow from source code to Kubernetes**.

---

# ⚡ At a Glance

| Capability           | Implementation               |
| -------------------- | ---------------------------- |
| Frontend             | React 18                     |
| Build                | Create React App / npm       |
| Web Server           | Nginx                        |
| Containerization     | Docker multi-stage build     |
| Runtime              | Amazon EKS                   |
| Container Registry   | Amazon ECR                   |
| Deployment           | ArgoCD / GitOps              |
| CI/CD                | GitHub Actions               |
| Testing              | Jest + React Testing Library |
| Linting              | ESLint                       |
| Formatting           | Prettier                     |
| SAST                 | CodeQL + Semgrep             |
| Dependency Security  | npm audit                    |
| Container Security   | Trivy                        |
| Image Signing        | Cosign                       |
| Cloud Authentication | GitHub OIDC                  |
| Configuration        | Kubernetes ConfigMap         |
| API Integration      | Axios                        |

---

# 🏗️ Application Architecture

```text
                              INTERNET
                                  │
                                  ▼
                         ┌─────────────────┐
                         │ AWS Load        │
                         │ Balancer        │
                         └────────┬────────┘
                                  │
                                  ▼
                         ┌─────────────────┐
                         │ NGINX Ingress   │
                         │   Controller    │
                         └────────┬────────┘
                                  │
                                  ▼
                       ┌────────────────────┐
                       │   MedPharma UI     │
                       │     React 18       │
                       │       + Nginx      │
                       └─────────┬──────────┘
                                 │
                        /api/*    │
                                 ▼
                       ┌────────────────────┐
                       │    API Gateway     │
                       │     Spring Boot    │
                       └─────────┬──────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
              ▼                  ▼                  ▼
        Auth Service       Drug Catalog       Inventory
              │                  │                  │
              └──────────────────┼──────────────────┘
                                 │
                                 ▼
                         Amazon RDS
                           PostgreSQL
```

The frontend is intentionally separated from backend services.

The browser communicates with the application through the frontend, while `/api/*` traffic is routed to the backend API gateway.

This allows the frontend to remain independently deployable while still participating in the same platform architecture.

---

# 🔄 Source-to-Production Delivery

The application follows a complete automated delivery pipeline:

```text
Developer
    │
    ▼
GitHub
    │
    ▼
GitHub Actions
    │
    ├── ESLint
    ├── Jest
    ├── CodeQL
    ├── Semgrep
    ├── npm audit
    │
    ▼
Docker Build
    │
    ▼
Trivy Image Scan
    │
    ▼
Amazon ECR
    │
    ▼
Cosign Keyless Signing
    │
    ▼
GitOps Repository
    │
    ▼
ArgoCD
    │
    ▼
Amazon EKS
    │
    ▼
Nginx
    │
    ▼
MedPharma React Application
```

The important design principle is:

> **Code changes create immutable artifacts, while GitOps controls what version is deployed.**

---

# 🧩 Frontend Architecture

The application follows a component-oriented React architecture.

```text
src/
│
├── components/
│   └── Reusable UI
│
├── pages/
│   └── Route-level Views
│
├── services/
│   └── API / Axios Clients
│
├── App.jsx
│
└── ...
```

### Components

Reusable presentation and functional UI elements.

### Pages

Route-level application views.

### Services

Centralized API communication and backend integration.

This separation keeps UI composition, routing, and API communication from becoming tightly coupled.

---

# 🐳 Production Container Architecture

The frontend uses a **multi-stage Docker build**.

```text
┌─────────────────────────────────┐
│        BUILD STAGE              │
│                                 │
│        node:20-alpine           │
│                                 │
│  npm install                    │
│       │                         │
│       ▼                         │
│  npm run build                  │
│       │                         │
│       ▼                         │
│     build/                      │
└───────────────┬─────────────────┘
                │
                │ Copy static assets
                ▼
┌─────────────────────────────────┐
│        RUNTIME STAGE            │
│                                 │
│        nginx:alpine             │
│                                 │
│      Static React Assets        │
│              │                  │
│              ▼                  │
│            Nginx                │
└─────────────────────────────────┘
```

This separates the build environment from the runtime environment.

The production container contains the compiled frontend and Nginx rather than the full Node.js build environment.

### Benefits

* Smaller runtime footprint
* Reduced attack surface
* Faster container startup
* Clear build/runtime separation
* No Node.js runtime required to serve compiled assets

---

# 🌐 Nginx Runtime

Nginx acts as the production web server for the compiled React application.

```text
Browser
   │
   ▼
Nginx
   │
   ├── React static assets
   │
   └── /api/*
          │
          ▼
      API Gateway
```

Nginx therefore handles both:

1. Static frontend delivery
2. API request routing

The frontend container does not need to run a Node.js development server in production.

---

# 🔀 API Routing

API traffic is separated from frontend traffic.

```text
GET /
    │
    ▼
React Application

GET /api/*
    │
    ▼
Nginx
    │
    ▼
API Gateway
    │
    ▼
Backend Microservices
```

This keeps the browser-facing architecture simple while allowing the backend to evolve independently.

---

# ☸️ Kubernetes Deployment

The frontend runs as a Kubernetes workload on Amazon EKS.

```text
Amazon EKS
│
└── MedPharma Namespace
    │
    ├── Deployment
    │    └── pharma-ui Pods
    │
    ├── Service
    │    └── pharma-ui
    │
    ├── ConfigMap
    │    └── Runtime configuration
    │
    └── Ingress
         └── /
```

The Kubernetes deployment configuration is managed through the MedPharma GitOps repository.

This keeps application source code separate from deployment state.

---

# 🔐 Runtime Configuration

Environment-specific configuration is injected through Kubernetes rather than hard-coded into the application source.

| Variable        | Purpose                 | Example               |
| --------------- | ----------------------- | --------------------- |
| `API_BASE_URL`  | Backend API base path   | `/api`                |
| `AUTH_BASE_URL` | Authentication API path | `/api/auth`           |
| `ENV`           | Runtime environment     | `dev` / `qa` / `prod` |

Configuration is managed through the GitOps environment values and Kubernetes ConfigMap.

```text
GitOps Values
      │
      ▼
Helm
      │
      ▼
Kubernetes ConfigMap
      │
      ▼
Frontend Pod
```

This allows the same container artifact to move through environments without rebuilding the application for each environment.

---

# 🛡️ Application Security Pipeline

Security is integrated directly into the CI pipeline.

```text
Source Code
     │
     ▼
┌───────────────────┐
│ ESLint            │
│ Jest               │
│ CodeQL             │
│ Semgrep            │
│ npm audit          │
└─────────┬─────────┘
          │
          ▼
      Docker Build
          │
          ▼
     Trivy Scan
          │
          ▼
       Amazon ECR
          │
          ▼
    Cosign Signing
          │
          ▼
      GitOps / EKS
```

### Static Analysis

**CodeQL**

Used for source-code security analysis.

**Semgrep**

Used for additional static analysis and security rules.

### Dependency Security

`npm audit` identifies vulnerable npm dependencies.

### Container Security

Trivy scans the resulting container image before deployment.

---

# 🔏 Container Image Signing

Container images are signed using **Cosign keyless signing**.

```text
GitHub Actions
      │
      ▼
GitHub OIDC
      │
      ▼
Fulcio
      │
      ▼
Signed Container Image
      │
      ▼
Rekor Transparency Log
```

The goal is to establish provenance between the CI workflow and the resulting container artifact.

Instead of treating a container image as an opaque binary, the pipeline can associate the artifact with its CI identity and source workflow.

---

# 🔑 AWS Authentication

GitHub Actions authenticates to AWS using **GitHub OIDC** rather than storing long-lived AWS access keys.

```text
GitHub Actions
      │
      ▼
GitHub OIDC Token
      │
      ▼
AWS IAM
      │
      ▼
Temporary Credentials
      │
      ▼
Amazon ECR
```

This reduces credential-management overhead and avoids maintaining permanent AWS access keys inside GitHub secrets.

---

# 🏷️ Immutable Image Versioning

Container images are tagged using the source commit:

```text
sha-<7-character-commit>
```

Example:

```text
pharma-ui:sha-a1b2c3d
```

This creates a direct relationship between:

```text
Git Commit
    ↓
Docker Image
    ↓
ECR Artifact
    ↓
GitOps Deployment
    ↓
Running Kubernetes Workload
```

That traceability is important when debugging deployments or determining exactly what code is running in an environment.

---

# 🚀 CI/CD Pipeline

The primary workflow runs on changes to the configured application branches.

```text
1. Checkout
      │
      ▼
2. Install Dependencies
      │
      ▼
3. ESLint
      │
      ▼
4. Jest + Coverage
      │
      ▼
5. CodeQL
      │
      ▼
6. Semgrep
      │
      ▼
7. npm audit
      │
      ▼
8. Docker Build
      │
      ▼
9. Trivy Scan
      │
      ▼
10. Push to ECR
      │
      ▼
11. Cosign Sign
      │
      ▼
12. Update GitOps
      │
      ▼
13. ArgoCD Sync
      │
      ▼
14. DEV
      │
      ▼
15. QA Promotion PR
```

The pipeline intentionally places security checks **before deployment**.

---

# 🔁 GitOps Promotion

After a successful build, the deployment process updates the GitOps configuration with the new image version.

```text
Frontend Repository
        │
        ▼
GitHub Actions
        │
        ▼
Amazon ECR
        │
        ▼
Update GitOps Image Tag
        │
        ▼
GitOps Repository
        │
        ▼
ArgoCD
        │
        ▼
EKS
```

The GitOps repository becomes the declarative record of which application version should run in Kubernetes.

---

# 🧪 Testing Strategy

The frontend uses Jest and React Testing Library.

### Interactive Development

```bash
npm test
```

### CI Test Run

```bash
npm test -- --watchAll=false
```

### Coverage

```bash
npm test -- --coverage
```

Testing is executed before the application is packaged into its production container.

---

# 🧹 Code Quality

The repository uses automated code-quality tooling.

### ESLint

```bash
npm run lint
```

### Prettier

Used for consistent source formatting.

### Testing

Jest + React Testing Library provide automated frontend validation.

The objective is to catch defects and maintainability issues before the application reaches the deployment pipeline.

---

# 💻 Local Development

## Prerequisites

* Node.js 20+
* npm 9+

## Install

```bash
npm install
```

## Start Development Server

```bash
npm start
```

Application:

```text
http://localhost:3000
```

## Run Tests

```bash
npm test
```

## Run Tests Once

```bash
npm test -- --watchAll=false
```

## Generate Coverage

```bash
npm test -- --coverage
```

## Lint

```bash
npm run lint
```

## Production Build

```bash
npm run build
```

---

# 🐳 Run with Docker

## Build

```bash
docker build -t pharma-ui:local .
```

## Run

```bash
docker run -p 80:80 pharma-ui:local
```

Application:

```text
http://localhost
```

---

# 📂 Repository Structure

```text
med-pharma-frontend/
│
├── public/
│   └── Static assets
│
├── src/
│   ├── components/
│   │   └── Reusable UI components
│   │
│   ├── pages/
│   │   └── Route-level pages
│   │
│   ├── services/
│   │   └── API / Axios clients
│   │
│   └── App.jsx
│
├── nginx.conf
│
├── Dockerfile
│
├── sonar-project.properties
│
└── .github/
    └── workflows/
        ├── ci.yml
        └── _reusable-update-gitops.yml
```

---

# 🧠 Engineering Decisions

## Why React?

React provides a component-based architecture suitable for building a modular application UI.

## Why Nginx?

Nginx provides a lightweight production runtime for serving the compiled React application.

## Why Multi-Stage Docker?

The Node.js environment is required to build the application but is unnecessary at runtime.

## Why Kubernetes?

EKS provides a standardized orchestration platform shared with the MedPharma backend services.

## Why GitOps?

Deployment configuration becomes version-controlled desired state rather than a series of manual cluster changes.

## Why Immutable Image Tags?

A deployment should identify exactly which source revision produced the running artifact.

## Why OIDC?

CI/CD should obtain short-lived AWS credentials rather than relying on permanent access keys.

## Why Cosign?

Container signing adds an additional layer of artifact provenance and supply-chain integrity.

---

# 📊 Delivery Model

The project separates three concerns:

```text
┌─────────────────────────────────────────┐
│           APPLICATION CODE              │
│                                         │
│ React · Components · Pages · API       │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│           CONTAINER ARTIFACT             │
│                                         │
│ Docker · ECR · Trivy · Cosign           │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│           DEPLOYMENT STATE               │
│                                         │
│ GitOps · Helm · ArgoCD · Kubernetes     │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│              RUNTIME                    │
│                                         │
│ AWS EKS · Nginx · MedPharma             │
└─────────────────────────────────────────┘
```

This separation makes the application easier to release, promote, troubleshoot, and roll back.

---

# 🔎 Deployment Traceability

A production-style deployment should answer:

> **What code is running?**

MedPharma maintains a traceable path:

```text
Commit SHA
    │
    ▼
GitHub Actions Run
    │
    ▼
Docker Image
    │
    ▼
ECR
    │
    ▼
Cosign Signature
    │
    ▼
GitOps Commit
    │
    ▼
ArgoCD Application
    │
    ▼
Kubernetes Pod
```

This creates an auditable relationship between source code and the deployed workload.

---

# 🌎 MedPharma Platform

The frontend is one component of a larger cloud-native platform.

```text
                         MEDPHARMA
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ▼                 ▼                 ▼
      Frontend           Backend          Infrastructure
          │                 │                 │
       React 18       Spring Boot          Terraform
          │                 │                 │
          ▼                 ▼                 ▼
       Docker            Docker              AWS
          │                 │                 │
          ▼                 ▼                 ▼
          ECR ◄─────────────┘                 │
           │                                  │
           ▼                                  ▼
        GitOps ─────────────────────────────► EKS
           │                                  │
           ▼                                  ▼
         ArgoCD                              RDS
                                              │
                                              ▼
                                          PostgreSQL
```

### Companion Repositories

* **[CloudTechs-ai/med-infra](https://github.com/CloudTechs-ai/med-infra)** — AWS infrastructure and Terraform
* **[CloudTechs-ai/med-pharma-backend](https://github.com/CloudTechs-ai/med-pharma-backend)** — Spring Boot microservices
* **[CloudTechs-ai/gitops](https://github.com/CloudTechs-ai/gitops)** — Kubernetes, Helm, and ArgoCD deployment configuration

---

# 🏆 What This Project Demonstrates

This repository demonstrates practical experience across the modern cloud application delivery lifecycle.

### Frontend Engineering

* React 18
* Component architecture
* API integration
* Automated testing
* Linting and formatting

### Container Engineering

* Multi-stage Docker builds
* Minimal production runtime
* Nginx
* Immutable image versioning

### Cloud Engineering

* AWS
* Amazon ECR
* Amazon EKS
* IAM
* OIDC

### Kubernetes

* Deployments
* Services
* ConfigMaps
* Ingress
* Helm
* ArgoCD

### DevSecOps

* CodeQL
* Semgrep
* npm audit
* Trivy
* Cosign
* GitHub OIDC

### Delivery Engineering

* GitHub Actions
* Automated artifact publishing
* GitOps
* Environment promotion
* Deployment traceability

---

# 🚀 Why This Is More Than a React Project

A conventional frontend project ends here:

```text
React
  ↓
npm build
  ↓
Website
```

MedPharma extends the engineering lifecycle:

```text
React
  ↓
Tests
  ↓
SAST
  ↓
Dependency Security
  ↓
Docker
  ↓
Container Security
  ↓
ECR
  ↓
Artifact Signing
  ↓
GitOps
  ↓
ArgoCD
  ↓
Kubernetes
  ↓
AWS EKS
  ↓
Production-Style Runtime
```

That is the central purpose of this project.

**The frontend is not simply built. It is engineered for delivery.**

---

# 🧭 Engineering Principles

| Principle                              | Implementation                  |
| -------------------------------------- | ------------------------------- |
| **Automate Everything Reasonable**     | GitHub Actions                  |
| **Test Before Deployment**             | Jest + React Testing Library    |
| **Shift Security Left**                | CodeQL + Semgrep + npm audit    |
| **Scan the Artifact**                  | Trivy                           |
| **Sign the Artifact**                  | Cosign                          |
| **Avoid Long-Lived Cloud Credentials** | GitHub OIDC                     |
| **Build Once, Promote the Artifact**   | Immutable ECR image             |
| **Use Git as Deployment State**        | GitOps                          |
| **Separate Build from Runtime**        | Multi-stage Docker              |
| **Keep Runtime Lightweight**           | Nginx                           |
| **Make Deployments Traceable**         | Commit SHA → ECR → GitOps → EKS |

---

# 🏁 End-to-End Platform

```text
                           DEVELOPER
                               │
                               ▼
                            GitHub
                               │
                               ▼
                       GitHub Actions
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
           Testing          Security          Build
              │                │                │
              └────────────────┼────────────────┘
                               │
                               ▼
                            Docker
                               │
                               ▼
                         Trivy Scan
                               │
                               ▼
                         Amazon ECR
                               │
                               ▼
                       Cosign Signature
                               │
                               ▼
                         GitOps Repo
                               │
                               ▼
                            ArgoCD
                               │
                               ▼
                           Amazon EKS
                               │
                               ▼
                            Nginx
                               │
                               ▼
                         React Frontend
                               │
                               │ /api/*
                               ▼
                         API Gateway
                               │
                               ▼
                      Backend Microservices
                               │
                               ▼
                         Amazon RDS
```

---

# 📌 Technology Summary

**React 18 · JavaScript · Axios · Jest · React Testing Library · ESLint · Prettier · Docker · Nginx · Kubernetes · Amazon EKS · Amazon ECR · GitHub Actions · GitHub OIDC · CodeQL · Semgrep · npm audit · Trivy · Cosign · Helm · ArgoCD · Terraform · AWS · PostgreSQL**

---

## Built as part of the CloudTechs.ai MedPharma Platform

**Cloud Engineering · DevSecOps · Kubernetes · AWS · Infrastructure as Code · GitOps**
