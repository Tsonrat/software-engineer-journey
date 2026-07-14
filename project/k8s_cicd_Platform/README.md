# K8s Deploy Tool — Kubernetes CI/CD Platform

一套自建的 Kubernetes CI/CD 管理平台，將 **Artifact、Template、Project Configuration、Deployment Package 與 Kubernetes Deployment** 整合成版本化、可追蹤且支援多環境的部署流程。

> 本 Repository 以專案架構、技術決策與開發經驗為主。公開文件不包含公司內部 Credential、實際 API Endpoint、資料表欄位、Registry Host 或儲存路徑。

---

# Project Overview｜專案簡介

平台目標是讓不同 Project 與 Runtime 的服務，透過統一介面完成：

- Artifact 與版本管理
- Helm、Dockerfile、Shell Template 管理
- DEV、UAT、PROD 環境設定
- Docker Base Image 與 OCI Artifact 管理
- Deployment Asset Preview
- Deployment Package Generation
- Docker Build 與 Helm Deploy Workflow
- Artifact Push Task 與操作紀錄
- Kubernetes Workload Image Usage Tracking

整體設計以 **Version Management、Environment Isolation、Deployment Reproducibility、Storage Abstraction** 為核心。

---

# Architecture Summary｜架構摘要

```text
Registry and Managed Artifacts
            │
            ▼
Artifact / Template Versions
            │
            ▼
Project Configuration
            │
            ▼
DEV / UAT / PROD
            │
            ▼
Deployment Asset Rendering
            │
            ▼
Deployment Package
            │
            ▼
Docker / Helm / Kubernetes
            │
            ▼
Task and Deployment History
```

前端將複雜的 Kubernetes、Harbor、Template 與 Environment Configuration 轉換成可操作的管理流程；後端則負責版本解析、驗證、外部系統整合、部署資產渲染與歷史追蹤。

---

# Technology Stack｜技術棧

## Frontend

- React
- TypeScript
- Vite
- React Router
- Axios
- Keycloak / OAuth 2.0 / OpenID Connect

## Backend

- Java
- Spring Boot
- Spring MVC
- Spring Data JPA
- Spring Security
- MySQL

## Platform

- Docker
- Kubernetes
- Helm
- Harbor Registry
- OCI Registry
- File Storage

---

# Documentation｜詳細文件

- [Frontend Development](./Frontend.md)  
  前端架構、Authentication、Routing、API Data Flow、多環境設定、非同步任務 UI、共用元件與互動設計。

- [Backend Development](./Backend.md)  
  後端分層架構、Domain Module、Deployment Package Generation、Artifact Push Task、Registry/Kubernetes 整合、Storage 與一致性設計。

---

# Key Engineering Decisions｜主要設計決策

- Artifact 與 Template 採用 Version-based Model，而不是直接覆蓋內容。
- Deployment Configuration 依 DEV、UAT、PROD 隔離。
- 不同 Runtime 使用對應的 Dockerfile Template 與 Validation Rule。
- 平台內部 Storage Layout 與 Runtime Deployment Package 分離。
- Metadata 儲存在 Database，Binary Content 交由 Storage Service 管理。
- Docker Base Image 從平台管理的 Artifact Version 選擇，避免未受控 Image Reference。
- 長時間 Registry 操作使用 Task State 與 Persistent Log 管理。

