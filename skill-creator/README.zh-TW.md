# Skill Creator - 正體中文說明文件

## 📋 目錄
- [Skill 概述](#skill-概述)
- [目錄結構](#目錄結構)
- [檔案說明](#檔案說明)
- [關鍵腳本分析](#關鍵腳本分析)
- [設計策略](#設計策略)
- [Claude Code 使用方式](#claude-code-使用方式)
- [技術架構](#技術架構)
- [使用的 Prompt 策略](#使用的-prompt-策略)
- [使用的工具](#使用的工具-tools)
- [最佳實踐](#最佳實踐)
- [總結](#總結)

---

## Skill 概述

### 這個 Skill 的功能
Skill Creator 是一個專門用於建立高品質 Claude Skills 的指南和工具套件。它提供完整的 skill 建立流程、模板系統和驗證工具,讓 Claude 能夠建立結構良好、易於維護的 skills,擴展 Claude 的能力,使其成為特定領域的專家。

主要功能包括:
- 提供結構化的六步驟 skill 建立流程
- 自動初始化 skill 目錄結構和模板
- 內建驗證機制,確保 skill 符合規範
- 自動打包 skill 為可分發的 zip 檔案
- 提供多種 skill 結構模式(工作流程型、任務型、參考型、功能型)
- 包含漸進式揭露(Progressive Disclosure)設計原則

### 解決的問題
在建立 Claude Skills 時,開發者經常遇到以下挑戰:

1. **不知從何開始**:缺乏清晰的 skill 建立流程和最佳實踐
2. **結構不一致**:每次建立 skill 都要重新思考目錄結構和檔案組織
3. **文檔品質低落**:SKILL.md 缺乏關鍵資訊或格式不正確
4. **資源組織混亂**:不知道腳本、參考資料和資產應該如何分類
5. **驗證困難**:缺乏自動化工具檢查 skill 是否符合規範
6. **分發麻煩**:手動打包 skill 檔案容易遺漏或出錯

Skill Creator 提供完整的自動化工具和清晰的指南,讓建立 skill 變得系統化、可靠且高效。

---

## 目錄結構

```
skill-creator/
├── SKILL.md                    # Skill 主要說明文件(英文)
├── LICENSE.txt                 # 授權條款
├── README.zh-TW.md            # 正體中文說明文件(本文件)
└── scripts/                    # 工具腳本目錄
    ├── init_skill.py          # Skill 初始化腳本
    ├── package_skill.py       # Skill 打包腳本
    └── quick_validate.py      # Skill 驗證腳本
```

---

## 檔案說明

### 核心檔案

#### 1. `SKILL.md`(210 行)
- **用途**:建立 skills 的完整指南
- **內容**:
  - Skills 的定義和用途
  - Skill 結構說明(SKILL.md、bundled resources)
  - 漸進式揭露設計原則
  - 六步驟建立流程
  - 資源類型說明(scripts、references、assets)
- **特點**:系統化的步驟式指引,確保 skill 品質

#### 2. `LICENSE.txt`
- **用途**:定義 Skill 的使用授權條款
- **重要性**:確保使用者了解使用限制和權利

#### 3. `README.zh-TW.md`(本文件)
- **用途**:為繁體中文使用者提供詳細的說明和教學
- **內容**:完整的架構分析、使用說明和設計理念

### Scripts 目錄

#### 1. `init_skill.py`(304 行)
**功能**:初始化新 skill 的目錄結構和模板

**主要功能**:

**`init_skill()` 函數**:
- 建立 skill 目錄
- 生成 SKILL.md 模板(包含 YAML frontmatter 和 TODO 佔位符)
- 建立資源目錄:`scripts/`、`references/`、`assets/`
- 在每個目錄中建立範例檔案

**模板內容**:

**SKILL_TEMPLATE**:
```yaml
---
name: {skill_name}
description: [TODO: Complete and informative explanation...]
---

# {skill_title}

## Overview
[TODO: 1-2 sentences explaining what this skill enables]

## Structuring This Skill
[TODO: Choose the structure that best fits this skill's purpose. Common patterns:

**1. Workflow-Based** (best for sequential processes)
**2. Task-Based** (best for tool collections)
**3. Reference/Guidelines** (best for standards or specifications)
**4. Capabilities-Based** (best for integrated systems)

...
```

**EXAMPLE_SCRIPT**:
- Python 腳本範例
- 包含文檔字串和主函數結構
- 提供其他 skills 的實際範例參考

**EXAMPLE_REFERENCE**:
- 參考文檔範例
- 說明何時使用 reference docs
- 提供結構建議(API Reference、Workflow Guide)

**EXAMPLE_ASSET**:
- 資產檔案說明
- 列舉常見資產類型(templates、images、fonts、boilerplate code)
- 解釋 assets 與其他資源的差異

**命令列介面**:
```bash
init_skill.py <skill-name> --path <output-directory>

# 範例
init_skill.py my-new-skill --path skills/public
init_skill.py my-api-helper --path skills/private
```

**輸出**:
```
✅ Created skill directory: /path/to/my-new-skill
✅ Created SKILL.md
✅ Created scripts/example.py
✅ Created references/api_reference.md
✅ Created assets/example_asset.txt

Next steps:
1. Edit SKILL.md to complete the TODO items
2. Customize or delete the example files
3. Run the validator when ready
```

**設計亮點**:
- 自動建立完整的目錄結構
- 提供詳細的模板和範例
- 包含最佳實踐和結構指引
- 設定腳本檔案的可執行權限(chmod 0o755)

#### 2. `package_skill.py`(111 行)
**功能**:打包 skill 為可分發的 zip 檔案

**主要流程**:

**1. 驗證階段**:
```python
# 檢查 skill 目錄存在
if not skill_path.exists():
    return None

# 檢查 SKILL.md 存在
skill_md = skill_path / "SKILL.md"
if not skill_md.exists():
    return None

# 執行完整驗證
valid, message = validate_skill(skill_path)
if not valid:
    print(f"❌ Validation failed: {message}")
    return None
```

**2. 打包階段**:
```python
with zipfile.ZipFile(zip_filename, 'w', zipfile.ZIP_DEFLATED) as zipf:
    for file_path in skill_path.rglob('*'):
        if file_path.is_file():
            arcname = file_path.relative_to(skill_path.parent)
            zipf.write(file_path, arcname)
            print(f"  Added: {arcname}")
```

**命令列介面**:
```bash
package_skill.py <path/to/skill-folder> [output-directory]

# 範例
package_skill.py skills/public/my-skill
package_skill.py skills/public/my-skill ./dist
```

**輸出**:
```
🔍 Validating skill...
✅ Skill is valid!

  Added: my-skill/SKILL.md
  Added: my-skill/scripts/example.py
  Added: my-skill/references/api_reference.md

✅ Successfully packaged skill to: my-skill.zip
```

**設計亮點**:
- 打包前自動驗證
- 保留目錄結構
- 使用 ZIP_DEFLATED 壓縮
- 提供詳細的檔案列表

#### 3. `quick_validate.py`(65 行)
**功能**:快速驗證 skill 是否符合規範

**驗證項目**:

**1. SKILL.md 存在性**:
```python
skill_md = skill_path / 'SKILL.md'
if not skill_md.exists():
    return False, "SKILL.md not found"
```

**2. YAML Frontmatter 格式**:
```python
if not content.startswith('---'):
    return False, "No YAML frontmatter found"

match = re.match(r'^---\n(.*?)\n---', content, re.DOTALL)
if not match:
    return False, "Invalid frontmatter format"
```

**3. 必要欄位檢查**:
```python
if 'name:' not in frontmatter:
    return False, "Missing 'name' in frontmatter"
if 'description:' not in frontmatter:
    return False, "Missing 'description' in frontmatter"
```

**4. 命名規範驗證**:
```python
# 必須是 hyphen-case (小寫字母、數字和連字號)
if not re.match(r'^[a-z0-9-]+$', name):
    return False, "Name should be hyphen-case"

# 不能以連字號開始/結束,不能有連續連字號
if name.startswith('-') or name.endswith('-') or '--' in name:
    return False, "Invalid hyphen usage"
```

**5. Description 驗證**:
```python
# 不能包含角括號
if '<' in description or '>' in description:
    return False, "Description cannot contain angle brackets"
```

**命令列介面**:
```bash
quick_validate.py <skill_directory>

# 範例
quick_validate.py skills/public/my-skill
```

**輸出**:
```
✅ Skill is valid!
# 或
❌ Name 'MySkill' should be hyphen-case (lowercase letters, digits, and hyphens only)
```

---

## 關鍵腳本分析

### init_skill.py 深度分析

#### 1. 模板設計哲學

**TODO 驅動開發**:
```markdown
## Overview
[TODO: 1-2 sentences explaining what this skill enables]

## Structuring This Skill
[TODO: Choose the structure that best fits this skill's purpose...]
```

**設計理念**:
- 提供清晰的填寫指引
- 防止遺漏關鍵資訊
- 教育開發者 skill 結構模式

#### 2. 四種 Skill 結構模式

**1. Workflow-Based(工作流程型)**:
```markdown
適合:序列化流程
範例:DOCX skill
結構:Overview → Workflow Decision Tree → Step 1 → Step 2...
```

**2. Task-Based(任務型)**:
```markdown
適合:工具集合
範例:PDF skill
結構:Overview → Quick Start → Task Category 1 → Task Category 2...
```

**3. Reference/Guidelines(參考型)**:
```markdown
適合:標準或規範
範例:Brand styling
結構:Overview → Guidelines → Specifications → Usage...
```

**4. Capabilities-Based(功能型)**:
```markdown
適合:整合系統
範例:Product Management
結構:Overview → Core Capabilities → Feature 1 → Feature 2...
```

**設計優點**:
- 提供多種選擇,適應不同需求
- 每種模式都有實際範例
- 可以混合使用

#### 3. 資源目錄的區分

**scripts/ - 可執行程式碼**:
```python
#!/usr/bin/env python3
"""
Example helper script for {skill_name}

This is a placeholder script that can be executed directly.
"""

def main():
    print("This is an example script")
    # TODO: Add actual script logic here

if __name__ == "__main__":
    main()
```

**特點**:
- 直接可執行
- 不需要載入到 context
- Token 高效
- 確定性可靠

**references/ - 參考文檔**:
```markdown
# Reference Documentation

## When Reference Docs Are Useful
- Comprehensive API documentation
- Detailed workflow guides
- Complex multi-step processes
- Information too lengthy for main SKILL.md

## Structure Suggestions
### API Reference Example
- Overview
- Authentication
- Endpoints with examples
```

**特點**:
- 需要時載入到 context
- 保持 SKILL.md 簡潔
- 適合詳細資訊

**assets/ - 輸出資源**:
```markdown
# Example Asset File

Asset files are NOT intended to be loaded into context,
but rather used within the output Claude produces.

## Common Asset Types
- Templates: .pptx, .docx
- Images: .png, .jpg, .svg
- Fonts: .ttf, .woff
- Boilerplate code: Project directories
```

**特點**:
- 不載入到 context
- 用於最終輸出
- 可以是任何檔案類型

#### 4. 名稱轉換邏輯

**`title_case_skill_name()` 函數**:
```python
def title_case_skill_name(skill_name):
    """Convert hyphenated skill name to Title Case for display."""
    return ' '.join(word.capitalize() for word in skill_name.split('-'))

# 範例:
# 'my-new-skill' → 'My New Skill'
# 'pdf-editor' → 'Pdf Editor'
# 'bigquery-helper' → 'Bigquery Helper'
```

**用途**:
- skill 名稱:hyphen-case(my-new-skill)
- skill 標題:Title Case(My New Skill)
- 保持一致性

### package_skill.py 深度分析

#### 1. 驗證優先策略

**打包前驗證**:
```python
print("🔍 Validating skill...")
valid, message = validate_skill(skill_path)
if not valid:
    print(f"❌ Validation failed: {message}")
    print("   Please fix the validation errors before packaging.")
    return None
print(f"✅ {message}\n")
```

**設計理念**:
- 防止打包無效的 skill
- 提早發現問題
- 提供清晰的錯誤訊息

#### 2. 目錄結構保留

**相對路徑計算**:
```python
for file_path in skill_path.rglob('*'):
    if file_path.is_file():
        # 計算 zip 中的相對路徑
        arcname = file_path.relative_to(skill_path.parent)
        zipf.write(file_path, arcname)
```

**範例**:
```
原始路徑: /home/user/skills/my-skill/scripts/example.py
skill_path: /home/user/skills/my-skill
skill_path.parent: /home/user/skills
arcname: my-skill/scripts/example.py
```

**優點**:
- 保留完整的目錄結構
- zip 解壓後直接可用
- 包含 skill 目錄名稱

#### 3. 可選輸出目錄

**彈性輸出**:
```python
if output_dir:
    output_path = Path(output_dir).resolve()
    output_path.mkdir(parents=True, exist_ok=True)
else:
    output_path = Path.cwd()

zip_filename = output_path / f"{skill_name}.zip"
```

**設計優點**:
- 預設輸出到當前目錄
- 支援自訂輸出位置
- 自動建立輸出目錄

### quick_validate.py 深度分析

#### 1. 正規表達式驗證

**Frontmatter 提取**:
```python
# 提取 YAML frontmatter
match = re.match(r'^---\n(.*?)\n---', content, re.DOTALL)
if not match:
    return False, "Invalid frontmatter format"

frontmatter = match.group(1)
```

**模式說明**:
- `^---\n`:開始於 `---` 和換行
- `(.*?)`:非貪婪匹配任意內容
- `\n---`:結束於換行和 `---`
- `re.DOTALL`:`.` 匹配包括換行符

**欄位提取**:
```python
# 提取 name
name_match = re.search(r'name:\s*(.+)', frontmatter)
if name_match:
    name = name_match.group(1).strip()

# 提取 description
desc_match = re.search(r'description:\s*(.+)', frontmatter)
if desc_match:
    description = desc_match.group(1).strip()
```

#### 2. 命名規範設計

**Hyphen-case 要求**:
```python
# 只允許小寫字母、數字和連字號
if not re.match(r'^[a-z0-9-]+$', name):
    return False, "Name should be hyphen-case"
```

**原因**:
- **一致性**:所有 skills 使用相同格式
- **URL 友好**:適合用於 URL 路徑
- **檔案系統相容**:避免特殊字符問題
- **可讀性**:清晰易懂

**禁止的模式**:
```python
# 不能以連字號開始或結束
# 不能有連續連字號
if name.startswith('-') or name.endswith('-') or '--' in name:
    return False, "Invalid hyphen usage"
```

**範例**:
```
✅ my-skill
✅ pdf-editor
✅ bigquery-helper
✅ skill-creator-v2

❌ MySkill (大寫)
❌ my_skill (底線)
❌ -my-skill (開始連字號)
❌ my-skill- (結束連字號)
❌ my--skill (連續連字號)
```

#### 3. Description 安全性

**角括號檢查**:
```python
if '<' in description or '>' in description:
    return False, "Description cannot contain angle brackets"
```

**原因**:
- 防止 HTML/XML 注入
- 避免解析問題
- 確保純文字描述

---

## 設計策略

### 1. 漸進式揭露(Progressive Disclosure)

**三層載入系統**:

**第一層:元資料(總是載入)**:
```yaml
name: pdf-editor
description: Edit, merge, and manipulate PDF files...
```
- 約 100 字
- 決定是否使用此 skill
- 總是在 context 中

**第二層:SKILL.md 主體(skill 觸發時載入)**:
```markdown
# PDF Editor

## Overview
This skill enables PDF manipulation...

## Quick Start
1. To merge PDFs...
2. To split PDFs...
```
- < 5,000 字
- 詳細的使用指南
- skill 被選擇時才載入

**第三層:Bundled Resources(按需載入/執行)**:
```
scripts/rotate_pdf.py     - 執行時不需載入
references/api_docs.md    - Claude 需要時才載入
assets/template.pdf       - 使用時不載入
```
- 無限制(腳本可以不載入就執行)
- 最大化 context 效率

**優點**:
- 節省 context window
- 提高載入速度
- 按需提供資訊

### 2. 模板驅動開發

**TODO 佔位符策略**:
```markdown
[TODO: Complete and informative explanation of what the skill does]
[TODO: 1-2 sentences explaining what this skill enables]
[TODO: Choose the structure that best fits this skill's purpose]
```

**設計理念**:
- **防止遺漏**:提醒填寫關鍵資訊
- **提供指引**:說明每個部分的用途
- **教育功能**:幫助理解最佳實踐
- **可搜尋**:容易找到未完成的部分

### 3. 自動化驗證和打包

**整合流程**:
```
init_skill.py
    ↓
[編輯 SKILL.md 和資源]
    ↓
quick_validate.py (可選)
    ↓
package_skill.py
    ├─ 自動驗證
    └─ 打包 zip
    ↓
my-skill.zip (可分發)
```

**優點**:
- 一致性:所有 skills 遵循相同規範
- 可靠性:自動驗證防止錯誤
- 效率:自動化減少手動工作

### 4. 資源分類策略

**三類資源的明確區分**:

| 資源類型 | 載入到 Context? | 用途 | 範例 |
|---------|----------------|------|------|
| **scripts/** | 否(可執行) | 確定性任務 | rotate_pdf.py |
| **references/** | 是(按需) | 詳細文檔 | api_docs.md |
| **assets/** | 否(用於輸出) | 模板資源 | logo.png, template.pptx |

**設計理念**:
- **清晰的界限**:每種資源有明確的用途
- **Context 優化**:減少不必要的載入
- **靈活性**:支援各種 skill 類型

### 5. 範例驅動設計

**每個模板都包含範例**:
```python
EXAMPLE_SCRIPT = '''
Example real scripts from other skills:
- pdf/scripts/fill_fillable_fields.py - Fills PDF form fields
- pdf/scripts/convert_pdf_to_images.py - Converts PDF pages to images
'''

EXAMPLE_REFERENCE = """
Example real reference docs from other skills:
- product-management/references/communication.md
- bigquery/references/ - API references and query examples
"""
```

**優點**:
- 具體展示實際用法
- 減少歧義
- 加速學習曲線

### 6. 六步驟建立流程

**結構化方法**:

1. **Understanding(理解)**:收集具體範例
2. **Planning(規劃)**:分析需要的資源
3. **Initializing(初始化)**:執行 init_skill.py
4. **Editing(編輯)**:撰寫 SKILL.md 和資源
5. **Packaging(打包)**:執行 package_skill.py
6. **Iterating(迭代)**:根據使用回饋改進

**優點**:
- 可追蹤的進度
- 明確的里程碑
- 系統化的品質保證

---

## Claude Code 使用方式

### 觸發時機

Claude Code 會在以下情況使用此 Skill:

1. **需要建立新 skill**
   - 範例:「建立一個 PDF 編輯 skill」
   - 範例:「我想建立一個 skill 來處理 BigQuery」
   - 範例:「幫我製作一個品牌指南 skill」

2. **需要更新現有 skill**
   - 範例:「改進這個 skill 的文檔」
   - 範例:「為這個 skill 添加新功能」
   - 範例:「重構這個 skill 的結構」

3. **需要驗證或打包 skill**
   - 範例:「驗證這個 skill 是否正確」
   - 範例:「打包這個 skill 為 zip 檔案」
   - 範例:「檢查 skill 的格式」

### 典型工作流程

#### 工作流程 1:建立全新的 Skill

```
步驟 1:理解 Skill 需求
使用者:「我想建立一個 skill 來處理 Excel 檔案」

Claude:
- 詢問具體範例:
  「這個 skill 應該支援什麼功能?編輯、讀取、建立圖表?」
  「可以給一些使用範例嗎?」
  「什麼樣的請求應該觸發這個 skill?」
- 避免一次問太多問題
- 收集 2-3 個具體使用案例

步驟 2:規劃資源
Claude 分析每個範例:
1. 讀取 Excel → 需要 scripts/read_excel.py
2. 建立圖表 → 需要 assets/chart_template.xlsx
3. 了解 Excel API → 需要 references/excel_api.md

建立資源清單:
- scripts/read_excel.py
- scripts/create_chart.py
- references/excel_api.md
- assets/chart_template.xlsx

步驟 3:初始化 Skill
Claude 執行:
bash scripts/init_skill.py excel-helper --path skills/

輸出:
✅ Created skill directory: skills/excel-helper
✅ Created SKILL.md
✅ Created scripts/example.py
✅ Created references/api_reference.md
✅ Created assets/example_asset.txt

步驟 4:編輯 Skill
Claude:
1. 刪除不需要的範例檔案
2. 建立實際的腳本:
   - scripts/read_excel.py
   - scripts/create_chart.py
3. 建立參考文檔:
   - references/excel_api.md
4. 添加資產:
   - assets/chart_template.xlsx
5. 更新 SKILL.md:
   - 填寫 frontmatter description
   - 撰寫 Overview
   - 選擇結構(Task-Based)
   - 撰寫使用指南
   - 刪除 TODO 佔位符

步驟 5:驗證和打包
Claude 執行:
python scripts/quick_validate.py skills/excel-helper

如果驗證通過:
python scripts/package_skill.py skills/excel-helper

輸出:
🔍 Validating skill...
✅ Skill is valid!

  Added: excel-helper/SKILL.md
  Added: excel-helper/scripts/read_excel.py
  Added: excel-helper/scripts/create_chart.py
  Added: excel-helper/references/excel_api.md
  Added: excel-helper/assets/chart_template.xlsx

✅ Successfully packaged skill to: excel-helper.zip

步驟 6:測試和迭代
Claude:
1. 使用 skill 處理實際任務
2. 注意問題或不足
3. 根據回饋改進:
   - 更新 SKILL.md
   - 添加或修改腳本
   - 補充參考文檔
4. 重新打包
```

#### 工作流程 2:更新現有 Skill

```
步驟 1:識別改進需求
使用者:「這個 PDF skill 需要添加合併功能」

Claude:
- 讀取現有的 SKILL.md
- 了解當前結構
- 識別需要的變更

步驟 2:規劃變更
Claude 分析:
- 需要新腳本:scripts/merge_pdfs.py
- 需要更新 SKILL.md:添加合併 PDF 章節
- 可能需要參考文檔:references/merge_options.md

步驟 3:實作變更
Claude:
1. 建立 scripts/merge_pdfs.py
2. 更新 SKILL.md:
   - 添加「Merge PDFs」章節
   - 更新 description 提及合併功能
   - 添加使用範例
3. (可選)建立 references/merge_options.md

步驟 4:驗證和重新打包
Claude 執行:
python scripts/package_skill.py skills/pdf-editor

輸出:
🔍 Validating skill...
✅ Skill is valid!

  Added: pdf-editor/SKILL.md
  Added: pdf-editor/scripts/merge_pdfs.py
  ...

✅ Successfully packaged skill to: pdf-editor.zip
```

#### 工作流程 3:驗證和修復 Skill

```
步驟 1:執行驗證
Claude 執行:
python scripts/quick_validate.py skills/my-skill

輸出:
❌ Name 'MySkill' should be hyphen-case (lowercase letters, digits, and hyphens only)

步驟 2:修復問題
Claude:
1. 讀取 SKILL.md
2. 修正 frontmatter:
   name: MySkill → name: my-skill
3. 更新檔案

步驟 3:重新驗證
Claude 執行:
python scripts/quick_validate.py skills/my-skill

輸出:
✅ Skill is valid!

步驟 4:打包
Claude 執行:
python scripts/package_skill.py skills/my-skill

輸出:
✅ Successfully packaged skill to: my-skill.zip
```

### 不使用此 Skill 的情況

- 使用現有的 skills(不需要建立新的)
- 簡單的一次性任務(不需要重複使用)
- 通用的程式設計任務(不需要專門的 skill)

---

## 技術架構

### 技術堆疊

**Python 生態系**:
```
Python 3.x         # 執行環境
pathlib            # 路徑操作
zipfile            # ZIP 壓縮
re                 # 正規表達式
sys                # 系統介面
```

**檔案格式**:
```
YAML               # Frontmatter 元資料
Markdown           # SKILL.md 內容
ZIP                # 打包格式
```

### 依賴關係圖

```
Skill Creator
├── init_skill.py
│   ├── SKILL_TEMPLATE (YAML + Markdown)
│   ├── EXAMPLE_SCRIPT (Python)
│   ├── EXAMPLE_REFERENCE (Markdown)
│   └── EXAMPLE_ASSET (Text)
├── package_skill.py
│   ├── quick_validate.py
│   └── zipfile
└── quick_validate.py
    └── re (正規表達式)
```

### Skill 結構模型

```
Skill 目錄
├── SKILL.md (必要)
│   ├── YAML Frontmatter
│   │   ├── name: (必要)
│   │   ├── description: (必要)
│   │   └── license: (可選)
│   └── Markdown 內容
│       ├── Overview
│       ├── 主要章節(依結構模式)
│       └── Resources 說明
└── Bundled Resources (可選)
    ├── scripts/
    │   └── *.py, *.sh
    ├── references/
    │   └── *.md
    └── assets/
        └── *.* (任何檔案)
```

### 驗證流程

```
驗證流程
    ↓
[檢查 SKILL.md 存在]
    ↓
[檢查 Frontmatter 格式]
    ↓
[提取 Frontmatter]
    ↓
[檢查必要欄位]
    ├─ name: 存在?
    └─ description: 存在?
    ↓
[驗證 name 格式]
    ├─ Hyphen-case?
    ├─ 無開始/結束連字號?
    └─ 無連續連字號?
    ↓
[驗證 description]
    └─ 無角括號?
    ↓
[回傳結果]
    ├─ ✅ Skill is valid!
    └─ ❌ Error message
```

---

## 使用的 Prompt 策略

### 1. 結構化提問

**六步驟流程**:
```markdown
## Step 1: Understanding the Skill with Concrete Examples
Skip this step only when the skill's usage patterns are already clearly understood.

## Step 2: Planning the Reusable Skill Contents
...

## Step 3: Initializing the Skill
...
```

**優點**:
- 清晰的流程
- 可跳過不適用的步驟
- 系統化的方法

### 2. TODO 驅動開發

**模板中的 TODO**:
```markdown
[TODO: Complete and informative explanation of what the skill does and when to use it]
[TODO: 1-2 sentences explaining what this skill enables]
[TODO: Choose the structure that best fits this skill's purpose...]
```

**設計理念**:
- 明確的任務清單
- 防止遺漏
- 提供填寫指引

### 3. 範例嵌入

**每個模板都包含範例**:
```markdown
Example real scripts from other skills:
- pdf/scripts/fill_fillable_fields.py - Fills PDF form fields
- pdf/scripts/convert_pdf_to_images.py - Converts PDF pages to images
```

**優點**:
- 具體展示實際用法
- 減少歧義
- 加速理解

### 4. 決策指引

**結構選擇指引**:
```markdown
[TODO: Choose the structure that best fits this skill's purpose. Common patterns:

**1. Workflow-Based** (best for sequential processes)
- Works well when there are clear step-by-step procedures
- Example: DOCX skill with "Workflow Decision Tree"

**2. Task-Based** (best for tool collections)
...

Patterns can be mixed and matched as needed.
```

**優點**:
- 提供多種選擇
- 說明適用場景
- 給予實際範例

### 5. 自我驗證提示

**驗證提示**:
```markdown
## Step 5: Packaging a Skill

The packaging script will:
1. **Validate** the skill automatically, checking:
   - YAML frontmatter format and required fields
   - Skill naming conventions
   - Description completeness
```

**設計理念**:
- 內建品質檢查
- 自動化驗證
- 清晰的錯誤訊息

### 6. 漸進式揭露說明

**三層設計的解釋**:
```markdown
### Progressive Disclosure Design Principle

Skills use a three-level loading system:
1. **Metadata** - Always in context (~100 words)
2. **SKILL.md body** - When skill triggers (<5k words)
3. **Bundled resources** - As needed (Unlimited*)

*Unlimited because scripts can be executed without reading into context window.
```

**優點**:
- 解釋設計原理
- 幫助理解 context 管理
- 指導資源組織

---

## 使用的工具(Tools)

此 Skill 主要依賴以下 Claude Code 工具:

### 1. **Bash Tool**
- 執行 init_skill.py
- 執行 package_skill.py
- 執行 quick_validate.py
- 建立目錄和檔案
- 設定檔案權限

### 2. **Write Tool**
- 建立 SKILL.md
- 建立腳本檔案
- 建立參考文檔
- 建立資產檔案

### 3. **Read Tool**
- 讀取 SKILL.md(現有 skill)
- 讀取腳本檔案
- 讀取參考範例
- 驗證檔案內容

### 4. **Edit Tool**
- 更新現有 SKILL.md
- 修改腳本
- 調整文檔
- 修正驗證錯誤

---

## 最佳實踐

### 1. Skill 建立最佳實踐

**描述撰寫**:
```yaml
# ✅ 好的描述
description: Guide for editing PDF files including rotation, merging, splitting, and form filling. Use this skill when users need to manipulate PDF documents or extract PDF content.

# ❌ 不好的描述
description: PDF tool

# ❌ 使用第二人稱
description: Use this skill when you need to edit PDFs

# ✅ 使用第三人稱
description: This skill should be used when editing PDFs is required
```

**特點**:
- 詳細說明功能
- 明確觸發條件
- 使用第三人稱
- 完整且資訊豐富

**命名規範**:
```
✅ pdf-editor
✅ excel-helper
✅ bigquery-connector
✅ brand-guidelines-2024

❌ PDFEditor
❌ excel_helper
❌ BigQueryConnector
❌ brand-guidelines-
```

**SKILL.md 結構**:
```markdown
# ✅ 清晰的結構
## Overview
簡短的概述(1-2 句)

## Quick Start
快速開始指南

## Main Sections
依據選擇的模式組織

## Resources
說明 bundled resources

# ❌ 缺乏結構
[一大段文字混在一起]
```

### 2. 資源組織最佳實踐

**scripts/ 使用**:
```python
# ✅ 好的腳本組織
scripts/
├── rotate_pdf.py        # 單一功能
├── merge_pdfs.py        # 單一功能
├── extract_text.py      # 單一功能
└── utils.py             # 共用工具

# ❌ 不好的組織
scripts/
└── all_pdf_operations.py  # 所有功能混在一起
```

**references/ 使用**:
```markdown
# ✅ 好的參考文檔
references/
├── api_reference.md     # API 文檔
├── workflow_guide.md    # 詳細工作流程
└── troubleshooting.md   # 常見問題

每個檔案 < 10k 字
SKILL.md 包含 grep 模式

# ❌ 不好的組織
references/
└── everything.md        # 50k 字的單一檔案
```

**assets/ 使用**:
```
# ✅ 好的資產組織
assets/
├── templates/
│   ├── report.pptx
│   └── invoice.docx
├── images/
│   └── logo.png
└── fonts/
    └── custom.ttf

# ❌ 不好的組織
assets/
├── file1.png
├── template.pptx
├── random.txt
└── ... (雜亂無章)
```

### 3. 驗證和打包最佳實踐

**打包前驗證**:
```bash
# ✅ 先驗證再打包
python scripts/quick_validate.py skills/my-skill
python scripts/package_skill.py skills/my-skill

# ❌ 直接打包(可能包含錯誤)
python scripts/package_skill.py skills/my-skill
```

**錯誤修復**:
```bash
# 1. 執行驗證
python scripts/quick_validate.py skills/my-skill
# 輸出: ❌ Name 'MySkill' should be hyphen-case

# 2. 修正問題
# 編輯 SKILL.md,修正 name

# 3. 重新驗證
python scripts/quick_validate.py skills/my-skill
# 輸出: ✅ Skill is valid!

# 4. 打包
python scripts/package_skill.py skills/my-skill
```

### 4. 迭代改進最佳實踐

**記錄使用問題**:
```markdown
使用 skill 時記錄:
- 哪些指示不清楚?
- 缺少什麼功能?
- 哪些腳本經常需要修改?
- 哪些參考文檔需要補充?
```

**系統化改進**:
```
1. 測試 skill
   ↓
2. 記錄問題
   ↓
3. 分析根本原因
   ↓
4. 實作改進
   ├─ 更新 SKILL.md
   ├─ 改進腳本
   └─ 補充文檔
   ↓
5. 重新打包
   ↓
6. 再次測試
```

### 5. Context 優化最佳實踐

**SKILL.md 簡潔性**:
```markdown
# ✅ 簡潔的 SKILL.md
## Overview
Brief explanation

## Quick Start
Essential steps

## Common Operations
Key workflows with references to detailed docs

See references/detailed_guide.md for comprehensive information.

# ❌ 過長的 SKILL.md
## Overview
## Every Single Detail
## All API Endpoints
## Complete Specifications
[10k+ words]
```

**參考文檔策略**:
```markdown
# ✅ 在 SKILL.md 中提供 grep 模式
For BigQuery schema details, see `references/schema.md`
Use grep pattern: "table: user_events"

# ❌ 無搜尋指引
See references/schema.md for details
```

---

## 總結

### 成功要素

1. **結構化流程**:六步驟建立流程確保品質
2. **自動化工具**:init、validate、package 三個腳本提高效率
3. **清晰的指引**:模板和範例減少歧義
4. **驗證機制**:自動檢查確保規範遵循
5. **漸進式揭露**:三層設計優化 context 使用
6. **靈活的結構**:四種模式適應不同需求

### 學習重點

對於想要建立 skills 的開發者:
- Skill 的結構和組成部分
- 漸進式揭露設計原則
- 資源分類(scripts、references、assets)
- YAML frontmatter 格式
- 命名規範(hyphen-case)
- 驗證和打包流程

### 適用場景

此 Skill 最適合用於:
- 建立新的 Claude Skills
- 更新和改進現有 skills
- 標準化 skill 格式
- 驗證 skill 品質
- 打包 skills 用於分發

### 技術亮點

- **模板系統**:完整的 SKILL.md 模板和範例檔案
- **自動驗證**:正規表達式驗證命名和格式
- **自動打包**:保留目錄結構的 ZIP 壓縮
- **資源分類**:清晰的 scripts/references/assets 區分
- **四種結構模式**:適應不同類型的 skills
- **漸進式揭露**:優化 context window 使用

### 企業價值

**提高開發效率**:
- 自動化工具減少手動工作
- 模板加速 skill 建立
- 驗證防止錯誤

**確保品質一致性**:
- 標準化的結構
- 自動驗證機制
- 清晰的最佳實踐

**降低維護成本**:
- 清晰的組織結構
- 完整的文檔
- 系統化的更新流程

---

## 參考資源

- [SKILL.md](./SKILL.md) - 英文原始文檔
- [scripts/init_skill.py](./scripts/init_skill.py) - 初始化腳本
- [scripts/package_skill.py](./scripts/package_skill.py) - 打包腳本
- [scripts/quick_validate.py](./scripts/quick_validate.py) - 驗證腳本

---

**最後更新**:2025-11-15
**文件版本**:1.0.0
**適用於**:Claude Code Skills
