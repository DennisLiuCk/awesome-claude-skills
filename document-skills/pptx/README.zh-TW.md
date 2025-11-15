# PPTX - PowerPoint 簡報處理正體中文說明文件

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
PPTX 是一個全方位的 Microsoft PowerPoint 簡報處理工具套件，提供簡報的建立、編輯、分析和範本重用功能。它使用 Python 和 JavaScript 庫來操作 Office Open XML (OOXML) 格式，使 Claude 能夠：
- 從零建立全新的 PowerPoint 簡報（使用 html2pptx）
- 編輯現有簡報並保留格式
- 基於範本建立新簡報（重排和替換內容）
- 提取簡報文字內容和結構
- 視覺化分析簡報（縮圖網格）
- 檢測文字溢出和形狀重疊問題
- 建立專業設計的簡報（避免 AI 風格）

### 解決的問題
在處理 PowerPoint 簡報時，經常遇到以下挑戰：

1. **設計一致性困難**：AI 生成的簡報常有「AI 風格」（過度置中、紫色漸層、統一圓角）
2. **範本重用複雜**：需要基於現有範本建立新簡報，但手動複製貼上效率低
3. **內容溢出問題**：文字超出文字框邊界，在簡報時才發現
4. **形狀重疊檢測**：手動檢查數十張投影片的重疊問題耗時
5. **批次處理需求**：需要產生多份相似簡報（如每月報告）
6. **視覺驗證困難**：需要快速預覽整份簡報的版面配置

這個 Skill 透過提供專門的腳本、視覺分析工具和完整的工作流程解決了這些問題，特別是針對專業簡報製作的需求。

---

## 目錄結構

```
pptx/
├── SKILL.md                      # Skill 主要說明文件（英文）
├── LICENSE.txt                   # 授權條款
├── README.zh-TW.md              # 正體中文說明文件（本文件）
├── html2pptx.md                 # html2pptx 工作流程指南
├── ooxml.md                     # OOXML 操作詳細文檔
└── scripts/                      # Python/JavaScript 腳本目錄
    ├── thumbnail.py              # 建立縮圖網格
    ├── inventory.py              # 提取文字庫存和檢測問題
    ├── replace.py                # 替換文字內容
    ├── rearrange.py              # 重新排列投影片
    └── html2pptx.js              # HTML 轉 PowerPoint
```

---

## 檔案說明

### 核心檔案

#### 1. `SKILL.md`
- **用途**：Skill 的主要說明文件，提供完整的使用指南
- **內容**：
  - 簡報讀取與分析方法
  - 建立新簡報的工作流程（無範本）
  - 編輯現有簡報的步驟
  - 基於範本建立簡報的詳細流程
  - 縮圖網格建立方法
  - 轉換為圖片的方法
- **特點**：提供決策樹和詳細的步驟說明

#### 2. `html2pptx.md`
- **用途**：html2pptx 工作流程的詳細指南
- **內容**：
  - HTML 轉 PowerPoint 的原理
  - 支援的 HTML 標籤和樣式
  - 圖表和表格的添加方法
  - 設計原則和色彩選擇
  - 版面配置最佳實踐
- **重要性**：建立無範本簡報時的必讀文檔

#### 3. `ooxml.md`
- **用途**：OOXML 格式和直接 XML 編輯指南
- **內容**：
  - OOXML 結構說明
  - XML 檔案關係
  - 驗證機制
- **適用**：需要直接編輯 XML 的進階場景

### Scripts 目錄

#### 1. `thumbnail.py`（451 行）
**功能**：建立簡報的縮圖網格，用於快速視覺預覽

**主要功能**：
- 將 PowerPoint 轉換為 PDF，再轉為圖片
- 建立網格佈局（3-6 欄可選）
- 支援隱藏投影片的處理（顯示為灰色斜線）
- 自動分頁（超過單頁容量時建立多個網格）
- 可選：標示文字占位符位置

**使用範例**：
```bash
# 基本用法（預設 5 欄）
python thumbnail.py presentation.pptx

# 指定輸出名稱和欄數
python thumbnail.py template.pptx analysis --cols 4

# 標示占位符（用於範本分析）
python thumbnail.py template.pptx --outline-placeholders
```

**輸出**：
```
# 單一網格
thumbnails.jpg

# 多個網格（大型簡報）
thumbnails-1.jpg
thumbnails-2.jpg
thumbnails-3.jpg
```

**網格容量**：
- 3 欄：最多 12 張投影片（3×4）
- 4 欄：最多 20 張投影片（4×5）
- 5 欄：最多 30 張投影片（5×6）【預設】
- 6 欄：最多 42 張投影片（6×7）

#### 2. `inventory.py`（1,021 行）
**功能**：提取簡報的文字庫存並檢測版面問題

**核心功能**：
1. **文字提取**：
   - 提取所有文字形狀的內容
   - 保留段落格式（對齊、項目符號、字型等）
   - 處理群組形狀（GroupShape）的絕對位置
   - 按視覺位置排序形狀

2. **問題檢測**：
   - **文字溢出**：文字超出文字框底部
   - **投影片溢出**：形狀超出投影片邊界
   - **形狀重疊**：兩個形狀重疊超過容忍值
   - **格式警告**：手動項目符號符號（•）

3. **智慧過濾**：
   - 自動跳過投影片編號
   - 過濾空白形狀
   - 可選：只顯示有問題的形狀

**使用範例**：
```bash
# 提取所有文字庫存
python inventory.py presentation.pptx inventory.json

# 只提取有問題的形狀
python inventory.py presentation.pptx issues.json --issues-only
```

**輸出格式**（inventory.json）：
```json
{
  "slide-0": {
    "shape-0": {
      "left": 1.5,
      "top": 2.0,
      "width": 7.5,
      "height": 1.2,
      "placeholder_type": "TITLE",
      "default_font_size": 44.0,
      "paragraphs": [
        {
          "text": "投影片標題",
          "alignment": "CENTER",
          "bold": true,
          "font_size": 44.0
        }
      ],
      "overflow": {
        "frame": {"overflow_bottom": 0.15}
      }
    }
  }
}
```

#### 3. `replace.py`（386 行）
**功能**：根據 JSON 定義替換簡報中的文字內容

**核心功能**：
1. **自動清除**：清除所有庫存中的文字形狀
2. **選擇性替換**：只替換 JSON 中定義的形狀
3. **格式保留**：應用段落和字型屬性
4. **驗證機制**：
   - 檢查 JSON 中的形狀是否存在
   - 檢測替換後的文字溢出
   - 檢查手動項目符號

**使用範例**：
```bash
python replace.py input.pptx replacement.json output.pptx
```

**replacement.json 格式**：
```json
{
  "slide-0": {
    "shape-0": {
      "paragraphs": [
        {
          "text": "新標題",
          "alignment": "CENTER",
          "bold": true,
          "font_size": 44.0
        }
      ]
    }
  }
}
```

**驗證輸出**：
```
ERROR: Replacement text made overflow worse in these shapes:
  - slide-0/shape-2: overflow worsened by 1.25" (was 0.00", now 1.25")

ERROR: Invalid shapes in replacement JSON:
  - Shape 'shape-99' not found on 'slide-0'. Available shapes: shape-0, shape-1, shape-4
```

#### 4. `rearrange.py`（232 行）
**功能**：根據索引序列重新排列投影片

**核心功能**：
1. **複製重複投影片**：自動處理重複使用的投影片
2. **刪除未使用投影片**：移除不在序列中的投影片
3. **重新排序**：按指定順序排列投影片
4. **保留關係**：維護圖片和媒體的關係

**使用範例**：
```bash
# 使用投影片 0, 34（兩次）, 50, 52
python rearrange.py template.pptx output.pptx 0,34,34,50,52
```

**處理流程**：
```
原始範本：73 張投影片
序列：0,34,34,50,52

步驟 1：複製投影片 34（建立副本）
步驟 2：刪除未使用的 68 張投影片
步驟 3：重新排序為指定順序
結果：5 張投影片
```

#### 5. `html2pptx.js`
**功能**：將 HTML 投影片轉換為 PowerPoint

**核心功能**：
- 使用 Playwright 渲染 HTML
- 擷取為高解析度圖片
- 建立 PowerPoint 投影片
- 支援圖表和表格的原生添加

**注意**：此檔案是 JavaScript 模組，需要在建立簡報的腳本中引用。

---

## 關鍵腳本分析

### thumbnail.py 深度分析

#### 1. 隱藏投影片處理

**問題**：PowerPoint 的隱藏投影片在轉 PDF 時會被跳過

**解決方案**：
```python
# 檢測隱藏投影片
hidden_slides = {
    idx + 1
    for idx, slide in enumerate(prs.slides)
    if slide.element.get("show") == "0"
}

# 建立占位符圖片
if slide_num in hidden_slides:
    placeholder_img = create_hidden_slide_placeholder(size)
    # 灰色背景 + 斜線圖案
```

**結果**：縮圖網格保留所有投影片的位置，隱藏投影片顯示為灰色斜線

#### 2. 自動分頁機制

**計算邏輯**：
```python
# 最大投影片數 = 欄數 × (欄數 + 1)
max_images_per_grid = cols * (cols + 1)

# 範例：5 欄 → 5 × 6 = 30 張
# 如果有 75 張投影片：
# 網格 1：投影片 0-29（30 張）
# 網格 2：投影片 30-59（30 張）
# 網格 3：投影片 60-74（15 張）
```

**檔案命名**：
```
# 單一網格
thumbnails.jpg

# 多個網格
thumbnails-1.jpg
thumbnails-2.jpg
thumbnails-3.jpg
```

#### 3. 占位符標示功能

**用途**：分析範本時視覺化文字占位符位置

**實作**：
```python
# 使用 inventory.py 提取占位符位置
placeholder_regions = get_placeholder_regions(pptx_path)

# 在縮圖上繪製紅色外框
for region in placeholder_regions[slide_idx]:
    # 轉換英寸座標為圖片像素
    # 繪製紅色矩形外框
```

**應用**：
```bash
python thumbnail.py template.pptx analysis --outline-placeholders
# 讀取 analysis.jpg，看到所有文字框位置
```

### inventory.py 深度分析

#### 1. 群組形狀的絕對位置計算

**問題**：PowerPoint 中的群組形狀（GroupShape）的子形狀位置是相對於群組的

**解決方案**：
```python
def collect_shapes_with_absolute_positions(
    shape, parent_left=0, parent_top=0
):
    """遞迴計算絕對位置"""
    if hasattr(shape, "shapes"):  # 群組形狀
        # 累積父群組的偏移
        abs_group_left = parent_left + shape.left
        abs_group_top = parent_top + shape.top

        # 處理子形狀
        for child in shape.shapes:
            result.extend(
                collect_shapes_with_absolute_positions(
                    child, abs_group_left, abs_group_top
                )
            )
```

**效果**：
- 正確計算嵌套群組中形狀的位置
- 支援任意深度的群組巢狀
- 位置資訊用於排序和重疊檢測

#### 2. 文字溢出估算

**挑戰**：準確估算文字是否溢出需要考慮多種因素

**實作策略**：
```python
def _estimate_frame_overflow(self):
    """使用 PIL 測量文字高度"""

    # 1. 計算可用空間（扣除邊距）
    usable_width_px, usable_height_px = self._get_usable_dimensions(text_frame)

    # 2. 對每個段落：
    for paragraph in text_frame.paragraphs:
        # a. 載入字型
        font = ImageFont.truetype(font_path, size=font_size)

        # b. 文字換行
        wrapped_lines = self._wrap_text_line(text, usable_width_px, draw, font)

        # c. 計算行高
        if para_data.line_spacing:
            line_height_px = para_data.line_spacing * 96 / 72
        else:
            line_height_px = font_size * 96 / 72  # 預設單倍行距

        # d. 累積總高度
        total_height_px += len(wrapped_lines) * line_height_px
        total_height_px += space_before + space_after

    # 3. 檢查溢出（超過 0.05 英寸才報告）
    if total_height_px > usable_height_px:
        overflow_inches = (total_height_px - usable_height_px) / 96.0
        if overflow_inches > 0.05:
            self.frame_overflow_bottom = overflow_inches
```

**考慮因素**：
- 字型名稱和大小
- 行距（line_spacing）
- 段落間距（space_before, space_after）
- 文字框邊距
- 文字換行

#### 3. 形狀重疊檢測

**演算法**：
```python
def calculate_overlap(rect1, rect2, tolerance=0.05):
    """計算兩矩形的重疊面積"""
    left1, top1, w1, h1 = rect1
    left2, top2, w2, h2 = rect2

    # 計算重疊尺寸
    overlap_width = min(left1 + w1, left2 + w2) - max(left1, left2)
    overlap_height = min(top1 + h1, top2 + h2) - max(top1, top2)

    # 檢查是否有意義的重疊（> 0.05 英寸）
    if overlap_width > tolerance and overlap_height > tolerance:
        overlap_area = overlap_width * overlap_height
        return True, round(overlap_area, 2)

    return False, 0
```

**輸出格式**：
```json
{
  "overlap": {
    "overlapping_shapes": {
      "shape-2": 0.45,  // 重疊面積（平方英寸）
      "shape-5": 0.12
    }
  }
}
```

#### 4. 視覺位置排序

**策略**：
```python
def sort_shapes_by_position(shapes):
    """上到下、左到右排序"""

    # 1. 按 top 位置排序
    shapes = sorted(shapes, key=lambda s: (s.top, s.left))

    # 2. 分組為行（垂直距離 < 0.5 英寸視為同一行）
    rows = []
    current_row = [shapes[0]]
    row_top = shapes[0].top

    for shape in shapes[1:]:
        if abs(shape.top - row_top) <= 0.5:
            current_row.append(shape)
        else:
            # 同一行內按 left 排序
            rows.extend(sorted(current_row, key=lambda s: s.left))
            current_row = [shape]
            row_top = shape.top

    # 3. 最後一行
    rows.extend(sorted(current_row, key=lambda s: s.left))
    return rows
```

**效果**：
- `shape-0`, `shape-1`, ... 按閱讀順序編號
- 符合人類視覺習慣
- 便於內容替換

### replace.py 深度分析

#### 1. 自動清除機制

**策略**：
```python
# 提取所有文字形狀的庫存
inventory = extract_text_inventory(pptx_file)

# 清除所有形狀
for shape_data in inventory:
    shape.text_frame.clear()

# 只替換 JSON 中定義的形狀
if "paragraphs" in replacement_shape_data:
    # 添加新段落
```

**優點**：
- 確保沒有遺留舊文字
- 明確定義哪些形狀有新內容
- 避免部分更新導致的不一致

#### 2. 項目符號處理

**問題**：PowerPoint 的項目符號需要特殊處理

**解決方案**：
```python
def apply_paragraph_properties(paragraph, para_data):
    if para_data.get("bullet", False):
        # 1. 設定層級
        paragraph.level = para_data.get("level", 0)

        # 2. 計算縮排（字型比例）
        font_size = para_data.get("font_size", 18.0)
        level_indent_emu = int((font_size * (1.6 + level * 1.6)) * 12700)
        hanging_indent_emu = int(-font_size * 0.8 * 12700)

        # 3. 添加項目符號字元
        buChar = OxmlElement("a:buChar")
        buChar.set("char", "•")
        pPr.append(buChar)

        # 4. 預設左對齊（項目符號不應置中）
        if "alignment" not in para_data:
            paragraph.alignment = PP_ALIGN.LEFT
```

**注意**：
- 不要在文字中包含 `•` 符號（會自動添加）
- 項目符號預設左對齊
- 縮排量與字體大小成比例

#### 3. 溢出驗證

**流程**：
```python
# 1. 記錄原始溢出狀況
original_overflow = detect_frame_overflow(inventory)

# 2. 替換文字
# ...

# 3. 檢查替換後的溢出
updated_overflow = detect_frame_overflow(updated_inventory)

# 4. 比較溢出是否惡化
for slide_key, shape_overflows in updated_overflow.items():
    for shape_key, new_overflow in shape_overflows.items():
        original = original_overflow.get(slide_key, {}).get(shape_key, 0.0)

        if new_overflow > original + 0.01:
            # 錯誤：溢出惡化
            raise ValueError(...)
```

**策略**：
- 允許原本就溢出的形狀保持溢出
- 禁止使溢出變得更嚴重
- 禁止原本沒溢出的形狀變成溢出

### rearrange.py 深度分析

#### 1. 投影片複製邏輯

**挑戰**：複製投影片需要處理圖片和媒體關係

**實作**：
```python
def duplicate_slide(pres, index):
    """完整複製投影片，包括圖片關係"""
    source = pres.slides[index]
    new_slide = pres.slides.add_slide(source.slide_layout)

    # 1. 收集所有圖片關係
    image_rels = {}
    for rel_id, rel in source.part.rels.items():
        if "image" in rel.reltype or "media" in rel.reltype:
            image_rels[rel_id] = rel

    # 2. 清除占位符（避免重複）
    for shape in new_slide.shapes:
        shape.element.getparent().remove(shape.element)

    # 3. 複製所有形狀
    for shape in source.shapes:
        new_el = deepcopy(shape.element)
        new_slide.shapes._spTree.insert_element_before(new_el, "p:extLst")

        # 4. 更新圖片參考
        blips = new_el.xpath(".//a:blip[@r:embed]")
        for blip in blips:
            old_rId = blip.get("{...}embed")
            if old_rId in image_rels:
                # 建立新關係
                new_rId = new_slide.part.rels.get_or_add(...)
                blip.set("{...}embed", new_rId)
```

**確保**：
- 完整複製形狀和屬性
- 正確處理圖片參考
- 避免占位符重複

#### 2. 三步驟處理

**為什麼需要三個步驟**：
```python
# 步驟 1：複製重複投影片
# 範例：序列 [0, 34, 34, 50, 52]
#       投影片 34 出現兩次，建立一個副本

# 步驟 2：刪除未使用投影片（從後往前）
# 保留的投影片：0, 34, 34的副本, 50, 52
# 刪除其他 68 張

# 步驟 3：重新排序
# 將保留的投影片排列為指定順序
```

**優點**：
- 清晰的三階段處理
- 避免索引錯誤（從後往前刪除）
- 可除錯（每步驟都有輸出）

---

## 設計策略

### 1. 三種簡報建立路徑

**路徑 A：無範本建立**
```
需求：從零建立簡報
工具：html2pptx
流程：
1. 分析內容和選擇設計
2. 建立 HTML 投影片
3. 使用 html2pptx.js 轉換
4. 視覺驗證
```

**路徑 B：範本重用**
```
需求：基於現有範本建立新簡報
工具：thumbnail + inventory + rearrange + replace
流程：
1. 分析範本（縮圖 + 庫存）
2. 選擇和重排投影片
3. 替換文字內容
4. 驗證溢出和重疊
```

**路徑 C：直接編輯**
```
需求：小幅修改現有簡報
工具：OOXML 編輯
流程：
1. 解壓 .pptx
2. 編輯 XML
3. 驗證
4. 重新打包
```

### 2. 視覺驗證優先

**驗證點**：
1. **範本分析**：縮圖網格快速預覽
2. **占位符檢查**：使用 --outline-placeholders
3. **內容驗證**：檢查溢出和重疊
4. **最終確認**：再次建立縮圖

**工具鏈**：
```bash
# 分析
python thumbnail.py template.pptx analysis --cols 4 --outline-placeholders

# 檢測問題
python inventory.py output.pptx issues.json --issues-only

# 驗證結果
python thumbnail.py output.pptx final
```

### 3. 防止 AI 風格

**SKILL.md 明確列出要避免的特徵**：
- ❌ 過度置中的版面
- ❌ 紫色漸層
- ❌ 統一的圓角
- ❌ Inter 字型（過度使用）

**設計原則**：
- ✅ 根據內容選擇色彩
- ✅ 考慮品牌識別
- ✅ 使用多樣化的版面配置
- ✅ 建立清晰的視覺層次

### 4. 批次處理支援

**範本映射**：
```python
template_mapping = [
    0,   # 標題投影片
    34,  # 內容佈局
    34,  # 再次使用相同佈局
    50,  # 引用佈局
    52,  # 結尾投影片
]
```

**自動化流程**：
```bash
# 1. 建立映射
# 2. 重排投影片
python rearrange.py template.pptx working.pptx 0,34,34,50,52

# 3. 提取庫存
python inventory.py working.pptx inventory.json

# 4. 準備替換內容
# 5. 替換
python replace.py working.pptx replacement.json output.pptx
```

---

## Claude Code 使用方式

### 觸發時機

Claude Code 會在以下情況使用此 Skill：

1. **建立全新簡報**
   - 範例：「建立一份產品發表簡報」
   - 範例：「製作季度報告簡報」

2. **基於範本建立簡報**
   - 範例：「使用這個範本建立新的月報」
   - 範例：「套用公司範本製作提案」

3. **編輯現有簡報**
   - 範例：「更新簡報中的數字」
   - 範例：「修改投影片標題」

4. **分析簡報結構**
   - 範例：「分析這份簡報的版面配置」
   - 範例：「檢查簡報是否有文字溢出」

5. **批次產生簡報**
   - 範例：「為每個分公司建立相同格式的報告」

### 典型工作流程

#### 工作流程 1：建立無範本簡報

```bash
# 步驟 1：讀取 html2pptx.md（完整檔案）
# Claude 了解設計原則和 HTML 語法

# 步驟 2：分析內容並選擇設計
# - 根據主題選擇色彩方案
# - 決定版面配置策略

# 步驟 3：建立 HTML 投影片檔案
# slide-1.html, slide-2.html, ...

# 步驟 4：建立轉換腳本
# create-presentation.js

# 步驟 5：執行轉換
node create-presentation.js

# 步驟 6：建立縮圖驗證
python thumbnail.py output.pptx
# 讀取 thumbnails.jpg，視覺檢查版面
```

#### 工作流程 2：基於範本建立簡報

```bash
# 步驟 1：分析範本
## a. 提取文字並建立縮圖
python -m markitdown template.pptx > template-content.md
python thumbnail.py template.pptx --outline-placeholders

## b. 讀取並建立庫存檔案
# Claude 讀取 template-content.md 和 thumbnails.jpg
# 建立 template-inventory.md，記錄每張投影片的用途

# 步驟 2：建立簡報大綱
# Claude 建立 outline.md，選擇要使用的範本投影片

# 步驟 3：重排投影片
python rearrange.py template.pptx working.pptx 0,34,34,50,52

# 步驟 4：提取文字庫存
python inventory.py working.pptx text-inventory.json
# Claude 讀取 text-inventory.json

# 步驟 5：準備替換內容
# Claude 建立 replacement-text.json

# 步驟 6：替換文字
python replace.py working.pptx replacement-text.json output.pptx

# 步驟 7：驗證
## a. 檢查問題
python inventory.py output.pptx issues.json --issues-only

## b. 建立最終縮圖
python thumbnail.py output.pptx final
```

#### 工作流程 3：檢測簡報問題

```bash
# 步驟 1：提取有問題的形狀
python inventory.py presentation.pptx issues.json --issues-only

# 步驟 2：Claude 讀取 issues.json
# 分析問題：
# - frame_overflow_bottom: 文字溢出
# - overlap: 形狀重疊
# - warnings: 格式問題

# 步驟 3：建議修正方案
```

### 不使用此 Skill 的情況
- Word 文件（使用 docx skill）
- PDF 文件（使用 pdf skill）
- Excel 試算表（使用 xlsx skill）

---

## 技術架構

### 技術堆疊

**Python 生態系**：
```
python-pptx     # PowerPoint 操作（讀寫、形狀、文字）
Pillow (PIL)    # 圖片處理（縮圖、驗證圖片）
markitdown      # PPTX 轉 Markdown
defusedxml      # 安全的 XML 解析
```

**JavaScript/Node.js 生態系**：
```
pptxgenjs       # PowerPoint 生成
playwright      # HTML 渲染
sharp           # SVG 光柵化和圖片處理
react-icons     # 圖示庫
```

**命令列工具**：
```
soffice         # LibreOffice（PPTX → PDF）
pdftoppm        # Poppler（PDF → 圖片）
```

### 依賴關係圖

```
簡報建立工作流程
├── 無範本路徑
│   ├── HTML 投影片建立
│   ├── html2pptx.js
│   │   ├── playwright（渲染）
│   │   ├── sharp（圖片處理）
│   │   └── pptxgenjs（PowerPoint 生成）
│   └── thumbnail.py（驗證）
├── 範本重用路徑
│   ├── thumbnail.py
│   │   ├── soffice（PPTX → PDF）
│   │   └── pdftoppm（PDF → 圖片）
│   ├── inventory.py
│   │   ├── python-pptx（讀取）
│   │   └── PIL（文字測量）
│   ├── rearrange.py
│   │   └── python-pptx（投影片操作）
│   └── replace.py
│       └── python-pptx（文字替換）
└── 直接編輯路徑
    └── OOXML 操作
        └── defusedxml
```

### OOXML 簡報結構

```
presentation.pptx（ZIP 檔案）
├── [Content_Types].xml
├── _rels/
│   └── .rels
├── ppt/
│   ├── presentation.xml          # 簡報元資料
│   ├── slides/
│   │   ├── slide1.xml            # 投影片內容
│   │   ├── slide2.xml
│   │   └── ...
│   ├── slideLayouts/             # 版面配置範本
│   ├── slideMasters/             # 母片範本
│   ├── theme/
│   │   └── theme1.xml            # 主題（色彩、字型）
│   ├── media/                    # 圖片和媒體
│   └── _rels/
│       └── slide1.xml.rels       # 投影片關係
└── docProps/
    ├── core.xml                  # 核心屬性
    └── app.xml                   # 應用程式屬性
```

---

## 使用的 Prompt 策略

### 1. 設計內容感知

**強制設計思考**：
```markdown
**CRITICAL**: Before creating any presentation, analyze the content and choose appropriate design elements:
1. **Consider the subject matter**: What is this presentation about?
2. **Check for branding**: If the user mentions a company/organization
3. **Match palette to content**: Select colors that reflect the subject
4. **State your approach**: Explain your design choices BEFORE writing code
```

**範例色彩方案**：
- 提供 18 種預設色彩方案（經典藍、珊瑚色、勃艮第奢華等）
- 鼓勵創造性選擇，避免自動駕駛

### 2. 版面配置指引

**內容結構匹配**：
```markdown
**CRITICAL: Match layout structure to actual content**:
- Single-column layouts: Use for unified narrative
- Two-column layouts: Use ONLY when you have exactly 2 distinct items
- Three-column layouts: Use ONLY when you have exactly 3 distinct items
- Count your actual content pieces BEFORE selecting the layout
```

**圖表/表格版面**：
```markdown
**When creating slides with charts or tables:**
- **Two-column layout (PREFERRED)**: Header spanning full width, then text in one column and chart in the other
- **Full-slide layout**: Let the chart take up the entire slide
- **NEVER vertically stack**: Don't place charts below text
```

### 3. 範本分析流程

**庫存檔案格式**：
```markdown
# Template Inventory Analysis
**Total Slides: [count]**
**IMPORTANT: Slides are 0-indexed (first slide = 0, last slide = count-1)**

## [Category Name]
- Slide 0: [Layout code] - Description/purpose
- Slide 1: [Layout code] - Description/purpose
[... EVERY slide must be listed individually ...]
```

### 4. 驗證強調

**多處強調驗證**：
```markdown
**Visual validation**: Generate thumbnails and inspect for layout issues
**IMPORTANT**: Check the thumbnail image for text cutoff, overlap, positioning issues
If issues found, adjust HTML margins/spacing/colors and regenerate
```

---

## 使用的工具（Tools）

此 Skill 主要依賴以下 Claude Code 工具：

### 1. **Read Tool**
- 讀取 SKILL.md、html2pptx.md、ooxml.md
- 讀取縮圖網格圖片（視覺分析）
- 讀取 inventory.json（了解形狀結構）
- 讀取驗證圖片（占位符標示）

### 2. **Write Tool**
- 建立 HTML 投影片檔案
- 建立 JavaScript 轉換腳本
- 建立 replacement-text.json
- 建立範本庫存檔案

### 3. **Bash Tool**
- 執行 thumbnail.py（建立縮圖）
- 執行 inventory.py（提取庫存）
- 執行 rearrange.py（重排投影片）
- 執行 replace.py（替換文字）
- 執行 Node.js 腳本（html2pptx）
- 執行 soffice 和 pdftoppm

### 4. **Edit Tool**
- 修正 replacement-text.json
- 調整 HTML 投影片
- 修改腳本參數

---

## 最佳實踐

### 1. 設計選擇

**✅ 建議做法**：
```
1. 分析內容主題
2. 考慮品牌識別（如提及公司）
3. 選擇符合主題的色彩
4. 解釋設計選擇
5. 建立多樣化的版面配置
```

**❌ 避免做法**：
```
自動使用紫色漸層
所有投影片置中對齊
統一使用圓角
不考慮內容就選擇顏色
```

### 2. 範本重用

**投影片選擇**：
```
✅ 選擇文字型版面配置
✅ 確認版面配置與內容數量匹配
✅ 驗證索引在範圍內（0-based）
✅ 建立庫存檔案供參考
```

**避免**：
```
❌ 使用比內容多的欄位版面
❌ 強制內容適應錯誤的版面
❌ 使用超出範圍的索引
❌ 跳過範本分析步驟
```

### 3. 文字替換

**replacement.json 建立**：
```json
{
  "slide-0": {
    "shape-0": {
      "paragraphs": [
        {
          "text": "標題文字",
          "alignment": "CENTER",
          "bold": true,
          "font_size": 44.0
        }
      ]
    },
    "shape-1": {
      "paragraphs": [
        {
          "text": "第一個要點",
          "bullet": true,
          "level": 0
        },
        {
          "text": "第二個要點",
          "bullet": true,
          "level": 0
        }
      ]
    }
  }
}
```

**注意事項**：
- ✅ 包含原始格式屬性（bold、alignment 等）
- ✅ 項目符號文字不包含 `•` 符號
- ✅ 項目符號時不設定 alignment（預設左對齊）
- ✅ 檢查 default_font_size 選擇合適字體大小
- ❌ 不要只提供純文字字串

### 4. 驗證流程

**完整驗證**：
```bash
# 1. 建立縮圖（快速視覺檢查）
python thumbnail.py output.pptx

# 2. 檢測問題（詳細分析）
python inventory.py output.pptx issues.json --issues-only

# 3. 讀取問題報告
# 如果有溢出或重疊，修正後重新執行
```

**問題類型**：
```json
{
  "overflow": {
    "frame": {"overflow_bottom": 0.25}  // 文字超出 0.25 英寸
  },
  "overlap": {
    "overlapping_shapes": {
      "shape-3": 0.45  // 與 shape-3 重疊 0.45 平方英寸
    }
  },
  "warnings": [
    "manual_bullet_symbol: use proper bullet formatting"
  ]
}
```

### 5. 批次處理

**模板化方法**：
```python
# 定義可重用的映射
MONTHLY_REPORT_TEMPLATE = [0, 34, 34, 50, 52]

# 建立多份報告
for month in ["Jan", "Feb", "Mar"]:
    # 1. 重排投影片
    rearrange(template, f"{month}-working.pptx", MONTHLY_REPORT_TEMPLATE)

    # 2. 準備該月份的資料
    data = load_monthly_data(month)

    # 3. 建立替換 JSON
    replacement = create_replacement_json(data)

    # 4. 替換
    replace(f"{month}-working.pptx", replacement, f"{month}-report.pptx")
```

---

## 總結

### 成功要素

1. **視覺驗證機制**：縮圖網格提供快速預覽和問題檢測
2. **問題自動檢測**：文字溢出、形狀重疊、格式警告
3. **範本重用支援**：完整的工作流程從分析到生成
4. **設計指引**：避免 AI 風格，鼓勵內容感知設計
5. **批次處理**：適合重複性簡報製作

### 學習重點

對於想要理解此 Skill 的開發者：
- OOXML 簡報結構
- 群組形狀的絕對位置計算
- 文字溢出估算（文字測量、換行、行距）
- 形狀重疊檢測演算法
- 投影片複製的圖片關係處理
- 視覺位置排序策略

### 適用場景

此 Skill 最適合用於：
- 📊 數據報告簡報（月報、季報、年報）
- 🎯 銷售提案和產品發表
- 📚 教育訓練教材
- 🏢 公司內部溝通簡報
- 🎨 基於範本的品牌簡報
- 📈 定期更新的儀表板簡報

### 技術亮點

- **群組形狀處理**：正確計算任意深度嵌套的形狀位置
- **文字溢出估算**：使用 PIL 精確測量文字高度
- **視覺驗證**：縮圖網格和占位符標示
- **批次友好**：模板化映射和自動化替換
- **問題導向**：`--issues-only` 標記只顯示需要關注的部分

---

## 參考資源

- [SKILL.md](./SKILL.md) - 英文原始文檔
- [html2pptx.md](./html2pptx.md) - html2pptx 工作流程指南
- [ooxml.md](./ooxml.md) - OOXML 操作詳細文檔
- [python-pptx 文檔](https://python-pptx.readthedocs.io/)
- [PptxGenJS 文檔](https://gitbrent.github.io/PptxGenJS/)

---

**最後更新**：2025-11-14
**文件版本**：1.0.0
**適用於**：Claude Code Skills
