# Template Skill - 正體中文說明文件

## Skill 概述

### 功能
Template Skill 是一個基礎模板，用於創建新的 Claude Code skills。它提供標準的 SKILL.md 結構和 YAML frontmatter 格式，幫助開發者快速開始建立自己的 skill。

### 用途
這個模板不是一個可執行的 skill，而是一個起點：

1. **學習 Skill 結構**：了解 skill 檔案應該如何組織
2. **快速開始**：複製此模板以建立新 skill
3. **保持一致性**：確保所有 skills 遵循相同的格式
4. **參考標準**：作為 skill 開發的參考指南

---

## 目錄結構

```
template-skill/
├── SKILL.md          # Skill 模板檔案（英文）
└── README.zh-TW.md   # 正體中文說明文件（本文件）
```

---

## 模板內容

### YAML Frontmatter

```yaml
---
name: template-skill
description: Replace with description of the skill and when Claude should use it.
---
```

**必要欄位**：
- **name**：Skill 的唯一識別名稱（使用連字號命名）
- **description**：Skill 的功能描述和使用時機

### 主要內容

```markdown
# Insert instructions below
```

**應包含的內容**：
- Skill 的詳細說明
- 使用時機和情境
- 具體的操作指令
- 範例和使用案例
- 任何特殊要求或限制

---

## 如何使用此模板

### 步驟 1：複製模板

```bash
cp -r template-skill/ my-new-skill/
cd my-new-skill/
```

### 步驟 2：更新 YAML Frontmatter

編輯 `SKILL.md`：

```yaml
---
name: my-new-skill
description: Describe what your skill does and when Claude should use it. Be specific about the trigger conditions.
---
```

### 步驟 3：撰寫 Skill 指令

在 `# Insert instructions below` 下方加入詳細指令：

```markdown
# My New Skill Instructions

## When to Use
[說明什麼情況下觸發此 skill]

## What It Does
[說明此 skill 的功能]

## How to Use
[提供使用範例和步驟]

## Examples
[提供具體的使用案例]
```

### 步驟 4：測試 Skill

將 skill 放置在 Claude Code 可以存取的位置，並測試觸發條件。

---

## 最佳實踐

### 命名規範

**使用連字號命名（kebab-case）**：
- ✓ `my-new-skill`
- ✓ `pdf-analyzer`
- ✓ `code-formatter`
- ✗ `myNewSkill`
- ✗ `My_New_Skill`
- ✗ `MYNEWSKILL`

### 描述撰寫

**清晰且具體**：
- 說明 skill 做什麼
- 說明何時/為何使用
- 包含觸發關鍵字或條件
- 保持簡潔（1-3 句）

**好的描述範例**：
```
description: Analyzes meeting transcripts to identify communication patterns
like conflict avoidance, filler words, and speaking ratios. Use when the
user wants feedback on their meeting behavior.
```

**不好的描述範例**：
```
description: A skill for meetings.
```

### 指令撰寫

**詳細且可行動**：
- 使用清晰的標題和章節
- 提供具體的步驟
- 包含範例輸入和輸出
- 說明任何限制或要求
- 使用項目符號和編號清單

---

## Skill 開發檢查清單

### 必須包含

- [ ] 正確的 YAML frontmatter
- [ ] 唯一且描述性的名稱
- [ ] 清晰的描述和觸發條件
- [ ] 詳細的指令
- [ ] 使用範例
- [ ] 任何必要的檔案或腳本

### 建議包含

- [ ] 使用案例和情境
- [ ] 限制或注意事項
- [ ] 相關 skills 或工具的連結
- [ ] 故障排除提示
- [ ] 版本資訊或變更日誌

### 測試

- [ ] Skill 正確觸發
- [ ] 指令清晰易懂
- [ ] 範例正常運作
- [ ] 錯誤處理適當
- [ ] 文件完整準確

---

## 相關資源

### 範例 Skills

查看其他 skills 作為參考：
- `brand-guidelines`：簡單的內容型 skill
- `competitive-ads-extractor`：複雜的多步驟 skill
- `file-organizer`：系統操作型 skill
- `canvas-design`：創意生成型 skill

### Skill 開發指南

- 參考 Claude Code 官方文件
- 查看 awesome-claude-skills 倉庫
- 研究現有 skills 的結構和模式
- 測試不同的觸發條件和指令

---

## 總結

### 何時使用此模板

**適合用於**：
- 創建新的 Claude Code skill
- 學習 skill 結構
- 確保格式一致性
- 快速原型開發

**開始步驟**：
1. 複製模板
2. 重新命名
3. 更新 frontmatter
4. 撰寫指令
5. 測試和迭代

### 關鍵原則

- **清晰**：明確說明何時和如何使用
- **具體**：提供詳細的指令和範例
- **一致**：遵循標準格式和命名
- **可測試**：確保 skill 可驗證和除錯

---

**最後更新**：2025-11-14
**文件版本**：1.0.0
**適用於**：Claude Code Skills
