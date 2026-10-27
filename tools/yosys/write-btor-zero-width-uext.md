---
title: Yosys 0.69 write_btor 對 zero-width 值輸出 `uext S -1 w`，rIC3 與 Pono 都讀不了
scope: tools/yosys
tool: yosys
status: active
created: 2026-10-27
updated: 2026-10-27
tags: [yosys, btor2, write_btor, ric3, pono, verilog-rename]
---

# write_btor 的 zero-width uext

2026-10-27 在 mazu（agentic-ic3-repro W6）發現：LLM-Miners `fifo_vis.sv`（srFIFO 的 `for` 迴圈 shift 寫法）
經 Yosys 0.69 `write_btor -s` 會輸出

```
251 uext 1 -1 1
252 uext 1 -1 1
253 eq 1 251 252
```

`-1` 指向 node 1，而 node 1 是 `sort`，所以是非法 BTOR2：
- rIC3 master（4d63899）panic：`deps/btor-rs/src/parse.rs:26:44 called Option::unwrap() on a None value`，exit code 仍為 0，輸出沒有 verdict。
- Pono：`line 252: 'sort' cannot be used as argument`。

語意上是 zero-width 值補零到 w bit，等於 w 個 0。修法：把 `N uext S -k w`（k 是 sort id）改寫成
`N const S 0…0`（寬度取 sort S）。修完後 orig fifo_vis 用 rIC3 portfolio 597 s 證得 SAFE。
建 bench 後應掃一遍「參數引用到 sort id」的行，不然 baseline 會整排 ERROR 卻看不出原因。

## 其他相關陷阱（同一次工作）
- rIC3 master portfolio log 的 `ic3_xxx proved at depth N` 只是各 worker 推進 frame，不是最終結果；最終只看單獨一行 `UNSAT`/`SAT`。
- 對 riscv-formal（`rvfi_macros.vh`、nerv）做 token 級改名時：(1) 很多 `` `define `` 以空行結尾，刪空行會讓後面的 `` `else``/`` `endif`` 被吞進 macro；(2) `` `` `` token pasting（`rvfi_``name`、`rvfi_csr_``NAME``_rmask`）會在展開時拼出原名，被拼出的名字和 macro 引數都不能改。
