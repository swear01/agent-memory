---
title: Spec2RTL CI must preserve the original project flow
scope: projects/spec2rtl
status: active
updated: 2026-10-18
---

# CI 的流程邊界

使用者明確要求：CI 必須先完整執行原本 Spec2RTL 的 orchestrator、skills 與工具流程，之後再檢查生成 RTL 的正確性與 PPA。被評估的就是原本流程，CI 不得自行改造它。

- 保留原本的步驟順序、必要產物、驗證、除錯與失敗處理規則；原本流程允許的並行仍可使用。
- 外部正確性檢查與 PPA 是流程完成後的驗收，不得為縮短時間自行提前插入中途、將結果回饋成新的 agent 修復流程，或取代原本的 testbench/assertions/alignment。
- 不得藉 CI 優化擅自修改 skills、提前 alignment/assertions、刪減必要步驟或增加原本沒有的設計任務。若需要改變原本流程，必須作為獨立提案，另取得使用者明確授權。
- 縮時應在保留原本流程的前提下評估資源、SDK 開銷、已授權的固定 baseline 快取與 benchmark 大小；不能只為達到時間目標就改變評估內容。
- 先前提出的「在 RTL/lint 後提前執行獨立 VCS 檢查」方案已被使用者否決，不得沿用為待執行計畫。

此要求由使用者於 2026-10-10 直接重申，並要求加入共享記憶；不是由 CI 失敗或模型推論自行得出的政策。


## 2026-10-18：本輪 CI 研究停止

使用者要求記錄結論後停止；等待新指示才繼續調查、修改或模型實驗。排程已刪除，不重建。保留既有部署、workflow、固定參照與全部失敗證據；這不是停用正常 GitHub CI 觸發。

- 已完成：skill PR #69 合併，controller `72719716` 部署至兩個 Mazu runner，91 項 native tests 通過。Grok4.7 Medium／Fast=false，Oracle Opus5.5 High／Fast=false。固定參照來源run38370030437保留，AT使用mapped area×cycles×STA通過週期，5%退步為warning。
- 最新run39025717999：180.28分鐘、FAIL。Keccak77.20分鐘後缺lint摘要；工具紀錄確認SpyGlass跑過，但模型沒有寫summary。BCH180.11分鐘逾時；347項生成的定向測試通過不等於獨立六個pattern通過。RTL未改、架構文件補正仍造成20.39分鐘alignment重驗；DC最後已跑36分15秒仍在mapping。兩design均無有效最終獨立功能／STA／AT結果。
- 前一輪run39012273541的Keccak獨立3/3與scored STA通過，雜湊AT卻比固定參照+362.15%：cycles不變、面積2.10倍、週期2.20倍。具體問題是動態lane讀寫的長mux/XOR路徑，以及共用round datapath承諾未落實而alignment漏判。
- 結論：CI已有可追溯的部署與固定參照，但生成品質、產物收尾、保守重驗及EDA成本仍限制可靠度與速度。沒有達成穩定完整CI／10–15分鐘目標；最新結果也不足以判定#69改善AT，或把全部問題歸因於Grok／skill無效。

不手修生成RTL、不重抽求綠、不自動重建baseline。CI-only維護的專案規則是隔離驗證後直接整合develop、不開PR；skill仍用獨立PR，#69由使用者明確授權合併。

證據：`<task-root>/outputs/skill69-promotion-20261017/closeout-20261018.md`、`final-result-39025717999.json`、`keccak-diagnosis.json`及該目錄的原始run資料；部署收據在`<task-root>/outputs/handoff-stability-20261017/deployment-v2.json`。
