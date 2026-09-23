---
title: "Skillshare：將 article-writing 從上游安裝改成自行維護"
scope: tools/skillshare
status: active
updated: 2026-09-23
tags:
  - article-writing
  - metadata
  - content-tampered
---

# 將上游安裝的 skill 改成自行維護

Skillshare v0.20.25 的 `<skillshare-source>/.metadata.json` 保存一般安裝 skill 的來源與內容雜湊。`article-writing` 全面重寫後，舊紀錄仍指向 ECC 的 `skills/article-writing`，使 audit 回報 `content-tampered`；一般 `update` 也會依此紀錄重新安裝上游內容。只改 `SKILL.md` 或其 frontmatter 不會移除這筆追蹤。

使用者明確要求將 `article-writing` 改成自己的版本。經核對後，只移除 metadata 的 `entries.article-writing`，保留剛重寫的內容、其他 skill 紀錄及各工具指向自有來源的同步連結。此變更已由 `shared-skills` PR #36 合併，merge commit 為 `10c3925`。

驗證結果：

- `skillshare list article-writing --json -g` 仍列出 skill，但不再有上游 `source` 與 `type`。
- `skillshare audit article-writing --format json --threshold high -g` 回報零 findings、零 warnings；沒有停用掃描或改寫雜湊來掩蓋差異。
- `skillshare update article-writing --dry-run --json -g` 回傳 `no valid skills to update`（exit 1），符合已無上游可更新項目的狀態。
- `skillshare sync -g --json` 完成後，本機六個工具仍讀取相同的自有來源，內容雜湊與重寫版本一致。

這是使用者授權的來源所有權切換，不是所有 integrity 警告的通用解法。只有決定自行維護的 skill 才移除其上游紀錄；其他差異仍需核對來源與變更原因。GitHub 保存、當前機器同步與其他機器部署要分別驗證，不能互相推定。
