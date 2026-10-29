---
title: Google Colab CLI GPU jobs
scope: tools/colab
tool: colab
status: active
updated: 2026-10-29
---

# Google Colab CLI (GPU jobs)

- Official: https://github.com/googlecolab/google-colab-cli
- Installed on Mac **SwairM5**: `uv tool install google-colab-cli` (v0.7.5).
- Auth via gcloud ADC (done 2026-10-26):
  `gcloud auth application-default login --scopes=openid,https://www.googleapis.com/auth/cloud-platform,https://www.googleapis.com/auth/userinfo.email,https://www.googleapis.com/auth/colaboratory`

## Usage
- One-shot: `colab --auth=adc run --gpu A100 script.py` — creates session, runs, auto-stops. Verified A100-SXM4-40GB in ~9s.
- Persistent: `colab new -s NAME --gpu A100`, `colab exec -s NAME -f file`, `colab stop`.
- **Always stop sessions** (they burn compute units).

## 平行跑（使用者偏好，2026-10-29）
- 可以拆開的工作（重複測試、多電路計時）要一次開多個 A100 session（`new -s NAME1`、`NAME2`…）分段跑；compute units 按 VM 小時算，總量不變但更快完成。使用者明確要求「之後要記得」。
- 每台各自跑完就 `colab stop`。
- 實測上限（Google AI Pro，2026-10-29）：**A100 同時最多 2 台**。第 3 台起 `colab new` 回「Allocation refused (precondition failed). This can mean too many active sessions, or a temporary usage or capacity limit…」，沒有 HTTP 碼。不必再試更多台；要更多平行度只能換別的加速器。

## Plan / quota
- Google AI Pro: 200 Colab compute units/month (A100 ≈ 5.3 CU/hr ≈ 37 h).
- H100 locked (needs AI Ultra). Trial memberships excluded.

## Pitfalls
- Default auth is oauth2 → pass `--auth=adc`.
- Invalid `--gpu` value silently becomes A100.
- HTTP 400 on `new` = no access to that accelerator.
- `colab new` 加上第一個 `exec` 可能超過 Claude Code Bash 的 120 s timeout，被移到背景；之後別 `sleep; cat`，用 `run_in_background` 或 Monitor 等。
- 2026-10-29：A100 子 session 在 `colab new` 之後 context 被 compact，回來就 idle，VM `culs-a100b` 空轉約 2 小時（/content 只有預設檔）。交給子 session 的 Colab 工作要附「卡住 15 分鐘就先 `colab stop` 再回報」，主 session 隔一段時間用 `colab --auth=adc ls` 看 VM 是否真的在跑東西。
- 2026-10-29：長時間背景工作（nohup）跑 2–3.5 小時後 Colab 會直接收回 VM（`colab sessions` 變空、`download` 回 not found），3 台裡 3 台都遇到；進度日誌要定期 `download` 回本機，別只留在 VM。VM 滿載時 `exec` 常回 `Connection was lost`／timeout，但 `download` 仍可用，輪詢進度用 `download` 一個日誌檔比 `exec` 穩。

## Alternative
- Colab MCP: https://github.com/googlecolab/colab-mcp (needs an open browser notebook).
