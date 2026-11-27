---
title: 實驗驗收容易誤判的證據界線
scope: global
status: active
confidence: high
tags:
  - evidence
  - experiments
  - benchmarks
  - verification
  - fail-closed
source_refs:
  - agent-memory:issue-10
  - agent-memory:issue-12
  - agent-memory:issue-13
  - agent-memory:issue-16
created: 2026-08-20
updated: 2026-11-28
generated_by: openai-codex/gpt-5.6-luna
---

- **程式有輸出，不代表證明有效。** 由生成器產生的候選先查語法、型別和 SAT／SMT；invariant 須依原模型驗證 initiation、consecution 與目標性質，vector 則用對應的等價性。保留原 verifier 的狀態空間，未驗的猜測不能變成限制條件。
- **Proxy 上升，不代表真實成果變好。** 用完整 oracle 驗證 outcome，列出跳過、不可達類別和分母。個別成功或 owner 回報只證明該次執行與結果，根因和泛化要有各自的對照。
- **舊產物混進新的實驗。** 每次保存 immutable inputs、設定、穩定 task keys 與獨立輸出目錄，按規則記 solved、failed、unknown、timeout。只比較同 build、dataset、seed、solver、parallelism 與 timeout policy 的可比集合。重跑保留 timeout grace 與 dump 路徑，否則觀測條件會改。
- **Smoke 零筆、parser 壞掉或 request 根本沒送，卻算 verifier outcome。** 把 provider、transport、parser、resource 和 telemetry failure 分開記錄。Fallback 如實標示。Dry run 只驗命令建構；replay miss 只驗路徑已到。兩組對照各自跑現行產物，確認 common base 的非實驗行為一致。
- **IDs 與雜湊相同，但衍生 manifest 丟了欄位。** 若宣稱保留 row identity，就比較完整原始物件；另外證明集合互斥及聯集。對有損版本留證據、版本化修正。折疊同一 NFS 或 exact duplicate，衝突測量保留個別來源。
- **掃程序或指令列拿到密鑰。** 發布前掃並就地替換敏感值，留範圍與前後 hash，驗證實驗語意、結果與負載數據未變。原始 snapshot 留在受控來源。

[整理前完整紀錄](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/global/workflows/experiment-evidence-boundaries.md)
