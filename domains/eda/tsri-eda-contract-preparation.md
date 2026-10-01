---
title: TSRI EDA 續用合約下載、預填與完成狀態判斷
scope: domains/eda
status: active
updated: 2026-10-01
tags: [tsri, eda, contracts, pdf]
---

# EDA 申請與製程申請分開處理

eTAS 的 Software Application 與製程／矽智財名單案件有不同流程。不能把製程的年度測驗、代理人規則或名單表單直接套到 EDA 軟體續用；以實際頁面、Application Guidelines、合約列表與所選項目為準。

成功建立申請後的 Pending 不等於授權核准。Contract Status 若仍顯示 Download Contract、沒有上傳或審查結果，只能回報申請已建立、合約尚待處理。不能從套件申請成功推論「所有可申請項目已申請滿」。

# 正式待簽檔與範本

2026-10-01 取得的廠商包以 readme 和 Sample 核實：Cadence 的 `Cadence_Legal_agreement_to_Sign.pdf` 為 1 頁，Siemens 的 `Siemens_EDA_HEP-Agreement_to_Sign.pdf` 為 3 頁，Synopsys 的 `synopsys_agreement_to_Sign.pdf` 為 1 頁。`Sample` 僅供填寫參考；`Complete` 是完整條款。Siemens 待簽包的三頁包含完整合約第 1、9 頁及 End-Use Certification，不能因頁碼跳號誤認原檔缺頁。

三家的範本要求英文填寫。左側 School／University／Licensee 為申請端，右側 TSRI、SISW 或 Synopsys 欄不由申請端代填。Printed Name／Typed Name 可預填；Signature 應留給實際簽署人本人。日期按實際簽署日，勿把下一授權年度直接當成當天年份。

當日 TSRI 的正式 PDF 為「EDA 軟體暨矽智財授權合約」，TSRI-Q4-ES01 版本 1.8，生效日期 2026-10-01，共 1 頁。中文姓名欄明列簽名或蓋章，製作預填件時留白供本人簽署。以上版本與頁數為查核快照；新年度應重新讀系統檔案。

# Siemens 與 Synopsys 的留白邊界

Siemens End-Use Certification 除基本資料，另需實際產品清單、最終用途及用途說明、限制身分聲明。範本上的 Yes／No 是填寫示例，不能據此宣稱個案用途或身分已核實。缺少事實時先完成基本資料，把待確認欄位列在交付說明，由實際簽署人確認；不猜测學術研究以外用途是否全部為 No。

Synopsys 的 Site No.、Agreement No. 與合約 effective date 不是教授簽署欄 Date 的同義欄位。沒有正式值時保留空白，不自動把預計簽署日期填入生效日。

# PDF 預填與下載核實

無 AcroForm 的原始檔可保留完整底稿，再合併文字 overlay；不要重排正式條款或將姓名畫進 Signature 欄。以來源檔雜湊核對原件未改，重新開啟輸出 PDF 核對頁數與預填值，再逐頁 render 檢查欄位溢出、字型及簽署空間。

下載按鈕可能只開 PDF 預覽，不會立即寫入 Downloads。等到實際 PDF 內容出現，再透過瀏覽器儲存／下載；檢查本機檔案存在、PDF 格式、頁數與可見內容，才能宣稱下載完成。對照檔案系統確認四份正式待簽件，不要把清單 TXT 或三個 ZIP 當成全部合約已齊。

各廠商 readme 及上述 TSRI 合約均要求掃描 PDF 上傳，不需要寄回紙本。下載、預填、簽署、上傳、送審、核准是不同狀態；只回報已完成且有證據的步驟。
