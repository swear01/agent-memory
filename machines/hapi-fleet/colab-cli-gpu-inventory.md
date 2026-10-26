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

## 最終狀態（2026-10-26 23:30 驗證）

| 機器 | OS | uv | gcloud | colab | ADC＋A100 測試 | 本機 GPU |
|---|---|---|---|---|---|---|
| NFS 共用 home（mazu/valkyrie/cthulhu/athena） | Ubuntu 26.04 x86_64 | 0.11.19（`~/.local/bin`） | 590.0.1（`~/google-cloud-sdk`） | uv tool `google-colab-cli` | 通過（從 valkyrie 驗證） | 4060 / 4090 / 4070 Ti / 4090 |
| zeus（swear02 home） | Ubuntu 26.04 x86_64 | 0.11.19 | 580.0.0（`~/google-cloud-sdk`） | 0.7.5 | 通過 | GTX 1080 Ti |
| swever（OCI，即 oracle） | Ubuntu 24.04 aarch64 | 0.13.5（`/usr/local/bin`） | 590.0.1（`/opt/google-cloud-sdk`，連結到 `/usr/local/bin`） | 0.7.5（`/opt/uv-tools`） | 通過 | 無 |
| swairM5 | macOS arm64 | 0.11.32 | Homebrew 567.0.0 | 0.7.5 | 原本就可用 | Apple Silicon |

- A100 測試：`colab --auth=adc run --gpu A100 check.py`（nvidia-smi）拿到 A100-SXM4-40GB，跑完自動 terminate，`colab --auth=adc sessions` 為空。
- ADC 帳號為使用者的個人 Google 帳號；每個 home 一份 `~/.config/gcloud/application_default_credentials.json`（NFS 四台共用一份）。
- 每次登入都提示沒有 quota project（可用 `gcloud auth application-default set-quota-project`），對 colab run 無影響。
- Runner PATH 注意：共用 home 與 zeus 的 gcloud 只加在 `~/.bashrc`／`~/.profile`，HAPI runner.env 的 PATH 不含 `~/google-cloud-sdk/bin`；HAPI 起的 agent 要自己加 PATH 或用絕對路徑。swever 在 `/usr/local/bin` 不受影響。
- athena 安裝當下離線，未單獨驗證（共用 home 理論上同樣可用）。

## 唯讀稽核手法

- Codex read-only sandbox 內 `nvidia-smi` 會回「could not communicate with the driver」（看不到 `/dev/nvidia*`），這不是 driver 壞掉；改用 `/proc/driver/nvidia/gpus/*/information` 與 `lspci` 判斷 GPU 型號。
- Codex read-only 內 `uv tool list` 會因 cache lock 失敗；改 `ls ~/.local/share/uv/tools/`。
- swever 的 Codex read-only 失敗（bwrap RTM_NEWADDR），改用 `--agent claude --model haiku --permission-mode plan` 可唯讀完成。
- spawn-peer 偶發 HTTP 502；序列化、上一個 session archive 後重試即可成功。
- Grok box 的 `/var/tmp/hapi-runner/runner.env` 可能被清掉，此時 `hapi-runner-ensure` 靜默退出（flock 目錄不存在），runner 不會上線；仍可用 `~/.secrets/env` 的 token + 公開 hub URL 呼叫 hapi CLI。

## 安裝過程與教訓

- NFS 共用 home（從 valkyrie 安裝）：`~/google-cloud-sdk` SDK 590.0.1（官方 tarball，不跑 install.sh）＋ `uv tool install google-colab-cli`；PATH 行加在 `~/.bashrc`／`~/.profile`。runner.env 的 PATH 不含 gcloud（未修改）。ADC 已登入，`colab --auth=adc run --gpu A100 check.py` 拿到 A100-SXM4-40GB，跑完自動釋放（`colab --auth=adc sessions` 為空）。
- zeus：沿用既有 SDK 580.0.0、colab 0.7.5，ADC＋A100 測試同樣通過。
- ADC 會提示沒有 quota project（`gcloud auth application-default set-quota-project`），對 colab run 不影響。
- swever：用 ubuntu 的 `sudo -n` 系統層安裝：uv 0.13.5 在 `/usr/local/bin`、SDK 590.0.1 在 `/opt/google-cloud-sdk`（symlink 到 `/usr/local/bin`）、colab 用 `UV_TOOL_DIR=/opt/uv-tools UV_TOOL_BIN_DIR=/usr/local/bin`。swever 沒有 tmux，pending login 改用 `screen -dmS colab-adc`。
- 手法：HAPI Codex `safe-yolo` spawn 一律 HTTP 502，Claude `acceptEdits` 會卡在 Bash 授權；安裝只能用使用者核准的 `yolo`。`colab` 沒有 `--version`，用 `--help` 驗證。`inspect-peer` 會截斷長訊息，URL 要用 `fold -w 150` 拿。pending login 放在 detached tmux/screen，之後用 `tmux send-keys` 貼 code，不需保持 HAPI session。
- 透過 HAPI ping 把使用者給的 OAuth code 交給 Codex session 輸入：valkyrie／zeus 的 session 有照做；swever 的 session 兩次都拒絕輸入（即使說明使用者已核准），ADC 仍未完成。Google 授權 code 很快過期，被拒後需要新的 URL／code。
- swever 最後改由使用者核准走 SSH（從 Mac 的 `oracle` alias；box 沒有 ssh）完成：`screen -S colab-adc -X stuff '<code>\n'` 貼 code，ADC 存好，A100-SXM4-40GB 測試通過並自動釋放。至此 shared home、zeus、swever 三處 Colab CLI＋ADC 都驗證可用。
- 坑：遠端 `pkill -f "application-default login"` 會殺到自己的 shell（同一行命令字串含該 pattern）。kill 與重新啟動要拆成兩個 ssh 命令，或用 `[a]` 技巧且命令內不再出現原字串。
- OpenCode 在 swever 上 `opencode-go/deepseek-flash` 回 Model not found，會悄悄改用其他模型；指定模型時要確認實際使用的模型。
