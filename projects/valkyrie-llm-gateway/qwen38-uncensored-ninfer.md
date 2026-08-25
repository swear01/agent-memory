---
title: Valkyrie Qwen3.8 Uncensored NInfer production
scope: projects/valkyrie-llm-gateway
project: valkyrie-llm-gateway
machines: [valkyrie, zeus]
tags: [qwen3.8, ninfer, bifrost, mtp, llm-inference]
status: active
created: 2026-08-25
updated: 2026-08-25
---

# Valkyrie Qwen3.8 Uncensored NInfer production

## 架構與呼叫邊界

- Valkyrie 執行 NInfer；Zeus 的 Bifrost 是唯一對使用者分發的 OpenAI-compatible gateway。
- 使用者模型 ID 以 Bifrost `/v1/models` 回傳的 `valkyrie-ninfer/qwen3.8-27b` 為準；未加 provider 前綴的 `qwen3.8-27b` 目前也能由既有路由解析，但不應據此判定模型清單缺失。
- Gateway 位址是實驗室私網端點，只能經 LAN、VPN 或 SSH tunnel；只分發 Bifrost Virtual Key，不分發 NInfer 後端憑證。

## 生產 artifact 與回滾

- 來源：`orcarouter/Qwen3.8-27B-Uncensored`，固定 commit `9878936be9458522b5aeed0e13476bb8426f57f0`。
- 來源快照為 BF16：18 個 safetensor shards、1,199 個 tensors；含 15 個 MTP tensors。
- 以 NInfer revision `b6172f2423a128142845745bd41e371267a95305` 轉換為 `qwen3_8_27b-v1` groupwise-int artifact。
- 生產檔：`/opt/valkyrie-infer/models/qwen3_8_27b.ninfer`，18,210,531,328 bytes，SHA256 `314e2812942b2b078f341530f6f5f5e46d79b08d668b77ed4d8f58e12fda41f4`。
- 轉換報告：同路徑加 `.conversion.json`，SHA256 `02214a8e95b7cdb8656a7a1c1ebc279aead6c73a869e41687ae6a4d62c3ea5d5`。
- 回滾檔：`qwen3_8_27b.stock-eec39564993d.ninfer`，SHA256 `eec39564993d6e9c7d5e383382a760f093465c9d163ec9a1bd6b80199514bf3e`。

## 已驗證狀態

- `valkyrie-infer.service` 已啟用且運作；既有設定保留 262,144 context、單一 concurrency、16 queue、MTP 3 draft tokens 與 LM-head draft。
- Direct NInfer 與 Zeus→Bifrost authenticated chat 均通過；MTP 實測 3.00 token/round、acceptance 66.7%，短測 decode 約 129–131 tok/s。
- Bifrost `/v1/models` 回 HTTP 200，模型 ID 是 provider-prefixed；先前「沒列出」是查詢腳本用未加前綴 ID 且誤期望頂層 `.object` 所造成的假陰性。

## 拒答與截斷的判讀

- 問模型「遇到惡意請求會不會拒絕」只測到自我描述，不能證明直接行為；必須用實際請求與完成狀態判讀。
- 將明確拒答、完成回答、`finish_reason=length` 截斷分開統計。若正文空白但 reasoning 持續到 token 上限，是截斷／未定，不是拒答。
- 安全的一般維運請求優先設 `reasoning_effort: "none"`；需短推理時用 `low`。預設 thinking 曾在安全唯讀請求上耗盡 4,096 tokens，而 `none` 能正常完成。
- 「uncensored」不代表更強或可放寬控制；仍保留私網、Virtual Key、日誌與授權邊界。
- 精確的 NInfer groupwise artifact 尚未執行完整 harmful-prompt benchmark；同家族 GGUF 與模型作者數據只能作旁證。

## 維護檢查

- 模型替換後依序核對 source commit、artifact SHA256、conversion report、systemd/Compose 狀態、`/v1/models` provider-prefixed ID、authenticated chat 與 MTP telemetry。
- 記憶與文件不得保存 API key、token、後端憑證或敏感測試內容。
