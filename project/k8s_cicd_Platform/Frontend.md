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

# Frontend Architecture｜前端架構設計

## Application Architecture｜應用程式架構

### Overview

前端應用程式採用分層方式組成，將登入驗證、路由權限、共用版型、頁面功能、API 串接與資料型別分開管理。

整體結構可以簡化為：

```text
Application
  └─ Authentication Provider
      └─ Router
          ├─ Authentication Callback
          └─ Authentication Guard
              └─ Main Layout
                  ├─ Top Navigation
                  ├─ Side Navigation
                  ├─ Breadcrumb
                  └─ Role Guard
                      └─ Page Component
                          ├─ Domain Component
                          ├─ Common Component
                          └─ API Layer
```

Authentication Provider 負責管理登入狀態、Token 與目前使用者資訊。

Router 負責將 URL 對應至不同功能；需要登入的功能會先通過 Authentication Guard，需要特定角色的功能則會再經過 Role Guard。

Main Layout 統一管理 Top Navigation、Side Navigation、Breadcrumb 與主要內容區域，實際頁面則渲染於 Layout 保留的子路由位置。

Page Component 負責整合畫面狀態、使用者操作與資料載入，並透過 API Layer 與 Backend 溝通。

### Technical Highlights

* Layered Frontend Architecture
* Authentication Context
* Nested Routing
* Route Guard
* Role-based Access Control
* Shared Layout
* Domain Component
* API Layer
* Type-safe Data Model

### Design Considerations

將不同責任拆開後，頁面不需要自行處理完整的登入流程、Token Header 或共用版型。

功能開發時，可以沿用既有的 Authentication、Layout、API Configuration、Loading Component 與 Error Handling Pattern，降低重複程式碼。

這種架構也讓權限判斷集中於路由與功能入口，而不是分散在每一個按鈕或頁面中。

---

## Routing and Permission Flow｜路由與權限流程

### Overview

平台使用巢狀路由組織功能頁面，並在不同層級加入登入驗證與角色驗證。

```text
User enters URL
  └─ Check authentication callback
      └─ Check authentication state
          └─ Check required role
              └─ Render shared layout
                  └─ Render target page
```

Authentication Callback 採用獨立流程處理，不會進入一般管理平台版型。

其他管理功能皆需要通過登入驗證。部分功能具有額外角色限制，使用者通過登入驗證後，系統仍會檢查目前角色是否符合 Route Requirement。

### Authentication Guard

Authentication Guard 主要負責：

* 確認 Authentication 初始化是否完成
* 確認使用者是否已登入
* 未登入時啟動登入流程
* 登入完成後顯示子路由
* Session 失效時清除登入狀態
* 避免驗證尚未完成時提前渲染頁面

### Role Guard

Role Guard 主要負責：

* 取得目前使用者角色
* 比對頁面要求的角色
* 無權限時阻止頁面載入
* 有權限時渲染目標功能
* 將頁面權限規則集中於路由層管理

透過 Route Guard，Authentication 與 Authorization 不需要重複寫在每一個 Page Component 中，也能避免只隱藏選單、卻仍可透過 URL 直接進入受限制頁面的問題。

---

## Layout Composition｜共用版型組成

### Overview

管理平台使用共用 Layout 統一頁面結構，主要包含：

* Top Navigation
* Side Navigation
* Breadcrumb
* Main Content
* Nested Route Outlet

```text
Main Layout
  ├─ Top Navigation
  ├─ Side Navigation
  └─ Content Area
      ├─ Breadcrumb
      └─ Nested Page Content
```

Top Navigation 負責顯示平台資訊、目前使用者與選單控制。

Side Navigation 依照目前使用者角色顯示可使用的功能入口，並透過目前 URL 判斷啟用中的選單項目。

Breadcrumb 顯示使用者目前所在的功能層級，Main Content 則透過 Nested Route Outlet 渲染實際的功能頁面。

### State Management

Layout 會管理全域版型所需的狀態，例如：

* Side Navigation 展開與收合
* Breadcrumb 內容
* 目前路由位置
* 使用者角色對應的 Navigation Item
* 頁面切換時的狀態清理

當路由切換時，系統會清除上一個頁面設定的 Breadcrumb，避免新頁面尚未完成載入時顯示舊的導覽資訊。

---

## Frontend Data Flow｜前端資料流

### Overview

前端功能通常由 Component 呼叫 API Layer，取得 Backend Response 後更新 State，再由 React 重新渲染畫面。

```text
User Action
  └─ Event Handler
      └─ API Function
          └─ Backend REST API
              └─ Axios Response
                  └─ Extract Response Data
                      └─ Update Component State
                          └─ Render UI
```

基本資料流程如下：

```tsx
const response = await loadData();
setData(response.data);
```

### Responsibility Boundary

Page Component 主要負責：

* 取得查詢條件
* 管理 Loading 與 Error State
* 呼叫 API Function
* 更新 Page State
* 控制 Modal 與 Detail View
* 操作完成後重新載入資料

API Layer 主要負責：

* Endpoint Path
* HTTP Method
* Query Parameter
* Request Body
* Response Type
* TypeScript Generic Type
* Blob 或 Text Response Configuration

Component 不需要知道共用的 Axios 設定、Authorization Header 或 Backend Base URL。當 API Endpoint 發生變化時，也能集中修改 API Layer。

---

# Key Implementations｜核心功能實作

## Authentication｜登入與權限驗證

### Overview

<img width="565" height="501" alt="image" src="https://github.com/user-attachments/assets/a2066309-cb64-45e0-98cf-c3a2e27890c2" />

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

<img width="973" height="694" alt="image" src="https://github.com/user-attachments/assets/4b7d8ea0-5510-4a73-90b0-1aa2ee963c20" />
<img width="1713" height="799" alt="image" src="https://github.com/user-attachments/assets/67f7a053-1846-4734-94df-b22395f2cf81" />
<img width="1148" height="708" alt="image" src="https://github.com/user-attachments/assets/f2f93afe-7cba-43ee-85f2-252efd2f0ca5" />

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
<img width="1379" height="452" alt="image" src="https://github.com/user-attachments/assets/48828823-706e-49ad-9d4f-50bacafe0499" />

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
<img width="1317" height="472" alt="image" src="https://github.com/user-attachments/assets/b1a81561-2e3b-4578-b3ea-e2fef463ded8" />
<img width="598" height="815" alt="image" src="https://github.com/user-attachments/assets/15a5a0d5-613a-4634-9688-25591011c6b5" />

Push Task 是由後端執行的非同步工作，因此前端不能將它當成一般同步 API 操作。

前端需要根據 Task Status 決定使用者可以進行的操作：

* PENDING：可以執行或刪除
* RUNNING：顯示執行中狀態，避免重複操作
* FAILED：顯示失敗原因，並提供 Retry
* SUCCESS：代表 Image 已完成同步

<img width="1655" height="104" alt="image" src="https://github.com/user-attachments/assets/e154a040-6898-4f27-a068-4f3e35772780" />
<img width="1644" height="111" alt="image" src="https://github.com/user-attachments/assets/241272a6-de73-4ebc-b700-27ab2545907c" />
<img width="934" height="641" alt="image" src="https://github.com/user-attachments/assets/54ff223c-3eaf-4406-aa79-be0cab87c7b3" />


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

# Frontend Interaction Patterns｜前端互動設計模式

## Modal Interaction Pattern｜Modal 互動模式

### Overview

平台中的新增、編輯、詳細資料與設定選擇，大多透過 Modal 完成。Modal 的控制責任分為 Page State 與 Modal State。

```text
Page Component
  ├─ Control whether Modal is open
  ├─ Store selected item
  ├─ Reload data after success
  └─ Pass callback to Modal

Modal Component
  ├─ Initialize form data
  ├─ Validate input
  ├─ Submit API request
  ├─ Display saving state
  ├─ Display error message
  └─ Notify Page after success
```

Page 通常管理 Modal 是否開啟、目前選擇的資料、新增或編輯模式，以及操作完成後的資料刷新。

Modal 則管理 Form State、Validation Error、API Error、Saving State、Preview State 與相依選項。

當 Modal 開啟時，會根據傳入資料初始化表單。儲存期間，按鈕會進入 Disabled 或 Loading 狀態，避免重複送出請求。成功後由 Modal 通知 Page，再由 Page 重新取得列表或詳細資料。

---

## UI Feedback and Transition｜操作回饋與畫面效果

### Overview

企業管理平台中的操作通常涉及 API 請求、非同步任務與資料重新整理，因此前端需要提供清楚的即時回饋。

平台使用不同層級的 Loading State，避免所有操作都以全頁 Loading 呈現。

```text
Page Loading
  └─ Initial data loading

Inline Loading
  └─ Background list refresh

Row Loading
  └─ Execute / Retry / Delete one item

Modal Loading
  └─ Save / Resolve / Preview
```

### List Transition

列表重新查詢時，可以保留舊資料並加入簡單的 Transition State：

```text
show
  └─ out
      └─ update data
          └─ in
              └─ show
```

這能避免資料重新載入時整張表格突然消失或閃爍。

### Status and Event Feedback

平台透過統一的 Status Badge 顯示 Task Status、Artifact Status、Deployment Status、Environment、Platform、Enabled State 與 File Kind。

在可點擊的 Table Row 中，操作按鈕會停止事件向上傳遞，避免點擊 Edit、Retry、Download 或 Delete 時，同時觸發 Row Detail。

```tsx
event.stopPropagation();
```

---

## Breadcrumb Navigation Pattern｜麵包屑導覽模式

### Overview

Breadcrumb 的內容由各個 Page Component 設定，共用 Layout 負責顯示。

```text
Page Component
  └─ Set breadcrumb items
      └─ Breadcrumb Context
          └─ Main Layout
              └─ Render breadcrumb
```

Page Component 負責決定目前功能的導覽層級、提供 Label 與可返回的 Route，並在 Detail Data 載入後更新動態名稱。

Layout 則負責統一 Breadcrumb 外觀、顯示可點擊與不可點擊項目，以及在路由切換時清除舊內容。

---

## Frontend Feature Development Flow｜前端功能開發流程

### Overview

平台中的一個完整前端功能通常不只包含單一頁面，而是由資料型別、API Layer、路由權限、頁面狀態、共用元件與操作回饋共同組成。

```text
Requirement
  └─ Define Domain Model
      └─ Define Request and Response Type
          └─ Implement API Function
              └─ Register Route and Permission
                  └─ Build Page State
                      └─ Compose UI Components
                          └─ Add Loading and Error Handling
                              └─ Add Navigation Entry
                                  └─ Test Frontend and Backend Contract
```

### Domain and API Contract

首先確認功能中的主要資料結構，包括 ID、Name、Status、Enum、Nullable Field、Request Payload、Response DTO 與 Pagination Structure。

接著使用 TypeScript 建立 Domain Model，並於 API Layer 定義 HTTP Method、Endpoint、Query Parameter、Request Body 與 Response Type。

### Route and Page State

根據功能需求設定是否需要登入、允許哪些角色存取、是否顯示於導覽選單，以及無權限時的處理方式。

Page Component 再依操作流程管理：

* List Data
* Detail Data
* Search Keyword
* Pagination
* Loading State
* Error State
* Selected Item
* Modal State
* Action Loading ID

### UI Composition and Refresh

畫面優先使用既有 Common Component 與 Domain Component，例如 Modal、Pagination、Status Badge、Inline Loading、Selector、Detail Panel 與 Data Table。

Create、Update、Delete、Execute 或 Retry 完成後，需要確認 List、Detail、Pagination Total、Selected Item、Related Domain Data 與 Modal Content 是否應重新載入，避免不同區域顯示不同版本的資料。

### Testing Considerations

功能完成後，主要確認：

* Request Payload 是否符合 Backend DTO
* Enum 值是否一致
* Nullable Field 是否安全處理
* API 失敗時是否顯示正確訊息
* Loading 期間是否避免重複操作
* 權限不足時是否阻止存取
* 操作完成後資料是否正確刷新
* 切換 Route 或 Environment 時是否殘留舊 State

### What I Learned

透過這套開發流程，我逐漸建立從 Domain Model、API Contract、Page State 到 UI Interaction 的完整思考方式。

一個前端功能並不只是完成畫面，而是需要同時考量權限、資料契約、操作狀態、錯誤回饋與後續維護性。

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

# Engineering Challenges｜工程挑戰與解決方式

## Authentication 與 Token Lifecycle

**Challenge**

需要處理 Keycloak 登入、Callback、Token 過期、權限驗證與未授權回應。

**Solution**

* 使用 OAuth 2.0 Authorization Code Flow with PKCE
* 驗證 Callback State，避免不合法回傳
* 解析 JWT，建立統一的使用者與角色資料
* 透過 Axios Interceptor 自動加入 Bearer Token
* Token 即將過期時使用 Refresh Token 更新
* 收到 `401 Unauthorized` 時清除 Session
* 使用 `RequireAuth` 與 `RequireRole` 保護 Route
* 避免 React Effect 重複交換 Authorization Code

---

## 前後端 DTO 與 API Response

**Challenge**

前後端 DTO 持續演進，而且 API Response 同時包含 JSON、Text、Blob 與 Pagination 等不同格式。

**Solution**

* 為 Request、Response 與 Domain Model 建立 TypeScript 型別
* API Function 保留 `AxiosResponse`
* Preview API 使用 Text Response
* 檔案下載使用 Blob Response
* 列表統一處理 Page、Size、Content 與 Total Elements
* API 依 Registry、Artifact、Template、Project 等 Domain 拆分
* 由 Component 管理 Loading、Error 與操作結果

---

## 多環境設定與資料隔離

**Challenge**

DEV、UAT、PROD 具有不同 Template、Values 與 Project File 設定，切換或儲存時不能互相覆蓋。

**Solution**

* 使用 `DeployEnvironmentType` 限制實際部署環境
* 切換環境時重新載入該環境的 Values 與 Project Files
* 儲存時只更新目前 Environment
* 保留其他 Environment 原有設定
* 修改單一 Template 時，保留其他 Template 的資料
* 將 ENV 與 MOUNT 統一儲存在 `configFiles`
* 將 Project File 的可用環境與實際部署環境分開處理

---

## Project File 的 ALL 規則

**Challenge**

Project File 的 `ALL` 代表所有環境皆可使用，但 Selected Configuration 不允許使用 `ALL` 作為部署環境。

**Solution**

前端會將 `ALL` 視為檔案的可用範圍，讓該檔案可以在 DEV、UAT、PROD 中被選擇。

實際儲存 Selected Configuration 時，仍使用目前選擇的 DEV、UAT 或 PROD，避免混淆「可用範圍」與「部署環境」。

---

## Template 與 Deployment Preview

**Challenge**

Helm、Dockerfile 與 Shell 具有不同設定需求，而且 Template、Version、Values 與 Preview 之間存在相依關係。

**Solution**

* Template 改變時重新載入 Version
* Version 改變時重新取得 Default Values
* 上游選擇改變時清除舊 Preview
* 支援 Public Template 與 Custom Template
* Dockerfile 強制選擇可用的 Base Image Version
* Helm 整合 ConfigMap、Secret 與 Manifest
* Shell 管理 Helm Wait 與 Deployment Completion Mode
* 呼叫不同 Preview API 顯示最終 Render 結果
* 解析 Manifest Resource，檢查 ConfigMap 與 Secret 命名衝突

---

## 非同步 Push Task

**Challenge**

Image Push 由後端非同步執行，前端需要根據任務狀態提供不同操作，並保持 List 與 Detail 資料一致。

**Solution**

* PENDING 提供 Execute 與 Delete
* RUNNING 顯示執行中狀態並禁止重複操作
* FAILED 顯示 Failure Reason 並提供 Retry
* 使用 Task ID 管理單筆 Loading State
* 操作完成後重新載入 Task List
* Detail Modal 開啟時同步更新 Task Detail
* 使用 Push Log 保留持久化的執行與失敗紀錄

---

## 共用元件與大型 Component

**Challenge**

隨著功能增加，頁面狀態與 Component 規模持續成長，也出現重複的 Modal、Status、Pagination 與 Loading 實作。

**Solution**

已逐步建立：

* Common Modal
* Status Badge
* Pagination
* Inline Loading
* Common Icons
* Project File Table
* Base Image Selector
* Template Version List
* Project Selected Config Panel

同時將純 UI 行為整理成 Common Component，將包含業務規則的功能保留為 Domain Component。

大型 Component 的拆分仍是持續進行的工作。後續會繼續將資料載入、表單驗證與不同 Template 的設定流程拆分，降低單一 Component 的複雜度。
