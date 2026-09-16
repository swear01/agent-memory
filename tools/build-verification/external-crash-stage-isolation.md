---
title: "外部工具崩潰先隔離命令階段，再下根因結論"
scope: tools/build-verification
status: active
updated: 2026-09-16
evidence_digest: 8ba01221cd555e6064892bde9d9dbb0b1cc5f3ebfbf33c9e1888ebcbaff019ec
---

# 外部工具崩潰先隔離命令階段，再下根因結論

FAN 發生 segmentation fault 後，歷史助手先猜 shell pipe，再斷言環境；後續逐段測試才回報 read_netlist 與 build_circuit 可用，run_atpg 崩潰。

以未變更輸入和分階段操作縮小失敗位置，保留每段輸出與退出狀態。曾經可用的腳本仍崩潰，不足以排除 binary、輸入或環境變動；定位到命令也不等於找到根因。來源只到重建調查，沒有恢復測試。
