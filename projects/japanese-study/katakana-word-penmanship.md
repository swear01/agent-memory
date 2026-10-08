---
title: 日文片假名單字練習紙：六字七字雙面版
scope: projects/japanese-study
status: active
created: 2026-10-08
updated: 2026-10-08
tags: [日文, 片假名, katakana, 練字紙, 官方詞表, 雙面列印]
---

# 片假名單字練習紙

## 使用者需求與固定版型

使用者常記得單字讀音及平假名，卻想不起片假名寫法；單獨反覆練假名效果不佳，改用實際外來語與人名、地名練習。中文意思作為提示，淡字描完後可遮住左側、看中文在右側回想寫法。

- A4 直向，每面 22 行、每行 14 個連續 13 mm 方格；中間沒有格子間隙。
- 左側 7 格淡字描一次，右側 7 格空白自行寫一次。統一練兩遍；六字詞每側剩下一格，不能再塞單字母補強。
- 每行分隔豎線固定在第七格後，不依詞長移動；整頁六字詞全部在前，七字詞全部在後，複習詞也留在各自字數組內。
- 中文意思只放最右側 18 mm 欄（含 2 mm 間距）；左、右頁邊距 5 mm，上下各 5.5 mm。沒有可見標題、頁碼或讀音。
- 長音「ー」、小假名各占一格；六／七字是實際書寫字元數，不能按音節數計算。
- 列印採 A4、黑白、100% 原尺寸、雙面長邊翻頁；PDF 內嵌 DuplexFlipLongEdge 與 PrintScaling=None，仍需向印表機明確指定並讀回實際設定。

## 已完成版本與來源

2026-10-08 版本共 313 個不同單字：184 個六字詞、129 個七字詞；加 39 行複習詞補滿 352 行，共 16 面、雙面 8 張紙。307 詞出現在官方來源，另外 6 詞出自既有《大學日本語》第一、二冊。

- 國立國語研究所 NINJAL「日本語教育基本語彙データベース」共 11,827 筆教育詞彙對照資料，原研究為 2009 年報告；不是當前常用頻率排行榜。官方頁面 mmsrv.ninjal.ac.jp/brfvep/，實際 CSV 檔名 rokusyutaisyo.csv（cp932）；來源頁面的連結文字與實際檔名拼法不同，應用實際 href。
- 國際交流基金 IRODORI 入門、初級 1、初級 2 合併詞表共 4,247 筆、3,676 個不同見出し；官方 resources.html 下 wordlist_all.xlsx。
- 國際交流基金 MARUGOTO 繁體中文詞表涵蓋入門到中級 2；中級 1 有 2,878 筆、中級 2 有 3,633 筆，跨冊需去重。
- 不把 Tシャツ、Jポップ、JR、SOS 的全片假名讀音當成慣用片假名拼寫。IRODORI Excel 的フランダンス與初級 2 PDF 的フラダンス不一致，採 PDF 拼法後為五字，不納入本版。
- 人名及少用地名允許納入，但缺乏可確認中文意思或疑似拼寫錯誤的資料不硬湊。
- 淡字沿用 KanjiVG r20260714 向量筆畫，© Ulrich Apel，CC BY-SA 3.0；原始來源及授權留在附檔與 PDF 中繼資料。

## 可重用檔案

工作根目錄：`<user-documents>/Codex/2026-10-08/new-chat-3/`。

- 最終 PDF：`outputs/片假名單字練習紙_六字七字_雙面.pdf`。
- 詞表、完整來源連結及授權：`outputs/詞彙與來源.txt`；版面預覽：`outputs/版面預覽.png`。
- 產生器：`work/build-katakana-words.py`；字詞與逐行配置：`work/katakana-word-manifest.json`；原始官方資料：`work/sources/`。
- PDF SHA-256：`502d7dc8b01713b01331564ee99c9f1d4ded1ad07f798007385a6dedc3ec9ae4`。
- 16 頁與全部格線配置已解析驗證，六字／七字交界頁已目視檢查。保留既有單假名及漢字練字紙；不要拿舊生成器覆蓋這版。

列印機的紙匣、IPP 與完成判定參照 `machines/swairM5/kyocera-ecosys-m6635cidn-ipp-print.md`。這次列印授權只限一份，不代表以後可自行重印。

## 2026-10-08 實際列印驗證

使用者要求本版雙面列印一份，並記錄記憶及 QMD。先核對實驗室 Kyocera ECOSYS M6635cidn 型號與既有設備 UUID，Validate-Job 接受後只送出一個 Print-Job，指定紙匣 2、A4、黑白、copies=1、sides=two-sided-long-edge、print-scaling=none。Get-Job-Attributes 讀回相同雙面與不縮放設定，最終 job-state=completed、job-state-reasons=job-completed-successfully、job-impressions-completed=16，因此本次完成 16 面／8 張 A4；未在現場目視檢查紙張。

本次直接經本地 en0 到達既有實驗室印表機，不能宣稱 VPN tunnel 已建立；曾執行既有連線捷徑，最後以斷線捷徑恢復原先無 VPN tunnel 狀態。實際設備識別與完成判定優先於推測網路狀態。
