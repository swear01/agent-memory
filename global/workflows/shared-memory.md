---
title: Shared persistent agent memory workflow
scope: global
status: active
created: 2026-08-18
updated: 2026-09-11
tags:
  - shared-memory
  - qmd
  - github
---

# Shared persistent agent memory workflow

這個 repository 是跨 agents、projects、sessions 與 machines 共用的 persistent engineering memory。Markdown 檔案是 canonical source。

使用 QMD 的 `memory` collection 搜尋與讀取記憶；不要把 QMD 的 SQLite/vector index 當成 canonical data。

GitHub repository `swear01/agent-memory` 負責在不同 machines 之間同步這些 Markdown 記憶。新增或更新已驗證的長期經驗後，先 `git pull --rebase`，再 commit、push，並重新執行 QMD update/embed。

記憶有變更時，QMD update/embed/query-check 成功只證明本機索引可用，不代表遠端已同步。完成前必須確認 working tree clean、目前 branch 不再顯示 `ahead`，且 local `HEAD` 等於 remote branch；若 push 缺少授權或失敗，應詢問或明確回報未完成，不得跳過後宣稱完成。若只有 QMD read/update 而 Markdown 沒有變更，則沒有 Git commit 或 push 可做。

這個 repository 的 CI 會檢查可達 Git 歷史；commit 前確認 author 與 committer 都使用 GitHub noreply email。個人信箱一旦進入遠端歷史，後續一般 commit 無法消除，必須重寫歷史。

QMD 2.8.3 會在 collection 根目錄執行 `update` hook，與呼叫 `qmd update` 時所在的 working directory 無關。`memory` collection 使用 `git pull --ff-only`，只接受可 fast-forward 的同步；若本機與遠端分歧，停止並人工處理，不讓索引更新自動 rebase Git 歷史。

QMD 2.8.3 的 `qmd update` 結尾會用預設 embedding model 計算 pending hashes；使用自訂 `QMD_EMBED_MODEL` 時可能誤報全部文件需要 vectors。執行一次 `qmd embed` 後，以其 `All content hashes already have embeddings` 結果及 `qmd status` 的 document/vector counts 為準，不因該提示重複 embed。

若 `qmd embed` 在 GPU VRAM 緊張時回報 `Failed to create any embedding context`，先用
`qmd doctor` 確認 device probe 與可用 VRAM；這不是 Markdown／Git 資料遺失。以
`QMD_FORCE_CPU=1 QMD_EMBED_PARALLELISM=1 qmd embed` 做單工 CPU retry，完成後再用
`qmd doctor` 驗證 `embedding freshness` 與 vector sample。不要因 auto-GPU context 失敗而略過
embedding 或重建 canonical Markdown。

# CI validator 的禁項（寫 note 前先自查）

`.github/workflows/validate-memory.yml` 用 `git ls-files` 掃 tracked files，所以**先
`git add` 再驗**，否則新檔不會被檢查；它同時掃描完整 Git 歷史（`git log -p`），
已進入歷史的 secret 無法靠改 working tree 修好。本機重現方式是從 workflow 抽出
`run:` 區塊當 script 執行（先 `git diff --check`，再跑其中的 python inline 檢查）。

- 非 README 的 `.md` 必須以 `---\n` 開頭並含第二個 `\n---\n`（frontmatter）。
- 絕對路徑禁：以 `home` 或 `Users` 為第一層的絕對路徑（CI 用「slash 接 home/」與
  「slash 接 Users/」兩種 regex 擋，連 CI 自己的 workflow 檔都用字串串接繞開），
  改用 `<remote-home>`、`<worktree-root>` 這類 placeholder。
- secret regex 不只抓真 key，也抓寫法。實測踩到的例子：把 shell 變數直接接在
  `Bearer` 後面會命中，因為 `<authorization 或 bearer> + 冒號或等號` 這條規則只把
  `Bearer <...>` 與 `${...}` 當成可接受的 value；照 repository 慣例寫 angle-bracket
  placeholder（`Bearer <api-key>`）才通過。
- 這類檢查是純 regex，不認 Markdown code span：把觸發字串原文寫進 backtick 裡一樣會
  讓 CI 失敗，只能用改寫或 placeholder。本節自身就是被 validator 退回三次才寫成的。
- 其他會命中的形式：`sk-` 加 20 字以上、GitHub token 前綴、AWS access key 前綴、
  Google API key 前綴、PEM private key 的 BEGIN/END 標頭行、JWT、以及
  「key/token/password/secret/credential/cookie 等關鍵字 + 冒號或等號 + 非 placeholder 值」。
- 一般 email 形式字串，以及 commit author/committer 的個人信箱都會失敗。
