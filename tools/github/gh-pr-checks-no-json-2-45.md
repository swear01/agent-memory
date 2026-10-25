---
title: gh 2.45 pr checks has no --json; poll loops silently never fire
scope: tools/github
tool: gh
machine: oracle
status: active
confidence: high
tags:
  - gh
  - ci
  - polling
created: 2026-10-25
updated: 2026-10-25
---

# gh 2.45 的 `gh pr checks` 沒有 `--json`

oracle 上的 `gh` 是 2.45.0（Ubuntu 套件）。`gh pr checks <n> --json name,bucket`
會直接失敗：`unknown flag: --json`。

如果 poll 迴圈寫成 `gh pr checks ... --json ... 2>/dev/null || echo '[]'`，錯誤被吞掉，
每輪都拿到空結果，迴圈永遠不會判定完成。2026-10-25 一個 30 分鐘的 Monitor 因此
零事件過期，實際上 CI 早就全過了。

改用 tab 分隔的純文字輸出判斷：

```bash
until out=$(gh pr checks <n> -R <owner>/<repo> 2>/dev/null) \
  && [ -n "$out" ] && ! grep -qP '\tpending\t' <<<"$out"; do sleep 30; done
echo "$out"
```

每行格式是 `name<TAB>pass|fail|pending<TAB>duration<TAB>url`。同一版 `gh` 的
`gh pr edit` 也會因 projectCards GraphQL 錯誤失敗，要改用
`gh api -X PATCH repos/<owner>/<repo>/pulls/<n>`。
