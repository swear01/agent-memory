---
title: CPAchecker VGuide provider routing
project: cpachecker
tags: [vguide, deepseek, muse, gateway]
status: active
updated: 2026-08-26
---

# VGuide provider routing

- 2026-08-26 `swear01/cpachecker` main merge commit `694e719e1a` 起，production
  VGuide 預設且唯一 live provider 是 `meta`，預設模型
  `muse-spark-1.2-contributor`，官方 endpoint
  `https://api.meta.ai/v1/chat/completions`，live key 只讀 `MODEL_API_KEY`。
- 正式 runner 固定以 CPA option 傳入 1024 completion tokens；舊的
  `VGUIDE_LLM_MAX_COMPLETION_TOKENS` 與 `DEEPSEEK_MODEL` env alias 已移除。
  Meta 在 thinking disabled 時送 `reasoning_effort=minimal`。
- DeepSeek live request 已 fail-closed 禁用；只允許在
  `VGUIDE_LLM_REPLAY_DIR` 啟用時重播歷史 exact response。不要再用本機
  `deepseek-gateway` 作 production VGuide live fallback。
- swear01 NFS hosts 的既有 Meta credential 在
  `~/.config/hapi-runner/meta.env`，變數名是 `META_API_KEY`。Production Java
  transport 需要通用名稱，因此只在 launch process 內做
  `MODEL_API_KEY="$META_API_KEY"` 映射；不要把 key 值複製到 repo、artifact、
  issue 或 shell command output。該檔案權限應維持 `0600`。
- 判讀實驗時同時查 `run_meta.json` 的 `llm_provider`、`model`、
  `llm_api_url`，以及 Java log 的 `VGuide LLM provider/model`；不要只憑 port
  或環境中的 key 推測 provider。
