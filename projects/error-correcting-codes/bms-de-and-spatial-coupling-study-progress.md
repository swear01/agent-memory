---
title: 1123 BMS、Density Evolution 與 Spatial Coupling 學習進度與作業紀錄
scope: projects/error-correcting-codes / 1123 BMS channels, irregular LDPC, spatial coupling
status: reviewed-by-user-not-assessed
updated: 2026-11-27
---

# 學習進度

- 來源為老師的 `Symbol1/Correct2026/1123.ipynb` 與本次逐步問答。使用者於 2026-11-23 至 2026-11-27 走完全部章節（Sum-Product Rule、BMS Channels、Density Evolution on BMS、Irregular LDPC Codes、Spatial Coupling、週五第 1–4 題、週一第 5–8 題與額外練習 1–2），並已理解後要求更新記憶與 QMD。
- 狀態為使用者自述理解；尚未以獨立測驗驗證熟練度。

# 關鍵直覺與已釐清的易混淆點

- **Sum-Product Rule 與對偶域**：
  - VN（平行條件獨立觀測）在 likelihood / LLR 域計算（LLR 相加）；CN（串聯 XOR 和）在 $\mathbb{F}_2$ Fourier / bias 域計算（$\beta = \tanh(L/2)$ 相乘，即 2 點 DFT 把 pmf convolution 變乘法）。
  - 樹上算精確 posterior；有 cycle 時同一份證據繞回重複計入導致 LLR 偏高（girth 越大越好）。
- **BMS Channels、LLR 對稱性與 Symmetric Gaussian**：
  - BPSK 送 `0` $\mapsto +1$、LLR 分子為 `x=0`，故機率 `0.5` 對應 LLR 中立點 `0`，送 `0` 與送 `1` 的分布以 `0` 左右鏡像對稱。
  - 通道對稱性在 LLR 座標下表現為 $\Pr(L=-t)=e^{-t}\Pr(L=t)$（非獨立新條件，而是鏡像對稱的座標變形）；若為高斯則鎖死為 Symmetric Gaussian $\mathcal{N}(m, 2m)$（變異數必須是平均值的 2 倍，否則高度比與刻度自相矛盾）。
  - **BSC 混合模型**：給定 $|L|=t$（與輸入無關），翻轉率鎖死為 $p=\frac{1}{1+e^t}$，且接收端從 $|L|$ 知道這次抽到的 $p$；故任何 BMS 皆等價於依 mixing distribution $g(p)$ 抽 $\operatorname{BSC}(p)$，證明不等式可「先對 BSC 證再取期望值」。
- **BMS 的 Density Evolution 夾擠與同容量極限**：
  - $B = \mathbb{E}[e^{-L/2}] = \mathbb{E}[\operatorname{sech}(L/2)]$ 在 VN 相乘（Laplace transform 對稱中心兼最低點 $s=1/2$）；$T = \mathbb{E}[|\tanh(L/2)|] = 1-2P_e$ 在 CN 相乘。
  - 一般 BMS 被夾在 $\operatorname{BEC}(B)$（上界）與 $\operatorname{BEC}(2P_e)$（下界）之間；同 capacity $C$ 下 BEC 的 $B$ 最小、BSC 的 $B$ 最大（$(3,6)$ 在 $C \ge 0.72$ 必成功）。
- **Irregular LDPC、Stability、Flatness 與 EXIT Area Theorem**：
  - **Degree-2 的兩難**：沒有 degree-2（$\lambda_2=0$）曲線在原點斜率為 $0$ 趴在地上浪費容量；放太多則原點斜率 $\epsilon\lambda_2\rho'(1) > 1$（如同傳染病 $R_0 > 1$ 連成 degree-2 鎖死迴圈），撞到 Stability condition 而在終點前反彈卡死。
  - **Right-regular 與代價**：固定 CN degree 為 $r$（圖右側整齊），用反函數 $1-(1-x)^{1/(r-1)}$ 展開截斷湊 $\lambda(x)$；在 EXIT chart（CN 曲線 $y=1-\rho(1-x)$ 與 VN 曲線 $x=\epsilon\lambda(y)$，僅 VN 經過通道故含 $\epsilon$）上，兩線夾出的通道面積精確正比於 $(1-R)-\epsilon$。把通道擠窄雖逼近容量，卻導致最大 VN degree 暴增（$N=256$）且迭代輪數暴衝（21 輪 $\to$ 253 輪）。
- **Spatial Coupling 與 Potential $U(x)$**：
  - 把 $M$ 個 regular code 排成鏈、VN 邊往後連 $w$ 格；兩端 CN 較輕率先解開，形成兩道解碼波往中間推進。
  - 位能 $U(x)$ 中只要 $x=0$ 比右邊卡死谷底更深（$U(x)>0$），波就會推過整條鏈，讓 regular + BP 達到 MAP 門檻 $\epsilon^{\text{pot}}$（如 $(6,12)$ 從 $0.3075$ 升至 $0.4999$），代價為 $O(w/M)$ rate 損失與 $O(M)$ 迭代輪數（且不可將 $w$ 開到接近 $M$，否則 rate 歸零且失去局部邊界）。
- **Poisson Channel 與均勻 CN 合併（週一作業 & 額外練習）**：
  - $\operatorname{PC}(\lambda, q)$ 中「接收端知道 $N$」使它成為真正的 BMS 混合通道；在 CN 合併下靠 Poisson 可加性封閉（$\lambda+\mu$），$T=e^{-2q\lambda}$ 對應 AWGN 在 VN 合併的 $B=e^{-m/4}$。
  - 第 7 題極限 $2q\lambda=s, \lambda\to\infty$ 時，光算均值不夠，須證 $\operatorname{Var}(p_N) = \frac{e^{-2s}}{4}(e^{s^2/\lambda}-1) \to 0$ 並透過 Chebyshev 不等式把偏離機率壓到 `0`，才證得收斂至 $\operatorname{BSC}((1-e^{-s})/2)$。
  - 額外練習 1 陷阱：$\operatorname{PC}(\lambda, W)$ 推廣至一般 BMS 在 CN 合併（$\boxdot$）成立（$N=0$ 為無雜訊 $L=+\infty$ 是乘法單位元素）；若換成 VN 合併（$\odot$），$N=0$ 必須改為完全抹除 $L=0$（加法單位元素），否則 $+\infty$ 會吸乾整條加法而破功。

# w12 作業紀錄

- 位置：`<project-root>/document/學習紀錄/碩一/錯誤更正碼/w12/`，沿用 `w11` 的 A4 11pt 20mm LaTeX 模板（皆為 2 頁 PDF）：
  - 週五 `1127`（第 1–4 題）：`1127fri.tex`、`1127fri.pdf`（搭配 `scripts/q4_awgn_mixing.py` 以 $p=1/(1+e^t)$ 複合 Simpson 積分與 `pgfplots` 繪製 `figures/awgn_{05,08,12}.dat`）。
  - 週一 `1130`（第 5–8 題）：`1130mon.tex`、`1130mon.pdf`（經 `hw-neon` [Gemini 4 Neon (High)] 精簡至 386 字並調整第 7 題直覺順序）。
- 已於 2026-11-27 透過內建瀏覽器繳交至 NTU COOL 課程 `錯誤更正碼 (EE5029)` 的 `1127` 與 `1130` 作業並確認顯示「已繳交！」。
