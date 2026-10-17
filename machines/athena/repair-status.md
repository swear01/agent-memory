---
title: Athena 送修未歸還與 fleet 操作排除
scope: machines/athena
status: active
updated: 2026-10-17
---

# 目前狀態

2026-10-17 使用者再次確認：Athena 已送修，至今尚未歸還。這是使用者提供的實體狀態，不需要以 SSH 重試驗證；先前在線／版本核對只作歷史證據。

# 操作規則

- 使用者明確確認歸還前，排除 SSH、ping、健康探測、重啟、升級、部署、skill／memory 同步及其他 fleet 維護；不定期探測回機、不把送修當連線故障重試。
- fleet 作業處理其他主機，另列「Athena：送修待回機」，不得宣稱八台或實驗室五台全數完成。保留機器身分、設定、來源與待辦。
- HAPI 控制腳本的無 target `status` 與 `all` 批次仍包含 Athena；目前改用明確的其他單機 target。指南已更新，腳本未修改。
- mazu 更新共用 NFS source 不代表 Athena 已載入、reload 或通過驗證；其主機驗證須延後。
- 回機後才核對 SSH／機器身分、Runner／supervisor、版本與待同步設定，再恢復 active fleet 範圍。

# 待回機補做

- [ARM memory compiler 依賴與 AWT／compiler 檢查](../hapi-fleet/eda-install-inventory.md)。
- [SpyGlass NSS 順序與帳號解析](../hapi-fleet/synopsys-spyglass-linux7-compatibility.md)。
- 送修期間其餘 fleet 設定、skills、memory 與版本的差異核對。
