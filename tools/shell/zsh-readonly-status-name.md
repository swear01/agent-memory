---
title: "zsh 腳本別把 status 當一般變數"
scope: tools/shell
status: active
updated: 2026-09-16
evidence_digest: 9eeeaac4e25f9fdaf4f5e4f4e1918a9a277c6ec984b6b3d03ce502af227d4d86
---

# zsh 腳本別把 status 當一般變數

助手回報 runData 已成功，但 drift check 的 shell 變數撞到 zsh readonly status，因而需要改別名重跑。

使用任務專屬變數名，並在實際目標 shell 驗證 wrapper；shell 自己的錯誤和被包裝程式結果分開記錄。

來源僅此診斷和重跑意圖，沒有新的 drift 通過證據。Prism 沒启动 Minecraft 與未 quote 路徑是獨立缺陷。
