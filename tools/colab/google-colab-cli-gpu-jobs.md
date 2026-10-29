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

## Plan / quota
- Google AI Pro: 200 Colab compute units/month (A100 ≈ 5.3 CU/hr ≈ 37 h).
- H100 locked (needs AI Ultra). Trial memberships excluded.

## Pitfalls
- Default auth is oauth2 → pass `--auth=adc`.
- Invalid `--gpu` value silently becomes A100.
- HTTP 400 on `new` = no access to that accelerator.
- `colab new` 加上第一個 `exec` 可能超過 Claude Code Bash 的 120 s timeout，被移到背景；之後別 `sleep; cat`，用 `run_in_background` 或 Monitor 等。
- 2026-10-29：A100 子 session 在 `colab new` 之後 context 被 compact，回來就 idle，VM `culs-a100b` 空轉約 2 小時（/content 只有預設檔）。交給子 session 的 Colab 工作要附「卡住 15 分鐘就先 `colab stop` 再回報」，主 session 隔一段時間用 `colab --auth=adc ls` 看 VM 是否真的在跑東西。

## Alternative
- Colab MCP: https://github.com/googlecolab/colab-mcp (needs an open browser notebook).
