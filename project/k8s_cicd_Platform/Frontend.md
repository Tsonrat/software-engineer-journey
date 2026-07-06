# Frontend Development｜前端開發

在這個 Kubernetes CI/CD 平台中，我主要參與前端管理介面的開發，將 Registry、Artifact、Template、Project 與 Deployment 等後端功能，轉換成使用者可以實際操作的流程。

前端除了呈現資料，也負責身分驗證、API 串接、多環境部署設定、檔案管理、部署預覽，以及非同步任務的狀態追蹤與錯誤回饋。

---

# Frontend Tech Stack｜前端技術

## React

使用 React 建立平台管理頁面與互動流程，並將 Modal、Status Badge、Pagination、Loading 與檔案表格拆分成共用元件。

主要應用於 Registry、Artifact、Template、Project、Deployment，以及 Push Task 等功能。

## TypeScript

使用 TypeScript 定義 Domain Model、API Request／Response、Component Props 與表單狀態，讓前端資料結構與 Spring Boot DTO 保持一致。

同時使用 Union Type 限制部署環境及任務狀態，降低 API 欄位、環境值與狀態值不一致的問題。

## Vite

使用 Vite 管理前端開發與正式建置流程，搭配環境變數區分不同執行環境，並透過 Proxy 將開發環境中的 `/api` 請求轉送至 Spring Boot Backend。

```md

# Frontend Highlights｜前端實作重點



本專案的前端不只是呈現後端資料，而是將身分驗證、多環境部署設定與非同步映像推送流程，整理成使用者可以操作的管理介面。



---



## Authentication｜登入與權限驗證



我實作了平台登入畫面，並串接 Keycloak 作為外部身分驗證服務。



前端採用 OAuth 2.0 Authorization Code Flow with PKCE，負責：



* 導向 Keycloak 登入頁面

* 處理登入 Callback

* 驗證 State 與交換 Access Token

* 解析 JWT 中的使用者資訊與角色

* 自動更新即將過期的 Token

* 在 Axios Request 中加入 Bearer Token

* 收到 `401 Unauthorized` 時清除登入狀態

* 根據使用者角色限制頁面與功能



這部分讓我實際接觸到前端登入流程、Token 生命週期與 Role-based Access Control，而不只是製作一個登入表單。



---



## REST API Integration｜API 串接



我使用 Axios 建立前端 API Layer，串接 Spring Boot Backend 提供的 REST API。



API 依照功能拆分，例如：



* Registry API

* Artifact API

* Artifact Version API

* Template API

* Project API

* Project Selected Config API

* Artifact Push Task API

* Deployment Log API



每個 API Function 都透過 TypeScript 定義 Request 與 Response 型別，並保留完整的 `AxiosResponse`：



```ts

const res = await getArtifactPushTasks();

const data = res.data;

```



除了一般 JSON 資料外，也處理了不同形式的 Response，例如：



* Helm、Dockerfile 與 Shell 純文字預覽

* 產生完成後的 Blob 檔案下載

* 分頁查詢結果

* API Loading 與錯誤狀態

* Axios Interceptor 驗證處理



開發環境則透過 Vite Proxy 將 `/api` 請求轉送至 Spring Boot Backend。



---



## Project Selected Config｜多環境部署設定



Project Selected Config 是前端流程中較複雜的一部分。



使用者可以分別設定 Helm、Dockerfile 與 Shell Template，並選擇：



* Public Template 或 Project Custom Template

* Template Version

* DEV、UAT、PROD 部署環境

* ConfigMap 與 Secret

* ENV 或 MOUNT 用途的 Project File

* Docker Base Image 與版本

* Template Values

* Deployment Package 路徑



前端需要同時管理 Template、Version、Environment、Project File 與 Preview 等多組狀態。



在環境切換時，我需要確保目前環境的選擇不會覆蓋其他環境已儲存的設定。例如修改 DEV 時，只更新 DEV 的 `configFiles`，保留 UAT 與 PROD 原有資料。



```ts

type DeployEnvironmentType =

  | 'DEV'

  | 'UAT'

  | 'PROD';

```



Project File 可以設定為 `ALL`，代表所有環境皆可使用；但實際選入 Project Selected Config 時，仍必須記錄為 DEV、UAT 或 PROD，避免部署環境語意混淆。



設定完成後，前端會呼叫 Preview API，即時呈現：



* Rendered Helm YAML

* Rendered Dockerfile

* Rendered Shell Script



讓使用者可以在儲存或部署之前確認最後產生的內容。



---



## Artifact Push Task｜映像推送任務



Docker Image 從外部 Registry 同步至內部 Harbor 並不是立即完成的操作，因此系統使用 Push Task 管理非同步執行流程。



我在前端實作 Push Task 的操作與狀態呈現，包含：



* 建立與編輯 Push Task

* 手動執行待處理任務

* 顯示 PENDING、RUNNING、SUCCESS、FAILED 狀態

* FAILED Task 重試

* 分頁查詢 Task

* 查看開始與完成時間

* 顯示失敗原因與 Retry Count

* 查看選擇的 OCI Platform 資訊

* 顯示來源 Registry 與目標 Harbor Image Path



前端會根據 Task 狀態提供不同操作，例如 PENDING 可以執行、FAILED 可以重試，而 RUNNING Task 則以狀態資訊為主。



這個功能讓使用者可以直接從平台掌握映像同步進度，也能在失敗時查看原因並重新執行。



---



## Frontend Tech Stack｜技術運用



* **React**：建立管理頁面、Modal、表單及多步驟操作流程。

* **TypeScript**：定義 API、Domain Model 與環境狀態，降低資料格式錯誤。

* **Vite**：管理開發環境、環境變數、API Proxy 與正式建置流程。



透過這些功能，我學習到如何將身分驗證、多環境設定與非同步任務等後端流程，轉換成清楚且可維護的前端操作介面。

```
