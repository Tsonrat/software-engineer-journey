
# K8s Deploy Tool — Kubernetes CI/CD Platform

一套自建的 Kubernetes CI/CD 部署平台，提供 **Artifact Management、Template Management、Deployment Package Generation、Kubernetes Deployment** 等功能，將部署流程模組化、版本化，並支援多環境部署。

---

# 🚀 Project Overview｜專案簡介

本專案為公司內部使用的 Kubernetes CI/CD 平台，目標是讓不同專案、不同 Runtime 的服務，都能透過統一的平台完成 Docker Build、Helm Deploy、版本管理與部署流程。

後端主要使用 **Java、Spring Boot、MySQL** 建立平台核心服務，涵蓋：

- Artifact Management
- Template Management
- Project Management
- Deployment Engine
- Storage Management

平台設計重點放在：

- Version-based Resource Management
- Environment Isolation
- Deployment Package Generation
- Storage Abstraction
- Kubernetes Integration

---

# 🏗 Architecture Overview｜整體架構

```text
Registry
   │
   ▼
Artifact Management
   │
   ▼
Artifact Version
   │
   ▼
Template Management
(Helm / Dockerfile / Shell)
   │
   ▼
Project Configuration
   │
   ▼
DEV / UAT / PROD
(Environment Isolation)
   │
   ▼
Deployment Package
   │
   ▼
Docker Build
   │
   ▼
Helm Deploy
   │
   ▼
Kubernetes
   │
   ▼
Deployment History
```

整個平台以 **Deployment Flow** 為核心，將 Artifact、Template、Project、Storage 與 Kubernetes 整合成一致的部署流程。

---

# ⚙ Core Modules｜核心模組

## Artifact Management

負責管理所有可部署資源，包括：

- Docker Image
- Helm Chart
- Dockerfile Template
- Shell Template

所有 Artifact 採用 **Version-based Model** 管理，而不是直接覆蓋資料，確保每一次部署都可以追蹤實際使用的版本。

---

## Template System

平台將 Deployment Template 分成三種類型：

- Helm Template
- Dockerfile Template
- Shell Template

不同 Runtime 可以共用相同部署流程，同時保留各自的 Build Logic 與 Validation Rules。

---

## Project Management

Project 作為 Deployment Unit。

每個 Project 可以設定：

- Deployment Template
- Config Files
- Docker Base Image
- Environment-specific Configuration

所有 Deployment Config 都與 Environment 綁定。

---

## Deployment Engine

Deployment Engine 負責：

- Template Rendering
- Deployment Package Generation
- Deployment Workflow
- Deployment History

平台不直接操作內部檔案，而是產生標準化 Deployment Package 進行部署。

---

## Storage System

Template、Config、Deployment Package 等檔案皆透過統一 Storage Layer 管理。

Storage Metadata 與 Binary Files 分離，降低 Database 負擔，也方便未來擴充不同 Storage Backend。

---

# 💻 Technology Stack｜技術棧

### Backend

- Java
- Spring Boot
- Spring MVC
- Spring Data JPA
- MySQL

### Platform

- Docker
- Kubernetes
- Helm
- Harbor Registry
- OCI Registry

### Infrastructure

- File Storage
- YAML
- Shell Script

---

# 🔄 Deployment Flow｜部署流程

```text
Artifact Upload
      │
      ▼
Artifact Version
      │
      ▼
Project Configuration
      │
      ▼
Template Rendering
      │
      ▼
Deployment Package
      │
      ▼
Docker Build
      │
      ▼
Helm Deploy
      │
      ▼
Kubernetes
      │
      ▼
Deployment Log
```

每一個 Deployment Step 都可追蹤、可驗證，也方便後續 Audit、Rollback 與擴充。

---

# 📦 Storage Design｜儲存架構

平台採用 **Metadata 與 Binary File 分離** 的 Storage Design。

```text
data/
└── {group}
    └── {project}
        ├── shared/
        │   └── config/
        │
        └── env/
            ├── DEV/
            ├── UAT/
            └── PROD/
```

部署時則重新組裝成標準 Deployment Package：

```text
deployment/
├── Dockerfile
├── run.sh
├── common.sh
├── values.yaml
├── config/
├── target/
└── deployment.json
```

---

# ⭐ Key Features｜核心特色

- Version-based Artifact Management
- Multi-environment Deployment Configuration
- Runtime-aware Dockerfile Template
- Helm Template Rendering
- Deployment Package Generation
- Deployment History & Audit Logging
- OCI Registry Integration
- Harbor Image Management
- Kubernetes Workload Scanning
- Storage Abstraction Layer

---

# 🧠 System Design Decisions｜系統設計決策

## Environment Isolation

Deployment Configuration 採用 **DEV / UAT / PROD** 完全隔離。

每個 Environment 擁有自己的 Deployment Config，避免不同環境互相影響。

---

## Version-based Artifact Model

Artifact 不直接覆蓋，而是建立新的 Version。

優點包括：

- Deployment Traceability
- Rollback Capability
- Version Comparison
- Audit History

---

## Runtime-specific Template Design

原本考慮使用 Generic Dockerfile Template。

但不同 Runtime 擁有不同 Build Process，因此最終改為 Runtime-specific Template，同時保留共用 Metadata 與 Upload Workflow。

---

## Deployment Package Separation

平台內部 Storage Structure 與 Runtime Deployment Structure 完全分離。

方便平台演進，同時保持 Deployment Format 穩定。

---

## Storage Abstraction

Storage Metadata 儲存在 MySQL。

Binary Files 則存放於 File System。

降低 Database 負擔，也方便未來替換 Storage Implementation。

---

## OCI Governance

Docker Base Image 採用平台管理的 Artifact Version 選擇，而不是手動輸入 Image Path。

確保所有 Deployment 都使用受控的 OCI Image。

---

# ✨ Engineering Highlights｜技術亮點

- Designed a version-based Artifact Management System.
- Implemented Environment Isolation for DEV / UAT / PROD.
- Built a Deployment Package Generator.
- Designed a unified Storage Abstraction Layer.
- Integrated Harbor Registry and OCI Registry.
- Implemented Kubernetes Workload Image Scanning.
- Applied Modular Architecture for backend services.

---

# 🚀 Future Improvements｜未來規劃

- GitOps Integration
- Role-Based Access Control (RBAC)
- Deployment Approval Workflow
- Slack / Microsoft Teams Notification
- Deployment Metrics Dashboard

---

# 📖 Summary｜專案總結

This project demonstrates backend system design beyond traditional CRUD applications, including **Artifact Version Management、Environment Isolation、Deployment Package Generation、Storage Abstraction、OCI Registry Integration** and **Kubernetes Deployment Workflow**.

重點並非單一 API，而是建立一套可擴充、可維護的 Kubernetes Deployment Platform。

