---
title: 錯誤更正碼作業規範、寫作風格與 macOS TeX 環境配置
scope: projects/error-correcting-codes
project: error-correcting-codes
status: active
confidence: high
evidence: >-
  完成台大碩一錯誤更正碼第二週與第三週週五題推導與 XeLaTeX 排版；
  確立得分導向、禁止論文式冗長總結與背景的極簡精準風格，使用標準英文術語與精準頁數控制。
created: 2026-09-18
updated: 2026-09-25
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
5. **篇幅嚴格控制與扣題邊界（不要寫成課本或維基百科）**：
   * 題目問什麼就答什麼。若題目只要求「將 row 排序使生成矩陣為前 $k$ 列並求 $[2^m, k=?, d=?]$」，給出排序向量、求值矩陣與參數總表即可。
   * **切忌自行附加題目未要求的額外章節或理論長篇推導**（例如滿秩性上三角證明、最小距離非零點引理證明、SPC 對偶性等延伸證明）。課堂作業不是撰寫維基百科條目，額外的代數推導與作業目標無關，會被視為冗餘干擾，應直接刪除。
   * 題目的不同部分銜接應自然（例如第 1 題模型與求值等價性應完整收斂於第 1 頁，第 2 題排列表格收斂於第 2 頁），整份作業嚴格控制在 2 頁俐落呈現。
6. **嚴禁將自用的內部驗證程式寫入正文**：
   * 在本地撰寫的驗證程式（如 `w<N>mon.rs`、高斯消去或窮舉驗證）是給自己確認答案用的。作業正文中切勿出現「附帶之驗證程式（w2mon.rs）完全驗證……」等文字，保持純粹的學生作答風格。
7. **編碼參數與糾錯能力敘述必須數學精確**：
   * 最小距離 $d$ 與錯誤更正能力 $t = \lfloor (d-1)/2 \rfloor$ 不可混淆。例如 $d=4$ 時 $t=1$（可更正 1 個位元錯誤，或偵測 2 個位元錯誤，即 SEC-DED），切勿誤寫為「具雙錯誤更正能力」。
   * 區分全體單項式求值矩陣（Evaluation Matrix $E \in \mathbb F_2^{2^m \times 2^m}$）與特定階數 RM 碼之生成矩陣（$G_{\mathrm{RM}(r, m)} = E_{1:k, :}$ 取前 $k$ 列子矩陣），符號與觀念需嚴格分明。
8. **作業寫作定位：純答題、得分導向（「你是在寫作業，不是寫論文」）**：
   * 這是課堂作業，目標是讓助教快速核對並拿到滿分。
   * 老師在講義（Jupyter Notebook）已有完整定義，作業開頭**不需鋪陳大段教科書式的背景符號定義**，結尾**嚴禁附加論文式的「理論總結與編碼意涵」**等冗餘論述。
   * 直接切入題 1～題 $N$ 作答，每題保留助教審查給分的核心算式即可（例如：左右兩端展開對比、特徵 2 向量加法 $(a+b)+b=a$ 求逆、三類單項式展開搭配 $X_i^2 = X_i$ 次數分析）。
   * 術語優先使用自然標準的英文單字（如 `codeword`、`evaluation points`、`pullback`、`bijection`、`degree`、`multilinear monomial`），避免不自然的中文生硬翻譯（如「碼字」）。
   * 精準控制頁數預算（如 11 題濃縮至剛好 2 頁），每題獨立完整，避免跨頁斷頭或孤兒頁。

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
