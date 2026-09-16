---
title: "Checker 解析缺口不等於候選被證明或否證"
scope: projects/pono-llm
status: active
updated: 2026-09-16
evidence_digest: 1274b22f6b34e6b4234293677afebc4a63c41a23ceba9cec17f32a723a96f7f7
---

# Checker 解析缺口不等於候選被證明或否證

歷史助手回報 frozen candidates 含 n-ary add，但第一版 checker 只支援 binary add，因此需要補語法支援再重跑。

解析器未支援合法輸入時，保留為 checker 實作或契約缺口，不產生 proof verdict。修正後仍須通過原本的語意驗證；來源沒有重跑結果，也不證明候選本身正確。相鄰 LLM capture 進度不作 checker 成功證據。
