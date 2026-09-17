---
title: "多指令技能應整合成母子架構而非污染全域命名空間"
scope: tools/skillshare
status: active
updated: 2026-09-17
---

# 多指令技能應整合成母子架構而非污染全域命名空間

當一個工具或工作流程包含多個子功能或操作模式（例如程式碼精簡工具包含 audit、review、debt、gain 等）時，不要在全域註冊多個獨立的 global skills。

## 問題與代價
1. **全域 Metadata 預算膨脹**：每個全域 skill 的 name 與 description 會常駐在所有 Agent 的系統提示詞中（每項約 50–100 tokens），多個拆分技能會造成 context window 浪費。
2. **工具選取歧義**：過多語意相近的全域技能容易導致 Agent 觸發判斷混淆或重複載入。

## 最佳實踐：母子技能（Parent-Child Sub-Skills）架構
1. **單一全域入口**：全域只保留一個母 skill 目錄（如 `ponytail`），進入點 `SKILL.md` 嚴格遵守 `<200 行` 規則。
2. **參數提示與路由宣告**：在 YAML frontmatter 使用 `argument-hint: "<subcommand> [args]"`，並於 description 提示子指令關鍵字。
3. **漸進式揭露（Progressive Disclosure）**：母 `SKILL.md` 僅維護簡潔的子指令分發表，具體執行邏輯分別撰寫於 `references/sub-<name>.md`。Agent 僅在用戶實際呼叫特定子指令時，才動態按需讀取該參考文件。
