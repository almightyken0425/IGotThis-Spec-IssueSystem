# 工作日曆資料結構

## 使用者自訂資料結構

### 日曆定義 CalendarDefinitions

- **欄位:**
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵；Company 即租戶邊界
  - `name`: String - Not Null, 日曆名稱；與 `companyId` 組成複合 Primary Key；一份日曆對應一個地區的行事曆
  - `weeklyOff`: Object - Not Null, JSON, 週規則；記每週固定休息的星期代碼清單

### 日曆例外 CalendarExceptions

- **欄位:**
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵；Company 即租戶邊界
  - `calendarName`: String - Not Null, Index, 所屬日曆名稱；指向同一 Company 的日曆定義
  - `date`: String - Not Null, 例外生效日，日期不含時間部分；與 `companyId`、`calendarName` 組成複合 Primary Key，同一日曆的同一日僅一筆例外
  - `isWorking`: Boolean - Not Null, 逐日雙向覆寫的方向；真代表該日由假日改為工作日，用於補班日；假代表該日由工作日改為假日，用於國定假日

---

## 日曆組成標準

- **儲存標準:**
  - 一份日曆由週規則與例外清單組成
  - 日曆定義收在 Company 的定義區，定義區每 Company 一份
  - 一個 Company 可收多份日曆
  - 個人請假不進日曆，同一 Company 內的天數計算保有共同基準

---

## 日曆選用標準

- **儲存標準:**
  - 預設日曆欄位為帳號實體的 `defaultCalendarName`
  - 日曆選用欄位為檢視實體的 `calendarName`
  - 兩欄位皆以名稱指向同一 Company 的日曆定義
- **計算與顯示標準:**
  - 日曆只影響派生的天數，不影響儲存的起訖日
  - 顯示天數之處標明單位與所用日曆
  - 生效日曆的解析規則由檢視層邏輯承載
