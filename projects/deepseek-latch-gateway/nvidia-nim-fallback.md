---
title: CLIProxyAPI 加入 NVIDIA NIM 備援與 insufficient-credits failover
scope: projects/deepseek-latch-gateway
project: deepseek-latch-gateway
tool: CLIProxyAPI v8.0.53
status: active
created: 2026-11-11
updated: 2026-11-11
tags: [gateway, cliproxyapi, nvidia-nim, priority, failover, request-scoped-errors, opencode-go, openrouter, vercel]
---

# 2026-11-11 變更（已驗證）

- NVIDIA NIM key 存 Vault `nvidia-nim`（key 名 `deepseek-flash-vault`，2027-11-11 到期）；`<home>/.secrets` 新增 bare `NVIDIA_API_KEY=`，並加進開頭 `unset` 行。
- `cliproxy.yaml` 的 `api-keys.openai-compatibility` 第一組新增 `nvidia-nim`：base `https://integrate.api.nvidia.com/v1`，只對應 `deepseek-ai/deepseek-v4.1-flash` → alias `deepseek-flash`（NIM 沒有 V4 Pro）。`priority: -1`，其他群組預設 0，所以 NIM 只當最後備援。
- 四個舊群組（opencode-1/2、command-code-1/2）加 `request-scoped-errors: [{status: 400, match: ["insufficient credits"], action: continue-and-cooldown}]`。CLIProxyAPI 預設 400 不 failover（只對 403/408/429/5xx），所以 Command Code 沒額度時新對話會直接 400；加這條後會換下一組並 cooldown。
- 套用範圍：Mac、mazu（swear01 NFS home，cthulhu/valkyrie 共用同檔）、su_zeus（swear02）、Oracle；各自重啟 `deepseek-gateway` 並實測 HTTP 200。**沒有做**：swop（Windows）、Athena（送修，只是共用 NFS 檔案已變）、zeus swear01 那個 gateway 未重啟。
- 回復點：各 home `<gateway-config-dir>/archive/nim-<timestamp>/`（含舊 `.secrets` 與 `cliproxy.yaml`）。
- 同日稍後 transfer_MAC PR #85–#88 已把這些併入模板／renderer：`DEEPSEEK_GATEWAY_NVIDIA=1`、`DEEPSEEK_GATEWAY_OPENROUTER=1`、Radeon 都是 opt-in、priority -1，全部群組都有 insufficient-credits 規則，另加 command-code-3；14:45 已重新 render 到 fleet，zeus swear01 也在 14:46 重啟。swop 從 Mac SSH timeout，HAPI 狀態也沒回應，尚未確認。

# 行為與坑

- CLIProxyAPI group `priority` 越大越優先，可以是負數。但已建立的 session affinity（含 LCP prompt-prefix hash）優先於 priority，測路由時要換新的 prompt 或 session，不然會沿用舊綁定。
- `save-cooldown-status: true` 會把 cooldown 寫進 `auth/*.cds`；swear01 NFS home 上幾台共用，一台遇到 NIM 500 就讓其他台一起冷卻。
- NIM 免費試用：首 byte 實測 65–115 s，非串流 280 s 會 timeout；五台同時打同一把 key 回 500。Opencode Go 沒帶 `x-opencode-session` 會回 400 `MissingSessionID`（真 client 有帶，測試要自己加）。
- build.nvidia.com 的「API Key Granted」對話框按 Copy 會關掉，key 只顯示一次；app 內建瀏覽器的剪貼簿跟 macOS 不互通（`pbpaste` 拿不到）。

# 2026-11-11 DeepSeek V4.1 Flash 供應商比價（每 1M tokens，離峰）

- OpenCode Go：$10/月可用 $100 額度（照官方價扣），等效約 input $0.015、output $0.06、cache $0.0003；每 workspace 只能訂一份（使用者用不同帳號）；5h $20、週 $50 上限。Go Plus $40 換 $200，只有 5 倍，比較不划算。
- Vercel AI Gateway 的列表價 $0.022/$0.30 是拼湊不同 provider 的最低值；同一家同時最便宜的是 `pareto` provider $0.075/$0.30、cache $0.0015（尖峰加倍，p50 TTFT 約 1.1 s、116 tok/s）。要指定 provider 才拿得到。Vercel 免費月額度不含 DeepSeek。
- OpenRouter 沒有 Vercel 的 pareto provider；OpenRouter 的 `openrouter/pareto-code` 是用 `min_coding_score` 自動挑模型的 router，跟 Vercel 的 pareto 無關。DeepInfra flex（0.8×）是 best-effort，實測 uptime 很低，不適合當主力。
- 免費：AMD Radeon Cloud（`developer.amd.com.cn`）有 `DeepSeek-V4.1-Flash-1017`；ModelScope 每日約 100 次。兩者都沒實測。AMD Radeon 後續已實測並全 fleet 啟用，見 `radeon-fleet-fallback.md`。
- 使用者已在 2026-11-11 儲值 OpenRouter（`OPENROUTER_OFFICIAL_API_KEY`）；DeepSeek 官方 key 餘額為負、已停用。
