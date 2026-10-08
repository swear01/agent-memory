---
title: Cursor Bugbot PR 自動審查設定與驗證界線
scope: tools/cursor GitHub PR review
status: saved settings verified; PR execution pending
updated: 2026-10-08
---

- 使用者要求每個 PR 自動 review。Cursor 個人設定的 Trigger Mode 已由 `Manual Only` 改為 `Every Push`，重新載入仍顯示「Bugbot will automatically review every push to a PR」。此模式也 review 後續推送，適合要求 review 對應最新 PR head 的流程；`Once Per PR` 只在開 PR 時 review，跳過後續 pushes。
- 這是帳號層級設定，套用到所有已啟用 Bugbot 的 repositories，不只本次指定的兩個。沒有另外啟用全部 repositories。
- `DVLab-NTU/spec2rtl`、`DVLab-NTU/hw-benchmark` 的 Bugbot 開關均已啟用，重新載入組織頁後分別用 repository name 搜尋，兩者仍為 checked。`swear01/swear-review` 原本已啟用。
- 目前管理入口為 `https://cursor.com/automations/from-cursor/bugbot`，組織頁在其 organization 路徑。GitHub App 的 repository access 不等於 Cursor repository 的 Bugbot 開關，也不等於自動 trigger；三者應分別核對。
- 此次僅改 Trigger Mode，Autofix 仍為 `Off`。Bugbot 頁面說明 runs 依 agent usage 計費；每次 push review 會增加使用量。
- 2026-10-08 核對兩個 DVLab repositories 都沒有 open PR，因此沒有實際新 PR 自動觸發或 latest-head review pass 的證據。設定保存與審查執行完成需分開報告。
- 使用者取消 spec2rtl 的 Swear App 整合；`DVLab-NTU/spec2rtl` PR #11 已於 `2026-10-08T03:14:53Z` 關閉，未合併。Swear App 維持 private；不要重新開該 PR 或擴大 App 可見性。取消 repo 整合不撤銷 Swear backend 已合併的修正，細節見 `tools/review/swear-ocr-context-coverage.md`。
- 官方文件：`https://cursor.com/docs/bugbot`。後續應重新核對 live settings 與當下 PR head，不能把此次保存設定當成未来的 review 結果。
