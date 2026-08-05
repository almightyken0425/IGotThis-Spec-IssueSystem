# 彙總邏輯: RollupLogic

## computeRollupValue 計算彙總值

- 依欄位宣告的算法，從下級工單算出彙總值
- **輸入:**
  - 目標工單
  - 可彙總欄位
- **性質:**
  - 純計算，不直接寫入欄位主值
- **收集下級:**
  - **執行:**
    - 沿彙總開關為真的關聯型別，取得目標工單持有的下級工單
  - **行為:**
    - 關聯型別的彙總開關由該 Company 定義區的關聯型別定義宣告
    - 收集範圍限同一 Company 租戶內的工單
- **套用算法:**
  - **執行:**
    - 依該 Company 定義區的欄位定義宣告的彙總算法計算
  - **行為:**
    - 可彙總欄位限四個，其餘欄位皆不彙總
    - `StartTime` 取下級最早值
    - `EndTime` 取下級最晚值
    - `StoryPoint` 取下級值加總
    - `WorkLog` 取下級工時填報加總
- **回傳:**
  - 彙總值，無可彙總下級時回傳空值

---

## resolveEffectiveMode 判定生效模式

- 判定可彙總欄位當下依自動或手動運作
- **輸入:**
  - 目標工單
  - 可彙總欄位
- **執行:**
  - **IF** 目標工單無彙總開關為真的下級工單:
    - **回傳:** manual，葉節點恆等同手動，`IssueFieldValues.rollupMode` 不適用
  - **IF** 欄位為 `WorkLog`:
    - **回傳:** auto，工時由執行者填報，持有端手填無意義
  - **ELSE:**
    - **回傳:** `IssueFieldValues.rollupMode` 當下的值

---

## initializeRollupMode 決定模式初值

- 可彙總欄位尚無 `IssueFieldValues.rollupMode` 值時，決定初始模式
- **輸入:**
  - 目標工單的可彙總欄位
- **執行:**
  - **IF** 欄位已有值:
    - **回傳:** manual
  - **ELSE:**
    - **回傳:** auto

---

## switchRollupMode 切換彙總模式

- 使用者隨時可在 auto 與 manual 之間切換 `IssueFieldValues.rollupMode`
- **輸入:**
  - 目標工單
  - 可彙總欄位
  - 目標模式
- **執行:**
  - **IF** 欄位為 `WorkLog`:
    - **回傳:** 錯誤，工時恆為 auto，不開放切換
  - **IF** 目標模式為 auto:
    - 原手動值不保留
    - `IssueFieldValues.rollupMode` 寫入 auto
    - 呼叫 `computeRollupValue`，結果寫入欄位主值
    - 欄位轉為唯讀
  - **ELSE:**
    - `IssueFieldValues.rollupMode` 寫入 manual
    - 欄位開放人工編輯
    - 下級異動不再覆蓋欄位主值

---

## refreshAutoRollup 下級異動重算

- 下級工單的可彙總欄位異動時，維護持有端的自動模式主值
- **輸入:**
  - 發生異動的下級工單與欄位
- **執行:**
  - 沿彙總開關為真的關聯型別，找出異動工單的持有端
  - 對持有端呼叫 `resolveEffectiveMode`
  - **IF** 生效模式為 auto:
    - 呼叫 `computeRollupValue`，結果寫入持有端欄位主值
    - 欄位維持唯讀
    - 持有端主值因此變動時，向上逐層重複本流程
  - **ELSE:**
    - 主值不覆蓋，偏離標示由 `evaluateDeviation` 承擔

---

## evaluateDeviation 計算偏離標示

- 手動模式欄位在背景計算彙總值，標示與主值的偏離
- **輸入:**
  - 目標工單
  - 生效模式為 manual 的可彙總欄位
  - 採用的工作日曆
- **性質:**
  - 背景計算，彙總值不作為主值
- **偏離判定:**
  - **執行:**
    - 呼叫 `computeRollupValue` 取得彙總值
    - **IF** 彙總值與主值有差距:
      - 標示偏離，附差距值
    - **ELSE:**
      - 不標示
  - **行為:**
    - 不設差距門檻，任何差距皆標示
  - **理由:**
    - 差距多大算嚴重因團隊而異，給數字比替團隊訂線準確
    - 缺此標示，下級超期在持有端完全看不出來
- **天數單位:**
  - **條件:**
    - 欄位為 `StartTime` 或 `EndTime`
  - **行為:**
    - 偏離天數以工作天計，扣除採用日曆的假日
    - 未設定日曆時退回日曆天計算
- **日曆選用:**
  - **行為:**
    - 採用日曆由呼叫端帶入，以名稱指向 `CalendarDefinitions`
- **回傳:**
  - 偏離標示與差距值，差距值附單位與所用日曆
  - 無差距時回傳無偏離
