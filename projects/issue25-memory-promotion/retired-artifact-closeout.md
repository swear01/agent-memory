---
title: Issue25：移除退役索引前核對替代檢索與程序引用
scope: projects/issue25-memory-promotion
status: active
updated: 2026-09-28
evidence_digest: c8e28f9f5a98bd013a20c6b92548c75a72195554dbea5ab497015f64d5e35abb
---

- **工作流程換版，舊 shard、assembled DB 或 QMD tree 還占用大空間。** 將原始來源、唯一模型答案、來源綁定和最終帳冊列為保留；按實際用途與相依性辨認可重建衍生品。核對絕對位置、歸屬與 symlink，查 worker 的 fd/maps/cwd/root/exe、子掛載、systemd unit 和設定引用。相同檔名或大小只作搜尋線索。
- **舊索引可能仍是 rollback。** 先確認替代檢索已驗證並核對核准範圍，再按唯一路徑移除退役項，讀回確認路徑消失。保留小型 hash manifest、設定及核驗報告，保全原始來源和正式索引；歷史釋放量與管理員備份位置見完整收據。

[完整原文](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/projects/issue25-memory-promotion/retired-artifact-closeout.md)
