---
title: Calibre v2lvs 容器 PID 1 授權失敗診斷
scope: domains/eda
status: converter-verified-signoff-pending
updated: 2026-10-09
---

# v2lvs 授權錯誤不代表沒有 Siemens 使用權

Mazu 的 Calibre 2026.3_27.19 在既有 Rocky 8 image 中實測。授權 server 與 vendor daemons 正常，`calibrelvs` 有有效且可 checkout 的項目；實際 v2lvs 成功 log 顯示 `calibrelvs license acquired.`。不要猜 feature 名稱 `calibre_v2lvs`，再把查無該名稱誤判成沒有 converter 授權。

研究 wrapper 的內層 `bash -lc 'source ...; exec v2lvs "$@"'` 使工具成為容器 PID 1。相同 image、主機、UID/GID、network、授權 pointer、input 和 mounts，只改成 `bash -lc 'source ...; v2lvs "$@"'`，便從授權失敗／exit 134 改為 exit 0，實際工具 PID 44。最終兩組 A/B 都重現 PID 1 exit 134、child exit 0；修後再一次 exit 0。這是此版本及環境的實測結果，不外推成所有 EDA 工具通則；更底層 vendor bug 尚未定位。

先比對既有 `/apps/eda/docker-shell`，它保留 caller 身分並讓 shell 作為 PID 1。外層 `exec docker run` 可保留；要避免的是內層把 v2lvs 直接變成 PID 1。只補容器使用者資料或改 HOME 並未修好；原 wrapper 的使用者名稱警告在只移除內層 exec 後仍存在，但 checkout 已成功，所以不能把警告當此失敗的根因。單獨補 `MGLS_LICENSE_FILE` 也不是必要修正：既有 pointer、不補該變數亦可成功。

修正只套用於 SRAM compiler 隔離研究 wrapper，不變更共用 `/apps`、授權環境、TSRI IP 登錄或服務。實際 controller 轉換得到 129,625-byte SPICE、2,805 個 X instances；與既有 docker-shell 轉換結果 SHA-256 相同。這證明此 Mazu 路徑能使用實際授權，不表示其他主機或所有產品也驗證。

TSRI 官方 EDA 須知要求登錄實驗室實體 IP 與學術網域。2026-10-09 使用者重新登入後，即時讀回 eTAS：四份合約全部審核通過、六項軟體全部已核准（含 Siemens EDA Tool Suite），IP 表五筆全部已啟用。Mazu 當時的公開 DNS 與實測 HTTPS 對外 IP 相同，且在已啟用表內；先前的 v2lvs 問題不需要靠修改 IP 登錄解決。這不是其他四台主機的 checkout 證據，後續仍須現場讀回狀態。

同次 Calibre 下載清單含 `2026.3_27.19`，與共用 `/apps/eda/siemens/calibre/2026.3_27.19` 現有安裝一致。此次流程沿用伺服器安裝及實際 TSRI floating license 即可，不需重新下載安裝包，也沒有執行軟體下載或 IP 匯入；IP 匯入頁明示會全量覆蓋既有資料，不能當成增量更新。

OpenFinRAM 512×8 生成 exit 0 並輸出 SPICE/GDS/LEF/estimated Liberty，35 個 GDS/LEF pins 同層金屬 audit 通過、0 fallback rectangles；完整 LVS 因 native 環境缺 tcsh 被跳過。輸出不是已通過 signoff 的 macro，DRC/LVS/PEX/characterization 與專案整合仍待驗證。
