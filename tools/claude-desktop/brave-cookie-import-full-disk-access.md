---
title: Claude 桌面版匯入 Brave cookie 需要授權 Claude.app 本體的完全取用磁碟
scope: tools/claude-desktop
tool: Claude desktop app
machine: SwairM5
status: active
confidence: high
evidence: 2026-10-19 在 SwairM5 以 TCC.db 與程序啟動時間確認，授權並重開後匯入成功。
created: 2026-10-19
updated: 2026-10-19
tags:
  - claude-desktop
  - brave
  - cookies
  - macos
  - tcc
  - full-disk-access
---

# Claude 桌面版匯入 Brave cookie 需要授權 Claude.app 本體的完全取用磁碟

## 症狀

在 Claude 桌面版的內建瀏覽器按 Import cookies，選 Brave 後出現：

```text
macOS blocked Claude from reading Brave's data. In System Settings → Privacy & Security, allow Claude under Files and Folders or Full Disk Access, then reopen Claude and try again.
```

## 根因

「完全取用磁碟」清單裡有兩個名稱相近的項目，很容易開錯：

- 小寫 `claude.app`，灰色空白圖示。這是內建的 Claude Code 執行檔，位於 `~/Library/Application Support/Claude/claude-code/<version>/<hash>/claude.app`，bundle id 是 `com.anthropic.claude-code`。
- 大寫 `Claude`，橘色圖示，位於 `/Applications/Claude.app`，bundle id 是 `com.anthropic.claudefordesktop`。**負責匯入 cookie 的是這一個。**

只開前者時，在 Claude Code 的 Bash 裡讀得到 Brave 的 `Default/Cookies`，但桌面版匯入仍然被擋。這會讓人誤以為權限已經生效。

另外，Claude.app 必須在授權後用 ⌘Q 完全結束再重開。只關視窗不會套用新權限。

## 診斷

```bash
sqlite3 "/Library/Application Support/com.apple.TCC/TCC.db" \
  "select client,auth_value,datetime(last_modified,'unixepoch','localtime') from access where service='kTCCServiceSystemPolicyAllFiles' and client like '%laude%'"
ps -axo pid,lstart,comm | grep "/Applications/Claude.app/Contents/MacOS/Claude$"
```

要同時看到 `com.anthropic.claudefordesktop|2`，而且 Claude 程序的啟動時間晚於 `last_modified`。讀取系統 TCC.db 需要呼叫端本身有完全取用磁碟權限。

## 解法

1. 系統設定 → 隱私權與安全性 → 完全取用磁碟 → ＋ → 加入 `/Applications/Claude.app`，打開開關。
2. ⌘Q 結束 Claude，再重開。
3. 重新匯入。macOS 可能跳出鑰匙圈提示，要求讀取「Brave Safe Storage」，輸入登入密碼後選「允許」。
