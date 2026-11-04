---
title: Swear Review 的 OCR 規格背景與不完整結果處理
scope: tools/review swear-review and Open Code Review
status: backend fix merged and deployed; Spec2RTL integration canceled
updated: 2026-11-04
---

- Swear Review backend 使用 Alibaba Open Code Review（OCR）；本次查驗的版本為 `1.12.9`；2026-10-21 起 Oracle 部署為 `1.13.2`（升級與 rollback 見 `projects/swear-review/ocr-upgrade-fixture-capture.md`）。OCR 支援 `--background-file`，Swear 原本沒有傳入，不能因此推斷 OCR 本身沒有規格背景能力。
- `swear01/swear-review` PR #10 已合併，merge commit `1cf91e07ac6a0bcd2e8394c4fe1ab764f5c3a280`。runner 會讀 `.opencodereview/background.md`，驗證 realpath 在 repository 內且為一般檔案，再傳 `--background-file`；越界或 dangling symlink、目錄不允許。PR head 的背景文件是 contributor 提供的資料，不能當成可信 review policy。
- 修正前 OCR `partial` 可能 exit 0，而 Swear 只檢查 `failed`，可能錯報成功。修正後 partial／nonzero exit／coverage failure 均列為不完整或失敗，保留 findings，且不覆蓋前次成功的 full-review SHA；不完整結果不能顯示成功 banner。Gate 關閉時，綠燈仍不能推論沒有 findings。
- OCR 的 `.gitignore` 排除先於 include 選擇。被 ignore、即使 force-track 的生成輸出仍可能不會送審，include rule 無法覆蓋；spec2rtl 的 output ignore 必須另行處理。Markdown 與 `*_tb.sv` 也不在此次預設選擇內，需明確 include，並保留原生 HDL 規則。
- 此次 backend 修正的 119 tests、typecheck、build 與 CI 通過；GitHub review 在修正 head 取得 clean 結果。部署後健康檢查通過。隔離 OCR probe 以 XOR `5A` 規格對上 RTL `5B`，得到 complete、selected 1/completed 1/failed 0 與具體規格違反 finding；這只證明該 probe 的完整流程，不代表完整設計審查通過。
- `Review skipped: no items were selected` 的成因用 OCR 選檔閘門判斷（`ocr review --preview` 會列出每個檔案的 reason）：
  副檔名不在 allowlist 為 `unsupported_ext`（`.md`、`.command` 不在；`.py`、`.sh`、`.v`、`.sv` 在），
  命中內建測試檔規則為 `default_path`（含 `**/test_*.py`、`**/*_test.py`）。這兩項在 1.12.9 與 1.13.2 相同。CVSD2026 PR #14 只改
  `tools/README.md`、`tools/adfp-chromium.command`、`tools/test_adfp_chromium.py`，三者全被濾掉。
  修法是在 repo 加 `.opencodereview/rule.json` 的 `include`（可繞過這兩個閘門，但不能繞過 secret
  路徑與 `vendor/`、`node_modules/`）；`include` 不是白名單，未命中的檔案仍照常篩選。
  CVSD2026 PR #15（`48c3264`）已加入 `{"include": ["**/*.command", "**/test_*.py"]}`，Markdown 刻意不納入；
  `ocr review --preview --from <base> --to <head> --repo <dir>` 不呼叫 LLM，可在合併前驗證選檔結果。
- Spec2RTL 整合 PR #11 已按使用者要求關閉，未合併；background/include/output ignore 調整並未因此發布到該 repo。後續使用 Cursor Bugbot 的保存設定見 `tools/cursor/bugbot-pr-review.md`。

## Verilog context（OCR 1.13.5）

- 不需額外 plugin：內建 `.v`／`.vh`／`.sv` 對應 Verilog／SystemVerilog 規則，涵蓋 latch、位寬與 signedness、時脈/reset、CDC、模擬與合成行為差異；這是模型 review 指引，不能代替模擬、lint、合成或形式驗證。
- `.opencodereview/background.md` 適合簡述已核准的時脈、reset 極性與同步方式、介面、延遲、位寬／signedness 合約，並指向 tracked 規格與驗證檔。OCR 建議不超過 2,000 字元，超過 8,000 字元會拒絕；仍須按上面的 contributor-controlled 信任界線核對。
- `tb_*.sv`／`*_tb.sv` 預設遭 `default_path` 排除；`.opencodereview/rule.json` 的 `include` 可恢復這類預設排除，不能恢復更早被 `.gitignore` 過濾的檔案。新增自訂 `rules` 預設會取代該檔案的內建規則；要補充 RTL 專案約束而保留 HDL 規則，設定 `merge_system_rule: true`。
- 真實無 LLM 的 `ocr review --preview` probe：DUT＋TB 起初 selected 1，加入 include 後 selected 2。這只證明選檔覆蓋，沒有模型準確率、RTL 正確性或合成結果的證據。
- 官方來源：[Verilog 規則](https://github.com/alibaba/open-code-review/blob/v1.13.5/internal/config/rules/rule_docs/verilog.md)、[規則覆寫與選檔](https://github.com/alibaba/open-code-review/blob/v1.13.5/pages/src/content/docs/en/review-rules.md)、[背景文字限制](https://github.com/alibaba/open-code-review/blob/v1.13.5/cmd/opencodereview/background_file.go)。
