---
title: "Merge 檢查不能用錯格式的 grep 判定無衝突"
scope: tools/git
status: active
updated: 2026-09-16
evidence_digest: 168c8300c3949f26670cec616d1e52e8ef42f197385550ae0e542763804dc73e
---

# Merge 檢查不能用錯格式的 grep 判定無衝突

歷史助手用只匹配行首衝突標記的規則漏看 merge-tree 的加號前綴，後讀原始輸出才確認多個 add/add 衝突。

先確認使用的 merge-tree 模式與輸出格式，再判定衝突；保留分歧基底和原始結果。單一文字搜尋沒命中不能取代工具狀態或結構化衝突資料。來源的後續 rebase 與分離功能只是建議，沒有完成整合證據。
