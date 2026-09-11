---
title: 用模態預算指紋驗證 model id 實際服務的模型
scope: domains/llm-inference
status: active
confidence: high
updated: 2026-09-11
tags: [model-alias, model-id, gateway, vision, prompt_tokens, verification]
---

# 問題

Gateway、proxy 與 aggregator（deepseek-gateway、Command Code、OpenCode Zen/Go 等）常把舊
model id 別名到新模型，或讓 id 名稱與實際服務模型不一致。供應商的 catalog metadata
（provider 的 `models-store.json`、`input:["text"]`、頁面上的 "Released" 日期）也是宣告，
不是服務事實。**model id 字串與 catalog 標示都不能當成「現在跑哪個模型」的證據。**

模型自述同樣沒有證據力。實測對同一條 gateway 路徑問 model/version：

- `content` 常常是空字串；答案只落在 `reasoning_content` 裡。
- reasoning 會抓到 system prompt 的日期幻覺（自稱 current date `2026-05-07`），
  再自行推論出 GPT-4 時代的 knowledge cutoff，最後迴避作答。

# 方法：modality budget 當指紋

純文字模型收到 image part 時，通常直接丟棄（或回 "I cannot see an image."），
因此 **`usage.prompt_tokens` 會對 image 不計價**。同一張圖送同一個 model id：
多模態模型會多出固定的 image token；被打回文字模型則明顯偏低。

實測（16x16 純色 PNG 或點陣數字圖，同一段 prompt）：

| model id | prompt_tokens | 行為 |
| --- | --- | --- |
| gateway → `deepseek-v4-flash`（OpenCode Go 額度用盡，實際走 Command Code） | 235 | 正確讀出圖中數字 |
| Command Code `deepseek/deepseek-v4-flash` | 235 | 正確讀出 |
| Command Code `deepseek/deepseek-v4.1-flash` | 235 | 正確讀出 |
| Command Code `deepseek/deepseek-v4-flash-fast` | 18 | `I cannot see an image.` |

`235` 與 `18` 的差距就是判別點：**舊 V4-Flash 是純文字模型**（官方 changelog 明講
「the text model DeepSeek-V4-Flash ignores the multimodal elements」），所以能讀圖的
`deepseek-v4-flash` 已經不是 V4-Flash，而是被別名到多模態的 V4.1-Flash。

輔助訊號：`system_fingerprint` 相同代表同一 upstream 部署；但不同 fp 不能單獨推論
是不同模型，只當旁證。

# 可重現的探針

不需要外部套件，用 stdlib 產生 PNG（`zlib` + `struct` 手寫 IHDR/IDAT/IEND），
再送 OpenAI-compatible `image_url`：

```bash
# 1) 產生確定內容的圖（點陣數字或純色塊），輸出 base64
# 2) 送同一段 prompt 到待測 model id，比較 usage.prompt_tokens
curl -s "$BASE_URL/chat/completions" -H 'Content-Type: application/json' \
  -H "Authorization: Bearer <api-key>" \
  -d "{\"model\":\"$MODEL\",\"max_tokens\":900,\"messages\":[{\"role\":\"user\",\"content\":[
      {\"type\":\"text\",\"text\":\"What number is shown in the image? Reply with only the digits.\"},
      {\"type\":\"image_url\",\"image_url\":{\"url\":\"data:image/png;base64,$IMG\"}}]}]}" \
  | python3 -c "import json,sys; d=json.load(sys.stdin); print(d['usage']['prompt_tokens'], d['choices'][0]['message'].get('content'))"
```

**判讀以 token 數為主，讀字為輔。** 點陣字形必須無歧義：帶斜線的 `0` 會讓
vision 模型把 `740` 讀成 `749` 或 `743`。字形混淆是模型的 OCR 誤差，不是路由證據。

# 一併要查的三件事

1. 官方 changelog / pricing 頁的 alias 與 retirement 說明（例如
   `deepseek-v4-flash` 為相容而暫時 route 到 V4.1 Flash，canonical id 是 `deepseek-flash`）。
2. 本地 routing 設定：實際上 request 有沒有走到預期的 upstream（gateway log 的
   `[Routing]` 與 `[Upstream Unreachable]` 行，以及 fallback endpoint 的 `upstream_model`）。
3. Fallback 路徑的 `upstream_model`：fallback 若寫死舊 id，主路徑換新模型時
   fallback 仍會回舊模型，兩條路徑的行為會不一致。

# 教訓

只看 model id 字串或 provider catalog 就宣告「你現在用的是 X」會出錯；必須用
能力指紋或 upstream 回應的 `model` 欄位交叉驗證。本檔案的方法就是被使用者更正後
才補上的驗證步驟。
