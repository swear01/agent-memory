---
title: Claude API 實驗用 fleet 環境
scope: tools/anthropic
status: active
updated: 2026-09-30
tags:
  - anthropic
  - fleet
  - credentials
---

# Claude API 實驗用 fleet 環境

## 已驗證狀態（2026-09-30）

- Mac、mazu、cthulhu、athena、valkyrie、zeus、oracle 共 7 台的帳號 `~/.secrets` 有 `ANTHROPIC_API_KEY` 與 `ANTHROPIC_WORKSPACE_ID`；檔案權限為 `0600`。四台 `swear01` Linux 主機共用 home，只從 mazu 寫一次，再逐台讀回。
- 七台各自啟動的新互動 shell 均取得兩個變數，且沒有 `ANTHROPIC_BASE_URL` 覆蓋。各主機對 `GET /v1/models?limit=1` 發送指定 workspace header，皆收到 HTTP 200、非空模型清單，回應 workspace header 與設定一致。這只證明金鑰、workspace 與網路可用；未執行付費模型生成，也未驗證舊程序重載。
- Windows `swop` 尚未部署：外網 SSH TCP/22 逾時，從 mazu 測試舊區網 SSH 也未開通。恢復可用連線後再部署並逐機驗證，不可宣稱 8/8 完成。

## 使用方式與安全界線

- 此 identity-linked key 需要每次 API 請求帶 `anthropic-workspace-id` header；只設定 `ANTHROPIC_API_KEY` 會讓模型清單請求回 HTTP 400。Anthropic CLI 可讀 `ANTHROPIC_WORKSPACE_ID`。Python SDK 要用 `default_headers={"anthropic-workspace-id": os.environ["ANTHROPIC_WORKSPACE_ID"]}`，不能假設 SDK 自動讀取這個變數。官方說明：<https://platform.claude.com/docs/en/manage-claude/authentication>。
- 真實金鑰存於各帳號的 `~/.secrets`，不可放入 Git、筆記、命令參數或工具輸出；本筆記也省略 workspace ID 的值。跨機複製金鑰時透過 SSH stdin 傳送，合併既有檔案並保持 `0600`，不覆蓋其他變數。
- 憑證稽核只能輸出變數名稱與狀態，不要整段顯示 shell 設定。這次檢查 `.bashrc` 時曾意外讓一把無關的既有 OpenAI key 出現在工具輸出；已告知使用者需要輪替，完成輪替尚未驗證。
