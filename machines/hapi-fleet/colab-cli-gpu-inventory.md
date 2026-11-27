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

# Colab CLI：ADC、Runner PATH 與 GPU 驗證

- NFS 共用 home 的 gcloud 在 `~/google-cloud-sdk`，只加到登入 shell。Zeus 的 swear02 home 是另一份；Oracle 走 `/opt/google-cloud-sdk`、`/usr/local/bin`，uv tools 在 `/opt/uv-tools`。HAPI runner.env 不一定讀 profile，先核對 process PATH，再用已安裝工具的絕對路徑。使用者 ADC 每個 distinct home 一份，不以裝好 CLI 判定登入完成。
- Colab CLI 無 `--version` 時先驗 `--help`，實際使用核准 ADC 及最小 `colab --auth=adc run --gpu A100 check.py`，以 job 內 `nvidia-smi` 驗配置，跑完 `colab --auth=adc sessions` 應空。當時 shared home（從 Valkyrie）／Zeus／Oracle 都成功，Athena 未單獨測。ADC 缺 quota project warning 當時未妨礙 Colab run，與真正授權或配額錯誤區分。
- 唯讀 sandbox 看不到 `/dev/nvidia*` 時，`nvidia-smi` 的 driver error 不能證明 host 驅動壞，先看 `/proc/driver/nvidia/gpus/*/information` 與 `lspci`。uv tool list 可能被 cache lock 阻擋，核對既有 tool 目錄。bwrap 出錯改用已授權的唯讀入口。Grok box 的 HAPI runner.env 從 `/var/tmp` 消失可令 runner 不上線，不把這當 Colab/ADC 問題。
- 官方登入如必須等待使用者交互，tmux 或沒有 tmux 時用 screen 保存待核准程序；從已授權入口完成，驗 ADC 與真實工作。OAuth code 短效，不放在聊天、Git 或 memory。等舊 session 不再需要後再做下一輪登入。

[整理前的完整證據與逐批紀錄](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/machines/hapi-fleet/colab-cli-gpu-inventory.md)
