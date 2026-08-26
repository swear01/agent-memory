---
title: CPAchecker VGuide provider routing
project: cpachecker
tags: [vguide, deepseek, muse, gateway]
status: active
updated: 2026-08-26
---

# VGuide provider routing

- `VGUIDE_LLM_PROVIDER` 預設為 `deepseek`。DeepSeek arm 使用
  `DEEPSEEK_API_KEY`，模型由 `VGUIDE_LLM_MODEL`／`DEEPSEEK_MODEL` 選擇；未指定
  API URL 時 Java client 直連 `https://api.deepseek.com/chat/completions`。
- 實驗若明確設定
  `VGUIDE_LLM_API_URL=http://127.0.0.1:35001/v1/chat/completions`，模型仍是
  DeepSeek，但 HTTP 流量經本機 `deepseek-gateway`，不是官方 API 直連。
- 2026-08-20 cthulhu gateway routing：`deepseek-v4-flash` 先走 OpenCode Go，
  再以 Command Code fallback；`deepseek-v4-pro` 走 Command Code Pro。這些是
  DeepSeek model 的上游 route，不是 Muse。
- Muse Spark 1.2 是獨立 arm：必須明確設定
  `VGUIDE_LLM_PROVIDER=meta`，使用 `MODEL_API_KEY`，預設模型
  `muse-spark-1.2`、預設 endpoint `https://api.meta.ai/v1/chat/completions`，固定
  `reasoning_effort=minimal`。
- swear01 NFS hosts 的既有 Meta credential 在
  `~/.config/hapi-runner/meta.env`，變數名是 `META_API_KEY`。Production Java
  transport 需要通用名稱，因此只在 launch process 內做
  `MODEL_API_KEY="$META_API_KEY"` 映射；不要把 key 值複製到 repo、artifact、
  issue 或 shell command output。該檔案權限應維持 `0600`。
- 判讀實驗時同時查 `run_meta.json` 的 `llm_provider`、`model`、
  `llm_api_url`，以及 Java log 的 `VGuide LLM provider/model`；不要只憑 port
  或環境中的 key 推測 provider。
