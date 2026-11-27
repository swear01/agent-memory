---
title: "新增 drag-drop 路徑要沿用排程附件限制"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 7dceb50df4b3cef03c63615888f65542d12861f3bbcbb78a7ba713af8ea51c64
---

# 新增 drag-drop 路徑要沿用排程附件限制

保存的 bot finding 指出 drag-drop 只檢查 inactive／sending，略過 attach button 與 paste 既有的 pendingSchedule guard，於是可能送出 backend schema 禁止的排程加附件組合。

在共同附件入口保留同一契約，並讓 drop overlay 與游標提示一致反映 disabled。來源回報 overlay 位置與離開行為通過，但未提供排程拒收或真正右側檔案上傳的端到端結果；不能用畫面測試代替整條附件路徑，也不採用其關於合成 DragEvent 絕不可能帶 File 的廣泛說法。

## 檔案 drop 的預設動作

在 dragover 與 drop 識別 `Files` payload 後才 preventDefault。附件 disabled 時仍須阻止瀏覽器開啟檔案，但不加入附件。非檔案拖放保留原生行為，例如把選取文字拖進 composer。歷史 review 確認兩個拖放問題已被處理，仍有 locale 問題，未重跑瀏覽器驗收。見 `git show 2505b5e:tools/hapi/file-drop-default-action-boundary.md`。
