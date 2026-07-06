#  Frontend

> **React + TypeScript + Vite frontend for the Enterprise Kubernetes CI/CD Platform.**
> **Enterprise Kubernetes CI/CD Platform 的 React 前端實作。**

---

#  Overview｜概述

This frontend is built with **React**, **TypeScript**, and **Vite**, following a layered architecture that separates Pages, Domain Components, API, Authentication, Layout, Types, and Utilities.

Instead of organizing code only by UI components, the project is structured around business domains such as **Project**, **Artifact**, **Template**, **Registry**, and **Deployment**, making the codebase easier to maintain and extend.

本專案前端採用 **React**、**TypeScript** 與 **Vite** 開發。

除了共用元件之外，也依照 **Project**、**Artifact**、**Template**、**Registry**、**Deployment** 等業務模組進行劃分，降低模組耦合並提升程式碼的可維護性與擴充性。

---

#  Tech Stack｜使用技術

* React
* TypeScript
* Vite
* Axios
* React Router
* Keycloak Authentication

---

#  Project Structure｜專案結構

```text
src/
├── api/
├── auth/
├── components/
├── layout/
├── pages/
├── styles/
├── types/
└── utils/
```

---

#  Folder Responsibilities｜資料夾職責

| Folder        | Responsibility                     |
| ------------- | ---------------------------------- |
| `pages/`      | 完整頁面與使用者操作流程                       |
| `components/` | 可重複使用的 UI 與 Domain Components      |
| `api/`        | Spring Boot REST API 存取層           |
| `auth/`       | Keycloak 登入與角色權限管理                 |
| `layout/`     | 共用版面配置（MainLayout、TopBar、SideMenu） |
| `types/`      | TypeScript Domain Models 與 API 型別  |
| `utils/`      | 共用工具函式                             |
| `styles/`     | 跨頁面共用樣式                            |

---

#  Domain Modules｜主要功能模組

目前前端主要包含以下功能模組：

* Registry Management
* Artifact Management
* Template Management
* Project Management
* Deployment Management
* Authentication
* Image Management

各模組皆擁有自己的 Pages、Components、API 與 TypeScript 型別，降低不同模組間的耦合，讓功能可以獨立演進。

