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

## Alternative
- Colab MCP: https://github.com/googlecolab/colab-mcp (needs an open browser notebook).
