---
title: "expandvars 不等於 Bash 預設值展開"
scope: tools/python
status: active
updated: 2026-09-16
evidence_digest: 0dc60b7bc04237fd2faee643776f1e9abc448ff5f8f0432593b29342d817b271
---

# expandvars 不等於 Bash 預設值展開

歷史助手檢查 YAML 載入流程時發現，設定使用 ${VAR:-default}，但載入器交給 os.path.expandvars；預期的 Bash 預設值語意沒有被該函式支援。來源止於發現問題並表示要修改，沒有修正後結果。

設定格式與讀取器必須使用相同的展開語法。若只需要 Python 的環境變數替換，就避免宣稱支援 Shell 的預設值運算；需要預設值時在設定層明確處理，並檢查變數存在、缺失及空字串的約定。不要為展開設定而執行整段 Shell 內容。
