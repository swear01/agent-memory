---
title: Claude Desktop Remote Control 手機 Offline：重新登入並接回原對話
scope: tools/anthropic
status: active
updated: 2026-11-15
---

手機 Claude Code 的本機對話顯示 Offline 或要求電腦重新登入時，先查 Mac 的 `~/Library/Logs/Claude/remote-control.log` 與 `main.log`。若有 `auth_stale (no_auth)` 或 `session_stale_relogin`，即使 App 有開、Remote Control 開關還亮著，桌面登入仍須更新。CLI 的 `claude auth status` 正常不能證明 Desktop 已登入。

1. 在 **Claude Desktop** 點 **Sign in again**，用同一帳號登入。這次流程在瀏覽器顯示 **Finish sign-in in the Claude app**，再點 **Open Claude** 返回 App。如果舊 OAuth 分頁最後卡在 `localhost` 的 `ERR_CONNECTION_REFUSED`，先確認本機回呼是否仍有程序接收；沒有就回 Desktop 發起新流程，不用舊分頁。
2. 等日誌出現 `auth stale ended`、`watch: connecting → connected` 和工作資料夾重新註冊。這只證明桌面服務恢復。**逐一檢查原本需要手機連線的對話**：一個對話顯示 Connected，其他對話仍可能 Offline。打開原對話；若右側找不到 Remote Control，開啟 **Session rail**，必要時先收起內建 **Browser** 面板。若 Remote Control 關閉就重新開啟，再用 **Open in Browser** 或 Claude Code 網頁清單逐筆確認 **Connected**。若網頁另外要求裝置驗證，用同一帳號重新登入網頁後再查。
3. 舊的登入錯誤會留在歷史訊息。這次已確認兩個原對話在網頁清單均顯示 Connected，並確認其中一個的歷史與輸入欄載入；未代送訊息，也未直接驗證手機。

Mac v5.11492.5 當次重開後一度畫面空白。一次性停用 GPU 後一開始也仍空白，稍後在操作 Help → Troubleshooting 時才出現登入按鈕；沒有證據表明 GPU 旗標或清快取是必要條件。不要為此先重置 App 資料。

來源：[Anthropic Desktop 認證排解](https://code.claude.com/docs/en/desktop#403-or-authentication-errors-in-the-code-tab)、[Remote Control 文件](https://code.claude.com/docs/en/remote-control)。
