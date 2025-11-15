# DOCX - Word 文件處理正體中文說明文件

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
DOCX 是一個全方位的 Microsoft Word 文件處理工具套件，提供文件創建、編輯、分析和追蹤修訂（Track Changes）的完整功能。它使用 Python 和 JavaScript 庫來操作 Office Open XML (OOXML) 格式，使 Claude 能夠：
- 建立全新的 Word 文件（使用 docx-js）
- 編輯現有文件並保留格式
- 實施專業的追蹤修訂工作流程（Redlining）
- 添加註解和批註
- 讀取和分析文件內容
- 轉換文件為 Markdown 格式

### 解決的問題
在處理 Word 文件時，經常遇到以下挑戰：

1. **格式保留困難**：直接操作 .docx 文件容易破壞原有格式和樣式
2. **追蹤修訂複雜**：專業的文件審核需要精確的追蹤修訂（Track Changes）功能
3. **XML 結構複雜**：OOXML 格式層級深，手動編輯容易出錯
4. **批量修改挑戰**：需要對多處內容進行一致性修改時效率低
5. **版本控制需求**：法律、學術、商業文件需要完整的修訂歷程

這個 Skill 透過提供結構化的文件庫和自動化工具解決了這些問題，讓 Claude 能夠像專業文件編輯器一樣精確操作 Word 文件。

---

## 目錄結構

```
docx/
├── SKILL.md                    # Skill 主要說明文件（英文）
├── LICENSE.txt                 # 授權條款
├── README.zh-TW.md            # 正體中文說明文件（本文件）
├── docx-js.md                 # docx-js 庫使用指南
├── ooxml.md                   # OOXML 操作詳細文檔
└── scripts/                    # Python 腳本目錄
    ├── __init__.py            # 套件初始化檔案
    ├── document.py            # Document 類別 - 高階文件操作
    └── utilities.py           # XMLEditor 類別 - 低階 XML 操作
```

---

## 檔案說明

### 核心檔案

#### 1. `SKILL.md`
- **用途**：Skill 的主要說明文件，提供完整的使用指南
- **內容**：
  - 文件讀取與分析方法
  - 建立新文件的工作流程
  - 編輯現有文件的步驟
  - Redlining（追蹤修訂）工作流程
  - 文件轉圖片的方法
- **特點**：提供決策樹幫助選擇正確的工作流程

#### 2. `docx-js.md`
- **用途**：docx-js 庫的詳細使用指南
- **內容**：
  - Document、Paragraph、TextRun 等組件的語法
  - 格式化選項（粗體、斜體、顏色等）
  - 表格、圖片、頁首頁尾的創建
  - 樣式和段落格式設定
- **重要性**：建立新文件時的必讀文檔

#### 3. `ooxml.md`
- **用途**：OOXML 格式和 Document 庫的深度指南
- **內容**：
  - Document 庫 API 說明
  - 追蹤修訂的 XML 模式
  - 註解系統的實作方式
  - DOM 直接操作方法
- **重要性**：編輯現有文件時的核心參考

### Scripts 目錄

#### 1. `document.py`（1,277 行）
**功能**：提供高階的 Word 文件操作介面

**主要類別**：

**`DocxXMLEditor` 類別**（繼承自 XMLEditor）：
- 自動注入 RSID（Revision Session ID）屬性
- 自動添加作者和時間戳資訊
- 提供追蹤修訂專用方法：
  - `suggest_deletion()` - 標記刪除
  - `revert_insertion()` - 拒絕插入
  - `revert_deletion()` - 拒絕刪除
  - `suggest_paragraph()` - 將段落標記為插入

**`Document` 類別**：
- 管理整個文件的操作
- 自動設定追蹤修訂基礎設施
- 提供註解管理功能：
  - `add_comment()` - 添加註解
  - `reply_to_comment()` - 回覆註解
- 內建驗證機制（Schema 和 Redlining 驗證）
- 臨時目錄管理，確保原始文件安全

**關鍵功能**：
```python
# 初始化文件
doc = Document('workspace/unpacked', author="John Doe")

# 查找節點
node = doc["word/document.xml"].get_node(tag="w:p", line_number=42)

# 添加追蹤刪除
doc["word/document.xml"].suggest_deletion(node)

# 添加註解
doc.add_comment(start=node, end=node, text="需要修改此段落")

# 儲存並驗證
doc.save()  # 自動驗證
```

**設計亮點**：
- 自動處理所有 OOXML 命名空間（w14, w16du, w16cex）
- 智慧生成唯一 ID（paraId, durableId, change ID）
- 保護性複製：操作在臨時目錄進行，原始文件不受影響

#### 2. `utilities.py`（375 行）
**功能**：提供低階 XML 操作工具

**主要類別**：

**`XMLEditor` 類別**：
- 解析 XML 並追蹤每個元素的原始行號
- 支援多種查找方式：
  - 按行號查找（`line_number=519`）
  - 按行號範圍查找（`line_number=range(100, 200)`）
  - 按屬性查找（`attrs={"w:id": "1"}`）
  - 按文字內容查找（`contains="特定文字"`）
- DOM 操作方法：
  - `replace_node()` - 替換節點
  - `insert_before()` / `insert_after()` - 插入節點
  - `append_to()` - 附加子節點
- 自動處理命名空間
- 支援 ASCII 和 UTF-8 編碼

**關鍵功能**：
```python
# 建立編輯器
editor = XMLEditor("word/document.xml")

# 按行號查找
elem = editor.get_node(tag="w:r", line_number=519)

# 按範圍查找
elem = editor.get_node(tag="w:p", line_number=range(100, 200))

# 按內容查找（支援 HTML 實體）
elem = editor.get_node(tag="w:t", contains="&#8220;Agreement")

# 替換節點
new_elem = editor.replace_node(elem, "<w:r><w:t>新文字</w:t></w:r>")

# 儲存變更
editor.save()
```

**技術亮點**：
- **行號追蹤**：使用 SAX 解析器追蹤每個元素的原始位置
- **智慧查找**：支援多條件組合查找
- **HTML 實體處理**：自動正規化搜尋字串（`&#8220;` ↔ `"`）
- **命名空間管理**：自動從根元素複製所有命名空間宣告

#### 3. `__init__.py`
- **用途**：使 scripts 目錄成為 Python 套件
- **內容**：空檔案，用於套件結構

---

## 關鍵腳本分析

### document.py 深度分析

#### 1. 自動屬性注入機制

**`_inject_attributes_to_nodes()` 方法**：
```python
def _inject_attributes_to_nodes(self, nodes):
    # 自動添加必要屬性：
    # - w:p 元素：w:rsidR, w:rsidRDefault, w:rsidP, w14:paraId, w14:textId
    # - w:r 元素：w:rsidR（或 w:rsidDel 如在刪除中）
    # - w:t 元素：xml:space="preserve"（如有前後空白）
    # - w:ins/w:del：w:id, w:author, w:date, w16du:dateUtc
    # - w:comment：w:author, w:date, w:initials
```

**設計理念**：
- Word 需要這些屬性來正確追蹤修訂歷程
- 自動化避免遺漏必要屬性
- 確保文件符合 OOXML 規範

#### 2. 追蹤修訂的精確實作

**最小化原則**：
```python
# ❌ 錯誤 - 替換整個句子
'<w:del><w:r><w:delText>期限為 30 天。</w:delText></w:r></w:del>'
'<w:ins><w:r><w:t>期限為 60 天。</w:t></w:r></w:ins>'

# ✅ 正確 - 只標記變更的部分
'<w:r w:rsidR="00AB12CD"><w:t>期限為 </w:t></w:r>'
'<w:del><w:r><w:delText>30</w:delText></w:r></w:del>'
'<w:ins><w:r><w:t>60</w:t></w:r></w:ins>'
'<w:r w:rsidR="00AB12CD"><w:t> 天。</w:t></w:r>'
```

**優點**：
- 審核者只看到真正變更的內容
- 保留原始 run 的 RSID，維持修訂歷程
- 符合專業文件編輯標準

#### 3. 註解系統的完整實作

**多檔案協調**：
```
word/comments.xml          - 主要註解內容
word/commentsExtended.xml  - 擴充資訊（回覆關係）
word/commentsIds.xml       - 唯一識別符對應
word/commentsExtensible.xml - 可擴展屬性
```

**自動化處理**：
- 生成唯一 ID（comment ID, para ID, durable ID）
- 維護四個檔案的同步
- 更新 relationships 和 content types
- 支援註解回覆的層級結構

#### 4. 驗證機制

**雙重驗證**：
```python
def validate(self):
    # Schema 驗證：確保 XML 符合 OOXML 規範
    schema_validator = DOCXSchemaValidator(...)

    # Redlining 驗證：確保追蹤修訂正確實作
    redlining_validator = RedliningValidator(...)

    if not schema_validator.validate():
        raise ValueError("Schema validation failed")
    if not redlining_validator.validate():
        raise ValueError("Redlining validation failed")
```

### utilities.py 深度分析

#### 1. 行號追蹤的實作

**SAX 解析器客製化**：
```python
def _create_line_tracking_parser():
    def startElementNS(name, tagName, attrs):
        orig_start_cb(name, tagName, attrs)
        cur_elem = dom_handler.elementStack[-1]
        # 儲存 expat 解析器的當前位置
        cur_elem.parse_position = (
            parser._parser.CurrentLineNumber,
            parser._parser.CurrentColumnNumber,
        )
```

**應用價值**：
- Claude 讀取檔案時看到行號
- 可以直接使用 `line_number=42` 找到對應元素
- 避免複雜的 XPath 表達式

#### 2. 智慧節點查找

**多條件組合**：
```python
# 單一條件
elem = editor.get_node(tag="w:p", line_number=42)

# 組合條件
elem = editor.get_node(
    tag="w:p",
    line_number=range(1, 50),
    contains="特定文字"
)

# 屬性查找
elem = editor.get_node(
    tag="w:del",
    attrs={"w:id": "1"}
)
```

**錯誤訊息智慧化**：
```python
# 根據使用的篩選器提供不同提示
if contains:
    hint = "文字可能分散在多個元素中或使用不同措辭。"
elif line_number:
    hint = "如果文件已修改，行號可能已變更。"
elif attrs:
    hint = "請驗證屬性值是否正確。"
```

---

## 設計策略

### 1. 分層架構

**三層設計**：
1. **高階介面層**（Document 類別）：
   - 業務邏輯操作（添加註解、追蹤修訂）
   - 自動化基礎設施設定
   - 驗證和錯誤處理

2. **中階操作層**（DocxXMLEditor 類別）：
   - 自動屬性注入
   - 追蹤修訂專用方法
   - RSID 和時間戳管理

3. **低階 XML 層**（XMLEditor 類別）：
   - 純粹的 DOM 操作
   - 行號追蹤
   - 命名空間處理

### 2. 防禦性程式設計

**保護原始文件**：
```python
# 建立臨時目錄結構
self.temp_dir = tempfile.mkdtemp(prefix="docx_")
self.unpacked_path = Path(self.temp_dir) / "unpacked"
shutil.copytree(self.original_path, self.unpacked_path)

# 操作在臨時目錄進行
# 只有明確呼叫 save() 才會寫回原始位置
```

**自動清理**：
```python
def __del__(self):
    """解構時自動清理臨時目錄"""
    if hasattr(self, "temp_dir") and Path(self.temp_dir).exists():
        shutil.rmtree(self.temp_dir)
```

### 3. 批次處理策略

**Redlining 工作流程**：
- 將修改分組為 3-10 個變更的批次
- 每批次測試後再進行下一批
- 便於除錯和進度追蹤

**分組建議**：
```
批次 1：章節 2 的修正
批次 2：章節 5 的更新
批次 3：日期修正
批次 4：當事人名稱變更
```

### 4. 驗證優先

**內建驗證**：
- save() 預設執行驗證
- Schema 驗證確保符合 OOXML 規範
- Redlining 驗證確保追蹤修訂正確
- 提供清楚的錯誤訊息

---

## Claude Code 使用方式

### 觸發時機

Claude Code 會在以下情況使用此 Skill：

1. **使用者要求建立 Word 文件**
   - 範例：「建立一份會議記錄文件」
   - 範例：「製作一個專案提案書」

2. **需要編輯現有 Word 文件**
   - 範例：「修改合約中的日期」
   - 範例：「更新報告中的數字」

3. **需要追蹤修訂（Redlining）**
   - 範例：「審核這份法律文件並標記修改」
   - 範例：「對合約提出修訂建議」

4. **需要添加註解**
   - 範例：「在文件中添加評論」
   - 範例：「對特定段落提出問題」

5. **需要分析文件內容**
   - 範例：「讀取這份 Word 文件的內容」
   - 範例：「檢查文件中的追蹤修訂」

### 典型工作流程

#### 工作流程 1：建立新文件

```bash
# 步驟 1：讀取 docx-js.md（完整檔案）
# Claude 會讀取 docx-js.md 了解語法

# 步驟 2：建立 JavaScript 檔案
# 使用 Document、Paragraph、TextRun 組件

# 步驟 3：執行腳本生成 .docx
node create-document.js
```

#### 工作流程 2：編輯現有文件（簡單修改）

```bash
# 步驟 1：讀取 ooxml.md（完整檔案）
# 步驟 2：解壓文件
python ooxml/scripts/unpack.py document.docx unpacked/

# 步驟 3：建立並執行 Python 腳本
python edit-script.py

# 步驟 4：重新打包
python ooxml/scripts/pack.py unpacked/ edited-document.docx
```

#### 工作流程 3：Redlining（追蹤修訂）

```bash
# 步驟 1：轉換為 Markdown 並識別修改
pandoc --track-changes=all document.docx -o current.md

# 步驟 2：規劃修改批次
# 批次 1：章節 2 修改（3-10 個變更）
# 批次 2：章節 5 更新
# ...

# 步驟 3：讀取 ooxml.md（完整檔案）
# 步驟 4：解壓文件
python ooxml/scripts/unpack.py document.docx unpacked/
# 注意建議的 RSID（用於追蹤修訂）

# 步驟 5：對每個批次
# 5a. 使用 grep 查找 XML 中的文字位置
grep -n "目標文字" unpacked/word/document.xml

# 5b. 建立並執行腳本（使用 Document 庫）
python batch-1-changes.py

# 步驟 6：完成所有批次後打包
python ooxml/scripts/pack.py unpacked/ reviewed-document.docx

# 步驟 7：驗證所有修改
pandoc --track-changes=all reviewed-document.docx -o verification.md
grep "原始詞句" verification.md  # 不應找到
grep "替換詞句" verification.md  # 應該找到
```

### 不使用此 Skill 的情況
- 簡單的文字檔案（使用 .txt）
- 需要 PDF 格式（使用 pdf skill）
- 試算表操作（使用 xlsx skill）
- 簡報製作（使用 pptx skill）

---

## 技術架構

### 技術堆疊

**Python 生態系**：
```
defusedxml         # 安全的 XML 解析（防止 XXE 攻擊）
minidom            # DOM 操作
openpyxl           # Excel 檔案操作（依賴）
```

**JavaScript/Node.js 生態系**：
```
docx               # 建立 Word 文件的 JavaScript 庫
```

**命令列工具**：
```
pandoc             # 文件格式轉換（DOCX ↔ Markdown）
soffice            # LibreOffice（DOCX → PDF 轉換）
pdftoppm           # Poppler 工具（PDF → 圖片）
```

### 依賴關係圖

```
Document 類別
├── DocxXMLEditor（繼承自 XMLEditor）
│   ├── defusedxml.minidom - DOM 操作
│   ├── RSID 自動生成
│   └── 屬性自動注入
├── 驗證器
│   ├── DOCXSchemaValidator - XSD 驗證
│   └── RedliningValidator - 追蹤修訂驗證
├── 檔案系統操作
│   ├── pack.py - 打包 DOCX
│   ├── unpack.py - 解壓 DOCX
│   └── tempfile - 臨時目錄管理
└── 範本檔案
    ├── people.xml
    ├── comments.xml
    ├── commentsExtended.xml
    ├── commentsIds.xml
    └── commentsExtensible.xml
```

### OOXML 文件結構

```
document.docx（ZIP 檔案）
├── [Content_Types].xml          - 內容類型定義
├── _rels/
│   └── .rels                     - 根關係
├── word/
│   ├── document.xml              - 主要文件內容
│   ├── comments.xml              - 註解
│   ├── people.xml                - 作者資訊
│   ├── settings.xml              - 文件設定（含 RSID）
│   ├── media/                    - 嵌入的圖片和媒體
│   └── _rels/
│       └── document.xml.rels     - 文件關係
└── docProps/
    ├── core.xml                  - 核心屬性（標題、作者等）
    └── app.xml                   - 應用程式屬性
```

### 追蹤修訂的 XML 模式

**插入**：
```xml
<w:ins w:id="1" w:author="Claude" w:date="2025-11-14T12:00:00Z">
  <w:r>
    <w:t>新增的文字</w:t>
  </w:r>
</w:ins>
```

**刪除**：
```xml
<w:del w:id="2" w:author="Claude" w:date="2025-11-14T12:00:00Z">
  <w:r>
    <w:delText>刪除的文字</w:delText>
  </w:r>
</w:del>
```

**段落刪除**（編號列表）：
```xml
<w:p>
  <w:pPr>
    <w:rPr><w:del/></w:rPr>
  </w:pPr>
  <w:del>
    <w:r><w:delText>段落內容</w:delText></w:r>
  </w:del>
</w:p>
```

---

## 使用的 Prompt 策略

### 1. 決策樹導引

**SKILL.md 提供清晰的決策路徑**：
```
需要做什麼？
├─ 讀取/分析 → 文字提取 or 原始 XML 存取
├─ 建立新文件 → 使用 docx-js
└─ 編輯現有文件
   ├─ 自己的文件 + 簡單修改 → 基本 OOXML 編輯
   ├─ 他人的文件 → Redlining 工作流程（強烈建議）
   └─ 法律/學術/商業/政府文件 → Redlining 工作流程（必須）
```

### 2. 強制完整讀取

**關鍵文件的讀取指示**：
```markdown
**MANDATORY - READ ENTIRE FILE**: Read [`docx-js.md`](docx-js.md)
(~500 lines) completely from start to finish.
**NEVER set any range limits when reading this file.**
```

**設計理念**：
- 確保 Claude 了解完整的 API 和模式
- 避免因部分資訊導致錯誤
- 特別重要的文件使用此策略

### 3. 範例驅動學習

**提供多種程式碼範例**：
```python
# 範例 1：基本操作
doc = Document('workspace/unpacked')
node = doc["word/document.xml"].get_node(tag="w:p", line_number=42)
doc.save()

# 範例 2：追蹤刪除
doc["word/document.xml"].suggest_deletion(node)

# 範例 3：註解
doc.add_comment(start=node, end=node, text="註解文字")
```

### 4. 錯誤預防提示

**最小化原則的強調**：
```markdown
**Principle: Minimal, Precise Edits**
When implementing tracked changes, only mark text that actually changes.
Repeating unchanged text makes edits harder to review and appears unprofessional.
```

**批次處理建議**：
```markdown
**Batching Strategy**: Group related changes into batches of 3-10 changes.
This makes debugging manageable while maintaining efficiency.
```

---

## 使用的工具（Tools）

此 Skill 主要依賴以下 Claude Code 工具：

### 1. **Read Tool**
- 讀取 SKILL.md、docx-js.md、ooxml.md
- 檢查腳本內容（document.py、utilities.py）
- 讀取解壓後的 XML 檔案
- 驗證生成的檔案

### 2. **Write Tool**
- 建立 Python 腳本（編輯工作流程）
- 建立 JavaScript 檔案（docx-js 工作流程）
- 撰寫驗證腳本

### 3. **Bash Tool**
- 執行 pandoc 轉換
- 執行 unpack.py 和 pack.py
- 執行 grep 查找 XML 內容
- 執行 soffice 和 pdftoppm（轉換為圖片）
- 執行 Python 和 Node.js 腳本

### 4. **Edit Tool**
- 修改現有腳本
- 調整配置檔

### 5. **Grep Tool**
- 在 XML 檔案中搜尋特定文字
- 驗證修改結果
- 查找特定 XML 元素

---

## 最佳實踐

### 1. 文件操作流程

**✅ 建議做法**：
```python
# 使用 Document 類別的保護機制
doc = Document('workspace/unpacked', author="審核者")

# 操作在臨時目錄進行
doc["word/document.xml"].suggest_deletion(node)

# 自動驗證後才儲存
doc.save()  # 失敗時會拋出例外
```

**❌ 避免做法**：
```python
# 直接修改原始 XML（危險）
with open('word/document.xml', 'w') as f:
    f.write(modified_xml)  # 可能破壞檔案
```

### 2. 追蹤修訂最佳實踐

**精確標記變更**：
- ✅ 只標記實際變更的文字
- ✅ 保留原始 run 的 RSID
- ❌ 不要替換整個句子或段落

**批次處理**：
- ✅ 分組為 3-10 個相關變更
- ✅ 按章節、類型或位置分組
- ✅ 每批次測試後再繼續
- ❌ 不要一次處理所有變更

**驗證流程**：
```bash
# 每批次後驗證
pandoc --track-changes=all output.docx -o verify.md
grep "預期內容" verify.md
```

### 3. 節點查找策略

**優先使用**：
```python
# 1. 章節/標題編號（最穩定）
# 2. 段落識別符（如有編號）
# 3. grep 模式（配合唯一周圍文字）
# 4. 文件結構（如「第一段」、「簽名區塊」）
```

**避免使用**：
```python
# ❌ Markdown 行號（不對應 XML 結構）
# ❌ 過於通用的搜尋文字（可能有多處）
```

### 4. 程式碼風格

**簡潔為上**：
```python
# ✅ 簡潔
doc = Document('unpacked')
node = doc["word/document.xml"].get_node(tag="w:p", line_number=42)
doc["word/document.xml"].suggest_deletion(node)
doc.save()

# ❌ 冗長
document_manager = Document('unpacked')
target_paragraph_node = document_manager["word/document.xml"].get_node(
    tag="w:p",
    line_number=42
)
document_manager["word/document.xml"].suggest_deletion(target_paragraph_node)
document_manager.save()
```

### 5. 錯誤處理

**預期驗證失敗**：
```python
try:
    doc.save()  # 自動驗證
except ValueError as e:
    # 檢查錯誤訊息
    # 修正問題
    # 重新嘗試
```

**使用 grep 驗證**：
```bash
# 確認修改已套用
grep "新文字" unpacked/word/document.xml

# 確認舊文字已移除
grep "舊文字" unpacked/word/document.xml  # 應該找不到
```

---

## 總結

### 成功要素

1. **分層設計**：三層架構（Document → DocxXMLEditor → XMLEditor）提供不同抽象層級
2. **自動化**：自動注入屬性、自動驗證、自動清理
3. **防禦性**：臨時目錄保護、驗證機制、清晰錯誤訊息
4. **專業性**：完整的追蹤修訂支援，符合法律和商業文件標準
5. **靈活性**：既有高階介面也有低階 DOM 存取

### 學習重點

對於想要理解此 Skill 的開發者：
- OOXML 格式的結構和命名空間系統
- DOM 操作和 SAX 解析的結合使用
- 追蹤修訂的 XML 模式
- 臨時檔案管理和防禦性程式設計
- 多檔案系統的協調（comments、relationships 等）

### 適用場景

此 Skill 最適合用於：
- 📄 合約審核和修訂
- 📋 法律文件處理
- 📝 學術論文協作
- 📊 商業提案編輯
- 🏢 政府文件處理
- 💼 專業報告撰寫

### 技術創新

- **行號追蹤**：將 Read tool 的行號直接對應到 XML 元素
- **自動屬性注入**：減少手動錯誤，確保符合規範
- **批次處理框架**：平衡效率和可除錯性
- **雙重驗證**：Schema + Redlining 確保品質

---

## 參考資源

- [SKILL.md](./SKILL.md) - 英文原始文檔
- [docx-js.md](./docx-js.md) - docx-js 庫詳細文檔
- [ooxml.md](./ooxml.md) - OOXML 和 Document 庫指南
- [Office Open XML 規範](http://www.ecma-international.org/publications/standards/Ecma-376.htm)
- [python-docx 文檔](https://python-docx.readthedocs.io/)
- [Pandoc 文檔](https://pandoc.org/)

---

**最後更新**：2025-11-14
**文件版本**：1.0.0
**適用於**：Claude Code Skills
