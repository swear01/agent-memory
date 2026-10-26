---
title: Fleet Colab CLI / gcloud / uv / GPU 盤點與唯讀 HAPI 稽核手法
scope: machines/hapi-fleet
project: hapi
status: active
confidence: high
evidence: 2026-10-26 逐機唯讀 HAPI session（codex gpt-6-luna low / Claude haiku plan）回報
created: 2026-10-26
updated: 2026-10-26
tags: [colab, gcloud, uv, gpu, hapi]
---

## 盤點結果（2026-10-26）

| 機器 | OS | uv | gcloud | colab | GPU |
|---|---|---|---|---|---|
| mazu | Ubuntu 26.04.1 x86_64 | 0.11.19（NFS home） | 無 | 無 | RTX 4060 |
| valkyrie | Ubuntu 26.04.1 x86_64 | 0.11.19（NFS home） | 無 | 無 | RTX 4090 |
| cthulhu | Ubuntu 26.04.1 x86_64 | 0.11.19（NFS home） | 無 | 無 | RTX 4070 Ti |
| athena | Ubuntu 26.04 x86_64 | 0.11.19（NFS home） | 未驗（離線） | 未驗 | RTX 4090（舊紀錄） |
| zeus | Ubuntu 26.04.1 x86_64 | 0.11.19 | `~/google-cloud-sdk` 存在但不在 PATH，無 ADC | 無 | GTX 1080 Ti |
| swever（OCI） | Ubuntu 24.04.4 aarch64 | 無 | 無 | 無 | 無 |
| swairM5 | macOS 27.0.1 arm64 | 0.11.32 | Homebrew SDK 567.0.0，ADC 存在 | `~/.local/bin/colab`（uv tool google-colab-cli） | Apple Silicon |

- 四台 NFS 共用 home 的 `~/.config/gcloud` 不存在、uv tools 目錄為空；一次在共用 home 裝 uv tool + gcloud（user-level SDK）即可涵蓋四台，但系統路徑仍是各機獨立。

## 唯讀稽核手法

- Codex read-only sandbox 內 `nvidia-smi` 會回「could not communicate with the driver」（看不到 `/dev/nvidia*`），這不是 driver 壞掉；改用 `/proc/driver/nvidia/gpus/*/information` 與 `lspci` 判斷 GPU 型號。
- Codex read-only 內 `uv tool list` 會因 cache lock 失敗；改 `ls ~/.local/share/uv/tools/`。
- swever 的 Codex read-only 失敗（bwrap RTM_NEWADDR），改用 `--agent claude --model haiku --permission-mode plan` 可唯讀完成。
- spawn-peer 偶發 HTTP 502；序列化、上一個 session archive 後重試即可成功。
- Grok box 的 `/var/tmp/hapi-runner/runner.env` 可能被清掉，此時 `hapi-runner-ensure` 靜默退出（flock 目錄不存在），runner 不會上線；仍可用 `~/.secrets/env` 的 token + 公開 hub URL 呼叫 hapi CLI。
