# 欄位系統資料結構

## 使用者自訂資料結構

### 欄位組定義 FieldSetDefs

- **欄位:**
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵；Company 即租戶邊界，定義每 Company 一份
  - `name`: String - Not Null, 欄位組識別名稱；單一名詞，不支援命名空間；與 `companyId` 組成複合 Primary Key
  - `system`: Boolean - Not Null, 系統內建旗標；為真代表不可刪除、不可停用

### 欄位定義 FieldDefs

- **欄位:**
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵；Company 即租戶邊界，定義每 Company 一份
  - `name`: String - Not Null, 識別名稱，不可改，程式依此尋找欄位；與 `companyId` 組成複合 Primary Key
  - `fieldSetName`: String - Foreign Key to FieldSetDefs, Not Null, 所屬欄位組
  - `kind`: String - Not Null, 欄位形狀，單值、多筆、關聯三者之一；單值與多筆的值落工單欄位值兩實體，關聯的值由工單關聯資料結構承載
  - `valueType`: String - Not Null, 值型別，值域限值型別原語清單
  - `system`: Boolean - Not Null, 系統欄旗標；為真代表欄位定義不可從系統移除
  - `readonly`: Boolean - Not Null, 唯讀旗標；為真代表值由系統寫入，使用者不可編輯
  - `rollupable`: Boolean - Not Null, 可彙總旗標；可否彙總由欄位定義宣告，不由使用者逐張工單決定
  - `rollupFn`: String | Null - Nullable, 彙總算法，最早、最晚、加總三者之一；不可彙總的欄位恆為 Null
  - `tracked`: Boolean - Not Null, 追蹤開關；為真的欄位每次異動寫進變更歷史
  - `label`: String - Not Null, 顯示名稱，可改，供團隊用語使用；只有一份，跟隨 Company 設定的語言，不隨帳號語言切換

### 工單欄位單值 IssueFieldValues

- **欄位:**
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵
  - `issueId`: String, UUID/GUID - Foreign Key to Issues, Not Null, 與 `fieldName` 組成複合 Primary Key
  - `fieldName`: String - Foreign Key to FieldDefs, Not Null, 對應欄位定義，形狀須為單值
  - `value`: Object - Not Null, JSON, 值格放一個值；無值的欄位不存列
  - `rollupMode`: String | Null - Nullable, 彙總模式，值域 auto、manual，僅可彙總欄位使用

### 工單欄位多筆記錄 IssueFieldRecords

- **欄位:**
  - `id`: String, UUID/GUID - Primary Key, 單筆可獨立定位，支援引用、編輯、刪除
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵
  - `issueId`: String, UUID/GUID - Foreign Key to Issues, Not Null, Index, 可見範圍等同所屬工單，不另設單筆權限
  - `fieldName`: String - Foreign Key to FieldDefs, Not Null, 對應欄位定義，形狀須為多筆；承載留言、附件、工時記錄與變更歷史
  - `value`: Object - Not Null, JSON, 單筆記錄內容
  - `authorId`: String - Foreign Key to Accounts, Not Null, 單筆的作者
  - `createdOn`: Number, Unix Timestamp ms - Not Null, 單筆的時間

---

## App 標準定義資料

### 值型別原語 ValueTypePrimitives

- **說明:**
  - 值型別的固定原語清單，使用者可組欄位配方、不可擴充原語
  - 系統級清單，不植入各 Company 定義區
- **檔案:**
  - `assets/definitions/ValueTypePrimitives.json`
- **欄位:**
  - `name`: `String` - 原語名稱
  - `usage`: `String` - 用途描述
  - `exampleFields`: `Array<String>` - 範例欄位

| 原語 | 用途 | 範例欄位 |
|---|---|---|
| 文字 | 短字串 | Title、BranchName |
| 長文 | 支援 markdown 與截圖 | Description |
| 數字 | 整數與小數 | StoryPoint |
| 日期 | 無時間部分 | StartTime、EndTime |
| 時間戳 | 含時分秒 | CreateTime、UpdateTime |
| 選項 | 固定選項集合 | Status、Resolution、Severity |
| 使用者 | 指向帳號 | Assignee、CreateBy |
| 布林 | 是與否 | ExternalTool |
| 工單參照 | 指向其他工單 | Children、Container |
| 檔案 | 附件本體 | Attachment |

### 標準欄位總表 StandardFieldCatalog

- **說明:**
  - 內建欄位的標準定義，各 Company 定義區的初始內容
  - Company 建立時植入該 Company 的欄位定義與欄位組定義
- **檔案:**
  - `assets/definitions/StandardFieldCatalog.json`
- **欄位:**
  - `name`: `String` - 欄位識別名稱
  - `fieldSetName`: `String` - 所屬欄位組
  - `kind`: `String` - 欄位形狀
  - `valueType`: `String` - 值型別原語
  - `mandatoryLevel`: `String` - 強制範圍
  - `writer`: `String` - 寫入者，系統或使用者
  - `rollupable`: `Boolean` - 可彙總旗標
  - `rollupFn`: `String | Null` - 彙總算法，最早、最晚、加總三者之一；不可彙總恆為 Null
  - `tracked`: `Boolean` - 追蹤開關預設值
  - `readonly`: `Boolean` - 唯讀旗標；系統寫入欄位為真
  - `usage`: `String` - 用途描述

| 強制範圍 | 語意 |
|---|---|
| 必帶 | 每種工單型別都有，配方不可取消 |
| 可選 | 建立工單型別時預設勾選，可自行取消 |
| 純自訂 | 使用者自行建立，不入本表 |

| 欄位 | 欄位組 | 形狀 | 值型別 | 強制範圍 | 寫入者 | 彙總算法 | 追蹤 | 唯讀 | 用途 |
|---|---|---|---|---|---|---|---|---|---|
| IssueKey | 基本 | 單值 | 文字 | 必帶 | 系統 | - | - | 是 | 唯一識別，前綴取自所屬工單集的 KEY |
| Title | 基本 | 單值 | 文字 | 必帶 | 使用者 | - | - | - | 工單辨識 |
| Description | 基本 | 單值 | 長文 | 必帶 | 使用者 | - | - | - | 改動說明 |
| Status | 基本 | 單值 | 選項 | 必帶 | 使用者 | - | 開 | - | 流程狀態 |
| Resolution | 基本 | 單值 | 選項 | 必帶 | 使用者 | - | - | - | 結案原因 |
| Assignee | 基本 | 單值 | 使用者 | 必帶 | 使用者 | - | 開 | - | 負責人 |
| Comment | 基本 | 多筆 | 長文 | 必帶 | 使用者 | - | - | - | 討論記錄 |
| ChangeLog | 基本 | 多筆 | 文字，`value` 為固定結構 JSON，含欄位名、舊值、新值、執行者、時間 | 必帶 | 系統 | - | - | 是 | 欄位變更歷史 |
| Archived | 基本 | 單值 | 布林 | 必帶 | 系統 | - | - | 是 | 軟刪除旗標，取代硬刪除 |
| CreateTime | 基本 | 單值 | 時間戳 | 必帶 | 系統 | - | - | 是 | 稽核 |
| CreateBy | 基本 | 單值 | 使用者 | 必帶 | 系統 | - | - | 是 | 稽核 |
| UpdateTime | 基本 | 單值 | 時間戳 | 必帶 | 系統 | - | - | 是 | 稽核 |
| UpdateBy | 基本 | 單值 | 使用者 | 必帶 | 系統 | - | - | 是 | 稽核 |
| Attachment | 基本 | 多筆 | 檔案 | 可選 | 使用者 | - | - | - | 佐證素材 |
| Children | 關聯 | 關聯 | 工單參照 | 必帶 | 使用者 | - | - | - | 母子關聯 |
| Container | 關聯 | 關聯 | 工單參照 | 主題單專屬 | 使用者 | - | - | - | 主題綁定 |
| Before | 關聯 | 關聯 | 工單參照 | 可選 | 使用者 | - | - | - | 排程依賴 |
| RelatedTo | 關聯 | 關聯 | 工單參照 | 可選 | 使用者 | - | - | - | 無方向的相關連結 |
| StartTime | 專案 | 單值 | 日期 | 可選 | 使用者 | 最早 | 開 | - | 開始日期 |
| EndTime | 專案 | 單值 | 日期 | 可選 | 使用者 | 最晚 | 開 | - | 結束日期 |
| StoryPoint | 專案 | 單值 | 數字 | 可選 | 使用者 | 加總 | 開 | - | 估點 |
| WorkLog | 專案 | 多筆 | 數字 | 可選 | 使用者 | 加總 | - | - | 工時填報 |
| ExternalTool | 專案 | 單值 | 布林 | 可選 | 使用者 | - | - | - | 標記外部工具產出，不計工時 |
| BranchName | 版控 | 單值 | 文字 | 可選 | 使用者 | - | - | - | 對應 branch |
| CommitNo | 版控 | 單值 | 文字 | 可選 | 使用者 | - | - | - | 指向 commit |
| Severity | 品質 | 單值 | 選項 | 可選 | 使用者 | - | - | - | 缺陷嚴重度 |
| TestCase | 品質 | 多筆 | 長文 | 可選 | 使用者 | - | - | - | 功能測試案例 |

- 必帶欄位共十四個，橫跨基本與關聯兩組
- `IssueKey` 與 `Title` 分開，標題可改而識別碼不可改
- `Container` 只進主題單型別的配方，一般工單型別不帶
- 關聯的後續方向不另設欄位，由反查取得
