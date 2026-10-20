---
title: LSV 專題 GPU AIG resynthesis 文獻與可行切點
scope: projects/ntu-lsv-course
status: literature-reviewed-experiment-proposed
updated: 2026-10-21
---

# GPU AIG resynthesis

使用者在挑選 LSV 期末專題時，偏好平行化方向，並希望和 spec2rtl／AI 輔助 RTL 工作有所區隔、但符合現有能力。經查作者論文頁、論文原文、CULS README、CUHK 研究紀錄與 ASP-DAC 2027 官方議程，GPU AIG 最佳化可按算子分成 rewriting → refactoring／balancing → resubstitution → don’t-care resubstitution。

## 核心文獻與程式狀態

- Shiju Lin et al., “NovelRewrite: Node-Level Parallel AIG Rewriting,” DAC 2022. 以 logical cuts 做節點層級 GPU rewriting。CULS 公開程式：<https://github.com/cuhk-eda/CULS>；作者／CUHK 紀錄：<https://research.cuhk.edu.hk/en/publications/novelrewrite-node-level-parallel-aig-rewriting-2/>。
- Tianji Liu and Evangeline F. Y. Young, “Rethinking AIG Resynthesis in Parallel,” DAC 2023. GPU refactoring 與 AND balancing 使用可平行替換且避免 data race 的框架，和 rewriting 組成 GPU `resyn2`。論文：<https://tefantasy.github.io/docs/dac23_gpu_logic_resyn.pdf>；CULS 提供基本實作。
- Lin Li, Rui Li, and Yajun Ha, “A Recursion and Lock Free GPU-Based Logic Rewriting Framework Exploiting Both Intranode and Internode Parallelism,” IEEE TCAD 2023, DOI 10.1109/TCAD.2023.3251741. 重點是同時挖掘單節點子程序及跨節點平行度、降低遞迴與鎖的限制。公開程式狀態尚未確認。
- Yang Sun et al., “Massively Parallel AIG Resubstitution,” DAC 2024. 平行 divisor 收集、候選評估、網路更新，並設計無循環相依的候選收集和衝突處理。論文：<https://tefantasy.github.io/docs/dac24_gpu_resub.pdf>；CULS 有基本 resubstitution 程式。
- Tianji Liu et al., “A Unified Parallel Framework for LUT Mapping and Logic Optimization,” 線上發表於 2024、正式刊於 TCAD 2025, 44(1):214–226, DOI 10.1109/TCAD.2024.3429079. 框架除 LUT mapping 外，也用預先計算資料庫做 GPU AIG optimization。作者頁將其列為 2024；正式卷期年為 2025。mapping-based AIG optimization 未公開原始碼。
- Yang Sun et al., “Scalable Parallel Logic Resubstitution with Advanced Don’t Care Optimization,” IEEE TCAD Early Access 2026, DOI 10.1109/TCAD.2026.3671670. GPU window-based k-resubstitution 加入 SDC／ODC、平行 divisor 收集／候選評估／更新，並平行計算 ODC 與漸進擴張 window。CULS README 將含 don’t-care 的進階版本列為未開源、可申請執行檔；勿和 DAC 2024 公開的基本 resubstitution 混為一談。
- Zhuofan Lin and Shiju Lin, “GPU-Accelerated Efficient Transduction for Logic Optimization,” DATE 2026. GPU 加速 transduction 所需的布林模擬與 compatible ODC。公開程式：<https://github.com/Lin-HKUST-Guangzhou/gpu-transduction>。論文測試基準約 16–18 個 primary inputs，故與一般大圖上的局部 rewriting／resubstitution 不應視為同一擴展範圍。

CULS README（檢視日期 2026-10-21）公開 rewriting、refactoring、balancing、基本 resubstitution、`resyn2` 和 `resyn2rs`；mapping-based AIG optimization 與進階 don’t-care resubstitution 則提供二進位檔申請方式，沒有公開原始碼。硬體依賴列為 CUDA 11.4–12.5；實際在 cthulhu 建置前需重新核對環境相容性。

## 適合延伸的專題假設

DAC 2024 的 cycle-free divisor 規則允許較低 level 節點，或相同 level 且較小 ID 的節點當 divisor，以保證同時替換後不形成循環。由於同層 ID 次序會限制可用候選，可以研究保持全域一致無環順序時，不同同層節點排序對候選數、衝突率、節點數、深度與執行時間的影響。這是待驗證的研究假設，不是已證明的改善；要以等價性檢查確認每組結果，並報告端到端成本與面積／深度品質，不能只比較 kernel 時間或節點數。

另一切點是衝突解決時候選批次的接受策略；再者可先 profile window 工作量是否不均，再決定是否做工作分派調整。不要把「平行化更多候選」直接當成更好的 QoR，衝突與共享邏輯可能改變最後結果。

ASP-DAC 2027 官方議程已列出但會議尚未舉行：Zewen Li et al., “GARDEN: GPU-Accelerated Rapid Divisor Extraction” 平行化全網路 divisor 生成／抽取；Ziang Ge et al., “Fast Delay-oriented Logic Optimization with Parallel SOP Generation” 平行化 minterm 層級 SOP 生成。這兩項適合作為後續文獻觀察，不應寫成已完成發表或已可取得原始碼。議程：<https://easychair.org/smart-program/ASP-DAC2027/2027-01-28.html>。

論文報告的加速倍率使用不同 GPU、基準、CPU 參考與演算法範圍，不能直接橫向排名。Literature review 已完成；CULS 尚未在本次 review 中建置或測試。尚未確認授課教師是否接受這個具體切點作為專題題目。
