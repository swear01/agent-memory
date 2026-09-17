---
title: 錯誤更正碼作業規範、寫作風格與 macOS TeX 環境配置
scope: projects/error-correcting-codes
project: error-correcting-codes
status: active
confidence: high
evidence: >-
  完成台大碩一錯誤更正碼第二週週五題推導與 XeLaTeX 排版，實測驗證 Rust 數值積分；
  解決 macOS BasicTeX user-mode 補包與 CJK 字型 fallback 問題，確立像人類好學生的寫作風格。
created: 2026-09-18
updated: 2026-09-18
tags:
  - error-correcting-codes
  - latex
  - xelatex
  - rust
  - homework-style
  - macos
---

# 課程與專案資訊

* **課程名稱**：台大碩一「錯誤更正碼」（Error Correcting Codes）
* **講義來源**：`https://github.com/Symbol1/Correct2026`（如 `0907.ipynb`、`0914.ipynb`）
* **作業作者**：黃思維（學號：r14k41044）
* **專案路徑**：`<project-root>/document/學習紀錄/碩一/錯誤更正碼/`

# 作業檔案命名規則

依據專案既有規範與檔案系統命名慣例：
* **週五題**：`w<N>fri.tex`、`w<N>fri.pdf`、`w<N>fri.rs`（Rust 驗證程式）、`w<N>fri.txt`（執行輸出）
* **週一題**：`w<N>mon.tex`、`w<N>mon.pdf`、`w<N>mon.rs`、`w<N>mon.txt`
* 作業撰寫與數值程式一律使用 **Rust**，不可使用未經授權的腳本語言替代輸出。

# 人類好作業的寫作風格準則

教授在講義中明確規定：「作業由助教負責改，太浪費時間的對的答案，助教有權力扣到零分。寫作 = 溝通 = 作業的一部份。」

為避免產生典型的 AI 贅字與過度樣板化（AI bloat），作業應遵守以下原則：
1. **文字是膠水，算式才是主體**：
   * 避免「首先讓我們探討……」、「綜上所述……」等說教或填充語言。
   * 只有在公式之間需要邏輯銜接時（如「等先驗下 MAP 即選取最小歐氏距離」、「因能量相同等價於內積最大」）才放簡短的文字橋樑。
2. **節奏不平均（簡單題俐落，難題才展開）**：
   * 簡單直觀題（如單 bit 直傳、重複碼）在 2 至 3 行內快速列式得出結果。
   * 容易卡住或不直觀的題目（如 SPC 排序後最小兩數和 $Y_{(1)}+Y_{(2)}>0$、負值連續積分、硬判決平手機率加權）才詳細寫明分類理由與組合加權。
3. **避免浮點數記憶力展示**：
   * 正文推導中，數值通常保留 3 到 4 位小數（例如 $A \simeq 1.282$、$P_{\mathrm{norm}} \simeq 1.642$）。
   * 僅在結尾的比較總表中視需要保留 4 位小數，切忌整篇作業千篇一律輸出 6 位小數。
4. **精簡結尾（簡評取代長篇大論）**：
   * 總表後僅需 2 至 3 句提綱挈領的 Comparison，說明核心物理直覺（如軟式重複碼無編碼增益、硬判決丟失振幅資訊造成 $1.3\sim 3\text{ dB}$ 損失、高雜訊下 SPC 速率懲罰），不必寫成結案報告。
5. **篇幅嚴格控制**：
   * 整理好的作業長度應控制在 2 至 3 頁內，版面飽滿整齊，方便助教迅速批改。

# macOS XeLaTeX 與環境設定陷阱

1. **BasicTeX 權限限制**：
   * macOS 預設路徑 `/usr/local/texlive/2026basic/` 無 root 權限無法直接 `tlmgr install`。
   * 必須使用使用者模式安裝：`tlmgr --usermode install <package>`（安裝至 `~/Library/texmf`）。
   * 若安裝新套件後出現 format mismatch 錯誤，須以 `fmtutil-user --all` 重新編譯使用者端的 format files。
2. **中文字型備援鏈（CJK Font Fallback）**：
   * macOS 環境通常沒有預裝 Linux/Windows 常見的 `FandolSong` 或 `Noto Serif CJK TC`。
   * 若模板中未設置有效的 CJK fallback，XeLaTeX 會退回 Latin Modern Roman 並回報 Missing character，導致中文字完全空白。
   * 必備安全的字型備援設置：
     ```latex
     \IfFontExistsTF{Noto Serif CJK TC}{
       \setCJKmainfont{Noto Serif CJK TC}
     }{
       \IfFontExistsTF{PingFang TC}{
         \setCJKmainfont{PingFang TC}[AutoFakeBold]
       }{
         \setCJKmainfont{Noto Sans CJK TC}[AutoFakeBold]
       }
     }
     ```
