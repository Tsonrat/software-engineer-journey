
# K8s Deploy Tool — Kubernetes CI/CD Platform

一套自建的 Kubernetes CI/CD 部署平台，提供 **Artifact Management、Template Management、Deployment Package Generation、Kubernetes Deployment** 等功能，將部署流程模組化、版本化，並支援多環境部署。

---

#  Project Overview｜專案簡介

本專案為公司內部使用的 Kubernetes CI/CD 平台，目標是讓不同專案、不同 Runtime 的服務，都能透過統一的平台完成 Docker Build、Helm Deploy、版本管理與部署流程。
後端主要使用 **Java、Spring Boot、MySQL** 建立平台核心服務，涵蓋：

- Artifact Management
- Template Management
- Project Management
- Deployment Engine
- Storage Management

平台設計重點放在：

- **Version-based Resource Management（版本化資源管理）**：所有部署資源皆採用版本管理，確保部署可追蹤、可回溯。
- **Environment Isolation（環境隔離）**：DEV、UAT、PROD 擁有獨立的部署設定，避免不同環境互相影響。
- **Deployment Package Generation（部署包產生）**：自動整合 Template、Config 與 Artifact，產生標準化部署包。
- **Storage Abstraction（儲存層抽象化）**：將檔案儲存與 Metadata 管理解耦，提高維護性與擴充性。
- **Kubernetes Integration（Kubernetes 整合）**：整合 Kubernetes 與 Helm，提供完整的部署與管理流程。

---

#  Architecture Overview｜整體架構

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

#  Core Modules｜核心模組

## Artifact Management

負責管理所有可部署資源，包括：

- Docker Image
- Helm Chart
- Dockerfile Template
- Shell Template

所有 Artifact 採用 **Version-based Model** 管理，而不是直接覆蓋資料，確保每一次部署都可以追蹤實際使用的版本。
Artifact 將會上傳至 Harbor 作為實際 digest 管理。

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

#  Technology Stack｜技術棧

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

#  Deployment Flow｜部署流程

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

#  Storage Design｜儲存架構

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

#  Key Features｜核心特色

- **Version-based Artifact Management（版本化 Artifact 管理）**：支援 Artifact 多版本管理、Current Version 切換與版本追蹤。
- **Multi-environment Deployment Configuration（多環境部署設定）**：DEV、UAT、PROD 各自擁有獨立的部署設定與資源配置。
- **Runtime-aware Dockerfile Template（依 Runtime 的 Dockerfile 模板）**：根據不同 Runtime（如 Spring Boot、WildFly）提供對應的 Dockerfile Template 與驗證機制。
- **Helm Template Rendering（Helm 模板渲染）**：依照 Project 設定與 Values 自動產生 Helm 部署內容。
- **Deployment Package Generation（部署包產生）**：自動整合 Template、Config、Artifact，建立標準化 Deployment Package。
- **Deployment History & Audit Logging（部署歷程與稽核紀錄）**：完整記錄部署結果、執行時間、錯誤訊息與 Metadata，方便追蹤與除錯。
- **OCI Registry Integration（OCI Registry 整合）**：整合 OCI Registry，統一管理 Artifact 的儲存、版本與部署來源。
- **Harbor Image Management（Harbor 映像管理）**：支援 Harbor Registry 的 Image 管理、同步與版本維護。
- **Kubernetes Workload Scanning（Kubernetes Workload 掃描）**：掃描 Kubernetes Cluster 中的 Workload，分析 Image 使用狀況與版本資訊。
- **Storage Abstraction Layer（儲存層抽象化）**：統一管理 Template、Config、Deployment Package 等檔案，並分離 Metadata 與實體檔案。

---

#  System Design Decisions｜系統設計決策

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

#  Engineering Highlights｜技術亮點

- **Designed a version-based Artifact Management System（設計版本化 Artifact 管理機制）**：支援多版本管理、Current Version 切換與完整的版本追蹤。
- **Implemented Environment Isolation for DEV / UAT / PROD（實作多環境隔離架構）**：讓不同環境擁有獨立的部署設定，避免跨環境互相影響。
- **Built a Deployment Package Generator（建立部署包產生機制）**：自動整合 Template、Config 與 Artifact，產生標準化 Deployment Package。
- **Designed a unified Storage Abstraction Layer（設計統一的儲存層抽象化）**：分離 Metadata 與實體檔案，提升系統維護性與擴充性。
- **Integrated Harbor Registry and OCI Registry（整合 Harbor 與 OCI Registry）**：統一管理 Artifact 儲存、版本控制與部署來源。
- **Implemented Kubernetes Workload Image Scanning（實作 Kubernetes Workload Image 掃描）**：分析 Cluster 中的 Image 使用情況，協助版本管理與資源追蹤。
- **Applied Modular Architecture for Backend Services（採用模組化後端架構）**：依照 Domain 拆分各模組，降低耦合度並提升系統可維護性與擴充能力。

---

本專案不只是一般的 CRUD 系統，而是以 Kubernetes CI/CD 為核心打造的平台型專案。
從 Artifact 管理、Template 系統、Deployment Package、Storage Layer，到多環境部署與 Kubernetes 整合，皆以模組化與可擴充性為設計核心。
透過版本化管理（Version-based Management）、環境隔離（Environment Isolation）與統一的部署流程（Deployment Workflow），讓平台能支援不同 Runtime、不同專案與不同部署需求，同時維持良好的可維護性與可追蹤性。


