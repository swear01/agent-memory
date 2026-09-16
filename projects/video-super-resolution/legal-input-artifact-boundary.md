---
title: "RPR 比較錨點不自動成為合法推論輸入"
scope: projects/video-super-resolution
status: active
updated: 2026-09-16
evidence_digest: 30f6a4ca66674888751dfd057ded0822f786215b31a56a5c96ac786c4958ad6e
---

# RPR 比較錨點不自動成為合法推論輸入

使用者要求文件避免把 RPR anchor 當可用輸入；歷史助手區分依賴 bitstream 的 decoder 產物與題目允許的 BL／EL，並提出自行升頻與後處理方案。

依正式輸入契約追溯每個 tensor 的取得路徑，再計算整條管線的成本。不能把某個 anchor 不合法擴張成所有升頻都禁止，也不能把插值、補幀或 refine 提案當成已達品質及 MAC 預算。來源的文件修改與後續模型成效是不同證據。
