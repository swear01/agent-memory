---
title: HAPI canonical gist 同步：現況與歷史分檔、API 快取延遲
scope: tool
tool: hapi
status: active
created: 2026-10-26
updated: 2026-10-26
tags: [hapi, gist, skill, docs-sync]
---

# HAPI canonical gist 同步

## 分檔規則（2026-10-26 起）

- Gist `hapi-readme.md` 只放現況：值改變就原地更新，不追加帶日期的 rollout 段落。
- 帶日期的 rollout、審計、事故紀錄寫進同一 gist 的 `hapi-history.md`；它不鏡像進 skill，agent 預設不載入。
- 原因：舊檔以流水帳方式累積到 92 KB，約 35% 是帶日期的完成敘述，現況被埋住，agent 也容易把舊版本號當成現況。拆分後現況檔約 25 KB，skill 的 `config.md` 只留機器值（約 10 KB）。
- shared-skills `hapi/tests/test_hapi_session_preservation.py` 限制 `config.md` < 12 KB、`gist-latest.md` < 30 KB；超過代表又開始往現況檔追加歷史。

## API 快取延遲

- `gh api -X PATCH gists/<id>` 之後，未認證的 `curl https://api.github.com/gists/<id>`（`hapi-ctl.sh fetch-gist` 使用）可能約一分鐘內仍回舊內容，`gh api` 已是新內容。
- 刷新鏡像前先輪詢 API 回傳的 `files["hapi-readme.md"].size` 等於新值，再跑 `fetch-gist`。
- `gh api ... --jq '.content'` 會多加一個結尾換行，不能拿來和鏡像做 byte 比對；以 `fetch-gist` 的輸出為準。

## 精簡大型文件的做法

- 刪除內容時用逐行檢查保證不遺失：原檔每一個非空行，都必須逐字出現在新現況檔或歷史檔中。
- 子 agent 標成「已被子 skill 涵蓋」的政策要抽查；本次有一筆 swop 備份位置與 Windows bootstrap 版本陷阱被漏掉，是人工複查時補回的。

Related: [[supervisor-safe-operations]]
