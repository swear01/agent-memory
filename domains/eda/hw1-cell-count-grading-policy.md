---
title: EDA HW1 hierarchical Verilog cell-count grading policy
scope: domains/eda coursework grading
status: active
created: 2026-10-07
updated: 2026-10-07
---

# 批改要求與文件一致性

- 本次已確認的課程批改標準接受 Python/C++；不要因教師推薦 Perl 就扣其他語言的分數。
- Program document 的核心要求為 Flow chart、Algorithm；沒有公布心得、字數或頁數要求時，不以心得長度評分。
- 已明確豁免直接實例統計及各 module 展開總數；module 清單、指定 top 的分類計數、總數及未知引用錯誤處理仍需檢查。後續其他作業須重新核對規格，不能自動套用豁免。
- 官方輸入必須能依學生繳交說明完成。若需要未記載的前處理，不把人工修正後的正確結果當成原始官方測試通過；保留自附範例或核心演算法正確的部分證據。
- 只展開指定 top 可達的實例。自動找到多個根 module 後把全部加總，會混入未使用設計，是與 parser 失敗不同的缺失。
- 報告聲稱讀取 cells.txt、提供未知 module 錯誤或命令列參數，但繳交程式未實作時，屬文件與實作不一致；可另評文件正確性。不要只因有文字與圖就判文件完整。
- 同一原因造成多個測試失敗，合併認定一次缺失，不按 testcase 數量累加扣分。缺失嚴重程度依核心要求與失敗影響判斷，不以達到特定平均分為目的。

# 基本輸入與測試證據

- cells.txt 決定 leaf 類型；不因 module 沒有子實例或缺定義就自動當成 leaf。已定義、未列為 leaf 且只有 continuous assignment 的 module 貢獻零個 leaf cell。
- 分號分隔的實例放在同一行仍是基本語法。以行首 regex 抓實例可能漏算後續實例；註解也應在解析 module 前排除。
- 未被指定 top 使用的 module 應出現在定義清單，但其 cell 不加入 top 統計。
- 測試副本與原繳交檔分開，核對原檔 hash。單檔程式可合併 Verilog 或依其介面改名，但不能偷偷修改 parser、移除 assign 或註解後算作直接通過。
- 以原始參考程式交叉核對預期結果。省略 leaf library 定義等診斷輸入應明列條件，不直接把所有差異當成新增扣分項；未知 module 測試應區分正確報錯與無關敘述先解析失敗。

# COOL／Canvas 未公開批改

- EMI 課程的扣分留言用英文，以 score line 加簡短 deduction bullets 列出缺失與分數。
- 儲存前確認 assignment 的 manual posting policy；該模式明列未發布分數和 submission comments 都不可見且不通知學生。儲存後核對 Hidden，未經新的明確授權不 Post Grades。
- 本次 SpeedGrader 以單純 fill 顯示數字時並未持久儲存；實際逐字輸入及 Submit 後才顯示 graded／Hidden。以重新載入後的分數和留言回讀為證，不能只看欄位文字或按鈕成功。
- 驗證自己新增的 grading feedback，勿把原有學生 submission comments 當成重複留言或刪除目標。
- 公開記憶只同步規則與工具經驗；學生身分、逐人成績、私人留言、原始作業和未公開測試資料均保留在私人來源。
