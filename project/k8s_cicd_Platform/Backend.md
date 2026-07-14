# Backend Development｜後端開發

本文件記錄我在 **K8s Deploy Tool — Kubernetes CI/CD Platform** 中負責的後端架構、核心流程與系統設計。

後端以 **Java、Spring Boot 與 MySQL** 建立平台核心服務，整合 Harbor、OCI Registry、Kubernetes、Helm 與檔案儲存，將 Artifact、Template、Project Configuration 與 Deployment 組成可追蹤、可驗證且可擴充的部署流程。

本文聚焦於後端工程實作，不公開公司內部 API 路徑、資料表欄位、Credential、實際儲存位置或環境設定。

---

# Project Overview｜專案概述

後端主要負責：

- Artifact 與 Artifact Version 管理
- Helm、Dockerfile 與 Shell Template 管理
- Project 與多環境部署設定
- Deployment Asset Rendering
- Deployment Package Generation
- Artifact Push Task 與持久化操作紀錄
- Harbor 與 OCI Registry 整合
- Kubernetes Workload Image Scanning
- Storage Metadata 與 Binary Content 管理
- Deployment History 與 Audit Information

平台設計重點包括：

- **Version-based Resource Management**：部署資源以版本為單位管理，避免直接覆蓋造成追蹤資訊遺失。
- **Environment Isolation**：DEV、UAT、PROD 擁有獨立的設定與資源選擇。
- **Deployment Reproducibility**：部署時解析明確的 Template Version、Artifact Version 與 Environment Configuration。
- **Storage Abstraction**：資料庫保存可查詢的 Metadata，實體檔案交由 Storage Service 管理。
- **External System Encapsulation**：Registry、Kubernetes 與 Storage 操作封裝在獨立服務中。
- **Operational Traceability**：長時間操作與部署結果皆保留狀態、錯誤與歷史紀錄。

---

# Backend Architecture｜後端架構設計

## Layered Architecture｜分層架構

後端採用分層架構，將 HTTP 處理、商業邏輯、資料存取與外部系統整合分離。

```text
HTTP Request
    │
    ▼
Controller
    │
    ▼
Application / Domain Service
    ├─ Validation and State Transition
    ├─ Repository Coordination
    ├─ Registry Integration
    ├─ Kubernetes Integration
    └─ Storage Coordination
    │
    ▼
Repository / External Client / Storage Service
    │
    ▼
Database / Registry / Kubernetes / File Storage
```

### Controller Responsibilities

Controller 保持精簡，主要負責：

- 接收 HTTP Request
- 綁定 Path、Query 與 Request Body
- 觸發輸入格式驗證
- 呼叫對應 Service
- 回傳 Response DTO 與 HTTP Status

Controller 不處理跨模組流程、狀態轉換或檔案組裝等商業邏輯。

### Service Responsibilities

Service 是後端主要的流程控制層，負責：

- Business Validation
- Resource State Validation
- Transaction Boundary
- Version Resolution
- Environment-specific Configuration Resolution
- Cross-domain Coordination
- External System Integration
- Deployment Orchestration
- Error Translation

### Repository and Mapper Responsibilities

Repository 專注於資料查詢與持久化，不負責 Deployment Flow 或外部系統操作。

Mapper 負責 Entity、Domain Data 與 DTO 之間的轉換，避免將商業規則放入單純的資料轉換程式中。

---

## Domain-oriented Module Design｜領域導向模組設計

後端依照業務領域拆分模組，而不是將所有 Controller、Service 與 Repository 集中在同一層目錄中。

```text
Backend Domains
├─ Registry and OCI Governance
├─ Artifact and Version Management
├─ Template Management
├─ Project Configuration
├─ Deployment Package Generation
├─ Artifact Push Workflow
├─ Kubernetes Image Usage
├─ Storage Management
└─ Deployment History
```

每個模組封裝自己的 API Contract、Validation、Service、Repository 與資料模型，再透過明確的 Service Interface 與其他領域互動。

這種設計可以降低不同功能間的直接耦合，也讓 Artifact、Template、Project 與 Deployment 等領域能各自演進。

---

# Core Modules｜核心模組

## Artifact and Version Management

平台管理的資源包含：

- Docker Image
- Helm Chart
- Dockerfile Template
- Shell Template

Artifact 表示可管理的邏輯資源，Artifact Version 則表示實際可使用、可部署或可追蹤的版本。

建立新版本時不會直接覆蓋既有版本，因此每次 Deployment 都能保存當時解析出的版本資訊、Registry Reference 與 Digest Metadata。

主要能力包括：

- Artifact Lifecycle Management
- Version Creation and Selection
- Current Version Management
- Digest and Registry Metadata Tracking
- Base Image Eligibility
- Usage Relationship Tracking
- Version-level Delete Validation

---

## Template Management

Deployment Template 分為：

- Helm Template
- Dockerfile Template
- Shell Template

不同 Template 類型具有各自的 Upload Validation、Version Metadata 與 Rendering Requirement。

Dockerfile Template 依 Runtime 與 Package Type 套用不同驗證規則，避免使用單一 Generic Template 隱藏不同 Build Process 的差異。

Template Version 會作為獨立資源管理，使 Project 可以明確選擇部署時使用的版本，而不是自動依賴可能變動的最新內容。

---

## Project Configuration

Project 是平台中的 Deployment Unit，負責連結：

- Template Selection
- Template Version
- Managed Artifact
- Docker Base Image Version
- ConfigMap and Secret Resources
- Environment-specific Values
- Deployment Parameters

DEV、UAT、PROD 各自擁有獨立的 Selected Configuration。

共享檔案可以被不同環境選用，但實際儲存 Deployment Configuration 時，仍會記錄明確的目標環境，區分「資源可用範圍」與「實際部署環境」。

---

## Registry and OCI Governance

平台整合 Harbor 與 OCI-compatible Registry，統一管理 Image 與 Template Artifact 的來源、版本與目標位置。

主要能力包括：

- Registry Connectivity and Metadata Management
- OCI Manifest Resolution
- Digest Tracking
- Tag and Version Synchronization
- Multi-platform Image Resolution
- Controlled Target Reference Generation
- Managed Base Image Selection

平台不讓 Deployment 任意輸入未受控的 Base Image Path，而是從已管理且符合狀態要求的 Artifact Version 中選擇，以維持版本可追蹤性。

---

## Kubernetes Workload Image Scanning

後端能掃描 Kubernetes Cluster 中 Workload 實際使用的 Container Image，並與平台管理的 Artifact Version 建立比對關係。

掃描結果可用於：

- 確認 Artifact Version 是否仍被 Workload 使用
- 分析 Base Image Usage
- 找出可能過期的 Image
- 評估 Image Cleanup Candidate
- 協助版本升級與影響範圍分析

Cluster Connection 與 Credential 由外部設定注入，不寫入程式碼或專案文件。

---

# Request Processing Flow｜請求處理流程

一般 API Request 會依序經過 Authentication、Validation、Business Service 與 Persistence 或 External Integration。

```text
Client Request
    │
    ▼
Authentication and Authorization
    │
    ▼
Request Binding and Validation
    │
    ▼
Business Service
    ├─ Load Domain Data
    ├─ Validate Current State
    ├─ Execute Business Rule
    └─ Coordinate External Resources
    │
    ▼
Repository / Registry / Kubernetes / Storage
    │
    ▼
Persist Result and Audit Information
    │
    ▼
Response DTO
```

後端會先驗證 Request Format，再於 Service 層檢查資源是否存在、目前狀態是否允許操作，以及相關版本或依賴是否有效。

對外部系統的錯誤不直接洩漏底層例外，而會轉換成平台能理解的錯誤訊息與適當 HTTP Status。

---

# Deployment Package Generation｜部署包產生流程

Deployment Package Generation 是平台最核心的後端流程之一。

```text
Project and Environment
    │
    ▼
Resolve Selected Templates
    │
    ▼
Resolve Template Versions
    │
    ▼
Resolve Environment-specific Values
    │
    ▼
Resolve Config and Managed Artifacts
    │
    ▼
Validate Resource Compatibility
    │
    ▼
Render Deployment Assets
    │
    ▼
Assemble Deployment Package
    │
    ▼
Persist Package Metadata and History
    │
    ▼
Preview / Download / Deploy
```

## Resolution Phase

後端依照 Project 與目前 Environment 解析：

- Selected Template and Version
- Environment-specific Values
- ConfigMap and Secret Selection
- Managed Application Artifact
- Docker Base Image Version
- Deployment Parameters

所有內容都需要在產生部署包前完成狀態與相依性驗證。

## Rendering Phase

系統會依照不同 Template 類型建立對應的 Deployment Asset：

- Helm Values and Rendered Manifest
- Runtime-aware Dockerfile
- Deployment Shell Workflow
- Selected Config Resources
- Managed Application Artifact

Preview 與 Package Generation 共用相同的解析規則，避免預覽結果與實際產出使用不同邏輯。

## Packaging Phase

平台內部的 Storage Structure 與最終 Runtime Package Structure 分離。

Packaging Layer 負責將分散在 Template、Project File、Artifact 與 Environment Configuration 中的資料重新組裝成標準部署包。

Package Binary 與 Deployment Metadata 分開保存，使系統可以查詢歷史、重新下載，並追蹤每次產出所使用的版本與環境資訊。

---

# Artifact Push Task Workflow｜映像推送任務流程

Image Synchronization 或 Push 可能涉及 Manifest 解析、多平台映像處理與 Registry 傳輸，不適合綁定在單次同步 HTTP Request 中。

後端將此類操作建模為 Task Workflow：

```text
Create Task
    │
    ▼
Validate Source Artifact
    │
    ▼
Resolve Source and Target References
    │
    ▼
Resolve Selected Platforms
    │
    ▼
Execute Copy / Push
    │
    ▼
Update Task Status
    │
    ├─ Success → Persist Result Log
    └─ Failure → Persist Error and Retry Information
```

Task Lifecycle 包含：

- PENDING
- RUNNING
- SUCCESS
- FAILED

設計重點包括：

- 使用狀態機限制不同狀態可執行的操作
- 執行前再次驗證 Source 與 Target
- 保存開始時間、完成時間與錯誤資訊
- 對可恢復錯誤提供 Retry Flow
- 將即時 Task 與持久化 Push Log 分開管理
- 系統啟動時處理不合理的未完成狀態，避免任務永久卡住

這讓前端可以顯示即時操作狀態，同時保留長期 Audit 與 Troubleshooting Information。

---

# Storage Architecture｜儲存架構

平台採用 Metadata 與 Binary Content 分離的設計。

```text
Business Service
    │
    ├─ Metadata Repository → Database
    │
    └─ Storage Service → Binary Storage
```

## Metadata

資料庫保存：

- Resource Ownership
- Version Relationship
- Storage Reference
- Content Type and Size
- Status
- Created and Updated Information
- Deployment and Audit Relationship

## Binary Content

Storage Service 統一負責：

- Upload
- Download
- Preview
- Copy
- Delete
- Package Output
- Path and Naming Validation

Business Service 不直接依賴實際 File System Path，而是透過 Storage Reference 取得內容。

這使儲存實作未來可以從 Local File Storage 替換為 Object Storage，而不需要重寫主要業務流程。

---

# Validation and Error Handling｜驗證與錯誤處理

## Validation Strategy

驗證分為兩層：

- **Request Validation**：必要欄位、格式、Enum 與基本限制。
- **Business Validation**：資源狀態、版本相依、使用關係、重複資料與外部系統條件。

例如刪除版本前，系統會確認該版本是否仍被 Project Configuration、Deployment 或 Kubernetes Workload 使用，而不是只檢查 Artifact 是否曾被任何地方使用。

## Error Handling

後端透過統一例外處理機制，將不同模組的錯誤轉換成一致的 API Error Response。

常見錯誤類型包括：

- Resource Not Found
- Duplicate Resource
- Invalid State Transition
- Version Still in Use
- Invalid Template Package
- External Registry Failure
- Kubernetes Access Failure
- Storage Operation Failure
- Unauthorized or Forbidden Operation

前端可以依照 HTTP Status 與 Error Message 顯示穩定且可理解的操作回饋。

---

# Transaction and Consistency｜交易與一致性

修改型流程使用 Transaction 保持 Database State 一致，讀取型流程則在適合的情況下使用 Read-only Transaction。

對同時涉及 Database 與外部系統的長時間操作，不將整段流程綁定在單一資料庫 Transaction 中，而是透過 Task Status 與 Audit Log 記錄執行進度。

主要原則包括：

- 執行不可逆操作前先驗證目前狀態
- 先建立可追蹤的 Task 或 History Record
- 外部操作完成後再更新最終狀態
- 失敗時保存 Error Context，而不是只回傳一次性錯誤
- 刪除與更新時檢查 Version-level Usage
- 避免單一 Template 更新覆蓋其他 Environment 或 Template 設定

---

# Security and Authorization｜安全與權限

後端 API 透過 Spring Security 與 JWT 驗證保護。

主要設計包括：

- OAuth 2.0 Resource Server
- JWT Authentication
- Role-based Authorization
- Protected Management API
- Credential Configuration Injection
- Unauthorized and Forbidden Response Handling

Registry、Kubernetes 或其他外部系統 Credential 由執行環境提供，不寫死在 Source Code 中，也不記錄於公開文件。

---

# Technology Stack｜使用技術

## Backend

- Java
- Spring Boot
- Spring MVC
- Spring Data JPA
- Spring Security
- MySQL

## Platform Integration

- Docker
- Kubernetes
- Helm
- Harbor Registry
- OCI Registry

## Data and Infrastructure

- File Storage
- YAML
- Shell Script
- REST API
- JWT

---

# System Design Decisions｜系統設計決策

## Version-based Resource Model

Artifact 與 Template 不直接覆蓋既有內容，而是建立新版本。

這讓 Deployment 可以保存明確的版本關係，並支援 Traceability、Comparison、Rollback Planning 與 Audit History。

---

## Environment Isolation

DEV、UAT、PROD 使用獨立的 Selected Configuration。

切換或修改單一環境時，不會覆蓋其他環境的 Template、Values 或 Project File Selection。

---

## Runtime-specific Dockerfile Template

不同 Runtime 與 Package Type 具有不同啟動方式、必要檔案與驗證規則，因此不強制合併成單一 Generic Dockerfile Template。

平台保留共用 Upload Workflow 與 Metadata Model，同時讓不同 Runtime 使用自己的 Build Logic。

---

## Deployment Package Separation

平台內部 Storage Layout 與 Runtime Deployment Package 完全分離。

內部結構可以隨平台需求演進，Packaging Layer 則維持穩定的輸出契約，降低部署腳本對內部儲存方式的依賴。

---

## Managed OCI Reference

Docker Base Image 與 Deployment Artifact 透過平台管理的 Artifact Version 選擇，不以自由文字保存未受控 Image Reference。

這能確認實際使用的 Registry、Tag、Digest 與 Version，提升 Deployment Governance。

---

## Task and Log Separation

Task 表示目前可操作的執行狀態，Log 表示持久化的歷史結果。

即使成功 Task 後續被清理，Push Log 與 Deployment History 仍能保留 Audit Information。

---

# Engineering Challenges｜工程挑戰與解決方式

## Cross-domain Deployment Orchestration

**Challenge**

Deployment Package 同時依賴 Project、Environment、Template Version、Artifact Version、Config Resource、Registry 與 Storage。

**Solution**

由 Deployment Service 統一協調不同領域，先完成 Resolution 與 Validation，再進行 Rendering、Packaging 與 History Persistence，避免 Controller 或單一 Repository 承擔跨領域邏輯。

---

## External System Consistency

**Challenge**

Registry Push、Kubernetes Scan 與檔案操作不屬於資料庫內部 Transaction，可能出現部分成功或連線失敗。

**Solution**

將長時間操作建模為 Task，保存中間狀態、最終結果與錯誤原因；在不可逆操作前進行狀態驗證，並透過 Retry 與 Audit Log 提供恢復能力。

---

## Version Usage Validation

**Challenge**

刪除 Artifact 或 Template Version 時，需要判斷實際被使用的是哪一個版本，而不是只判斷父層資源是否曾被關聯。

**Solution**

建立 Version-level Usage Tracking，並在 Project Configuration、Deployment 與 Workload Usage 等來源之間維持清楚的使用關係。

---

## Storage and Business Data Separation

**Challenge**

若 Business Service 直接依賴 File System Path，將導致路徑規則、檔案生命週期與業務流程高度耦合。

**Solution**

透過 Storage Service 與 Storage Metadata 隔離實體檔案，Business Domain 僅保存 Storage Reference，並由 Storage Layer 統一處理檔案操作。


我逐漸理解後端架構的重點不只是拆分 Controller、Service 與 Repository，而是讓商業規則、狀態轉換、外部整合與資料生命週期都有清楚的責任邊界。

Deployment Package Generation、Artifact Push Workflow 與 Kubernetes Image Usage Tracking，也讓我累積了 Platform Engineering 與 CI/CD Domain 的實作經驗。
