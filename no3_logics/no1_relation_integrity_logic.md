# 關聯完整性邏輯: RelationIntegrityLogic

## writeRelationTypeDefinition 寫入關聯型別定義

- 危險組合由組合檢查擋下，不由權限擋
- **輸入:**
  - 關聯型別定義資料
- **執行:**
  - **IF** `RelationTypeDefinitions.symmetric` 為真，且 `exclusive` 與 `ordered` 任一為真:
    - **回傳:** 驗證失敗；對稱即無持有端，唯一與排序皆無基準
  - **IF** `RelationTypeDefinitions.rollup` 為真，且 `exclusive` 與 `acyclic` 任一為假:
    - **回傳:** 驗證失敗；非獨佔會重複計算，有環會無限遞迴
  - **ELSE:**
    - 寫入 `RelationTypeDefinitions` 表

---

## deleteRelationTypeDefinition 刪除關聯型別定義

- **輸入:**
  - 關聯型別識別碼
- **執行:**
  - **IF** 該定義的 `RelationTypeDefinitions.system` 為真:
    - **回傳:** 拒絕刪除；內建關聯型別不可刪除
  - **ELSE:**
    - 刪除該筆 `RelationTypeDefinitions` 記錄

---

## createRelation 建立工單關聯

- 寫入前依關聯型別的開關逐項檢查，任一失敗即中止
- **輸入:**
  - 關聯資料，含持有端工單、被指端工單、關聯型別
- **獨佔檢查:**
  - **條件:**
    - 關聯型別的 `RelationTypeDefinitions.exclusive` 為真
  - **執行:**
    - **IF** 被指端在同關聯型別下已存在關聯記錄:
      - 中止寫入
      - **回傳:** 驗證失敗；被指端只能被一個持有端指到
  - **理由:**
    - 主題綁定為獨佔，一張工單只能被一個主題單綁
    - 允許一單多主題 → 彙總重複計算 → 數字失真
- **禁環檢查:**
  - **條件:**
    - 關聯型別的 `RelationTypeDefinitions.acyclic` 為真
  - **執行:**
    - 寫入前自被指端出發，沿同型別關聯查可達性
    - 可達性判斷含零步，持有端等於被指端視為成環
    - **IF** 可達持有端:
      - 中止寫入
      - **回傳:** 驗證失敗；寫入後會成環
  - **理由:**
    - 環在寫入時擋，事後補救成本極高
- **母子邊界檢查:**
  - **條件:**
    - 關聯型別為 Children
  - **執行:**
    - **IF** 持有端與被指端所屬工單集不同:
      - 中止寫入
      - **回傳:** 驗證失敗；母子鏈不得跨工單集
  - **理由:**
    - 跨工單集的連結只有主題綁定一條路
    - 母子再跨 → 兩套機制搶同一件事
- **寫入關聯:**
  - **條件:**
    - 前列檢查全數通過
  - **執行:**
    - 新增一筆記錄至 `IssueRelations` 表
  - **欄位:**
    - `exclusive`: 寫入時複製 `RelationTypeDefinitions.exclusive`
    - `ordinal`: `RelationTypeDefinitions.ordered` 為假時維持預設值
- **反查索引維護:**
  - **行為:**
    - 反查索引隨關聯寫入同步更新
    - 索引為衍生資料，任何時點可由關聯記錄重建

---

## computeIssueLevel 計算工單層級

- 層級由母子結構計算，不由工單型別決定
- **輸入:**
  - 目標工單
- **性質:**
  - 純衍生計算，層級不落任何儲存欄位
- **執行:**
  - 自目標工單沿 Children 反查往上追至根
  - 工單集內無父工單的工單為第一層
  - 自根每往下一段 Children 關聯，層級加一
  - 主題單為錨點，不佔層，不參與計算
- **回傳:**
  - 目標工單的層級
