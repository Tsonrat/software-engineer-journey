# Frontend Development｜前端開發

在這個 Kubernetes CI/CD 平台中，我主要參與前端管理介面的開發，將 Registry、Artifact、Template、Project 與 Deployment 等後端功能，轉換成使用者可以實際操作的流程。
前端不只負責呈現資料，也需要處理多環境設定、模板選擇、檔案管理、部署預覽、狀態追蹤及 API 錯誤回饋。

---

## React

我使用 React 建立平台的操作頁面與共用元件，負責的功能包含：

* Registry 與 Artifact 管理介面
* Docker Image Tag、Push Task 與執行紀錄
* Helm、Dockerfile、Shell Template 管理
* Project、ConfigMap 與 Secret 檔案管理
* DEV、UAT、PROD 多環境部署設定
* Deployment Preview、執行狀態與結果查看

在 Project Selected Config 流程中，我需要處理不同環境的設定切換，並確保切換環境時不會覆蓋其他環境已選擇的 ConfigMap 或 Secret。
我也將 Modal、Status Badge、Pagination、Loading 與檔案表格等功能拆成共用元件，減少不同頁面之間的重複實作。

---

## TypeScript

我使用 TypeScript 定義前端的 Domain Model、API Response、Component Props 與表單狀態。
例如，針對 Registry、Artifact、Template、Project File、Selected Config 與 Deployment Log 建立對應型別，讓前端資料結構能與 Spring Boot DTO 保持一致。
在多環境設定中，我也使用 Union Type 限制可選值：

```ts
type DeployEnvironmentType =
  | 'DEV'
  | 'UAT'
  | 'PROD';
```

這能避免將只代表「所有環境皆可使用」的 ALL，錯誤儲存成實際部署環境。
API 層則使用 Axios 與後端溝通，並保留 AxiosResponse，由 Component 讀取 res.data。透過明確的型別設計，可以在開發階段提早發現 API 欄位、環境值或狀態值不一致的問題。

---

## Vite

我使用 Vite 建立前端的開發與建置流程，並分離開發環境與正式環境設定。
開發時透過 Vite Proxy，將 /api 請求轉送至本機 Spring Boot Backend，避免在每個 API Function 中寫死後端位址。

```ts
server: {
  proxy: {
    '/api': {
      target: 'http://localhost:8080'
    }
  }
}
```

專案也使用 .env 與 .env.production 管理不同環境的設定，正式建置時先執行 TypeScript 型別檢查，再由 Vite 產生部署所需的靜態檔案。
