# Frontend Development｜前端開發

本文件記錄我在 **Enterprise Kubernetes CI/CD Platform** 專案中的前端開發內容。

本專案採用 **React、TypeScript 與 Vite** 開發企業管理平台，目標是將 Kubernetes 部署流程、Artifact 管理、多環境設定及部署自動化等功能，整合成一致且容易操作的 Web 管理介面。

除了功能開發之外，我也持續重構元件架構、改善使用者操作流程、建立共用元件，並思考前後端責任分工與整體系統設計，希望打造一套具備良好維護性與擴充性的企業級前端架構。

---

# Project Overview｜專案概述

在此專案中，我負責前端管理平台的設計與開發，主要涵蓋：

* 使用者登入與權限驗證
* Registry 管理
* Artifact 與 Artifact Version 管理
* Docker Image Tag 管理
* Helm、Dockerfile 與 Shell Template 管理
* Project 與 Project Group 管理
* ConfigMap、Secret 與 Project File 管理
* DEV、UAT、PROD 多環境部署設定
* Deployment Preview
* Artifact Push Task
* Artifact Push Log
* Deployment Log
* API Layer 設計
* TypeScript Domain Model 建立
* 共用元件設計與重構

整個平台以 React Component 為基礎，搭配 TypeScript 建立前端 Domain Model，並透過 Axios 與 Spring Boot Backend 提供的 REST API 進行整合。

前端不只負責呈現後端資料，也需要將 Kubernetes、Harbor、OCI Artifact 與 Template Rendering 等較複雜的概念，轉換成使用者能夠理解與操作的流程。

---

# Key Implementations｜核心功能實作

## Authentication｜登入與權限驗證

### Overview

平台採用 Keycloak 作為統一身分驗證服務。

前端負責登入導向、Callback 處理、Token 管理、使用者狀態管理，以及依照角色限制頁面與功能存取。

這部分不只是製作登入畫面，也包含完整的 Authentication 與 Authorization Flow。

### Implementation

* Keycloak OpenID Connect
* OAuth 2.0 Authorization Code Flow with PKCE
* Login Redirect
* Callback Handling
* Authorization Code Exchange
* JWT Parsing
* Access Token Management
* Refresh Token Flow
* Logout Flow
* Role-based Route Protection
* Axios Authorization Header
* Unauthorized Session Handling

### Technical Highlights

* OAuth 2.0
* OpenID Connect
* PKCE
* JWT
* Authentication
* Authorization
* Route Guard
* Token Lifecycle
* Axios Interceptor

### Challenges

登入功能的難點不只是在畫面上提供登入按鈕，而是管理完整的 Token Lifecycle。

前端需要在導向 Keycloak 前產生 PKCE Verifier、Challenge 與 State，並在 Callback 時驗證 State，避免接受不合法的登入回傳。

取得 Access Token 後，前端會解析 JWT 中的使用者資料與角色，並透過 Axios Interceptor 統一加入 Authorization Header。

當 Token 即將過期時，系統會嘗試使用 Refresh Token 取得新的 Access Token；如果更新失敗或 API 回傳 `401 Unauthorized`，則清除登入狀態並重新進入驗證流程。

不同 Keycloak Client 或 Realm 中的角色格式也可能不同，因此前端需要整理 Realm Role、Resource Role 與角色名稱格式，轉換成平台使用的統一權限模型。

另外，React 開發模式下 Effect 可能重複執行，因此 Callback 頁面也需要避免同一組 Authorization Code 被交換兩次。

### What I Learned

透過此功能，我實際理解了前端 Authentication Flow、JWT Token 管理、Token Refresh，以及 Role-based Access Control 在企業管理平台中的應用。

我也了解到登入功能不只是畫面與 API 串接，而是需要完整考量登入狀態、Token 過期、未授權回應及路由保護。

---

## REST API Integration｜REST API 串接

### Overview

前端透過 Axios 建立 API Layer，與 Spring Boot Backend 提供的 REST API 進行整合。

API Function 依照 Domain 拆分，讓 Component 專注於畫面與互動流程，API Layer 則負責 Endpoint、Request Parameter、Request Body 與 Response Type。

### Implementation

* API Layer Design
* Axios Configuration
* Axios Interceptor
* Request / Response Type
* Authorization Header
* Query Parameter
* Pagination API
* Text Preview Response
* Blob Download
* Loading State
* Error Handling

API 依照功能拆分，例如：

* Registry API
* Artifact API
* Artifact Version API
* Artifact Push Task API
* Artifact Push Log API
* Helm Template API
* Dockerfile Template API
* Shell Template API
* Project API
* Project File API
* Project Selected Config API
* Deployment API
* Deployment Log API

前端 API Function 會保留完整的 `AxiosResponse`：

```ts
const res = await getArtifactPushTasks();
const data = res.data;
```

### Technical Highlights

* Axios
* AxiosResponse
* Axios Interceptor
* Type-safe API
* Loading State
* Error State
* Pagination
* Blob Download
* Text Response
* Frontend and Backend Contract

### Challenges

不同 API 的資料形式並不完全相同。

除了一般 JSON DTO，也包含純文字 Preview、Blob 檔案下載與分頁資料，因此前端需要依照 Response 類型進行不同處理。

例如：

* Helm、Dockerfile 與 Shell Preview 使用文字格式回傳
* Rendered Deployment Asset 使用 Blob 接收並建立下載流程
* 列表 API 需要處理 Page、Size、Content 與 Total Elements
* Detail API 需要處理 Modal Loading 與查詢失敗狀態

另一項挑戰是前後端 DTO 會隨功能持續演進，因此需要同步維護 TypeScript Request 與 Response 型別，避免欄位名稱、Nullable 狀態及 Enum 值不一致。

API 發生錯誤時，前端也需要區分：

* 頁面載入失敗
* 表單儲存失敗
* 單一操作失敗
* Authentication 失效
* 檔案下載失敗
* Preview 產生失敗

同時也需要正確管理 Loading State，避免使用者在 API 尚未完成時重複送出請求。

### What I Learned

透過 API Layer 的設計，我逐漸理解 Component 與 API Function 的責任分工，以及如何維持前後端資料契約一致。

我也學會處理 JSON、Text、Blob 與 Pagination 等不同 Response 類型，並建立較容易維護的前端資料流。

---

## Project Selected Configuration｜多環境部署設定

### Overview

Project Selected Configuration 是整個平台中最重要且較複雜的前端功能之一。

使用者可以針對不同部署環境選擇 Helm、Dockerfile 與 Shell Template，設定 Template Values、Project Files 與 Docker Base Image，並在部署前預覽最後產生的內容。

支援的部署環境包含：

* DEV
* UAT
* PROD

### Implementation

* Public Template Selection
* Project Custom Template Selection
* Template Version Selection
* Docker Base Image Selection
* Docker Base Image Version Selection
* ConfigMap Selection
* Secret Selection
* ENV File Selection
* MOUNT File Selection
* Manifest File Selection
* Values Configuration
* Environment Switching
* Default Values Preview
* Deployment Preview
* Configuration Validation
* Environment-specific Save Flow

設定完成後，可以預覽：

* Rendered Helm YAML
* Rendered Dockerfile
* Rendered Shell Script

### Technical Highlights

* Multi-environment Configuration
* Environment Isolation
* Type-safe State Management
* Dependent Select Flow
* Template Rendering Preview
* Domain Model Design
* Configuration Validation
* Environment-specific Data
* Public and Custom Template Source

### Challenges

此功能最大的挑戰，是在同一個操作流程中管理三種 Template、三個部署環境，以及多組彼此相依的選項。

Template Source、Template、Version、Values 與 Preview 之間具有先後依賴關係。當上游選擇改變時，前端需要清除已經失效的 Version 或 Preview，避免畫面保留舊資料。

切換 DEV、UAT、PROD 時，前端需要重新載入該環境的 Values 與 Project Files，並只更新目前環境的 ConfigMap、Secret 與 Template 設定，保留其他環境既有資料。

前端使用明確的 TypeScript Union Type 限制可選部署環境：

```ts
type DeployEnvironmentType =
  | 'DEV'
  | 'UAT'
  | 'PROD';
```

Project File 中的 `ALL` 代表該檔案可供所有環境使用，但實際加入 Selected Configuration 時，仍需要記錄為 DEV、UAT 或 PROD。

因此，前端需要同時處理：

* Project File 的可用環境
* Selected Configuration 的實際部署環境

另外，ENV 與 MOUNT 類型的 Project File 會統一儲存在 `configFiles`，前端需要依照檔案用途、環境與選擇狀態正確建立 Payload。

Helm、Dockerfile 與 Shell 也具有不同的設定需求：

* Helm 需要整合 ConfigMap、Secret、ENV、MOUNT 與 Manifest
* Dockerfile 必須選擇可用的 Base Image Version
* Shell 包含 Helm Wait Timeout 與部署完成條件
* Public Template 與 Custom Template 具有不同的 Version 載入方式

儲存其中一種 Template 時，還需要保留未被編輯的其他 Template 設定。例如修改 Helm Configuration 時，不能意外覆蓋 Dockerfile 或 Shell Configuration。

前端也會解析 Manifest Preview 中的 ConfigMap 與 Secret 名稱，用來檢查選取的 Resource 是否發生命名衝突。

### What I Learned

透過此功能，我學會如何管理較複雜的前端狀態，以及如何設計多環境部署設定流程。

我也更加理解 State 不只是畫面資料，而是需要反映 Template、Version、Environment、Project File 與 Preview 之間的業務關係。

這個功能也讓我學會如何避免不同環境互相覆蓋設定，並維持前後端資料結構的一致性。

---

## Artifact Push Task｜映像推送任務

### Overview

平台透過 Push Task 管理 Docker Image 從外部 Registry 同步至內部 Harbor 的非同步流程。

由於 Image Push 可能需要一定的執行時間，前端需要提供任務狀態、操作按鈕、失敗原因與重試功能，讓使用者能夠掌握映像同步結果。

### Implementation

* Task Creation
* Task List
* Task Detail
* Task Execution
* Retry Flow
* Delete Flow
* Status-based Action
* Pagination
* Per-task Loading State
* Failure Reason
* Retry Count
* OCI Platform Information
* Push Log

Task Status 包含：

* PENDING
* RUNNING
* SUCCESS
* FAILED

### Technical Highlights

* Asynchronous Workflow
* Task State Tracking
* Status-based UI
* Retry Mechanism
* Per-item Loading State
* Error Feedback
* Task Detail
* Audit Log

### Challenges

Push Task 是由後端執行的非同步工作，因此前端不能將它當成一般同步 API 操作。

前端需要根據 Task Status 決定使用者可以進行的操作：

* PENDING：可以執行或刪除
* RUNNING：顯示執行中狀態，避免重複操作
* FAILED：顯示失敗原因，並提供 Retry
* SUCCESS：代表 Image 已完成同步

執行或重試期間，前端使用 Task ID 管理單筆操作的 Loading State，避免一個 Task 執行時鎖定整張列表，也避免相同 Task 被重複送出。

操作完成後，需要同步重新取得 Task List；如果使用者已經開啟 Detail Modal，也需要更新目前 Detail，避免列表與詳細資料顯示不同狀態。

Push Task Detail 會呈現：

* Registry
* Harbor Project
* Artifact Path
* Image Tag
* Task Status
* Priority
* Retry Count
* Started Time
* Finished Time
* Failure Reason
* Selected OCI Platforms

SUCCESS Task 可能會被快速清理，因此系統另外透過 Push Log 保留持久化的執行紀錄與失敗資訊。

### What I Learned

這是我第一次實作非同步工作流程的前端管理介面。

透過此功能，我開始理解如何根據後端任務狀態提供不同操作，並透過 Loading、Status Badge、Failure Reason 與 Retry Flow，給予使用者清楚的執行回饋。

我也了解到即時任務與持久化操作紀錄具有不同用途，因此 Task 與 Log 應該分開設計與呈現。

---

## Reusable Components｜共用元件

### Overview

隨著專案功能逐漸增加，我開始重構大型頁面，並將重複的 UI 與操作邏輯整理成 Common Components 和 Domain Components。

Common Components 負責通用的 UI 行為；Domain Components 則保留特定業務領域的操作流程。

### Implementation

Common Components 例如：

* Modal
* Status Badge
* Pagination
* Inline Loading
* Toggle Switch
* Common Icons
* Form Style

Domain Components 例如：

* Base Image Selector
* Template Version List
* Project File Table
* Project File Upload Modal
* Project Selected Config Panel
* Project Selected Config Modal
* Artifact Push Task Detail
* Deployment Detail

### Technical Highlights

* Component Composition
* Domain Component
* Common Component
* Reusable UI
* Props Design
* Event Handling
* UI Consistency
* Separation of Concerns

### Challenges

共用元件的挑戰，是在重用性與 Domain-specific Requirement 之間取得平衡。

例如 Status Badge 需要同時支援：

* Task Status
* Artifact Status
* Platform
* Environment
* File Kind
* Enabled State

Pagination 需要由不同列表共用，但查詢條件與資料載入仍由各自頁面管理。

Modal 則需要維持一致的外觀與開關行為，同時允許 Artifact、Template、Project File 與 Deployment 等功能放入不同的表單內容。

在可點擊的 Table Row 中加入 Preview、Edit、Retry、Download 與 Delete 等操作按鈕時，也需要處理 Event Propagation，避免點擊按鈕時同時觸發 Row Detail。

我將純 UI 行為整理成 Common Components，並將 Base Image Selector、Template Version List 與 Project Selected Config 等具有業務語意的功能保留為 Domain Components，降低頁面之間的重複實作。

### What I Learned

透過持續重構，我開始理解 Component Design 不只是將程式碼拆成不同檔案，而是需要思考元件責任、Props 邊界與重用範圍。

合理的元件拆分可以降低耦合、維持 UI 一致性，也能讓後續需求更容易擴充。

---

# Frontend Technology｜前端技術運用

## React

使用 React 建立平台管理頁面與互動流程，並透過 Component、Hook 與 React Router 管理：

* 頁面結構
* 表單狀態
* Modal
* Environment Switching
* Loading State
* Error State
* Authentication State
* Route Protection
* API Data Flow

React 在此專案中的主要角色，是將後端提供的 Domain Function 轉換成完整且可操作的使用者流程。

---

## TypeScript

使用 TypeScript 定義：

* Domain Model
* API Request
* API Response
* Component Props
* Form State
* Environment Type
* Task Status
* Template Type
* Nullable Field

透過明確的型別定義，讓前端資料結構能與 Spring Boot DTO 保持一致，並在開發階段提早發現欄位名稱、Enum 值或資料格式錯誤。

TypeScript 在此專案中不只是提供基本型別檢查，也用來表達不同 Domain 之間的規則與限制。

---

## Vite

使用 Vite 建立前端的開發與正式建置流程。

主要用途包含：

* Development Server
* Hot Module Replacement
* Environment Variable
* Development / Production Mode
* API Proxy
* Production Build

開發環境透過 Vite Proxy 將 `/api` 請求轉送至本機 Spring Boot Backend，避免在每個 API Function 中寫死後端位址。

```ts
server: {
  proxy: {
    '/api': {
      target: 'http://localhost:8080',
      changeOrigin: true
    }
  }
}
```

專案也使用 `.env` 與 `.env.production` 管理不同環境的設定。

正式建置時，會先執行 TypeScript 型別檢查，再由 Vite 產生部署所需的靜態檔案。

---

# Engineering Challenges｜工程挑戰

在專案開發過程中，我遇到許多具有挑戰性的問題，例如：

* Keycloak Authentication Flow 與 Token Lifecycle 管理
* Role-based Route Protection
* 前後端 DTO 持續演進與 TypeScript 型別同步
* JSON、Text、Blob 與 Pagination 等不同 API Response
* DEV、UAT、PROD 多環境設定的資料隔離
* Project File 的 `ALL` 可用範圍與實際部署環境差異
* ENV 與 MOUNT 設定統一至 `configFiles`
* Helm、Dockerfile 與 Shell Template 的不同設定需求
* Public Template 與 Custom Template 的切換
* Deployment Preview 整合多種 Template
* Docker Base Image 與 Version 相依選擇
* ConfigMap 與 Secret 命名衝突檢查
* 非同步 Push Task 的狀態式操作流程
* Task List 與 Detail Modal 的資料同步
* 大型 Component 的狀態管理與持續重構
* Common Component 與 Domain Component 的責任邊界
* API Layer 與 UI Component 的責任分工

這些問題促使我持續調整資料結構、Component 邊界與使用者操作流程，也讓我逐步建立企業級前端開發的工程思維。

