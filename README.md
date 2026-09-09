# 🚀 Ultimate-GitOps

## 📌 Overview

`Ultimate-GitOps` is the **GitOps repository** for the Ultimate CI/CD project.

This repository contains the Kubernetes deployment manifests required to run the application in the Kubernetes cluster.

The main purpose of this repository is to maintain the **desired state of the application** separately from the application source code.

Instead of directly deploying Kubernetes resources from the CI pipeline, the CI pipeline updates the deployment configuration in this repository.

**Git becomes the source of truth for the deployment state.**

---

# 🏗️ Why a Separate GitOps Repository?

The application source-code repository and deployment configuration have different responsibilities.

### Application Repository

Contains:

```text
Application Code
Dockerfile
CI/CD Pipeline
Tests
Source Code
```

### GitOps Repository

Contains:

```text
Kubernetes Manifests
Deployment Configuration
Service Configuration
Application Image Version
```

This separation makes the deployment process easier to manage, track, and audit.

---

# 🔄 GitOps Deployment Flow

The complete deployment flow is:

```text
                Application Repository
                         │
                         │ Code Push
                         ▼
                ┌─────────────────┐
                │ Jenkins /       │
                │ GitHub Actions  │
                └────────┬────────┘
                         │
                         ▼
                 Build & Test
                         │
                         ▼
                    SonarQube
                         │
                         ▼
                  Docker Build
                         │
                         ▼
                    Docker Hub
                         │
                         │ New Image
                         ▼
              ┌─────────────────────┐
              │   Ultimate-GitOps   │
              │                     │
              │ deployment.yml      │
              │                     │
              │ image: app:5        │
              └──────────┬──────────┘
                         │
                         │ Git Commit
                         ▼
                    ┌─────────┐
                    │ Argo CD │
                    └────┬────┘
                         │
                  Continuously
                     watches
                         │
                         ▼
                  Kubernetes Cluster
                         │
                         ▼
                    Application
```

---

# 👀 Argo CD Watches This Repository

Argo CD is configured to monitor this Git repository.

For example:

```text
Ultimate-GitOps
│
└── manifests
    └── deployment.yml
```

Argo CD continuously compares:

```text
Git Repository
      VS
Kubernetes Cluster
```

The Git repository represents the **desired state**.

The Kubernetes cluster represents the **actual state**.

If they are different, Argo CD can synchronize the Kubernetes cluster with the state defined in Git.

---

# 📦 Deployment Manifest

The `deployment.yml` contains the Kubernetes Deployment configuration.

Example:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: ultimate-app

spec:
  replicas: 2

  selector:
    matchLabels:
      app: ultimate-app

  template:
    metadata:
      labels:
        app: ultimate-app

    spec:
      containers:
        - name: ultimate-app
          image: suryabhaskarvempala/ultimate-cicd:5
          ports:
            - containerPort: 3000
```

The important part for CI/CD is the Docker image:

```yaml
image: suryabhaskarvempala/ultimate-cicd:5
```

---

# 🔁 Automatic Image Version Update

When a new application version is created, the CI pipeline builds and pushes a new Docker image.

For example:

```text
Version 5
suryabhaskarvempala/ultimate-cicd:5
```

The CI pipeline then updates the GitOps manifest:

```yaml
image: suryabhaskarvempala/ultimate-cicd:5
```

to:

```yaml
image: suryabhaskarvempala/ultimate-cicd:6
```

The change is committed and pushed to this repository.

```text
CI Pipeline
     │
     ▼
Docker Image :6
     │
     ▼
Update deployment.yml
     │
     ▼
Git Commit
     │
     ▼
Ultimate-GitOps
```

---

# 🔄 Continuous Delivery with Argo CD

Once the GitOps repository changes:

```text
deployment.yml
      │
      │ image changed
      ▼
Argo CD detects difference
      │
      ▼
Argo CD synchronizes
      │
      ▼
Kubernetes Cluster
      │
      ▼
New Application Version
```

Therefore, the CI pipeline does not need to directly execute Kubernetes deployment commands.

Instead:

**CI changes Git.**

**Argo CD changes Kubernetes.**

This is the core GitOps model.

---

# 🎯 Source of Truth

The GitOps repository acts as the **source of truth for the desired Kubernetes state**.

For example, if Git contains:

```yaml
image: suryabhaskarvempala/ultimate-cicd:10
```

then the desired application version is:

```text
ultimate-cicd:10
```

Argo CD works to ensure that the Kubernetes cluster matches that desired state.

---

# 🧩 Repository Structure

```text
Ultimate-GitOps/
│
├── manifests/
│   ├── deployment.yml
│   └── service.yml
│
└── README.md
```

### `deployment.yml`

Defines:

* Application Deployment
* Number of replicas
* Container image
* Container port
* Pod configuration

### `service.yml`

Defines how the application is exposed inside the Kubernetes cluster.

### `README.md`

Documents the GitOps deployment architecture and workflow.

---

# 🔐 GitOps Benefits

This approach provides:

* **Version-controlled deployments**
* **Auditable configuration changes**
* **Clear deployment history**
* **Easy rollback using Git**
* **Separation of CI and CD**
* **Automated Kubernetes synchronization**
* **Single source of truth**
* **Reduced manual deployment operations**

---

# 🚀 Final Architecture

```text
                  ┌──────────────────────┐
                  │   Application Repo   │
                  │                      │
                  │ Java / Node / Code   │
                  └──────────┬───────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │ Jenkins / GitHub     │
                  │ Actions              │
                  └──────────┬───────────┘
                             │
                 ┌───────────┴───────────┐
                 ▼                       ▼
           SonarQube                 Docker Hub
           Code Quality             Docker Image
                                         │
                                         ▼
                              ┌──────────────────────┐
                              │   Ultimate-GitOps    │
                              │                      │
                              │ deployment.yml       │
                              │ service.yml          │
                              └──────────┬───────────┘
                                         │
                                         │ Argo CD watches
                                         ▼
                              ┌──────────────────────┐
                              │       Argo CD        │
                              └──────────┬───────────┘
                                         │
                                         │ Sync
                                         ▼
                              ┌──────────────────────┐
                              │ Kubernetes Cluster   │
                              │                      │
                              │ Pods                 │
                              │ Services             │
                              └──────────┬───────────┘
                                         │
                                         ▼
                                    Application
```

## 💡 In One Line

> **CI builds the application and updates the desired state in Git; Argo CD continuously watches that GitOps repository and synchronizes the Kubernetes cluster to match it.**
