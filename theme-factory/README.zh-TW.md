# Theme Factory - 正體中文說明文件

## 📋 目錄
- [Skill 概述](#skill-概述)
- [目錄結構](#目錄結構)
- [檔案說明](#檔案說明)
- [主題分析](#主題分析)
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
Theme Factory 是一個專業的主題設計工具套件,提供 10 個精心策劃的配色和字型組合,可以應用於各種 artifacts,包括簡報、文件、報告、HTML 頁面等。每個主題都包含協調的色彩調色盤和互補的字型配對,確保視覺一致性和專業性。

主要功能包括:
- 10 個預設專業主題
- 完整的色彩調色盤(含 hex codes)
- 互補的字型配對(標題和內文)
- 主題展示 PDF 供視覺選擇
- 支援自訂主題生成
- 適用於多種場景的主題選擇指引

### 解決的問題
在為簡報或文件設計視覺風格時,經常遇到以下挑戰:

1. **配色困難**:不知道如何選擇協調的顏色組合
2. **字型配對問題**:不確定哪些字型組合看起來專業
3. **缺乏一致性**:不同頁面使用不同的顏色和字型
4. **時間浪費**:每次都要重新思考設計風格
5. **業餘外觀**:隨意選擇的配色導致不專業的視覺效果
6. **缺乏場景適配性**:不知道什麼風格適合什麼場合

Theme Factory 提供精心設計的主題,讓應用專業視覺風格變得簡單快速。

---

## 目錄結構

```
theme-factory/
├── SKILL.md                    # Skill 主要說明文件(英文)
├── LICENSE.txt                 # 授權條款
├── README.zh-TW.md            # 正體中文說明文件(本文件)
├── theme-showcase.pdf          # 主題視覺展示檔案
└── themes/                     # 主題定義目錄(10個主題)
    ├── ocean-depths.md        # 海洋深度主題
    ├── sunset-boulevard.md    # 日落大道主題
    ├── forest-canopy.md       # 森林樹冠主題
    ├── modern-minimalist.md   # 現代極簡主題
    ├── golden-hour.md         # 黃金時刻主題
    ├── arctic-frost.md        # 北極冰霜主題
    ├── desert-rose.md         # 沙漠玫瑰主題
    ├── tech-innovation.md     # 科技創新主題
    ├── botanical-garden.md    # 植物園主題
    └── midnight-galaxy.md     # 午夜銀河主題
```

---

## 檔案說明

### 核心檔案

#### 1. `SKILL.md`(60 行)
- **用途**:主題應用指南
- **內容**:
  - 使用說明(4步驟流程)
  - 10個主題列表
  - 應用流程說明
  - 自訂主題指引
- **特點**:簡潔明瞭的步驟式指導

#### 2. `theme-showcase.pdf`
- **用途**:視覺化展示所有主題
- **內容**:每個主題的視覺範例
- **使用方式**:顯示給使用者選擇主題
- **重要性**:提供視覺參考,便於決策

#### 3. `README.zh-TW.md`(本文件)
- **用途**:為繁體中文使用者提供詳細說明
- **內容**:完整的主題分析和使用指南

### Themes 目錄

每個主題檔案包含:
- **主題名稱和描述**:說明主題的視覺特色和氛圍
- **色彩調色盤**:4種協調的顏色 + hex codes
- **字型配對**:標題和內文字型
- **最佳使用場景**:適合的應用場合

#### 主題結構範例(ocean-depths.md):
```markdown
# Ocean Depths

A professional and calming maritime theme...

## Color Palette
- **Deep Navy**: `#1a2332` - Primary background color
- **Teal**: `#2d8b8b` - Accent color
- **Seafoam**: `#a8dadc` - Secondary accent
- **Cream**: `#f1faee` - Text and light backgrounds

## Typography
- **Headers**: DejaVu Sans Bold
- **Body Text**: DejaVu Sans

## Best Used For
Corporate presentations, financial reports...
```

---

## 主題分析

### 10 個專業主題

#### 1. Ocean Depths(海洋深度)
**視覺特色**:專業且平靜的海洋主題

**色彩調色盤**:
- 深海軍藍 `#1a2332`
- 藍綠色 `#2d8b8b`
- 海沫色 `#a8dadc`
- 奶油色 `#f1faee`

**字型**:DejaVu Sans Bold / DejaVu Sans

**最佳使用場景**:
- 企業簡報
- 財務報告
- 專業諮詢文件
- 建立信任的內容

**設計理念**:海洋色調傳達穩定、深度和專業性

#### 2. Sunset Boulevard(日落大道)
**視覺特色**:溫暖且充滿活力的日落色彩

**適合場景**:
- 創意簡報
- 活動宣傳
- 品牌發布
- 熱情洋溢的內容

**設計理念**:暖色調營造能量和樂觀氛圍

#### 3. Forest Canopy(森林樹冠)
**視覺特色**:自然且扎根的大地色調

**適合場景**:
- 環保相關簡報
- 永續發展報告
- 自然產品介紹
- 健康醫療內容

**設計理念**:綠色和棕色傳達自然、成長和穩定

#### 4. Modern Minimalist(現代極簡)
**視覺特色**:乾淨且當代的灰階調色盤

**色彩調色盤**:
- 炭灰色 `#36454f`
- 板岩灰 `#708090`
- 淺灰色 `#d3d3d3`
- 白色 `#ffffff`

**字型**:DejaVu Sans Bold / DejaVu Sans

**最佳使用場景**:
- 科技簡報
- 建築作品集
- 設計展示
- 現代商業提案
- 數據視覺化

**設計理念**:灰階提供最大靈活性和現代感

#### 5. Golden Hour(黃金時刻)
**視覺特色**:豐富且溫暖的秋季調色盤

**適合場景**:
- 高級品牌簡報
- 奢侈品介紹
- 成就展示
- 溫馨慶祝內容

**設計理念**:金色調傳達價值、溫暖和成功

#### 6. Arctic Frost(北極冰霜)
**視覺特色**:清涼且清新的冬季靈感主題

**適合場景**:
- 清新產品發布
- 科學研究簡報
- 清潔技術介紹
- 極簡風格內容

**設計理念**:冷色調營造清晰、純淨和專注氛圍

#### 7. Desert Rose(沙漠玫瑰)
**視覺特色**:柔和且精緻的塵土色調

**適合場景**:
- 時尚簡報
- 美容產品介紹
- 精品品牌展示
- 優雅風格內容

**設計理念**:柔和粉色調傳達精緻和優雅

#### 8. Tech Innovation(科技創新)
**視覺特色**:大膽且現代的科技美學

**適合場景**:
- 科技產品發布
- 創新提案
- 新創公司簡報
- 未來趨勢分析

**設計理念**:鮮明對比和大膽配色展現創新精神

#### 9. Botanical Garden(植物園)
**視覺特色**:清新且有機的花園色彩

**適合場景**:
- 有機產品介紹
- 園藝相關內容
- 自然教育簡報
- 清新品牌展示

**設計理念**:綠色調傳達生命力和自然美

#### 10. Midnight Galaxy(午夜銀河)
**視覺特色**:戲劇性且宇宙感的深色調

**適合場景**:
- 高端品牌簡報
- 神秘產品介紹
- 藝術展示
- 戲劇性內容

**設計理念**:深色調營造深度、神秘和高級感

---

## 設計策略

### 1. 視覺優先選擇流程

**4步驟流程**:
```
步驟 1:顯示 theme-showcase.pdf
    ↓
步驟 2:詢問使用者偏好
    ↓
步驟 3:等待明確選擇
    ↓
步驟 4:應用選定主題
```

**設計理念**:
- 視覺勝於文字描述
- 使用者主導選擇
- 明確確認避免誤解

### 2. 精心策劃的主題集合

**選擇標準**:
- **多樣性**:涵蓋不同氛圍(專業、創意、自然、科技等)
- **實用性**:每個主題都有明確的使用場景
- **協調性**:每個主題的顏色和字型都經過專業配對
- **專業性**:適合商業和專業環境

### 3. 色彩系統設計

**4色調色盤**:
- **主要顏色**:背景或主導色
- **強調色**:突出重點元素
- **次要強調色**:輔助視覺層次
- **文字/亮色**:確保可讀性

**設計理念**:
- 4種顏色足夠建立視覺層次
- 不會過於複雜
- 易於一致應用

### 4. 字型配對策略

**標題 + 內文模式**:
- **標題**:DejaVu Sans Bold(粗體,引人注目)
- **內文**:DejaVu Sans(易讀,專業)

**選擇 DejaVu Sans 的原因**:
- 廣泛可用
- 高可讀性
- 專業外觀
- 跨平台兼容

### 5. 自訂主題支援

**靈活性設計**:
```markdown
## Create your Own Theme
To handle cases where none of the existing themes work...
generate a new theme similar to the ones above.
```

**設計理念**:
- 預設主題涵蓋大多數需求
- 特殊需求可生成自訂主題
- 保持相同的結構和品質標準

---

## Claude Code 使用方式

### 觸發時機

Claude Code 會在以下情況使用此 Skill:

1. **需要為 artifact 套用主題**
   - 範例:「為這個簡報套用專業主題」
   - 範例:「這個文件需要更好的配色」
   - 範例:「讓這個頁面看起來更專業」

2. **使用者要求選擇主題**
   - 範例:「顯示可用的主題」
   - 範例:「我想看看有什麼配色選項」

3. **建立新 artifact 時提供主題選項**
   - 範例:「建立一個簡報」→「要套用主題嗎?」

### 典型工作流程

#### 工作流程 1:為現有簡報套用主題

```
步驟 1:顯示主題展示
Claude:「我可以為您的簡報套用專業主題。讓我顯示可用的主題選項。」
[顯示 theme-showcase.pdf]

步驟 2:詢問偏好
Claude:「這裡有 10 個主題可供選擇:
1. Ocean Depths - 專業且平靜
2. Sunset Boulevard - 溫暖且充滿活力
3. Modern Minimalist - 乾淨且當代
...

您偏好哪一個主題?」

步驟 3:等待選擇
使用者:「我喜歡 Modern Minimalist」

步驟 4:應用主題
Claude:
1. 讀取 themes/modern-minimalist.md
2. 提取顏色和字型
3. 應用到簡報:
   - 背景色:#36454f 或 #ffffff
   - 標題:DejaVu Sans Bold,顏色:#36454f
   - 內文:DejaVu Sans,顏色:#708090
   - 強調元素:#d3d3d3
4. 確保一致性
5. 完成更新

Claude:「已套用 Modern Minimalist 主題!我使用了:
- 炭灰色作為主要顏色
- 板岩灰和淺灰色作為強調色
- DejaVu Sans Bold 作為標題字型
- DejaVu Sans 作為內文字型」
```

#### 工作流程 2:建立新簡報並套用主題

```
步驟 1:建立基本簡報
Claude:建立簡報基本結構

步驟 2:詢問是否需要主題
Claude:「簡報已建立!您想套用主題嗎?」

使用者:「是的」

步驟 3-4:與工作流程 1 相同
[顯示主題 → 選擇 → 應用]
```

#### 工作流程 3:自訂主題

```
使用者:「我需要一個適合醫療簡報的主題,但現有主題都不太合適」

Claude:
步驟 1:了解需求
「我來為您建立一個醫療主題。您希望傳達什麼氛圍?
例如:信任感、清潔感、專業感?」

使用者:「清潔、專業、帶點溫暖」

步驟 2:生成自訂主題
Claude 建立:
# Medical Care

A clean and professional theme with warm accents for healthcare.

## Color Palette
- **Medical Blue**: `#0077be` - Trust and professionalism
- **Warm Teal**: `#20b2aa` - Caring and healing
- **Soft Gray**: `#e8e8e8` - Clean and sterile
- **White**: `#ffffff` - Purity and clarity

## Typography
- **Headers**: DejaVu Sans Bold
- **Body Text**: DejaVu Sans

## Best Used For
Healthcare presentations, medical reports, hospital communications.

步驟 3:顯示供審核
Claude:「我建立了一個醫療主題,使用醫療藍和溫暖的藍綠色。
可以嗎?需要調整嗎?」

步驟 4:應用主題
[與標準流程相同]
```

### 不使用此 Skill 的情況

- 使用者明確要求特定的顏色/字型(非主題套件)
- 技術文檔(不需要視覺主題)
- 程式碼專案(非視覺設計)

---

## 技術架構

### 資源類型

```
純文檔型 Skill
├── theme-showcase.pdf(視覺資產)
└── themes/*.md(主題定義)
```

**無程式碼**:
- 不包含可執行腳本
- 純粹的定義和資產
- 依賴 Claude 的理解和應用能力

### 主題應用流程

```
使用者請求
    ↓
顯示 theme-showcase.pdf
    ↓
使用者選擇主題
    ↓
讀取 themes/{選定主題}.md
    ↓
提取顏色和字型
    ↓
應用到 artifact
    ├─ 更新背景色
    ├─ 更新文字顏色
    ├─ 更新標題字型
    ├─ 更新內文字型
    └─ 更新強調元素
    ↓
確保一致性
    ↓
完成
```

### 主題定義結構

```yaml
主題檔案(.md)
├── 標題(主題名稱)
├── 描述(氛圍和特色)
├── Color Palette
│   ├── 主要顏色(名稱 + hex + 用途)
│   ├── 強調色(名稱 + hex + 用途)
│   ├── 次要強調色(名稱 + hex + 用途)
│   └── 文字/亮色(名稱 + hex + 用途)
├── Typography
│   ├── Headers(字型名稱)
│   └── Body Text(字型名稱)
└── Best Used For(使用場景列表)
```

---

## 使用的 Prompt 策略

### 1. 視覺優先策略

**SKILL.md 強調**:
```markdown
1. **Show the theme showcase**: Display the `theme-showcase.pdf`
```

**設計理念**:
- 圖片勝於千言萬語
- 視覺化幫助決策
- 減少描述的歧義

### 2. 明確確認流程

**步驟式指引**:
```markdown
1. Show the theme showcase
2. Ask for their choice
3. Wait for selection
4. Apply the theme
```

**優點**:
- 防止誤解
- 使用者主導
- 清晰的流程

### 3. 場景導向描述

**每個主題包含使用場景**:
```markdown
## Best Used For
Corporate presentations, financial reports...
```

**優點**:
- 幫助使用者選擇合適主題
- 提供決策指引
- 減少不當應用

### 4. 結構化主題定義

**一致的格式**:
```markdown
# [主題名稱]
[描述]
## Color Palette
## Typography
## Best Used For
```

**優點**:
- 易於解析
- 一致的應用
- 清晰的資訊層次

---

## 使用的工具(Tools)

此 Skill 主要依賴以下 Claude Code 工具:

### 1. **Read Tool**
- 讀取 theme-showcase.pdf(顯示給使用者)
- 讀取 themes/*.md(提取主題定義)
- 讀取 SKILL.md(了解流程)

### 2. **Edit Tool**
- 更新 artifact 的顏色
- 更新 artifact 的字型
- 確保一致性

### 3. **Write Tool**(如需建立自訂主題)
- 建立新的主題定義檔案

---

## 最佳實踐

### 1. 主題選擇最佳實踐

**考慮因素**:
```
受眾:
- 企業/專業 → Ocean Depths, Modern Minimalist
- 創意/活潑 → Sunset Boulevard, Botanical Garden
- 科技/創新 → Tech Innovation, Arctic Frost
- 高端/奢華 → Golden Hour, Midnight Galaxy

內容類型:
- 財務報告 → Ocean Depths
- 產品發布 → Tech Innovation, Sunset Boulevard
- 環保簡報 → Forest Canopy, Botanical Garden
- 數據視覺化 → Modern Minimalist
```

### 2. 主題應用最佳實踐

**一致性原則**:
```
✅ 全面應用:
- 所有標題使用相同字型和顏色
- 所有內文使用相同字型和顏色
- 強調元素一致使用強調色
- 背景色一致

❌ 不一致應用:
- 部分頁面使用主題,部分不使用
- 隨意混用主題顏色
- 標題字型不統一
```

**對比度檢查**:
```
✅ 確保可讀性:
- 深色背景 + 淺色文字
- 淺色背景 + 深色文字
- 足夠的對比度

❌ 避免:
- 深色背景 + 深色文字
- 淺色背景 + 淺色文字
```

### 3. 自訂主題最佳實踐

**保持相同結構**:
```markdown
# [主題名稱]

[描述 - 傳達氛圍和適用場景]

## Color Palette
- **顏色名稱**: `#hexcode` - 用途說明
[4種顏色,涵蓋深淺和用途]

## Typography
- **Headers**: [字型名稱]
- **Body Text**: [字型名稱]

## Best Used For
[具體的使用場景列表]
```

**顏色選擇原則**:
```
1. 選擇協調的色相
2. 包含深淺對比
3. 至少一個中性色(白/灰/黑)
4. 明確每個顏色的用途
```

---

## 總結

### 成功要素

1. **精心策劃的主題**:10個專業主題涵蓋多種場景
2. **視覺化展示**:PDF 展示幫助快速決策
3. **結構化定義**:一致的主題格式易於應用
4. **靈活性**:支援自訂主題生成
5. **使用場景指引**:幫助選擇合適主題

### 學習重點

對於想要理解此 Skill 的使用者:
- 配色原則(4色調色盤)
- 字型配對策略
- 視覺一致性的重要性
- 場景適配的考量
- 如何建立自訂主題

### 適用場景

此 Skill 最適合用於:
- 簡報設計
- 文件美化
- 報告格式化
- HTML 頁面樣式
- 品牌展示
- 專業溝通

### 技術亮點

- **10個專業主題**:涵蓋商業、創意、科技等場景
- **視覺優先**:PDF 展示減少描述歧義
- **結構化定義**:易於解析和應用
- **DejaVu Sans 字型**:廣泛可用且專業
- **4色調色盤**:簡單而強大的配色系統

---

## 參考資源

- [SKILL.md](./SKILL.md) - 英文原始文檔
- [theme-showcase.pdf](./theme-showcase.pdf) - 主題視覺展示
- [themes/](./themes/) - 主題定義目錄

---

**最後更新**:2025-11-15
**文件版本**:1.0.0
**適用於**:Claude Code Skills
