# 部署狀態資料結構

## Local State

### 部署設定 DeploymentConfig

- **說明:**
  - 部署形態與授權驗證參數的本地設定
- **欄位:**
  - `deployMode`: String - Not Null, 部署形態，值域 cloud、selfhosted；cloud 為雲端託管，selfhosted 為自架部署
  - `licenseKey`: String | Null - Nullable, 授權金鑰，授權驗證依據；Null 代表尚未設定
  - `gracePeriodDays`: Number - Not Null, 寬限期天數設定值；具體數值屬商品化階段定案
  - `renewIntervalDays`: Number - Not Null, 授權驗證間隔天數設定值；具體數值屬商品化階段定案

### 授權狀態 LicenseState

- **說明:**
  - 授權驗證進度與結果的本地狀態
- **欄位:**
  - `lastVerifiedOn`: Number | Null, Unix Timestamp ms - Nullable, 上次驗證成功時間；Null 代表尚未驗證成功
  - `graceStartedOn`: Number | Null, Unix Timestamp ms - Nullable, 寬限期起算時間；Null 代表未處於寬限期
  - `licenseStatus`: String - Not Null, 授權狀態，值域 valid、grace、readonly；valid 為驗證通過，grace 為驗證未過且寬限中，readonly 為寬限逾期

---

## 部署狀態標準

- **儲存標準:**
  - 兩實體皆為部署本地狀態，僅存部署端
  - 不入使用者資料模型，不與雲端使用者資料同步
  - 不帶租戶鍵，狀態屬部署實例，不屬任一 Company
