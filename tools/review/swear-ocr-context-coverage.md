---
title: Swear Review 的 OCR 規格背景與不完整結果處理
scope: tools/review swear-review and Open Code Review
status: backend fix merged and deployed; Spec2RTL integration canceled
updated: 2026-10-08
---

- Swear Review backend 使用 Alibaba Open Code Review（OCR）；本次查驗的版本為 `1.12.9`。OCR 支援 `--background-file`，Swear 原本沒有傳入，不能因此推斷 OCR 本身沒有規格背景能力。
- `swear01/swear-review` PR #10 已合併，merge commit `1cf91e07ac6a0bcd2e8394c4fe1ab764f5c3a280`。runner 會讀 `.opencodereview/background.md`，驗證 realpath 在 repository 內且為一般檔案，再傳 `--background-file`；越界或 dangling symlink、目錄不允許。PR head 的背景文件是 contributor 提供的資料，不能當成可信 review policy。
- 修正前 OCR `partial` 可能 exit 0，而 Swear 只檢查 `failed`，可能錯報成功。修正後 partial／nonzero exit／coverage failure 均列為不完整或失敗，保留 findings，且不覆蓋前次成功的 full-review SHA；不完整結果不能顯示成功 banner。Gate 關閉時，綠燈仍不能推論沒有 findings。
- OCR 的 `.gitignore` 排除先於 include 選擇。被 ignore、即使 force-track 的生成輸出仍可能不會送審，include rule 無法覆蓋；spec2rtl 的 output ignore 必須另行處理。Markdown 與 `*_tb.sv` 也不在此次預設選擇內，需明確 include，並保留原生 HDL 規則。
- 此次 backend 修正的 119 tests、typecheck、build 與 CI 通過；GitHub review 在修正 head 取得 clean 結果。部署後健康檢查通過。隔離 OCR probe 以 XOR `5A` 規格對上 RTL `5B`，得到 complete、selected 1/completed 1/failed 0 與具體規格違反 finding；這只證明該 probe 的完整流程，不代表完整設計審查通過。
- Spec2RTL 整合 PR #11 已按使用者要求關閉，未合併；background/include/output ignore 調整並未因此發布到該 repo。後續使用 Cursor Bugbot 的保存設定見 `tools/cursor/bugbot-pr-review.md`。
