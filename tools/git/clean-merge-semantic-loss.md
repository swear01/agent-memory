---
title: "無文字衝突的 merge 仍可能刪掉被依賴的功能"
scope: tools/git
status: active
updated: 2026-09-16
evidence_digest: 0948e75bab1a18e03a4bee72cd4d230c331ccd902c7aeb76634ca849b22403e1
---

# 無文字衝突的 merge 仍可能刪掉被依賴的功能

助手回報 trace.py 自動合併後失去 insn_trigram_counts，另一側 consumer 與測試仍依賴該欄位；該檔不是未解衝突項，因此只处理衝突清單漏掉問題。

核對自動合併檔案相對於整合前版本的語意差異與 consumer。依具體內容恢復應保留的功能，不能一律採 ours，也不能把沒有 conflict marker 當整合正確。

來源有恢復及測試通過的歷史自述，本次未重跑或驗證遠端 tree。不得由此授權 reset 或覆寫其他工作。
