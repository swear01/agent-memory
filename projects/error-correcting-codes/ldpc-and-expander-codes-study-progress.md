---
title: 1116 LDPC 與 Expander Codes 學習進度與作業紀錄
scope: projects/error-correcting-codes / 1116 LDPC and expander codes
status: reviewed-by-user-not-assessed
updated: 2026-11-22
---

# 學習進度

- 來源為老師的 `Symbol1/Correct2026/1116.ipynb` 與本次逐步問答。使用者於 2026-11-22 走完全部章節（LDPC 構造、Density Evolution、Bit-Flipping、Belief Propagation / LLR / Min-sum、Lifting 與應用、延伸話題，以及週五／週一 Barg–Zémor Expander Code 作業 7 題），並表示已理解後要求更新記憶與 QMD。
- 狀態為使用者自述理解；尚未以獨立測驗驗證熟練度。

# 關鍵直覺與已釐清的易混淆點

- **Density Evolution（BEC）的兩張不同圖**：
  - `numerical-threshold-minimum.png` 畫的是 $g(x) = \frac{x}{[1 - (1-x)^{d_c-1}]^{d_v-1}}$（無 45 度對角線；通道擦除率 $\epsilon$ 是水平線，須全程低於 $g(x)$ 的谷底 $\epsilon^* \approx 0.4294$）。對 $d_v \ge 3$，因分母為 $O(x^{d_v-1})$，左端 $x \to 0$ 時 $g(x) \to \infty$（雙重保險雪崩收尾），呈現 U 型；對 $d_v = 2$ 無平方，曲線遞增，有解析解 $\epsilon^* = \frac{1}{d_c-1}$。
  - Cobweb 階梯圖畫的是 $x_{\ell+1} = f(x_\ell)$ 與對角線 $y = x$；因 $f$ 遞增且 $x_1 = f(\epsilon) \le \epsilon = x_0$，由歸納法 $x_\ell$ 單調遞減有下界 $0$，必收斂而不震盪。
- **Codeword vs. Stopping set vs. Trapping set**：
  - Codeword support：每個相連 check 連到集合的邊數皆為偶數（$0, 2, 4, \dots$）。
  - Stopping set（BEC / peeling）：每個相連 check 連到集合的邊數皆不為 $1$（$0, 2, 3, \dots$）；不一定為 codeword（高斯消去法仍可能解出）。
  - Trapping set（BSC / AWGN / bit-flipping / BP）：允許少數 degree-1 外部 checks，但被內部 even-degree（亮綠燈）checks 壓過而困住局部解碼器，造成 error floor。
- **Belief Propagation 與 LLR**：
  - 邊上傳遞雙向 extrinsic 訊息；$\beta = \tanh(L/2)$ 使 VN 合併從 $\frac{\beta+\gamma}{1+\beta\gamma}$ 變為 LLR 直接相加，CN 為 tanh rule，硬體常以 min-sum 近似。
- **Expander Codes（Barg–Zémor 作業第 1–7 題）**：
  - Bit 放在二部圖的「邊」上（總長 $N = m\Delta$）；左右頂點各為長度 $\Delta$、相對距離至少 $\delta$（最小距離 $\ge \delta\Delta$）的小線性碼 $A, B$。
  - 第 4 題核心：$S$ 定義為右邊解碼前「至少有一條錯邊」的左邊頂點，故 **100% 錯邊左端點皆在 $S$**；$T$ 中每個右邊頂點因未能由碼 $B$ 修回，至少有 $\delta\Delta/2$ 條錯邊連向 $S$。
  - Expander Mixing Lemma 的棋盤密度直覺：兩邊同除以面積 $|S||T|$ 得 $\frac{e(S,T)}{|S||T|} - \frac{\Delta}{m} \le \frac{\lambda}{\sqrt{|S||T|}}$，即稀疏圖中**禁止出現又大又黑的長方形**；同除以 $|T|$ 直接推出第 4 題上界 $|T| \le \bigl(\frac{\lambda}{\Delta(\delta/2 - |S|/m)}\bigr)^2 |S|$。
  - 第 7 題挑選順序避免循環依賴：先定 $r_A = 1 - h(p) - \epsilon/2$、$r_B = 1 - \epsilon/2$ $\to$ 由 Shannon 結果鎖死與 $\Delta$ 無關的 $\delta = \min(\delta_A, \delta_B)$ $\to$ 選夠大常數 $\Delta$ 同時滿足 $\Delta > 128/\delta^2$ 與 $P_A < \delta/4$ $\to$ 固定上述參數讓 $m \to \infty$。

# w11 作業紀錄

- 位置：`<project-root>/document/學習紀錄/碩一/錯誤更正碼/w11/`，沿用 `w10` 的 A4 11pt 20mm LaTeX 模板，各編譯為單頁 PDF：
  - 週五 `1120`（第 1–4 題）：`1120fri.tex`、`1120fri.pdf`
  - 週一 `1123`（第 5–7 題）：`1123mon.tex`、`1123mon.pdf`
- 已於 2026-11-22 透過內建瀏覽器繳交至 NTU COOL 課程 `錯誤更正碼 (EE5029)` 的 `1120` 與 `1123` 作業並確認顯示「已繳交！」。
