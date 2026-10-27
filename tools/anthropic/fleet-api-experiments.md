---
title: Claude API 實驗用 fleet 環境
scope: tools/anthropic
status: active
updated: 2026-10-27
tags:
  - anthropic
  - fleet
  - credentials
---

# Claude API 實驗用 fleet 環境

## 2026-10-27 起：預設改用個人 org 的 key

- `~/.secrets` 現在有兩把：`ANTHROPIC_API_KEY_SWEAR`（Swear's Individual Org 的 key `swear-mac`，用的是 $100 credits，2026-11-20 到期；不需要 workspace header）與 `ANTHROPIC_API_KEY_LAB` + `ANTHROPIC_WORKSPACE_ID_LAB`（下方原本那把 identity-linked key 改名）。`ANTHROPIC_API_KEY` 預設等於 SWEAR，沒有 SWEAR 時才退回 LAB。
- 切換：shell 函式 `anthropic-use-lab`（同時設定 workspace ID）、`anthropic-use-swear`（unset workspace ID）。SDK 與 Claude Code 都不會在額度用完時自動換 key。
- 已驗證：Mac（使用者測過兩把）與 mazu 的共用 home（2026-10-27 用 SWEAR 打 `count_tokens` 與 Opus 5.5 生成成功，沒帶 workspace header）。其他主機沒逐台驗證。
- **改檔前就啟動的 shell 和程序，env 裡仍是舊值**（`ANTHROPIC_API_KEY` = lab key）。要指定扣哪把的程式，不要依賴繼承的 env：agentic-ic3-repro 的 `scripts/run_matrix.py` 改從 mazu `~/.config/agentic-ic3/anthropic.env`（0600）讀 key。
- vault（Vaultwarden）裡沒有 Console API key，只有 `Claude Code OAuth token`（`claude setup-token` 訂閱 token，不是 API key）。

## 已驗證狀態（2026-09-30）

- Mac、mazu、cthulhu、athena、valkyrie、zeus、oracle 共 7 台的帳號 `~/.secrets` 有 `ANTHROPIC_API_KEY` 與 `ANTHROPIC_WORKSPACE_ID`；檔案權限為 `0600`。四台 `swear01` Linux 主機共用 home，只從 mazu 寫一次，再逐台讀回。
- 七台各自啟動的新互動 shell 均取得兩個變數，且沒有 `ANTHROPIC_BASE_URL` 覆蓋。各主機對 `GET /v1/models?limit=1` 發送指定 workspace header，皆收到 HTTP 200、非空模型清單，回應 workspace header 與設定一致。這只證明金鑰、workspace 與網路可用；未執行付費模型生成，也未驗證舊程序重載。
- Windows `swop` 尚未部署：外網 SSH TCP/22 逾時，從 mazu 測試舊區網 SSH 也未開通。恢復可用連線後再部署並逐機驗證，不可宣稱 8/8 完成。

## 使用方式與安全界線

- 此 identity-linked key 需要每次 API 請求帶 `anthropic-workspace-id` header；只設定 `ANTHROPIC_API_KEY` 會讓模型清單請求回 HTTP 400。Anthropic CLI 可讀 `ANTHROPIC_WORKSPACE_ID`。Python SDK 要用 `default_headers={"anthropic-workspace-id": os.environ["ANTHROPIC_WORKSPACE_ID"]}`，不能假設 SDK 自動讀取這個變數。官方說明：<https://platform.claude.com/docs/en/manage-claude/authentication>。
- 真實金鑰存於各帳號的 `~/.secrets`，不可放入 Git、筆記、命令參數或工具輸出；本筆記也省略 workspace ID 的值。跨機複製金鑰時透過 SSH stdin 傳送，合併既有檔案並保持 `0600`，不覆蓋其他變數。
- 憑證稽核只能輸出變數名稱與狀態，不要整段顯示 shell 設定。這次檢查 `.bashrc` 時曾意外讓一把無關的既有 OpenAI key 出現在工具輸出；已告知使用者需要輪替，完成輪替尚未驗證。

## Claude Code 訂閱入口（2026-10-17）

上述 API 金鑰仍供真正 API 程式使用。Claude Code 已另外部署[訂閱啟動入口](claude-subscription-launcher.md)，只移除 Claude 子程序認證變數；不要因這份 API 環境筆記把 Claude 改回 API key 登入。新版 fleet 範圍與 Athena 送修排除以該筆記為準。

## 個人 key 優先、lab key 備援（2026-10-27）

- 個人 org：Claude Console「Swear's Individual Org」（使用者個人 Google 帳號；預設進的是 DVlab Team org，建 key 前要先切換）。$100 promo credit，2026-11-20 到期。key 名稱 `swear-mac`，Default workspace，前綴 `sk-ant-usr`（user-linked key，可正常呼叫 API）。
- `~/.secrets` 結構：原 lab 值改名為 `ANTHROPIC_API_KEY_LAB`、`ANTHROPIC_WORKSPACE_ID_LAB`；新增 `ANTHROPIC_API_KEY_SWEAR`；`ANTHROPIC_API_KEY="${ANTHROPIC_API_KEY_SWEAR:-$ANTHROPIC_API_KEY_LAB}"`；預設不設 `ANTHROPIC_WORKSPACE_ID`（lab workspace ID 不適用個人 org）。切換函式 `anthropic-use-lab` / `anthropic-use-swear`。
- SDK 與 Claude Code 沒有額度用完自動換 key 的機制；要全自動需 LiteLLM proxy fallback。
- 已部署並以 `GET /v1/models` 兩把皆 HTTP 200：M5、M1（新建 `~/.secrets` 並由 `.zshrc` source）、swear01 共用 home（mazu/zeus/cthulhu/valkyrie）、oracle。備份 `~/.secrets.bak-20261027`。athena、swop SSH 逾時未部署。
- 教訓：用 box 終端機存 key 時，`xclip`/`xsel` 不存在，誤把 key 貼到命令列而出現在截圖；下次先 `stty -echo; cat > file` 再貼。
