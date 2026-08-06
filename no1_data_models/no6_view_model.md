# 檢視層資料結構

## 使用者自訂資料結構

### 檢視 Views

- **欄位:**
  - `id`: String, UUID/GUID - Primary Key
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵
  - `name`: String - Not Null, 檢視名稱
  - `ownerId`: String - Foreign Key to Accounts, Not Null, Index, 擁有者帳號；一個帳號可建立多張檢視
  - `viewType`: String - Not Null, 檢視形態，名稱由使用者自訂、系統不解讀語意，如開發順序表、看板、燃盡圖
  - `sourceMgmtIds`: Object - Not Null, JSON, 資料來源 Mgmt 識別清單，儲存形態恆為展開後清單
  - `filterConfig`: Object | Null - Nullable, JSON, 篩選條件
  - `displayLevel`: Number - Not Null, 顯示層級，由工單母子結構起算
  - `columnConfig`: Object | Null - Nullable, JSON, 欄位顯示設定；記錄顯示哪幾欄、欄的先後與欄寬；清單表格依賴此項
  - `calendarName`: String | Null - Nullable, 工作日曆選用，以名稱指向定義區的日曆定義；Null 代表跟隨帳號的預設日曆

**與實例樹的關係**: 檢視層只讀實例樹，不寫入。
- 排序與欄位顯示等檢視自身狀態存檢視層
- 手動排序由檢視排序值承載；分享由權限資料結構的 ViewShares 承載

### 檢視排序值 ViewSortEntries

- **欄位:**
  - `id`: String, UUID/GUID - Primary Key
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵
  - `viewId`: String, UUID/GUID - Foreign Key to Views, Not Null, Index, 所屬檢視；排序值只在該檢視內有意義
  - `issueId`: String, UUID/GUID - Foreign Key to Issues, Not Null, Index, 排序對象工單；同檢視內每工單至多一筆
  - `sortValue`: Number - Not Null, 手動排序值，同檢視內決定先後

**與工單的關係**: 排序值存檢視層，工單不持有排序欄位。
- 無記錄的工單屬該檢視的未排序區
- 被篩選條件排除的工單保留排序值
