# K8s Deploy Tool — Kubernetes CI/CD Platform

## K8s Deploy Tool｜Kubernetes CI/CD 管理平台

A full-stack Kubernetes CI/CD platform designed to standardize artifact management, template versioning, multi-environment deployment, and Kubernetes delivery workflows.

一套企業級 Kubernetes CI/CD 管理平台，整合 **Artifact、Template、Project Configuration、Deployment Package** 與 **Kubernetes Deployment**，提供版本化、可追蹤且支援多環境的部署流程。

> **Repository Notice｜Repository 說明**
>
> This repository focuses on software architecture, engineering decisions, and implementation experience.
>
> 本 Repository 著重於系統架構、工程設計與開發經驗分享，已移除所有公司內部敏感資訊（如 API Endpoint、Credential、Registry Host、Storage Path、Namespace 等）。

---

# Documentation｜詳細文件

- [Frontend Development](./Frontend.md)  
  前端架構、Authentication、Routing、API Data Flow、多環境設定、非同步任務 UI、共用元件與互動設計。

- [Backend Development](./Backend.md)  
  後端分層架構、Domain Module、Deployment Package Generation、Artifact Push Task、Registry/Kubernetes 整合、Storage 與一致性設計。

---

# Table of Contents｜目錄

- Project Overview｜專案簡介
- Architecture Summary｜架構摘要
- Technology Stack｜技術棧
- Documentation｜文件導覽
- Engineering Principles｜工程設計理念

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

The platform follows a version-based deployment workflow.

Artifacts and templates are managed centrally, combined with project configurations, rendered into deployment packages, and finally deployed to Kubernetes.

平台採用版本化部署流程，所有 Artifact 與 Template 經由平台管理後，搭配 Project Configuration 產生 Deployment Package，最後完成 Docker Build、Helm Deployment 與 Kubernetes 部署。

---

# Technology Stack｜技術棧

## Frontend

- React
- TypeScript
- Vite
- React Router
- Axios
- OAuth 2.0 / OpenID Connect

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
- OCI-compatible Registry
- OpenTelemetry
- Grafana
- Prometheus
- Loki
- Tempo

---

# Key Engineering Decisions｜主要設計決策

- Artifact 與 Template 採用 Version-based Model，而不是直接覆蓋內容。
- Deployment Configuration 依 DEV、UAT、PROD 隔離。
- 不同 Runtime 使用對應的 Dockerfile Template 與 Validation Rule。
- 平台內部 Storage Layout 與 Runtime Deployment Package 分離。
- Metadata 儲存在 Database，Binary Content 交由 Storage Service 管理。
- Docker Base Image 從平台管理的 Artifact Version 選擇，避免未受控 Image Reference。
- 長時間 Registry 操作使用 Task State 與 Persistent Log 管理。

