---
title: "Skill description 的冒號空格須符合 YAML scalar 語法"
scope: tools/skillshare
status: active
updated: 2026-09-16
evidence_digest: 0b613b999db461a8fbe7537cec6daa2e2154056e98b6c255164f07ef9bffe88e
---

# Skill description 的冒號空格須符合 YAML scalar 語法

歷史調查回報數個 SKILL.md 的單行未引號 description 含冒號加空格，Pi parser 報 Nested mappings 並略過 skill；使用 block scalar 的其他 description 沒有同一問題。

讓 description 使用合法的引號或 block scalar，先用實際 loader 解析，再同步 canonical source 並核對可用 skill 清單。檔案存在不代表載入成功。

來源有最小 parser/loader 重現的自述，修法當時尚未實施；本次沒有變更 skill 或呼叫同步。
