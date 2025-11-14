# Awesome Claude Skills - 正體中文使用指南

## 專案來源說明

本專案 Fork 自：[ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)

本繁體中文文件為學習用途，旨在協助正體中文使用者更好地理解和使用 Claude Skills。

---

## 目錄

1. [專案概述](#專案概述)
2. [什麼是 Claude Skills](#什麼是-claude-skills)
3. [專案目錄結構](#專案目錄結構)
4. [Skill 分類與清單](#skill-分類與清單)
5. [安裝與使用方式](#安裝與使用方式)
6. [Skill 架構詳解](#skill-架構詳解)
7. [如何建立自訂 Skill](#如何建立自訂-skill)
8. [漸進式揭露原則](#漸進式揭露原則)
9. [貢獻指南](#貢獻指南)
10. [授權資訊](#授權資訊)
11. [常見問題](#常見問題)

---

## 專案概述

**Awesome Claude Skills** 是一個精選的 Claude 技能市集，包含 27 個實用的 Skills，旨在提升使用者在 Claude.ai、Claude Code 和 Claude API 上的生產力。本專案同時也是一個框架和範本，讓開發者可以建立並分享自己的 Skills。

### 專案統計資訊

- **總計 Skills 數量**：27 個（24 個活躍 Skills + 3 個元 Skills）
- **授權方式**：Apache 2.0（專案本身）；個別 Skills 可能有專屬授權
- **維護團隊**：ComposioHQ (tech@composio.dev)
- **專案性質**：開源社群驅動

---

## 什麼是 Claude Skills

Claude Skills 是一種模組化的方式，用於擴展 Claude AI 助理的能力。每個 Skill 包含特定領域的知識、工作流程和工具，能夠：

- **自動化複雜任務**：將多步驟流程整合為單一指令
- **專業領域知識**：提供特定領域的最佳實踐和指引
- **工具整合**：連接外部服務和 API
- **可重用性**：一次建立，多處使用
- **跨平台相容**：在 Claude.ai、Claude Code 和 API 上均可使用

### Skills 的運作方式

Skills 採用「漸進式揭露」（Progressive Disclosure）設計原則：

1. **第一層 - 元資料**（約 100 字）：始終載入至上下文
   - 從 YAML frontmatter 提取的 `name` 和 `description`
   - Claude 用於判斷何時啟動該 Skill

2. **第二層 - SKILL.md 主體**（少於 5000 字）：觸發時載入
   - 完整的指示和工作流程
   - 對附帶資源的參照

3. **第三層 - 附帶資源**：按需載入
   - 腳本可在不載入至上下文的情況下執行
   - 參考文件僅在 Claude 判斷需要時載入
   - 素材檔案直接用於輸出

---

## 專案目錄結構

```
awesome-claude-skills/
│
├── .claude-plugin/
│   └── marketplace.json              # 市集配置檔，定義所有 Skills 和分類
│
├── .git/                              # Git 版本控制目錄
│
├── README.md                          # 主要文件（英文，277 行）
├── README.zh-TW.md                    # 正體中文使用指南（本文件）
├── CONTRIBUTING.md                    # 貢獻指南（185 行）
│
├── [24 個 Skill 目錄]/                # 個別 Skill 實作
│   ├── SKILL.md                      # 必需：Skill 定義檔，包含 YAML frontmatter
│   ├── LICENSE.txt                   # 可選：Skill 專屬授權
│   ├── scripts/                      # 可選：可執行的輔助腳本
│   │   ├── init_skill.py
│   │   ├── package_skill.py
│   │   └── ...
│   ├── references/                   # 可選：按需載入的文件資料
│   │   ├── api_documentation.md
│   │   ├── best_practices.md
│   │   └── ...
│   ├── assets/                       # 可選：範本、字型、圖片等
│   │   ├── templates/
│   │   ├── images/
│   │   └── ...
│   └── examples/                     # 可選：範例檔案
│       ├── example_usage.md
│       └── ...
│
├── document-skills/                   # 特殊：文件處理 Skills 集合（4 個）
│   ├── docx/                         # Word 文件處理
│   ├── pdf/                          # PDF 文件處理
│   ├── pptx/                         # PowerPoint 簡報處理
│   └── xlsx/                         # Excel 試算表處理
│
├── skill-creator/                     # 元 Skill：用於建立新 Skills
│   ├── SKILL.md
│   └── scripts/
│       ├── init_skill.py             # 初始化新 Skill
│       ├── package_skill.py          # 打包和驗證 Skill
│       └── quick_validate.py         # 快速驗證
│
├── skill-share/                       # 元 Skill：透過 Slack 分享 Skills
│   └── SKILL.md
│
└── template-skill/                    # 最小範本：建立新 Skills 的起點
    └── SKILL.md
```

### 目錄結構說明

#### .claude-plugin/ 目錄
包含市集配置檔 `marketplace.json`，定義所有可用的 Skills、分類和元資料。此檔案控制 Skills 在市集中的顯示方式。

#### 個別 Skill 目錄
每個 Skill 都有自己的目錄，遵循一致的結構：

- **SKILL.md**（必需）：主要 Skill 定義檔
- **scripts/**（可選）：Python、Bash 或 JavaScript 腳本
- **references/**（可選）：API 文件、指南等參考資料
- **assets/**（可選）：範本、圖片、字型等靜態檔案
- **examples/**（可選）：使用範例和示範

#### document-skills/ 目錄
特殊的文件處理 Skills 集合，每個子目錄對應一種文件格式，提供深度的 OOXML 整合能力。

#### 元 Skills
- **skill-creator**：協助使用者建立新的 Skills
- **skill-share**：協助使用者透過 Slack 分享 Skills
- **template-skill**：提供最基本的 Skill 範本

---

## Skill 分類與清單

Skills 依據功能分為五大類別：

### 一、商業與行銷（Business & Marketing）

共 5 個 Skills，專注於品牌、廣告和業務發展。

#### 1. brand-guidelines
- **功能**：Anthropic 品牌色彩和字體規範
- **適用情境**：需要遵循 Anthropic 品牌識別系統時
- **複雜度**：簡單（73 行）

#### 2. competitive-ads-extractor
- **功能**：廣告庫分析工具
- **適用情境**：研究競爭對手的廣告策略
- **複雜度**：中等

#### 3. domain-name-brainstormer
- **功能**：網域名稱腦力激盪與可用性檢查
- **適用情境**：為新專案尋找合適的網域名稱
- **複雜度**：中等

#### 4. internal-comms
- **功能**：內部溝通範本
- **適用情境**：撰寫公司內部公告或溝通文件
- **複雜度**：中等

#### 5. lead-research-assistant
- **功能**：潛在客戶識別與資格審查
- **適用情境**：進行業務開發和潛在客戶研究
- **複雜度**：中等

### 二、溝通與寫作（Communication & Writing）

共 2 個 Skills，專注於內容創作和溝通分析。

#### 6. content-research-writer
- **功能**：基於研究的內容創作
- **適用情境**：撰寫經過充分研究的文章或部落格
- **複雜度**：複雜（538 行）
- **特色**：包含完整的研究和寫作工作流程

#### 7. meeting-insights-analyzer
- **功能**：會議記錄行為分析
- **適用情境**：分析會議記錄，提取關鍵洞察和行動項目
- **複雜度**：中等

### 三、創意與媒體（Creative & Media）

共 6 個 Skills，專注於視覺和多媒體內容創作。

#### 8. canvas-design
- **功能**：視覺藝術創作（PNG/PDF 格式）
- **適用情境**：建立圖形設計和視覺內容
- **複雜度**：中等

#### 9. image-enhancer
- **功能**：圖片品質提升
- **適用情境**：改善圖片解析度、清晰度或色彩
- **複雜度**：中等

#### 10. slack-gif-creator
- **功能**：為 Slack 建立動畫 GIF
- **適用情境**：建立自訂的 Slack 表情符號或動畫
- **複雜度**：複雜（646 行）
- **特色**：包含完整的動畫生成流程

#### 11. theme-factory
- **功能**：為 artifacts 建立專業主題
- **適用情境**：需要為網頁或應用程式套用視覺主題
- **複雜度**：中等
- **特色**：預設 10 種專業主題

#### 12. video-downloader
- **功能**：從平台下載影片
- **適用情境**：下載 YouTube 或其他平台的影片
- **複雜度**：中等

#### 13. algorithmic-art
- **功能**：生成式藝術創作
- **適用情境**：建立演算法驅動的藝術作品
- **複雜度**：中等

### 四、開發（Development）

共 7 個 Skills，專注於軟體開發工具和工作流程。

#### 14. artifacts-builder
- **功能**：React + Tailwind + shadcn/ui artifacts 建立工具
- **適用情境**：快速建立互動式網頁應用程式原型
- **複雜度**：中等
- **特色**：
  - `init-artifact.sh`：建立 React + TypeScript + Vite 專案
  - 預安裝 40+ shadcn/ui 元件
  - `bundle-artifact.sh`：打包成單一 HTML 檔案
  - 明確警告避免「AI 陳腔濫調」美學

#### 15. changelog-generator
- **功能**：Git commit 轉換為更新日誌
- **適用情境**：自動生成版本更新說明
- **複雜度**：中等

#### 16. developer-growth-analysis
- **功能**：聊天歷史分析，提供成長洞察
- **適用情境**：分析個人編碼模式，找出改進機會
- **複雜度**：複雜（322 行）
- **特色**：
  - 讀取 Claude Code 聊天歷史（~/.claude/history.jsonl）
  - 分析編碼模式和知識缺口
  - 從 HackerNews 精選學習資源
  - 透過 Slack DM 發送個人化報告

#### 17. mcp-builder
- **功能**：MCP 伺服器建立指南
- **適用情境**：建立 Model Context Protocol 伺服器
- **複雜度**：中等至複雜（328 行）
- **特色**：
  - 四階段工作流程：研究 → 實作 → 審查 → 評估
  - 語言特定指南（Python/TypeScript）
  - 以代理為中心的設計原則
  - 包含 10 個問題的評估框架

#### 18. skill-creator
- **功能**：Skill 建立框架
- **適用情境**：建立新的 Claude Skills
- **複雜度**：中等（209 行）
- **特色**：定義完整的 6 步驟建立流程

#### 19. webapp-testing
- **功能**：基於 Playwright 的測試
- **適用情境**：自動化網頁應用程式測試
- **複雜度**：中等（95 行）

#### 20. template-skill
- **功能**：基本 Skill 範本
- **適用情境**：作為建立新 Skill 的起點
- **複雜度**：非常簡單（7 行）

### 五、生產力與組織（Productivity & Organization）

共 7 個 Skills，專注於檔案管理和文件處理。

#### 21. file-organizer
- **功能**：智能檔案組織
- **適用情境**：自動整理混亂的檔案目錄
- **複雜度**：中等

#### 22. invoice-organizer
- **功能**：發票/收據組織
- **適用情境**：管理和分類財務文件
- **複雜度**：中等

#### 23. raffle-winner-picker
- **功能**：密碼學安全的隨機抽選
- **適用情境**：公平地選擇抽獎或比賽獲勝者
- **複雜度**：中等（159 行）

#### 24. document-skills-docx
- **功能**：Word 文件操作
- **適用情境**：建立、編輯、轉換 .docx 檔案
- **複雜度**：複雜
- **特色**：
  - 追蹤修訂和註解
  - 校訂工作流程
  - 深度 OOXML 整合

#### 25. document-skills-pdf
- **功能**：PDF 處理
- **適用情境**：PDF 文字提取、表單填寫、合併/分割
- **複雜度**：複雜
- **特色**：完整的 PDF 操作工具集

#### 26. document-skills-pptx
- **功能**：PowerPoint 建立/編輯
- **適用情境**：自動生成或修改簡報
- **複雜度**：複雜
- **特色**：
  - 投影片生成
  - 主題套用
  - OOXML 架構支援

#### 27. document-skills-xlsx
- **功能**：Excel 試算表操作
- **適用情境**：資料處理、公式、圖表建立
- **複雜度**：複雜
- **特色**：
  - 公式支援
  - 圖表生成
  - 資料轉換

---

## 安裝與使用方式

### 方式一：在 Claude.ai 上使用

1. 在聊天介面中點擊 Skills 圖示
2. 從市集中瀏覽和新增 Skills
3. 或上傳自訂 Skills
4. Skills 會根據對話內容自動啟動

**優點**：
- 圖形化介面，易於使用
- 無需設定
- 立即可用

### 方式二：在 Claude Code 中使用

#### 步驟 1：建立 Skills 目錄

```bash
mkdir -p ~/.config/claude-code/skills/
```

#### 步驟 2：複製 Skills

```bash
# 複製單一 Skill
cp -r /path/to/awesome-claude-skills/skill-name ~/.config/claude-code/skills/

# 或複製所有 Skills
cp -r /path/to/awesome-claude-skills/* ~/.config/claude-code/skills/
```

#### 步驟 3：啟動 Claude Code

```bash
claude
```

Skills 會自動載入並可在對話中使用。

**優點**：
- 與開發環境深度整合
- 可存取本地檔案系統
- 適合開發工作流程

### 方式三：透過 Claude API 使用

```python
import anthropic

client = anthropic.Anthropic(api_key="your-api-key")

response = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    skills=["skill-id-here"],
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "您的提示詞"}
    ]
)

print(response.content)
```

**優點**：
- 程式化整合
- 適合自動化工作流程
- 可嵌入應用程式

---

## Skill 架構詳解

### SKILL.md 檔案結構

每個 Skill 的核心是 `SKILL.md` 檔案，採用以下結構：

```markdown
---
name: skill-name
description: Claude 應該在何時使用此 Skill 以及其功能
license: 可選的授權資訊
---

# Skill 名稱

## 何時使用此 Skill

[說明在什麼情境下應該使用此 Skill]

## 工作流程

[詳細的步驟說明]

1. 第一步
2. 第二步
3. ...

## 範例

### 範例 1：[描述]

**使用者提示**：
```
[使用者的實際提示]
```

**預期輸出**：
```
[Skill 的輸出]
```

## 參考資源

[列出 references/ 目錄中的可用文件]

## 腳本

[列出 scripts/ 目錄中的可用腳本及其用途]
```

### YAML Frontmatter 欄位

- **name**（必需）：Skill 的唯一識別名稱，使用小寫和連字號
- **description**（必需）：簡短描述（約 100 字），說明何時使用此 Skill
- **license**（可選）：授權資訊，如與專案主授權不同

### 目錄結構範例

以 `mcp-builder` Skill 為例：

```
mcp-builder/
├── SKILL.md                           # 主 Skill 檔案（328 行）
├── references/
│   ├── mcp_overview.md               # MCP 概述
│   ├── python_mcp_server.md          # Python 伺服器指南
│   ├── typescript_mcp_server.md      # TypeScript 伺服器指南
│   ├── best_practices.md             # 最佳實踐
│   └── evaluation_harness.md         # 評估框架
├── scripts/
│   ├── init_mcp_server.py            # 初始化 MCP 伺服器
│   └── validate_server.py            # 驗證伺服器
└── examples/
    ├── simple_server.py              # 簡單伺服器範例
    └── complex_server.ts             # 複雜伺服器範例
```

---

## 如何建立自訂 Skill

### 使用 skill-creator Skill（推薦）

#### 步驟 1：啟動 skill-creator

在 Claude Code 或 Claude.ai 中，告訴 Claude：

```
我想建立一個新的 Skill，協助我 [描述您的需求]
```

skill-creator 會引導您完成以下六個步驟：

#### 步驟 2：理解（Understanding）

- 收集具體的使用範例
- 說明 Skill 應該解決的問題
- 定義預期的輸入和輸出

#### 步驟 3：規劃（Planning）

- 識別可重用的資源
- 決定需要哪些腳本、參考文件或素材
- 設計工作流程

#### 步驟 4：初始化（Initialization）

執行初始化腳本：

```bash
cd ~/.config/claude-code/skills/skill-creator
python scripts/init_skill.py my-new-skill
```

這會建立基本的目錄結構。

#### 步驟 5：編輯（Editing）

- 撰寫 `SKILL.md`
- 建立必要的腳本（在 `scripts/` 中）
- 新增參考文件（在 `references/` 中）
- 準備素材檔案（在 `assets/` 中）

#### 步驟 6：打包（Packaging）

執行打包和驗證腳本：

```bash
python scripts/package_skill.py
```

這會驗證您的 Skill 結構並準備分發。

#### 步驟 7：迭代（Iteration）

- 在真實場景中測試 Skill
- 根據使用經驗改進
- 收集回饋並精煉

### 手動建立 Skill

如果您偏好手動建立：

#### 1. 建立目錄結構

```bash
mkdir -p ~/.config/claude-code/skills/my-skill/{scripts,references,assets,examples}
```

#### 2. 建立 SKILL.md

```bash
touch ~/.config/claude-code/skills/my-skill/SKILL.md
```

#### 3. 填寫基本內容

```markdown
---
name: my-skill
description: 簡短描述此 Skill 的用途和適用情境
---

# My Skill

## 何時使用此 Skill

[說明]

## 工作流程

[步驟]

## 範例

[具體範例]
```

#### 4. 新增可選元件

根據需要新增腳本、參考文件、素材或範例。

---

## 漸進式揭露原則

漸進式揭露（Progressive Disclosure）是 Claude Skills 的核心設計原則，旨在優化上下文使用和效能。

### 三層載入機制

#### 第一層：輕量元資料（始終載入）

- **內容**：YAML frontmatter 中的 `name` 和 `description`
- **大小**：約 100 字
- **目的**：讓 Claude 判斷何時啟動此 Skill
- **影響**：對上下文影響極小

#### 第二層：主要指示（觸發時載入）

- **內容**：SKILL.md 的主體內容
- **大小**：少於 5000 字
- **目的**：提供完整的工作流程和指示
- **觸發條件**：Claude 判斷需要使用此 Skill 時

#### 第三層：附帶資源（按需載入）

- **內容**：scripts/、references/、assets/、examples/
- **載入策略**：
  - **腳本**：可執行而不載入至上下文
  - **參考文件**：僅在 Claude 判斷需要時載入
  - **素材檔案**：直接用於輸出，不載入至上下文
  - **範例**：根據需要參考

### 為什麼使用漸進式揭露

#### 優點

1. **上下文效率**：避免不必要的資訊佔用上下文視窗
2. **更快的回應**：減少需要處理的資訊量
3. **可擴展性**：可以包含大量資源而不影響效能
4. **模組化**：每個元件都有明確的用途
5. **靈活性**：根據實際需求載入資源

#### 實踐建議

- **保持 SKILL.md 簡潔**：核心指示應清晰且精簡
- **善用 references/**：將詳細的 API 文件或規範放在此處
- **腳本優先**：對於確定性任務，使用腳本而非指示
- **範例要具體**：提供真實世界的使用範例

---

## 貢獻指南

### 貢獻新 Skill 的要求

#### 1. 基於真實使用案例

- Skill 必須解決真實存在的問題
- 不接受純假設性或示範性的 Skills
- 應該來自實際使用經驗

#### 2. 良好的文件

- 清晰的 SKILL.md 檔案
- 包含具體的使用範例
- 說明何時應該使用此 Skill
- 提供預期的輸入和輸出範例

#### 3. 易於使用

- 盡可能對非技術使用者友善
- 提供清晰的錯誤訊息
- 在執行破壞性操作前請求確認

#### 4. 跨平台測試

- 在 Claude.ai 上測試
- 在 Claude Code 上測試
- 在 Claude API 上測試（如適用）

#### 5. 安全性

- 在執行破壞性操作前確認
- 不應包含硬編碼的機密資訊
- 遵循安全最佳實踐

#### 6. 可攜性

- 應該能在不同環境中運作
- 避免依賴特定的系統配置
- 清楚說明任何外部依賴

### 提交 Pull Request 的流程

#### 步驟 1：Fork 專案

在 GitHub 上 fork [ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills) 專案。

#### 步驟 2：建立分支

```bash
git checkout -b add-my-skill
```

分支命名建議：`add-[skill-name]`

#### 步驟 3：新增 Skill

```bash
# 建立 Skill 目錄
mkdir my-skill

# 新增 SKILL.md 和其他檔案
# ...
```

#### 步驟 4：更新 README.md

在適當的分類下新增您的 Skill：

```markdown
### My Skill
**描述**：簡短描述
**適用情境**：何時使用
```

#### 步驟 5：更新 marketplace.json

在 `.claude-plugin/marketplace.json` 中新增您的 Skill：

```json
{
  "name": "my-skill",
  "description": "簡短描述",
  "source": "./my-skill",
  "category": "適當的分類"
}
```

#### 步驟 6：提交變更

```bash
git add .
git commit -m "Add [My Skill] skill

- 簡短描述功能
- 說明解決的問題
"
```

#### 步驟 7：推送並建立 PR

```bash
git push origin add-my-skill
```

然後在 GitHub 上建立 Pull Request。

### Pull Request 檢查清單

在提交 PR 前，確認：

- [ ] SKILL.md 包含有效的 YAML frontmatter
- [ ] description 清楚說明何時使用此 Skill
- [ ] 包含至少一個真實世界的使用範例
- [ ] 所有腳本都有執行權限（如適用）
- [ ] 在至少兩個平台上測試過（Claude.ai、Claude Code 或 API）
- [ ] README.md 已更新
- [ ] marketplace.json 已更新
- [ ] 沒有包含機密資訊
- [ ] 遵循專案的程式碼風格
- [ ] 包含適當的授權資訊（如與主專案不同）

---

## 授權資訊

### 專案授權

**Awesome Claude Skills** 專案本身採用 **Apache License 2.0** 授權。

這意味著您可以：

- 商業使用
- 修改
- 分發
- 專利使用
- 私人使用

條件是：

- 包含授權和版權聲明
- 說明變更
- 包含 NOTICE 檔案（如有）
- 提供 Apache 2.0 授權文字

### 個別 Skill 授權

每個 Skill 可能有自己的授權：

- 檢查每個 Skill 目錄中的 `LICENSE.txt` 檔案
- 檢查 `SKILL.md` 的 YAML frontmatter 中的 `license` 欄位
- 如未指定，則遵循專案的 Apache 2.0 授權

### 使用注意事項

- 在商業使用前，請檢查特定 Skill 的授權
- 某些 Skill 可能有專屬授權
- 遵守每個 Skill 的授權條款

---

## 常見問題

### Q1：什麼是 Claude Skills？

**A**：Claude Skills 是一種模組化的方式，用於擴展 Claude AI 的能力。每個 Skill 包含特定領域的知識、工作流程和工具，可以在 Claude.ai、Claude Code 和 Claude API 上使用。

### Q2：如何知道應該使用哪個 Skill？

**A**：每個 Skill 的 `description` 欄位會說明其適用情境。您也可以：
- 瀏覽本文件的「Skill 分類與清單」章節
- 閱讀每個 Skill 的 SKILL.md 檔案
- 在 Claude.ai 市集中搜尋
- 直接詢問 Claude 推薦適合的 Skill

### Q3：我可以修改現有的 Skill 嗎？

**A**：可以。Skills 是開源的（除非特定 Skill 有不同授權）。您可以：
- Fork 專案並修改
- 建立自己的版本
- 提交 Pull Request 改進原始 Skill

### Q4：如何在 Claude Code 中載入 Skills？

**A**：將 Skills 複製到 `~/.config/claude-code/skills/` 目錄，然後啟動 Claude Code。Skills 會自動載入。

### Q5：Skills 會佔用多少上下文視窗？

**A**：採用漸進式揭露設計：
- 第一層（元資料）：約 100 字，始終載入
- 第二層（主要指示）：少於 5000 字，觸發時載入
- 第三層（附帶資源）：按需載入

這樣設計可以最小化上下文使用。

### Q6：我可以在 Claude API 中使用 Skills 嗎？

**A**：可以。使用 `skills` 參數指定要使用的 Skills：

```python
response = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    skills=["skill-id"],
    messages=[...]
)
```

### Q7：如何建立自己的 Skill？

**A**：最簡單的方式是使用 `skill-creator` Skill，它會引導您完成整個流程。或者參考本文件的「如何建立自訂 Skill」章節。

### Q8：Skills 可以執行系統命令嗎？

**A**：可以。Skills 中的腳本（在 `scripts/` 目錄中）可以執行系統命令。但出於安全考量，重要操作前應該請求使用者確認。

### Q9：如何分享我建立的 Skill？

**A**：您可以：
- 提交 Pull Request 到主專案
- 在您的 GitHub 上發布
- 使用 `skill-share` Skill 透過 Slack 分享
- 在 Claude 社群論壇分享

### Q10：Skills 支援哪些程式語言？

**A**：
- **SKILL.md**：使用 Markdown 格式
- **腳本**：Python、Bash、JavaScript/TypeScript 最常見
- **參考文件**：通常是 Markdown
- 理論上可以使用任何可執行的語言

### Q11：如何除錯 Skill？

**A**：
- 檢查 SKILL.md 的 YAML frontmatter 是否正確
- 使用 `skill-creator` 中的 `quick_validate.py` 驗證
- 在 Claude Code 中測試並查看輸出
- 檢查腳本的執行權限
- 查看錯誤訊息並逐步除錯

### Q12：Skills 可以存取網路嗎？

**A**：可以。許多 Skills 整合外部服務，例如：
- Slack（透過 Rube MCP）
- Google Sheets
- HackerNews
- YouTube
- 廣告庫
- 網域註冊商

### Q13：如何更新 Skill？

**A**：
- 如果從市集安裝：檢查市集是否有更新版本
- 如果手動安裝：從 GitHub 拉取最新版本並覆蓋
- 如果是自己的 Skill：編輯檔案後重新載入 Claude Code

### Q14：Skills 之間可以互相調用嗎？

**A**：可以。Skills 可以參考和建構於其他 Skills 之上。這就是為什麼有 `skill-creator` 和 `skill-share` 等元 Skills。

### Q15：我可以為企業內部建立私有 Skills 嗎？

**A**：可以。您可以：
- 建立內部 Skills 儲存庫
- 不發布到公開市集
- 在企業內部分發
- 根據企業需求客製化

---

## 外部資源

### 官方資源

- **Skills 概述**：[anthropic.com/news/skills](https://anthropic.com/news/skills)
- **Skills 使用指南**：[support.claude.com](https://support.claude.com)
- **Skills API 文件**：[docs.claude.com/en/api/skills-guide](https://docs.claude.com/en/api/skills-guide)
- **Claude 社群**：[community.anthropic.com](https://community.anthropic.com)
- **Skills 市集**：[claude.ai/marketplace](https://claude.ai/marketplace)

### 相關專案

- **官方 Anthropic Skills 儲存庫**：[github.com/anthropics/skills](https://github.com/anthropics/skills)
- **Notion Skills**：[notion.so/notiondevs](https://notion.so/notiondevs)
- **ComposioHQ 原始專案**：[github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)

### 學習資源

- **Lenny's Newsletter**：50 種使用 Claude Code 的方式
- **Claude Code 文件**：完整的 Claude Code 使用指南
- **MCP 文件**：Model Context Protocol 規範

---

## 總結

**Awesome Claude Skills** 是一個成熟、結構良好的專案，提供：

### 專案優勢

1. **豐富的 Skills 集合**：27 個涵蓋各領域的實用 Skills
2. **完善的文件**：清晰的貢獻指南和使用說明
3. **強大的框架**：skill-creator 提供完整的建立流程
4. **真實世界應用**：所有 Skills 都源自實際使用案例
5. **高效的設計**：漸進式揭露最小化上下文使用
6. **跨平台相容**：支援 Claude.ai、Claude Code 和 API

### 適用對象

- **個人開發者**：提升個人生產力和開發效率
- **企業團隊**：標準化工作流程和溝通
- **內容創作者**：加速內容創作過程
- **軟體工程師**：自動化開發任務

### 如何開始

1. **探索**：瀏覽 README.md 和本文件，找到相關的 Skills
2. **安裝**：將 Skills 安裝到 `~/.config/claude-code/skills/`
3. **客製化**：根據需求修改現有 Skills
4. **建立**：使用 skill-creator 建立新的 Skills
5. **貢獻**：透過 Pull Request 回饋社群

### 下一步行動

- 選擇一個與您需求相關的 Skill 開始使用
- 在實際工作流程中測試 Skills
- 根據經驗提供回饋
- 考慮建立和分享您自己的 Skills

---

## 聯絡資訊

### 專案維護

- **維護團隊**：ComposioHQ
- **Email**：tech@composio.dev
- **GitHub**：[github.com/ComposioHQ/awesome-claude-skills](https://github.com/ComposioHQ/awesome-claude-skills)

### 回報問題

如果您發現 bug 或有功能請求：

1. 檢查現有的 [Issues](https://github.com/ComposioHQ/awesome-claude-skills/issues)
2. 如果沒有相關 issue，建立新的 issue
3. 提供詳細的重現步驟和環境資訊

### 參與討論

- **Claude 社群論壇**：分享經驗和最佳實踐
- **GitHub Discussions**：技術討論和問題
- **Pull Requests**：貢獻程式碼和改進

---

## 版本資訊

- **專案版本**：1.0.0
- **文件版本**：1.0.0（正體中文）
- **最後更新**：2025-11-14
- **文件語言**：正體中文（繁體中文）
- **翻譯目的**：學習用途

---

**感謝您使用 Awesome Claude Skills！**

本文件旨在協助正體中文使用者更好地理解和使用 Claude Skills。如有任何問題或建議，歡迎透過 GitHub Issues 回饋。
