# CLAUDE.md

本 repo 為 IGotThis 產品 `no1_issue_system` module 的 **Module Spec git**，承載工單系統的落地層規格。

## 配對結構

- 上游：頂層 Product git 的提案層與規劃層，需求編號 Q-01 至 Q-07
- 本 repo：落地層 Spec
- 下游：`no5_product_development/no1_issue_system/` 的 Module Impl git，repo 待建
- 權威配對表由 `decision_framework_router` skill 的 `products_registry.md` 維護

## 目錄說明

- `no1_data_models/` — Model 層：容器骨架、關聯、欄位、權限、日曆的資料結構
- `no2_screens/` — View 層：清單表格、看板、開發順序表等畫面規格
- `no3_logics/` — Logic 層：禁環檢查、彙總計算、工單編號、狀態流程等行為規則

## 內容來源

- 內部設計定案四份位於 KnowledgeVault 的產品管理指南 IGotThis 目錄
- 遷入方式為 spec 化重寫：DDL 等實作細節不進 spec，租戶鍵修訂一併落入
- 遷入前本 repo 為骨架狀態

---

## 撰寫規範

所有規格文件依循 `universal_writing_linter` skill 的通用政策與 `spec_writer` skill 的分層政策。Model、View、Logic 三層的技術深度邊界見 `spec_writer` 的對應政策檔。任何改動前先 consult `decision_framework_router` skill 的上游 review 四問。
