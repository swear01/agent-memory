---
title: Mazu 長備份可重建資料清理邊界
scope: machines/mazu
machine: mazu
tags: [backup, archive, venv, conda, git, cmake, deduplication]
status: active
created: 2026-09-04
updated: 2026-10-12
---

# Canonical Home 製作、讀回和 NFS 瓶頸

## 封存與還原的閘門

- 長備份只留不可替代內容。保留核心程式碼、Git 及必要資料，不在來源 repo 做 gc/repack/prune。大型 log、dataset、checkpoint 和影音依使用者保留範圍，能生成不等同能重建。使用者指定 `E-Syn/sym_reg/aigfuzz/` 原樣保留，即使在 gitignore 或造成小檔案瓶頸。排除已辨識 EDA 安裝包／軟體目錄時，PDK/cell library、RTL、研究工具原始碼、資料和個人腳本保留，不做名稱全域排除。
- Python 環境留 Python major/minor、平台、顯式版本清單、原 requirements/lockfile、`pyvenv.cfg`、`.egg-link` 和 `direct_url.json`。Editable／VCS／local source 要有原始碼及 commit，不能只留 `pip freeze`。Conda 保留 name/channel/platform、explicit specs 及本地來源依賴；CMake 保留 toolchain、cache 關鍵參數及精確 Git 來源。先驗 dossier 可用再排除環境／產物。
- Canonical 只有一棵 `home/<account>/`，現役 base 加舊版必要差異。外層 tar 用 POSIX/PAX、numeric owner、xattr/ACL、sparse、verbatim NUL manifest 和 no-recursion；不要從目錄名稱推斷檔案類型。SQLite pathname 用 BLOB 保留非 UTF-8 bytes。掃描要用 `entry.is_dir(follow_symlinks=False)` 並傳遞錯誤，以免 `os.walk(followlinks=False)` 分類 symlink 時仍觸發 autofs。Current 檔被過濾時，仍要保留只有 overlay 封存能提供的版本。
- 私有未加密備份需要移除 credentials 和不需要的私人狀態。Redaction 針對實際值，token 可藏在 Git remote、reflog、作業 log，PEM 可藏在 JSON escapes。公開 fixture 要以官方套件／source 的 decoded SHA 驗證，不能因路徑有 test 判定。處理敏感材料不可發往外部服務。
- 原生 7-Zip volume 是一個邏輯 tar.7z 的分卷；從第一卷串流即可驗證或解出，不先把全部卷合併。Writer 尚未完成時可能 seek 改寫前卷，不能先發布。更換輸出碟時只搬已 finalized 且 FD 未開啟的數字卷，保留第一卷和暫存檔，驗 SHA 與 fsync 後才用 symlink 原子換入；提前建立未來卷 symlink 會出現 EEXIST。分卷不等同中斷續壓或容錯。
- Validator 須 drain FIFO 到 EOF，避免 tar 尾 padding 令 tee SIGPIPE。僅將明確接受的 current ENOENT 對到精確 manifest 路徑，不能把 FIFO、修改中或其他 tar 錯誤吞掉。外層 inventory／CRC trailer 容量／`7zz l` 可用於診斷，不能證明巢狀 `cph.tgz` 或截斷卷 payload 完整。重開後 job inactive／success 或最後進度都不能證明完成。另查 archive test、完整解壓長度、required payload hash、完整冷讀回和匹配的 COMPLETE marker。
- 舊卷只可在覆蓋、大小、校驗、實際冷讀回及精確批次授權成立後移除。不跑新的試產、不續 partial，也不清已發佈成品來回收空間。讀回、記錄、歸位與原件清除分開驗證。恢復位置與磁碟 UUID 見 [外接碟識別](external-backup-drives.md)。

## 確認備份真的在等哪一段

- 用 `ethtool` 查當下本機 NIC 與直連設備的協商速率。先查 route、kernel carrier changes 與 NFS server not responding，再確認慢的是哪段。100 Mbps 曾限制 Mazu，授權後 renegotiate 恢復 1 Gbps，但未確認實體根因。NetworkManager `auto-negotiate=no` 配空 speed/duplex 不等同 live autoneg disabled。兩個 NAS IP 若同走本機慢介面，換 IP 無法避開。
- NFS per-mount `nfsiostat` 第一筆是累積值，取短區間差分加 tar bytes、process wait channel、7zz CPU 與磁碟速率。慢小檔案可同時具大檔 direct I/O 正常、7zz 幾乎 idle、無重傳。tar/tee 阻塞寫 pipe 而 7zz 持續吃 CPU 則是當下壓縮瓶頸。整段平均、全機 swap、單次 iowait 不等於此程序瓶頸。權限不足的 pidstat `-1` 以有權限的 proc counters 重查。
- NFS server `maxread/maxwrite` 可低於 client 指定值，RPC 應用保留來源埠，否則 `NFS4ERR_PERM` 不證明不支援屬性。先確認實際 mount 與共用 BDI，大量循序讀取才考慮暫調 `read_ahead_kb`（Mazu 當時 128 → 1024 KiB），結束恢復原值。這會影響共用 mount，且不保證加速小檔／隨機讀取。NAS 的 DSM API 可另看負載，首份速率可能為零。

[整理前的完整證據與逐批紀錄](https://github.com/swear01/agent-memory/blob/54da0f0adfdb3dacb3bca59a729b12975d44ec9b/machines/mazu/long-backup-cleanup.md)
