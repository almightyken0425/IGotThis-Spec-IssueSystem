# 權限資料結構

## 使用者自訂資料結構

### 帳號 Accounts

- **欄位:**
  - `id`: String, UUID/GUID - Primary Key
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵，`Company` 即租戶邊界
  - `name`: String - Not Null, 帳號顯示名稱
  - `defaultCalendarName`: String | Null - Nullable, 預設工作日曆名稱，指向同一 `Company` 的日曆定義；檢視未選用日曆時跟隨此欄位；Null 代表尚未設定
  - `tags`: Object | Null, JSON - Nullable, 標籤群組識別碼對標籤值識別碼的對應；標籤不影響權限
  - `createdOn`: Number, Unix Timestamp ms - Not Null
  - `updatedOn`: Number, Unix Timestamp ms - Not Null, 資料最後更新時間

### 帳號 Role 清單 AccountRoles

- **欄位:**
  - `id`: String, UUID/GUID - Primary Key
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵，`Company` 即租戶邊界
  - `accountId`: String, UUID/GUID - Foreign Key to Accounts, Not Null, Index, 持有 Role 的帳號；一個帳號可掛多個 Role
  - `roleId`: String, UUID/GUID - Foreign Key to Roles, Not Null, Index, 被指派的 Role；同一個 Role 可分配給多個帳號
  - `createdOn`: Number, Unix Timestamp ms - Not Null
  - `updatedOn`: Number, Unix Timestamp ms - Not Null, 資料最後更新時間

### 授權角色 Roles

- **欄位:**
  - `id`: String, UUID/GUID - Primary Key
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵，Role 定義每 `Company` 一份
  - `roleTitle`: String - Not Null, Role 名稱
  - `levelId`: String, UUID/GUID - Foreign Key to LevelDefinitions, Not Null, 授權等級；掛 Role 整體，不掛範圍清單的單項
  - `typeAdmin`: Boolean - Not Null, 公司層開關的型別維護；管欄位、欄位組、工單型別配方、狀態流程、結案原因、關聯型別、日曆定義
  - `orgAdmin`: Boolean - Not Null, 公司層開關的組織結構；管 `Team` 與 `Product` 的建立與刪除
  - `permAdmin`: Boolean - Not Null, 公司層開關的權限管理；管 Role 定義、等級定義、標籤群組定義
  - `tags`: Object | Null, JSON - Nullable, 標籤群組識別碼對標籤值識別碼的對應；標籤不影響權限
  - `createdOn`: Number, Unix Timestamp ms - Not Null
  - `updatedOn`: Number, Unix Timestamp ms - Not Null, 資料最後更新時間

### 範圍清單 RoleScopes

- **欄位:**
  - `id`: String, UUID/GUID - Primary Key
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵，`Company` 即租戶邊界
  - `roleId`: String, UUID/GUID - Foreign Key to Roles, Not Null, Index, 所屬 Role；一筆一個範圍項目，可多筆
  - `scopeKind`: String - Not Null, 範圍種類，可為 `company`、`team`、`product`、`mgmt`；整層項目與明列單項可混用，粒度下限為 `Mgmt`
  - `scopeId`: String, UUID/GUID - Not Null, 依 `scopeKind` 指向對應容器，多型指向不宣告單一外鍵；`company` 項填所屬 `Company` 識別碼
  - `createdOn`: Number, Unix Timestamp ms - Not Null
  - `updatedOn`: Number, Unix Timestamp ms - Not Null, 資料最後更新時間

### 等級定義 LevelDefinitions

- **欄位:**
  - `id`: String, UUID/GUID - Primary Key
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵，等級定義每 `Company` 一份
  - `name`: String - Not Null, 等級名稱；內建列為觀看、回報、成員、管理，另可自行加列
  - `system`: Boolean - Not Null, 系統欄；標記內建列，內建列不可刪除
  - `canRead`: Boolean - Not Null, 可讀取工單
  - `canComment`: Boolean - Not Null, 可留言
  - `canCreate`: Boolean - Not Null, 可建單
  - `canEditOwn`: Boolean - Not Null, 可改自己的工單
  - `canEditAny`: Boolean - Not Null, 可改別人的工單
  - `canArchive`: Boolean - Not Null, 可封存
  - `canStructure`: Boolean - Not Null, 可改結構；指新增 `IssueSet`、改 `Mgmt` 名稱、改 `IssueSet` 的 KEY；新增 `Mgmt` 歸該 `Product` 範圍的管理級，不屬公司層開關
  - `canAssignRole`: Boolean - Not Null, 可分派 Role；指把現成的 Role 指給帳號
  - `createdOn`: Number, Unix Timestamp ms - Not Null
  - `updatedOn`: Number, Unix Timestamp ms - Not Null, 資料最後更新時間

### 標籤群組 TagGroups

- **欄位:**
  - `id`: String, UUID/GUID - Primary Key
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵，標籤群組定義每 `Company` 一份
  - `name`: String - Not Null, 標籤群組名稱；內建為帳號的部門與 Role 的分類
  - `target`: String - Not Null, 掛載對象，可為 `account`、`role`
  - `system`: Boolean - Not Null, 系統欄；標記內建群組，內建群組不可刪除
  - `createdOn`: Number, Unix Timestamp ms - Not Null
  - `updatedOn`: Number, Unix Timestamp ms - Not Null, 資料最後更新時間

### 標籤值 TagValues

- **欄位:**
  - `id`: String, UUID/GUID - Primary Key
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵，隨標籤群組定義每 `Company` 一份
  - `tagGroupId`: String, UUID/GUID - Foreign Key to TagGroups, Not Null, Index, 所屬標籤群組
  - `value`: String - Not Null, 標籤值；群組帶固定值清單，不接受自由文字
  - `createdOn`: Number, Unix Timestamp ms - Not Null
  - `updatedOn`: Number, Unix Timestamp ms - Not Null, 資料最後更新時間

### 費率歷史 RateHistories

- **欄位:**
  - `id`: String, UUID/GUID - Primary Key
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵，`Company` 即租戶邊界
  - `tagValueId`: String, UUID/GUID - Foreign Key to TagValues, Not Null, Index, 職級標籤值；費率綁職級標籤，不綁帳號
  - `effectiveOn`: Number, Unix Timestamp ms - Not Null, Index, 生效日；同一職級的費率為一串記錄，各帶生效日
  - `hourlyRate`: Number - Not Null, 每小時費率金額
  - `createdOn`: Number, Unix Timestamp ms - Not Null
  - `updatedOn`: Number, Unix Timestamp ms - Not Null, 資料最後更新時間

### 檢視分享 ViewShares

- **欄位:**
  - `id`: String, UUID/GUID - Primary Key
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵，分享對象限同一 `Company` 內
  - `viewId`: String, UUID/GUID - Foreign Key to Views, Not Null, Index, 被分享的檢視表，指向檢視層共用骨架
  - `targetKind`: String - Not Null, 分享對象種類，可為 `account`、`team`、`company`
  - `targetId`: String, UUID/GUID - Not Null, 依 `targetKind` 指向分享對象，多型指向不宣告單一外鍵；`company` 項填所屬 `Company` 識別碼
  - `shareLevel`: String - Not Null, 分享層級，可為 `readonly`、`editable`、`owner`，對應唯讀、可編輯、擁有者
  - `createdOn`: Number, Unix Timestamp ms - Not Null
  - `updatedOn`: Number, Unix Timestamp ms - Not Null, 資料最後更新時間

---

## App 標準定義資料

### 標準等級 StandardLevels

- **說明:**
  - 內建四等級的八開關真值表，各 `Company` 等級定義表的初始內容；內建列標系統欄，不可刪除
  - Company 建立時植入該 Company 的等級定義表
- **檔案:**
  - `assets/definitions/StandardLevels.json`
- **欄位:**
  - `name`: `String` - 等級名稱
  - `canRead`: `Boolean` - 可讀取工單
  - `canComment`: `Boolean` - 可留言
  - `canCreate`: `Boolean` - 可建單
  - `canEditOwn`: `Boolean` - 可改自己的工單
  - `canEditAny`: `Boolean` - 可改別人的工單
  - `canArchive`: `Boolean` - 可封存
  - `canStructure`: `Boolean` - 可改結構
  - `canAssignRole`: `Boolean` - 可分派 Role

| 等級 | 讀 | 留言 | 建單 | 改自己的 | 改別人的 | 封存 | 改結構 | 分派 Role |
|---|---|---|---|---|---|---|---|---|
| 觀看 | 是 | 否 | 否 | 否 | 否 | 否 | 否 | 否 |
| 回報 | 是 | 是 | 是 | 是 | 否 | 否 | 否 | 否 |
| 成員 | 是 | 是 | 是 | 是 | 是 | 否 | 否 | 否 |
| 管理 | 是 | 是 | 是 | 是 | 是 | 是 | 是 | 是 |
