---
title: CVSD HW2 雙處繳交與驗證收尾
scope: projects/ntu-cvsd-adfp
status: v2 submitted-and-verified; grading pending
updated: 2026-10-20
---

- **找不到有效版本或拿舊排程當成績。** 2026-10-20 v2 兩端已繳交，課程 `verify_adfp.sh` 四組 PASS，lint 0 errors／4 warnings，p1 3,935、p2–p4 各 96,302 cycles。私有 repo `hw2/SUBMISSION.md` 為完整收據入口。先核 ADFP home、COOL 回執及下載檔，再查助教收件、p5–p7、lint 分數與最終成績。
- **硬體推算自動省掉 projection／index 採樣。** Q41 允許 vertex/index 反向讀，Q36 可在 v/n/f 固定後收完 Mv 與 Mp[0][0]（第 17 word）開始。寬高推導 Mp[0][0] 有歧義，本案兩公式可同時過公開資料卻在 51% 寬高組合不同。p2–p4 被 index 96,330 words 限制，合規下界 96,298；Q51 只有證明畫面不變才可略過。valid 前預讀的舊成績不採用，公開模型不代表 hidden tests。
- **一端封裝省 report、另一端照抄。** 官方 `studentid_hw2_vk.tar` 要小寫學號並在 Linux 打包；ADFP 包原課程報告，COOL 可省。兩端都核解包檔案、必要 `01_rtl`、file list／include，不帶助教 testbench／memory、DesignWare 或 build outputs。用 receipt 下載原檔比對，選檔或「正在繳交」都不算。COOL 沒彈 chooser 時，先監看 filechooser，再對實際 input 按 Enter。
- **Guacamole 有合成 drag/drop 錯誤，或遠端腳本跑到別處。** 用實際 file input 上傳，選定目標 terminal，tcsh 下要跑 Bash 語法就明確 `bash -c`。`_Py_LegacyLocaleDetected` 表示課程啟動器選到不相容 Python，可用 Bash 驗 lint summary，不改共享環境。VCS license 暫時拒絕時用 `-licwait`／`+vcs+lic+wait` 和重試，與 RTL failure 分開。

[完整原文](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/projects/ntu-cvsd-adfp/hw2-closeout.md)
