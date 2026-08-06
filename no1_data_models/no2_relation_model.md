# 關聯資料結構

## 使用者自訂資料結構

### 關聯型別定義 RelationTypeDefinitions

- **欄位:**
  - `id`: String, UUID/GUID - Primary Key
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵；Company 即租戶邊界，關聯型別定義每 Company 一份
  - `name`: String - Not Null, 識別名稱，同 Company 內唯一；程式與關聯記錄依此對應型別
  - `exclusive`: Boolean - Not Null, 獨佔開關；為真時被指端只能被一個持有端指到
  - `acyclic`: Boolean - Not Null, 禁環開關；為真時同型別關聯不得成環
  - `ordered`: Boolean - Not Null, 有序開關；為真時關聯使用序位值
  - `symmetric`: Boolean - Not Null, 對稱開關；為真時關聯無方向，查詢時雙向合併
  - `rollup`: Boolean - Not Null, 彙總開關；為真時被指端的值往上累計至持有端
  - `system`: Boolean - Not Null, 系統旗標；內建四關聯為真，不可刪除
  - `createdOn`: Number, Unix Timestamp ms - Not Null
  - `updatedOn`: Number, Unix Timestamp ms - Not Null, 資料最後更新時間，同步依據

### 工單關聯 IssueRelations

- **欄位:**
  - `id`: String, UUID/GUID - Primary Key
  - `companyId`: String, UUID/GUID - Foreign Key to Companies, Not Null, Index, 租戶鍵；Company 即租戶邊界
  - `fromIssueId`: String, UUID/GUID - Foreign Key to Issues, Not Null, Index, 持有端工單；關聯只存持有端這一側
  - `toIssueId`: String, UUID/GUID - Foreign Key to Issues, Not Null, Index, 被指端工單；被指端不持有回指欄位
  - `relationTypeId`: String, UUID/GUID - Foreign Key to RelationTypeDefinitions, Not Null, 關聯型別
  - `ordinal`: Number - Not Null, Default 0, 序位；同一持有端同型別內的排列位置，型別的有序開關為假時維持預設值
  - `exclusive`: Boolean - Not Null, 獨佔複製欄；值恆同關聯型別定義的獨佔開關，讓被指端的唯一約束靜態成立
  - `createdOn`: Number, Unix Timestamp ms - Not Null
  - `updatedOn`: Number, Unix Timestamp ms - Not Null, 資料最後更新時間，同步依據

---

## App 標準定義資料

### 標準關聯型別 StandardRelationTypes

- **說明:**
  - 系統內建的關聯型別種子；Company 建立時載入該 Company 的關聯型別定義，並標系統旗標
- **檔案:**
  - `assets/definitions/StandardRelationTypes.json`
- **欄位:**
  - `id`: `Number`
  - `name`: `String` - 識別名稱，值為 Children、Container、Before、RelatedTo 之一
  - `exclusive`: `Boolean` - 獨佔開關
  - `acyclic`: `Boolean` - 禁環開關
  - `ordered`: `Boolean` - 有序開關
  - `symmetric`: `Boolean` - 對稱開關
  - `rollup`: `Boolean` - 彙總開關
- 內建四關聯的用途與開關組合如下

| name | 用途 | 獨佔 | 禁環 | 有序 | 對稱 | 彙總 |
|---|---|---|---|---|---|---|
| Children | 母子關聯，同一工單集內的縱向持有 | 是 | 是 | 是 | 否 | 是 |
| Container | 主題綁定，主題單跨工單集綁定成員 | 是 | 是 | 否 | 否 | 是 |
| Before | 前後關聯，承載排程依賴 | 否 | 是 | 否 | 否 | 否 |
| RelatedTo | 無方向的相關連結 | 否 | 否 | 否 | 是 | 否 |

---

## 關聯儲存標準

- **儲存標準:**
  - 值型別為工單參照的欄位，一律存為工單關聯記錄
  - 純值欄位不落工單關聯
  - 關聯只存持有端一列，單向儲存
  - 寫入路徑只有一條 → 不會出現半邊更新
  - 同一持有端、被指端、關聯型別的組合只存一列
  - 前後關聯只存前對後單一方向
  - 對稱關聯同樣只存一列，不另存反向列
  - 正查索引以持有端、關聯型別、序位為鍵
  - 反查索引以被指端、關聯型別為鍵
  - 獨佔複製欄為真的列，被指端加關聯型別的組合唯一
- **計算與顯示標準:**
  - 反查由系統維護的反查索引承擔
  - 反查索引為衍生資料，任何時點可由關聯記錄重建
  - 反查慢屬效能問題 → 由索引解決，不由資料模型承擔
  - 對稱關聯查詢時合併兩個方向
  - 後續方向不獨立儲存，由反查取得
