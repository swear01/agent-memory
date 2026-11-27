---
title: "Java generator 不能直接使用 JVM binary class name"
scope: projects/magic-storage
status: active
updated: 2026-09-16
evidence_digest: 37119240624b0d0d5aa12b9f0c47dd879c54ca7625e990b62f281c9903bdb204
---

- **Scanner 能識別 class，產生的 Java 卻無法引用。** 保留 JVM binary identity 和獨立 `source_class`，沿 `InnerClasses` 的完整 owner chain 正規化 Java source。合法 top-level 名稱可含 `$`；末端類名看似具名也可能有 anonymous／local／synthetic owner，整條 chain 有不可定址成員就排除。
- **修 owner 規則後仍重用舊分類。** 讓語意變更使 classifier cache 失效（歷史修補 2→3），用負例核對 scanner/audit、contract 對應和所有 generator shapes。歷史 owner 修補回報 RED→GREEN 及全套測試；未證實 ATM10／IE 實際掃描、完整 generator migration 或發布。

[原始紀錄](https://github.com/swear01/agent-memory/blob/2505b5e7b1170ecc3a61858e10894b2af847b5b7/projects/magic-storage/jvm-binary-versus-source-class.md) · [完整 owner chain 與快取](https://github.com/swear01/agent-memory/blob/2505b5e7b1170ecc3a61858e10894b2af847b5b7/projects/magic-storage/nested-class-source-addressability.md)
