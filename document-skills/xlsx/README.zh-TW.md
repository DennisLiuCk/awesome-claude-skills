# XLSX - Excel 試算表處理正體中文說明文件

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
XLSX 是一個全方位的 Microsoft Excel 試算表處理工具套件，提供試算表的建立、編輯、分析和公式計算功能。它整合了 Python 庫和 LibreOffice，使 Claude 能夠：
- 建立包含公式和格式的新試算表
- 讀取和分析試算表資料
- 編輯現有試算表並保留公式
- 自動重新計算所有公式
- 檢測和報告公式錯誤
- 處理財務模型（符合業界標準）
- 執行資料分析和視覺化

### 解決的問題
在處理 Excel 試算表時，經常遇到以下挑戰：

1. **公式不計算**：Python 建立的試算表中公式只是字串，沒有計算值
2. **公式錯誤難查**：手動檢查數百個儲存格的 #REF!、#DIV/0! 等錯誤
3. **格式與資料分離**：需要同時處理資料操作和格式設定
4. **財務模型標準**：財務模型需要遵循特定的顏色編碼和格式規範
5. **大規模公式驗證**：需要確保所有公式都正確無誤
6. **硬編碼 vs 公式**：容易誤將計算值硬編碼，失去動態性

這個 Skill 透過提供自動化的公式重計算、錯誤檢測和財務模型規範解決了這些問題。

---

## 目錄結構

```
xlsx/
├── SKILL.md                      # Skill 主要說明文件（英文）
├── LICENSE.txt                   # 授權條款
├── README.zh-TW.md              # 正體中文說明文件（本文件）
└── recalc.py                     # 公式重計算和錯誤檢測腳本
```

---

## 檔案說明

### 核心檔案

#### 1. `SKILL.md`
- **用途**：Skill 的主要說明文件，提供完整的使用指南
- **內容**：
  - Excel 檔案處理需求（零公式錯誤、格式標準）
  - 財務模型的顏色編碼和格式規範
  - 公式建構規則和錯誤預防
  - 使用 pandas 和 openpyxl 的工作流程
  - 公式驗證檢查清單
- **特點**：強調使用公式而非硬編碼值

#### 2. `recalc.py`（178 行）
- **用途**：使用 LibreOffice 重新計算試算表中的所有公式
- **核心功能**：
  - 自動設定 LibreOffice 巨集
  - 執行公式重計算
  - 掃描所有儲存格檢測錯誤
  - 回報錯誤位置和統計
- **重要性**：Python 建立試算表後的必要步驟

---

## 關鍵腳本分析

### recalc.py 深度分析

#### 1. LibreOffice 巨集自動設定

**問題**：LibreOffice 需要巨集才能自動化重計算

**解決方案**：
```python
def setup_libreoffice_macro():
    """首次執行時自動設定巨集"""

    # 1. 決定巨集目錄（跨平台）
    if platform.system() == 'Darwin':  # macOS
        macro_dir = '~/Library/Application Support/LibreOffice/4/user/basic/Standard'
    else:  # Linux
        macro_dir = '~/.config/libreoffice/4/user/basic/Standard'

    # 2. 檢查是否已存在
    macro_file = os.path.join(macro_dir, 'Module1.xba')
    if os.path.exists(macro_file):
        with open(macro_file, 'r') as f:
            if 'RecalculateAndSave' in f.read():
                return True  # 已設定

    # 3. 建立目錄（如果不存在）
    if not os.path.exists(macro_dir):
        subprocess.run(['soffice', '--headless', '--terminate_after_init'])
        os.makedirs(macro_dir, exist_ok=True)

    # 4. 寫入巨集
    macro_content = '''<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE script:module PUBLIC "-//OpenOffice.org//DTD OfficeDocument 1.0//EN" "module.dtd">
<script:module xmlns:script="http://openoffice.org/2000/script" script:name="Module1" script:language="StarBasic">
    Sub RecalculateAndSave()
      ThisComponent.calculateAll()
      ThisComponent.store()
      ThisComponent.close(True)
    End Sub
</script:module>'''

    with open(macro_file, 'w') as f:
        f.write(macro_content)
```

**優點**：
- 完全自動化，使用者不需手動設定
- 跨平台支援（macOS、Linux）
- 只設定一次，後續使用無需重複
- 失敗時有明確錯誤訊息

#### 2. 公式重計算流程

**執行邏輯**：
```python
def recalc(filename, timeout=30):
    """重計算並檢測錯誤"""

    # 1. 驗證檔案存在
    if not Path(filename).exists():
        return {'error': f'File {filename} does not exist'}

    # 2. 確保巨集已設定
    if not setup_libreoffice_macro():
        return {'error': 'Failed to setup LibreOffice macro'}

    # 3. 建立 LibreOffice 命令
    cmd = [
        'soffice', '--headless', '--norestore',
        'vnd.sun.star.script:Standard.Module1.RecalculateAndSave?language=Basic&location=application',
        abs_path
    ]

    # 4. 處理 timeout（跨平台）
    if platform.system() == 'Linux':
        cmd = ['timeout', str(timeout)] + cmd
    elif platform.system() == 'Darwin':
        # macOS 使用 gtimeout（如果有安裝）
        try:
            subprocess.run(['gtimeout', '--version'], ...)
            cmd = ['gtimeout', str(timeout)] + cmd
        except:
            pass  # 不使用 timeout

    # 5. 執行重計算
    result = subprocess.run(cmd, capture_output=True, text=True)

    if result.returncode != 0 and result.returncode != 124:
        return {'error': ...}

    # 6. 檢測錯誤（見下節）
    return check_errors(filename)
```

**重要細節**：
- 使用絕對路徑（LibreOffice 要求）
- timeout 機制防止卡住
- 錯誤碼 124 是 timeout，不視為失敗

#### 3. 全面的錯誤檢測

**掃描策略**：
```python
# 載入試算表（data_only=True 讀取計算值）
wb = load_workbook(filename, data_only=True)

# 定義所有 Excel 錯誤類型
excel_errors = ['#VALUE!', '#DIV/0!', '#REF!', '#NAME?', '#NULL!', '#NUM!', '#N/A']
error_details = {err: [] for err in excel_errors}

# 掃描所有工作表的所有儲存格
for sheet_name in wb.sheetnames:
    ws = wb[sheet_name]
    for row in ws.iter_rows():  # 不限制範圍
        for cell in row:
            if cell.value and isinstance(cell.value, str):
                for err in excel_errors:
                    if err in cell.value:
                        location = f"{sheet_name}!{cell.coordinate}"
                        error_details[err].append(location)
                        break
```

**關鍵設計**：
- `iter_rows()`：掃描所有儲存格，無範圍限制
- 字串檢查：錯誤值是字串（如 "#REF!"）
- 座標記錄：完整位置（工作表!儲存格）
- 分類統計：按錯誤類型分組

#### 4. 錯誤報告格式

**JSON 輸出**：
```python
# 建立結果摘要
result = {
    'status': 'success' if total_errors == 0 else 'errors_found',
    'total_errors': total_errors,
    'total_formulas': formula_count,
    'error_summary': {}
}

# 添加非空錯誤類別
for err_type, locations in error_details.items():
    if locations:
        result['error_summary'][err_type] = {
            'count': len(locations),
            'locations': locations[:20]  # 最多顯示 20 個
        }
```

**輸出範例**：
```json
{
  "status": "errors_found",
  "total_errors": 3,
  "total_formulas": 156,
  "error_summary": {
    "#REF!": {
      "count": 2,
      "locations": ["Sheet1!B5", "Sheet1!C10"]
    },
    "#DIV/0!": {
      "count": 1,
      "locations": ["Sheet2!D15"]
    }
  }
}
```

**成功範例**：
```json
{
  "status": "success",
  "total_errors": 0,
  "total_formulas": 156
}
```

#### 5. 公式計數

**統計公式數量**：
```python
# 重新載入（data_only=False 保留公式）
wb_formulas = load_workbook(filename, data_only=False)

formula_count = 0
for sheet_name in wb_formulas.sheetnames:
    ws = wb_formulas[sheet_name]
    for row in ws.iter_rows():
        for cell in row:
            # 公式以 '=' 開頭
            if cell.value and isinstance(cell.value, str) and cell.value.startswith('='):
                formula_count += 1

wb_formulas.close()
```

**提供脈絡**：
- 公式總數：了解試算表複雜度
- 錯誤比例：3 個錯誤 / 156 個公式 ≈ 2%
- 驗證指標：檢查是否所有公式都已處理

---

## 設計策略

### 1. 公式優先原則

**強制使用公式**：

**❌ 錯誤做法**（硬編碼值）：
```python
# Python 計算並硬編碼
total = df['Sales'].sum()
sheet['B10'] = total  # 寫入 5000

# 問題：如果 Sales 資料變更，B10 不會更新
```

**✅ 正確做法**（使用公式）：
```python
# 讓 Excel 計算
sheet['B10'] = '=SUM(B2:B9)'

# 優點：Sales 資料變更時，總和自動更新
```

**適用所有計算**：
- 總和、平均值、最大/最小值
- 百分比、比率、差異
- 查表（VLOOKUP、INDEX/MATCH）
- 條件計算（IF、SUMIF、COUNTIF）

### 2. 零錯誤容忍

**嚴格要求**：
```markdown
**Zero Formula Errors**
- Every Excel model MUST be delivered with ZERO formula errors
- All #REF!, #DIV/0!, #VALUE!, #N/A, #NAME? must be fixed
```

**驗證流程**：
```bash
# 1. 建立試算表
python create_model.py

# 2. 重計算
python recalc.py model.xlsx

# 3. 檢查回報
# 如果 status == "errors_found"，修正錯誤並重新執行
```

### 3. 財務模型規範

**顏色編碼標準**（業界慣例）：

| 用途 | 顏色 | RGB | 說明 |
|------|------|-----|------|
| 硬編碼輸入 | 藍色 | (0, 0, 255) | 使用者會變更的數字 |
| 公式計算 | 黑色 | (0, 0, 0) | 所有公式和計算 |
| 工作表內連結 | 綠色 | (0, 128, 0) | 從同一檔案其他工作表提取 |
| 外部連結 | 紅色 | (255, 0, 0) | 連結到其他檔案 |
| 重要假設 | 黃色背景 | (255, 255, 0) | 需要注意的儲存格 |

**數字格式標準**：

```python
# 年份：格式化為文字
sheet['A1'].number_format = '@'  # "2024" 不是 "2,024"

# 貨幣：千位分隔，單位在標題
sheet['B2'].number_format = '$#,##0'  # 標題："Revenue ($mm)"

# 零值：顯示為連字號
sheet['C3'].number_format = '$#,##0;($#,##0);-'

# 百分比：一位小數
sheet['D4'].number_format = '0.0%'

# 倍數：valuation 倍數格式
sheet['E5'].number_format = '0.0x'

# 負數：括號表示
# $#,##0;($#,##0);-  → 正數:1,234  負數:(1,234)  零值:-
```

**假設值文檔**：
```python
# 硬編碼值需要註解來源
sheet['B10'] = 1250000  # 營收數字
sheet['B10'].comment = "Source: Company 10-K, FY2024, Page 45, Revenue Note, [SEC EDGAR URL]"
```

### 4. 公式錯誤預防

**檢查清單**：

**參考驗證**：
```python
# ✅ 測試 2-3 個樣本參考
# 確認公式提取正確的值

# ✅ 確認欄位映射
# Excel 欄位 64 = BL，不是 BK

# ✅ 記住 Excel 行號從 1 開始
# DataFrame row 5 = Excel row 6
```

**常見陷阱**：
```python
# ❌ NaN 值
# 使用 pd.notna() 檢查空值

# ❌ 除以零
# 檢查分母後再使用除法

# ❌ 錯誤參考
# 驗證所有儲存格參考指向正確位置

# ❌ 跨工作表參考
# 使用正確格式：Sheet1!A1
```

**測試策略**：
```python
# ✅ 從小開始
# 先測試 2-3 個儲存格的公式

# ✅ 驗證相依性
# 檢查公式參考的所有儲存格是否存在

# ✅ 測試邊界情況
# 包含零、負數、極大值
```

---

## Claude Code 使用方式

### 觸發時機

Claude Code 會在以下情況使用此 Skill：

1. **建立包含公式的試算表**
   - 範例：「建立財務模型」
   - 範例：「製作銷售分析表」

2. **讀取和分析 Excel 資料**
   - 範例：「讀取這個試算表的資料」
   - 範例：「分析銷售數據趨勢」

3. **編輯現有試算表**
   - 範例：「更新試算表中的公式」
   - 範例：「添加新的計算欄位」

4. **驗證公式正確性**
   - 範例：「檢查試算表是否有公式錯誤」
   - 範例：「驗證所有計算都正確」

5. **批次處理 Excel 檔案**
   - 範例：「處理所有月報表格」
   - 範例：「合併多個 Excel 檔案」

### 典型工作流程

#### 工作流程 1：建立包含公式的試算表

```python
# 步驟 1：使用 openpyxl 建立試算表
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill

wb = Workbook()
sheet = wb.active

# 步驟 2：添加標題和資料
sheet['A1'] = 'Month'
sheet['B1'] = 'Revenue ($mm)'
sheet['C1'] = 'Growth %'

# 假設資料（藍色）
sheet['B2'] = 100
sheet['B2'].font = Font(color='0000FF')

# 步驟 3：添加公式（黑色，預設）
sheet['B3'] = '=B2*(1+C2)'  # 下個月營收
sheet['C3'] = '=(B3-B2)/B2'  # 成長率

# 步驟 4：格式設定
sheet['B2'].number_format = '$#,##0'
sheet['C3'].number_format = '0.0%'

# 步驟 5：儲存
wb.save('revenue_model.xlsx')
```

```bash
# 步驟 6：重計算公式（必要！）
python recalc.py revenue_model.xlsx

# 步驟 7：檢查輸出
# 如果 status == "success"，完成
# 如果 status == "errors_found"，修正錯誤並重試
```

#### 工作流程 2：讀取和分析資料

```python
import pandas as pd

# 使用 pandas 讀取
df = pd.read_excel('sales_data.xlsx')

# 分析
print(df.head())
print(df.describe())

# 視覺化
import matplotlib.pyplot as plt
df.plot(x='Month', y='Sales')
plt.savefig('sales_trend.png')
```

#### 工作流程 3：編輯保留公式

```python
from openpyxl import load_workbook

# 載入現有檔案
wb = load_workbook('existing_model.xlsx')
sheet = wb.active

# 修改值（保留公式）
sheet['B2'] = 150  # 更新假設值

# 添加新公式
sheet['D10'] = '=SUM(D2:D9)'

# 儲存
wb.save('updated_model.xlsx')
```

```bash
# 重計算
python recalc.py updated_model.xlsx
```

#### 工作流程 4：驗證公式

**場景：檢查大型模型**
```bash
# 執行驗證
python recalc.py financial_model.xlsx > validation.json

# 讀取結果
cat validation.json
```

**解讀結果**：
```json
{
  "status": "errors_found",
  "total_errors": 5,
  "total_formulas": 342,
  "error_summary": {
    "#REF!": {
      "count": 3,
      "locations": ["Summary!B15", "Summary!C15", "Detail!F23"]
    },
    "#DIV/0!": {
      "count": 2,
      "locations": ["Ratios!D5", "Ratios!D10"]
    }
  }
}
```

**修正步驟**：
```python
# 1. 修正 #REF! 錯誤（無效參考）
wb = load_workbook('financial_model.xlsx')
sheet = wb['Summary']

# 檢查 B15 的公式
print(sheet['B15'].value)  # 例如：=Sheet2!A1（但 Sheet2 不存在）

# 修正
sheet['B15'] = '=Detail!A1'  # 正確的工作表名稱

# 2. 修正 #DIV/0! 錯誤
sheet = wb['Ratios']
sheet['D5'] = '=IF(D3=0, "-", D2/D3)'  # 添加零值檢查

# 3. 儲存並重新驗證
wb.save('financial_model.xlsx')
```

```bash
python recalc.py financial_model.xlsx
# 預期：status == "success"
```

### 不使用此 Skill 的情況
- Word 文件（使用 docx skill）
- PDF 文件（使用 pdf skill）
- PowerPoint 簡報（使用 pptx skill）
- CSV 檔案的簡單讀寫（可以直接使用 pandas，不需重計算）

---

## 技術架構

### 技術堆疊

**Python 生態系**：
```
pandas          # 資料分析和操作
openpyxl        # Excel 讀寫、公式、格式
```

**外部工具**：
```
LibreOffice     # 公式重計算引擎
```

**可選工具**：
```
matplotlib      # 資料視覺化
seaborn         # 進階視覺化
xlsxwriter      # 替代 Excel 寫入（更多格式選項）
```

### 依賴關係圖

```
Excel 試算表工作流程
├── 建立/編輯
│   ├── openpyxl
│   │   ├── Workbook（建立）
│   │   ├── load_workbook（編輯）
│   │   ├── Font, PatternFill（格式）
│   │   └── Alignment, Border（格式）
│   └── pandas
│       ├── read_excel（讀取）
│       ├── DataFrame（操作）
│       └── to_excel（寫入）
├── 公式重計算
│   └── recalc.py
│       ├── LibreOffice Calc（引擎）
│       └── openpyxl（錯誤檢測）
└── 驗證
    └── openpyxl.load_workbook
        ├── data_only=True（計算值）
        └── data_only=False（公式）
```

### Excel 檔案結構

```
workbook.xlsx（ZIP 檔案）
├── [Content_Types].xml
├── _rels/
│   └── .rels
├── xl/
│   ├── workbook.xml             # 工作簿定義
│   ├── worksheets/
│   │   ├── sheet1.xml           # 工作表 1
│   │   ├── sheet2.xml           # 工作表 2
│   │   └── ...
│   ├── sharedStrings.xml        # 共享字串
│   ├── styles.xml               # 樣式定義
│   ├── calcChain.xml            # 計算鏈
│   └── _rels/
│       └── workbook.xml.rels
└── docProps/
    ├── core.xml                 # 核心屬性
    └── app.xml                  # 應用程式屬性
```

---

## 使用的 Prompt 策略

### 1. 強制公式使用

**SKILL.md 強調**：
```markdown
## CRITICAL: Use Formulas, Not Hardcoded Values

**Always use Excel formulas instead of calculating values in Python and hardcoding them.**
This ensures the spreadsheet remains dynamic and updateable.
```

**錯誤與正確範例**：
- 提供並列的 ❌ 和 ✅ 程式碼範例
- 清楚說明硬編碼的問題
- 強調公式的優點

### 2. 零錯誤要求

**顯著標示**：
```markdown
## All Excel files

### Zero Formula Errors
- Every Excel model MUST be delivered with ZERO formula errors (#REF!, #DIV/0!, #VALUE!, #N/A, #NAME?)
```

**驗證流程**：
```markdown
5. **Recalculate formulas (MANDATORY IF USING FORMULAS)**: Use the recalc.py script
   ```bash
   python recalc.py output.xlsx
   ```
6. **Verify and fix any errors**:
   - The script returns JSON with error details
   - If `status` is `errors_found`, check `error_summary` for specific error types and locations
```

### 3. 財務模型規範

**詳細表格**：
- 顏色編碼標準（藍色輸入、黑色公式等）
- 數字格式規則（年份為文字、貨幣千位分隔等）
- 公式建構規則（假設值放在獨立儲存格）

**文檔要求**：
```markdown
#### Documentation Requirements for Hardcodes
- Comment or in cells beside (if end of table). Format: "Source: [System/Document], [Date], [Specific Reference], [URL if applicable]"
```

### 4. 驗證檢查清單

**結構化指引**：
```markdown
## Formula Verification Checklist

Quick checks to ensure formulas work correctly:

### Essential Verification
- [ ] **Test 2-3 sample references**: Verify they pull correct values
- [ ] **Column mapping**: Confirm Excel columns match
- [ ] **Row offset**: Remember Excel rows are 1-indexed

### Common Pitfalls
- [ ] **NaN handling**: Check for null values
- [ ] **Division by zero**: Check denominators
- [ ] **Wrong references**: Verify all cell references
```

---

## 使用的工具（Tools）

此 Skill 主要依賴以下 Claude Code 工具：

### 1. **Write Tool**
- 建立 Python 腳本（試算表建立和編輯）
- 建立資料檔案（CSV、JSON）

### 2. **Bash Tool**
- 執行 recalc.py（公式重計算）
- 執行 Python 腳本
- 安裝依賴（pip install）

### 3. **Read Tool**
- 讀取 SKILL.md
- 讀取 recalc.py 輸出（validation.json）
- 檢查腳本內容

### 4. **Edit Tool**
- 修正 Python 腳本
- 調整公式定義

---

## 最佳實踐

### 1. 公式 vs 硬編碼

**永遠使用公式**：
```python
# ✅ 公式（推薦）
sheet['B10'] = '=SUM(B2:B9)'
sheet['C5'] = '=(B5-B4)/B4'
sheet['D20'] = '=AVERAGE(D2:D19)'

# ❌ 硬編碼（避免）
sheet['B10'] = sum(values)  # 值不會更新
sheet['C5'] = 0.15          # 失去計算邏輯
sheet['D20'] = 42.5         # 無法追蹤來源
```

### 2. 公式錯誤預防

**檢查除數**：
```python
# ❌ 可能除以零
sheet['E5'] = '=D5/C5'

# ✅ 安全處理
sheet['E5'] = '=IF(C5=0, "-", D5/C5)'
```

**驗證參考**：
```python
# ❌ 可能無效參考
sheet['F10'] = '=SUM(F2:F9)'  # 如果只有 F2:F8 有資料

# ✅ 先驗證範圍
if last_data_row == 8:
    sheet['F10'] = '=SUM(F2:F8)'
```

**測試公式**：
```python
# ✅ 先測試幾個
sheet['B2'] = '=A2*2'
sheet['B3'] = '=A3*2'

# 儲存並重計算
wb.save('test.xlsx')

# 驗證
import subprocess
result = subprocess.run(['python', 'recalc.py', 'test.xlsx'], capture_output=True)
# 檢查是否成功

# 如果成功，批次應用
for row in range(2, 100):
    sheet[f'B{row}'] = f'=A{row}*2'
```

### 3. 財務模型格式

**顏色編碼**：
```python
from openpyxl.styles import Font

# 藍色：輸入
sheet['B2'] = 1000
sheet['B2'].font = Font(color='0000FF')

# 黑色：公式（預設，無需設定）
sheet['B3'] = '=B2*1.1'

# 綠色：工作表內連結
sheet['B4'] = '=Assumptions!B10'
sheet['B4'].font = Font(color='008000')

# 紅色：外部連結
sheet['B5'] = "='[other_file.xlsx]Sheet1'!A1"
sheet['B5'].font = Font(color='FF0000')

# 黃色背景：重要假設
from openpyxl.styles import PatternFill
sheet['B6'] = 0.05
sheet['B6'].fill = PatternFill(start_color='FFFF00', fill_type='solid')
```

**數字格式**：
```python
# 年份：文字格式
sheet['A1'] = '2024'
sheet['A1'].number_format = '@'

# 貨幣：千位分隔，零值為連字號
sheet['B2'].number_format = '$#,##0;($#,##0);-'

# 百分比：一位小數
sheet['C3'].number_format = '0.0%'

# 倍數
sheet['D4'].number_format = '0.0x'
```

### 4. 重計算工作流程

**完整流程**：
```bash
# 1. 建立試算表
python create_model.py

# 2. 重計算（必要）
python recalc.py model.xlsx > result.json

# 3. 檢查結果
cat result.json

# 4. 如果有錯誤
# 4a. 讀取錯誤位置
# 4b. 修正公式
# 4c. 重新執行步驟 2

# 5. 直到 status == "success"
```

**除錯技巧**：
```bash
# 指定較長的 timeout（預設 30 秒）
python recalc.py large_model.xlsx 60

# 檢查特定工作表
python -c "
from openpyxl import load_workbook
wb = load_workbook('model.xlsx', data_only=False)
sheet = wb['Summary']
print(sheet['B15'].value)  # 查看公式
"
```

### 5. 大型試算表優化

**讀取優化**：
```python
# ✅ 只讀取需要的欄位
df = pd.read_excel('large_file.xlsx', usecols=['A', 'C', 'E'])

# ✅ 指定資料類型
df = pd.read_excel('file.xlsx', dtype={'id': str, 'amount': float})

# ✅ 只讀取特定工作表
df = pd.read_excel('file.xlsx', sheet_name='Summary')
```

**寫入優化**：
```python
from openpyxl import Workbook

# ✅ 使用 write_only 模式（大量資料）
wb = Workbook(write_only=True)
ws = wb.create_sheet()

for row in data:
    ws.append(row)

wb.save('large_output.xlsx')
```

### 6. 公式驗證策略

**漸進式驗證**：
```python
# 1. 建立小範圍測試
for row in range(2, 5):  # 只測試 3 行
    sheet[f'B{row}'] = f'=A{row}*2'

wb.save('test.xlsx')

# 2. 驗證
python recalc.py test.xlsx

# 3. 如果成功，擴大範圍
for row in range(2, 1000):
    sheet[f'B{row}'] = f'=A{row}*2'
```

**自動化測試**：
```python
import subprocess
import json

def verify_formulas(filename):
    """驗證公式無誤"""
    result = subprocess.run(
        ['python', 'recalc.py', filename],
        capture_output=True,
        text=True
    )

    data = json.loads(result.stdout)

    if data['status'] == 'errors_found':
        print(f"Found {data['total_errors']} errors:")
        for err_type, info in data['error_summary'].items():
            print(f"  {err_type}: {info['count']} errors")
            for loc in info['locations'][:5]:
                print(f"    - {loc}")
        return False

    print(f"✓ All {data['total_formulas']} formulas verified successfully")
    return True

# 使用
if not verify_formulas('model.xlsx'):
    print("Please fix errors and try again")
```

---

## 總結

### 成功要素

1. **公式優先**：強制使用 Excel 公式而非硬編碼，確保動態性
2. **自動重計算**：使用 LibreOffice 自動計算所有公式值
3. **全面錯誤檢測**：掃描所有儲存格，報告所有錯誤類型和位置
4. **財務標準**：提供業界認可的顏色編碼和格式規範
5. **零容忍驗證**：要求零公式錯誤，確保試算表品質

### 學習重點

對於想要理解此 Skill 的開發者：
- openpyxl 的公式處理（字串 vs 計算值）
- LibreOffice 巨集自動化
- 跨平台腳本設計（macOS、Linux）
- Excel 錯誤類型和檢測
- 財務模型的業界標準
- data_only 參數的使用

### 適用場景

此 Skill 最適合用於：
- 💼 財務模型（DCF、預算、預測）
- 📊 資料分析和報表
- 📈 業務儀表板
- 🔢 科學計算試算表
- 📋 庫存和資產管理
- 💰 會計和稅務計算

### 技術亮點

- **自動化巨集設定**：首次執行自動配置 LibreOffice
- **全面掃描**：不限範圍檢查所有儲存格
- **詳細報告**：JSON 格式提供結構化錯誤資訊
- **跨平台支援**：處理 macOS/Linux 的差異
- **公式計數**：提供脈絡了解試算表複雜度

---

## 參考資源

- [SKILL.md](./SKILL.md) - 英文原始文檔
- [recalc.py](./recalc.py) - 公式重計算腳本
- [openpyxl 文檔](https://openpyxl.readthedocs.io/)
- [pandas 文檔](https://pandas.pydata.org/docs/)
- [Excel 公式參考](https://support.microsoft.com/excel)

---

**最後更新**：2025-11-14
**文件版本**：1.0.0
**適用於**：Claude Code Skills
