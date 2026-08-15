# IGotThis 工單系統 Spec 規則

- 本 repo 是 Module Spec git
- 產品為 IGotThis
- module id 為 `no1_issue_system`
- 本 repo 承載落地層規格

## 多層配對

- Product git 承載提案與規劃
- Design git 仲裁視覺與互動
- 本 repo 仲裁資料與邏輯
- Impl git 承載產品實作
- 配對以 `decision_framework_router` 的註冊表為準

---

## Spec 分層

- `no1_data_models/`
  - 承載容器與關聯
  - 承載欄位與權限
  - 承載日曆資料結構
  - 套用 `spec_writer` Model 政策
- `no2_screens/`
  - 承載清單與看板
  - 承載開發順序畫面
  - 套用 `spec_writer` Screen 政策
- `no3_logics/`
  - 承載禁環檢查
  - 承載彙總與編號
  - 承載狀態流程
  - 套用 `spec_writer` Logic 政策

---

## 內容邊界

- 來源為既有內部設計定案
- 遷入方式為 spec 化重寫
- DDL 不得進入 Spec
- 租戶鍵修訂已納入規格

---

## 原生工作規則

- 任何改動先使用 `decision_framework_router`
- 所有 Spec 改動使用 `spec_writer`
- Markdown 改動使用 `universal_writing_linter`
- 行為變動要檢查 Impl
- 視覺狀態變動要檢查 Design
- 跨層 branch 名稱必須一致
- 配對 commit 內容必須一致

---

## 相容與漂移控制

- `AGENTS.md` 是本目錄的規則真相
- `CLAUDE.md` 只保留 Claude Code 入口
- 產品規則不得複製回相容入口
- 漂移檢查確認相容入口只含導向規則
