# ☸️ Enterprise Kubernetes CI/CD Platform

> **An internal enterprise Kubernetes CI/CD platform built with Java, Spring Boot, React, TypeScript, Docker, Kubernetes, Helm, Harbor, and Keycloak.**
> **一套以 Java、Spring Boot、React、TypeScript、Docker、Kubernetes、Helm、Harbor 與 Keycloak 打造的企業級 Kubernetes CI/CD 平台。**

---

# 📖 Overview｜專案介紹

This project is an internal enterprise Kubernetes CI/CD platform designed to streamline application deployment through reusable deployment templates, environment-specific configurations, centralized artifact management, and deployment automation.

The platform aims to improve deployment consistency, reduce manual operations, and provide a scalable and maintainable deployment workflow for enterprise applications.

本專案是一套企業內部 Kubernetes CI/CD 平台，透過可重複使用的部署模板、多環境部署設定、Artifact 管理與部署流程自動化，協助開發人員快速且一致地完成 Kubernetes 應用部署。

平台的目標是提升部署一致性、降低人工操作成本，並建立更具擴充性與維護性的企業部署流程。

---

# ✨ Key Features｜主要功能

## 🚀 Deployment

* Multi-environment Deployment (DEV / UAT / PROD)
* Deployment Package Generation
* Deployment Preview (YAML / Dockerfile / Shell)

支援 DEV / UAT / PROD 多環境部署、自動產生 Deployment Package，並提供部署前預覽功能。

---

## 📦 Artifact Management

* Docker Images
* Helm Charts
* Dockerfile Templates
* Shell Templates
* Artifact Version Management

集中管理部署相關 Artifact，並支援版本管理。

---

## ⚙️ Configuration Management

* ConfigMap Management
* Secret Management
* Docker Base Image Selection
* Environment-specific Configuration

提供 ConfigMap、Secret、多環境設定與 Docker Base Image 管理。

---

## 🔐 Authentication & Authorization

* Keycloak Authentication
* Role-based Access Control (RBAC)

透過 Keycloak 提供登入驗證與權限管理。

---

# 🛠 Tech Stack｜使用技術

## Backend

* Java
* Spring Boot
* Spring Data JPA
* MySQL

## Frontend

* React
* TypeScript
* Vite

## DevOps

* Docker
* Kubernetes
* Helm
* Harbor

## Security

* Keycloak

---

# 📚 Documentation｜專案文件

| Document            | Description                               |
| ------------------- | ----------------------------------------- |
| Architecture.md     | System Architecture / 系統架構                |
| Frontend-Journey.md | My React Learning Journey / React 成長紀錄    |
| Backend-Journey.md  | Backend Development Journey / 後端開發歷程      |
| Decisions.md        | Architecture & Design Decisions / 架構與設計決策 |

---

# 🌱 Learning Journey｜學習歷程

This project represents an important milestone in my software engineering journey.

Before starting this project, my primary experience was focused on backend development using Java and Spring Boot.

Throughout the development process, I gradually expanded into frontend development with React and TypeScript while gaining hands-on experience with Docker, Kubernetes, Helm, Harbor, and Keycloak.

Instead of learning these technologies through small demo projects, I applied them directly to a real enterprise platform by implementing deployment configuration, template management, artifact management, deployment preview, reusable UI components, and multi-environment deployment workflows.

這個專案不只是 Kubernetes 部署平台，也是我軟體工程學習旅程中的重要里程碑。

在開始這個專案之前，我的主要開發經驗集中在 Java 與 Spring Boot 後端。

隨著平台逐步完成，我開始接觸 React、TypeScript，並進一步學習 Docker、Kubernetes、Helm、Harbor 以及 Keycloak，逐漸建立完整的全端開發能力。

我並非透過 Todo List 或教學範例學習 React，而是在企業級平台開發中，完成部署設定、模板管理、Artifact 管理、Deployment Preview、多環境部署，以及可重複使用元件等功能，逐步理解完整的 Full Stack 開發流程。

---

# 👨‍💻 Personal Milestones｜個人成長

* ✅ First enterprise-level React application
* ✅ First full-stack enterprise platform
* ✅ First experience building a production-oriented Kubernetes platform
* ✅ Learned React and TypeScript through real-world development
* ✅ Gained hands-on experience with Docker, Kubernetes, Helm, Harbor, and Keycloak
* ✅ Improved software architecture, system design, and full-stack development skills

