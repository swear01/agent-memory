---
title: "改 frontmatter 不可讓貪婪取代吞掉正文"
scope: tools/editing
status: active
updated: 2026-09-16
evidence_digest: 0b613b999db461a8fbe7537cec6daa2e2154056e98b6c255164f07ef9bffe88e
---

# 改 frontmatter 不可讓貪婪取代吞掉正文

助手承認更新 description 時用的 regex 貪婪匹配，連 body 都被吃掉，隨後改採只定位該欄位的修改。

編輯 metadata 時限定首個 frontmatter 區塊及指定欄位，核對正文前後完整不變；需要復原時先保存未提交内容，不直接用 Git 版本覆蓋整份工作。

來源是助手承認與復原自述，沒有最終完整 diff 或測試；不把後續 YAML parse 失敗併成同一原因。
