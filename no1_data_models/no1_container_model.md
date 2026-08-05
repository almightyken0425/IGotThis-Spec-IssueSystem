# 容器骨架資料結構

## 使用者自訂資料結構

### 公司 Companies

- **欄位:**
  - `id`: String, UUID/GUID - Primary Key, 租戶鍵；Company 即租戶邊界
  - `name`: String - Not Null, 名稱由使用者自訂，系統不解讀語意

**與定義區的關係**: 定義區歸屬 Company，與 Team 平行，每 Company 一份。
- 收欄位定義與欄位組定義
- 收關聯型別定義與其開關旗標
- 收工單型別的欄位組配方
- 收狀態流程定義、結案原因選項與日曆定義
- 工單集不設可收型別限制，唯主題單位置例外

**與檢視層的關係**: Company 下轄實例樹、定義區、檢視層三支柱；檢視層讀實例樹，資料結構由檢視層資料模型承載。

### 團隊 Teams

- **欄位:**
  - `id`: String, UUID/GUID - Primary Key
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵兼歸屬外鍵；嚴格樹狀歸屬，僅屬一個 Company
  - `name`: String - Not Null, 名稱由使用者自訂，系統不解讀語意

### 產品 Products

- **欄位:**
  - `id`: String, UUID/GUID - Primary Key
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵
  - `teamId`: String, UUID/GUID - Foreign Key to Teams, Not Null, Index, 歸屬外鍵；僅屬一個 Team，不允許多重歸屬；跨團隊檢視由權限解決，不由結構解決
  - `name`: String - Not Null, 名稱由使用者自訂，系統不解讀語意

### 管理域 Mgmts

- **欄位:**
  - `id`: String, UUID/GUID - Primary Key
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵
  - `productId`: String, UUID/GUID - Foreign Key to Products, Not Null, Index, 歸屬外鍵；僅屬一個 Product，不允許多重歸屬
  - `name`: String - Not Null, 名稱由使用者自訂，系統不解讀語意；Requirement 收需求端，Project 收執行端
  - `containerIssueSetId`: String, UUID/GUID - Foreign Key to IssueSets, Not Null, 主題單位置現任工單集

### 工單集 IssueSets

- **欄位:**
  - `id`: String, UUID/GUID - Primary Key
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵
  - `mgmtId`: String, UUID/GUID - Foreign Key to Mgmts, Not Null, Index, 歸屬外鍵；僅屬一個 Mgmt，不允許多重歸屬
  - `name`: String - Not Null, 名稱由使用者自訂，系統不解讀語意
  - `key`: String - Not Null, 工單編號前綴，每 Company 內唯一
  - `nextSeq`: Number - Not Null, Default 1, 工單流水號序列；只前進不回收

**與工單的關係**: 嚴格樹下接工單，資料結構由工單與型別資料結構承載。

---

## 租戶鍵標準

- **儲存標準:**
  - 全實體帶租戶鍵，`companyId` 指向 Companies
  - Company 自身以 `id` 承載租戶鍵，不另設欄位
  - 定義與實例皆以 Company 為邊界，不跨 Company 共用
