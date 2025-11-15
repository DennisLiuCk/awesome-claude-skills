# Artifacts Builder - 正體中文說明文件

## 📋 目錄
- [Skill 概述](#skill-概述)
- [目錄結構](#目錄結構)
- [檔案說明](#檔案說明)
- [關鍵腳本分析](#關鍵腳本分析)
- [設計策略](#設計策略)
- [Claude Code 使用方式](#claude-code-使用方式)
- [技術架構](#技術架構)

---

## Skill 概述

### 這個 Skill 的功能
Artifacts Builder 是一個專門用於建立複雜、多組件 Claude.ai HTML artifacts 的工具套件。它使用現代前端網頁技術（React、Tailwind CSS、shadcn/ui）來建立功能豐富的互動式網頁應用程式，並將其打包成單一 HTML 檔案，可直接在 Claude 對話中作為 artifact 分享。

### 解決的問題
在使用 Claude 建立網頁應用程式時，經常遇到以下挑戰：

1. **重複性設定工作**：每次建立新專案都需要重新配置 React、TypeScript、Tailwind CSS 等工具
2. **組件管理困難**：手動安裝和配置 UI 組件庫（如 shadcn/ui）耗時且容易出錯
3. **打包複雜度**：將多個檔案的 React 應用程式打包成單一 HTML 檔案需要複雜的配置
4. **環境相容性**：不同 Node.js 版本可能導致相容性問題
5. **設計一致性**：避免產生「AI 風格」的設計（過度置中、紫色漸層、統一圓角等）

Artifacts Builder 透過自動化腳本和預配置模板解決了這些問題，讓開發者可以專注於應用程式邏輯和使用者體驗。

---

## 目錄結構

```
artifacts-builder/
├── SKILL.md                      # Skill 主要說明文件（英文）
├── LICENSE.txt                   # 授權條款
├── README.zh-TW.md              # 正體中文說明文件（本文件）
└── scripts/                      # 可執行腳本目錄
    ├── init-artifact.sh          # 專案初始化腳本
    ├── bundle-artifact.sh        # 打包腳本
    └── shadcn-components.tar.gz  # 預打包的 shadcn/ui 組件
```

---

## 檔案說明

### 核心檔案

#### 1. `SKILL.md`
- **用途**：Skill 的主要說明文件
- **內容**：
  - YAML frontmatter（名稱、描述、授權）
  - 快速開始指南
  - 步驟說明（初始化、開發、打包、分享、測試）
  - 設計指南
  - 參考連結
- **特點**：使用命令式語氣，提供清晰的工作流程

#### 2. `LICENSE.txt`
- **用途**：定義 Skill 的使用授權條款
- **重要性**：確保使用者了解使用限制和權利

#### 3. `README.zh-TW.md`（本文件）
- **用途**：為繁體中文使用者提供詳細的說明和教學
- **內容**：完整的架構分析、使用說明和設計理念

### Scripts 目錄

#### 1. `init-artifact.sh`
**功能**：自動初始化一個完整配置的 React + TypeScript 專案

**主要步驟**：
1. 檢測 Node.js 版本並選擇相容的 Vite 版本
2. 使用 `pnpm create vite` 建立 React + TypeScript 專案
3. 安裝 Tailwind CSS 及相關依賴
4. 配置 PostCSS 和 Tailwind 設定檔
5. 設定路徑別名（`@/` → `./src/`）
6. 安裝所有 Radix UI 依賴（40+ 組件）
7. 解壓預打包的 shadcn/ui 組件
8. 建立 components.json 配置檔

**使用範例**：
```bash
bash scripts/init-artifact.sh my-dashboard
```

**關鍵技術**：
- Node.js 版本偵測：確保 Node 18+ 相容性
- 跨平台 sed 語法處理：macOS 和 Linux 相容
- pnpm 自動安裝：如果未安裝則自動安裝
- 一鍵完成：從零到完整配置的專案

#### 2. `bundle-artifact.sh`
**功能**：將 React 應用程式打包成單一 HTML 檔案

**主要步驟**：
1. 驗證專案結構（package.json 和 index.html）
2. 安裝打包依賴（parcel、html-inline 等）
3. 建立 Parcel 配置檔（支援路徑別名）
4. 使用 Parcel 建置專案
5. 使用 html-inline 將所有資源內嵌到單一 HTML
6. 輸出 bundle.html

**使用範例**：
```bash
cd my-dashboard
bash ../scripts/bundle-artifact.sh
```

**輸出**：`bundle.html` - 包含所有 JavaScript、CSS 和依賴的自包含 HTML 檔案

#### 3. `shadcn-components.tar.gz`
**功能**：預打包的 shadcn/ui 組件檔案

**內容**：
- 40+ 個 shadcn/ui UI 組件
- 已配置好的組件檔案
- 包含所有必要的 TypeScript 類型定義

**優點**：
- 節省安裝時間
- 確保組件版本一致性
- 避免網路問題導致的安裝失敗

---

## 關鍵腳本分析

### init-artifact.sh 深度分析

#### 1. Node.js 版本管理策略
```bash
NODE_VERSION=$(node -v | cut -d'v' -f2 | cut -d'.' -f1)

if [ "$NODE_VERSION" -lt 18 ]; then
  echo "❌ Error: Node.js 18 or higher is required"
  exit 1
fi

if [ "$NODE_VERSION" -ge 20 ]; then
  VITE_VERSION="latest"
else
  VITE_VERSION="5.4.11"  # Node 18 相容版本
fi
```

**設計理念**：
- 確保最低版本要求（Node 18+）
- 針對不同 Node 版本選擇相容的 Vite 版本
- 避免版本不相容導致的錯誤

#### 2. 跨平台相容性處理
```bash
if [[ "$OSTYPE" == "darwin"* ]]; then
  SED_INPLACE="sed -i ''"
else
  SED_INPLACE="sed -i"
fi
```

**設計理念**：
- macOS 和 Linux 的 sed 語法不同
- 自動偵測作業系統並使用正確語法
- 提高腳本的可移植性

#### 3. Tailwind CSS 主題配置
腳本自動建立完整的 Tailwind 配置，包括：
- CSS 變數系統（支援深色模式）
- shadcn/ui 色彩系統
- 自訂動畫（accordion-down、accordion-up）
- 響應式斷點
- 插件整合（tailwindcss-animate）

#### 4. TypeScript 路徑別名配置
```javascript
config.compilerOptions.baseUrl = '.';
config.compilerOptions.paths = { '@/*': ['./src/*'] };
```

**優點**：
- 簡化 import 路徑（`@/components/Button` 而非 `../../components/Button`）
- 提高程式碼可讀性
- 方便重構和檔案移動

### bundle-artifact.sh 深度分析

#### 1. 專案驗證機制
```bash
if [ ! -f "package.json" ]; then
  echo "❌ Error: No package.json found."
  exit 1
fi

if [ ! -f "index.html" ]; then
  echo "❌ Error: No index.html found."
  exit 1
fi
```

**設計理念**：
- 確保在正確的專案目錄執行
- 驗證必要檔案存在
- 提供清晰的錯誤訊息

#### 2. Parcel 配置策略
```json
{
  "extends": "@parcel/config-default",
  "resolvers": ["parcel-resolver-tspaths", "..."]
}
```

**設計理念**：
- 支援 TypeScript 路徑別名
- 擴展預設配置而非完全覆蓋
- 確保與專案設定一致

#### 3. 打包優化
```bash
pnpm exec parcel build index.html --dist-dir dist --no-source-maps
pnpm exec html-inline dist/index.html > bundle.html
```

**設計理念**：
- 移除 source maps 減少檔案大小
- 使用 html-inline 內嵌所有資源
- 生成完全自包含的 HTML 檔案

---

## 設計策略

### 1. 漸進式揭露（Progressive Disclosure）
Skill 遵循三層載入系統：

1. **元資料層**（總是載入）
   - Skill 名稱：artifacts-builder
   - 描述：說明使用時機和功能

2. **SKILL.md 層**（當 Skill 被觸發時載入）
   - 工作流程說明
   - 步驟指南
   - 設計準則

3. **資源層**（按需載入/執行）
   - 腳本可直接執行，不需載入到 context
   - 組件包在需要時解壓

### 2. 自動化優先
- 最小化手動配置
- 一鍵完成複雜設定
- 智慧偵測和調整（Node 版本、作業系統等）

### 3. 最佳實踐內建
- 預配置 40+ shadcn/ui 組件
- TypeScript 嚴格模式
- 路徑別名設定
- 深色模式支援
- 響應式設計基礎

### 4. 設計品質指引
SKILL.md 明確指出要避免的「AI slop」特徵：
- ❌ 過度置中的版面配置
- ❌ 紫色漸層
- ❌ 統一的圓角
- ❌ Inter 字型（過度使用）

這確保產生的 artifacts 具有專業、獨特的視覺風格。

---

## Claude Code 使用方式

### 觸發時機
Claude Code 會在以下情況使用此 Skill：

1. **使用者要求建立複雜的互動式網頁應用**
   - 範例：「建立一個待辦事項管理應用」
   - 範例：「製作一個數據儀表板」

2. **需要現代 UI 組件的專案**
   - 範例：「使用 shadcn/ui 建立一個表單」
   - 範例：「建立一個帶有對話框和選單的應用」

3. **需要狀態管理或路由的應用**
   - 範例：「建立多頁面的網頁應用」
   - 範例：「製作一個具有複雜狀態的應用」

### 不使用此 Skill 的情況
- 簡單的單檔 HTML/JSX artifacts
- 不需要 React 或組件庫的靜態頁面
- 純 CSS/HTML 展示頁面

### 典型工作流程

#### 步驟 1：初始化專案
Claude 執行：
```bash
bash scripts/init-artifact.sh user-dashboard
cd user-dashboard
```

#### 步驟 2：開發應用程式
Claude 編輯生成的檔案：
- `src/App.tsx` - 主要應用邏輯
- `src/components/` - 自訂組件
- 使用預裝的 shadcn/ui 組件

#### 步驟 3：打包
Claude 執行：
```bash
bash ../scripts/bundle-artifact.sh
```

#### 步驟 4：分享
Claude 將 `bundle.html` 內容分享給使用者作為 artifact

#### 步驟 5：測試（可選）
如果需要，Claude 可以使用 Playwright 或 Puppeteer 測試應用

---

## 技術架構

### 技術堆疊
```
React 18           # UI 框架
TypeScript         # 類型安全
Vite              # 開發伺服器和建置工具
Parcel            # 打包工具（用於生成單一 HTML）
Tailwind CSS 3.4  # CSS 框架
shadcn/ui         # UI 組件庫
Radix UI          # 無障礙 UI 基礎組件
pnpm              # 套件管理器
```

### 依賴關係圖
```
專案根目錄
├── React 18 + TypeScript
│   ├── Vite（開發）
│   └── Parcel（打包）
├── 樣式系統
│   ├── Tailwind CSS
│   ├── PostCSS
│   └── tailwindcss-animate
└── UI 組件
    ├── shadcn/ui（40+ 組件）
    ├── Radix UI（底層組件）
    ├── lucide-react（圖示）
    ├── class-variance-authority（樣式變體）
    └── 其他工具（clsx、tailwind-merge 等）
```

### 打包流程
```
原始碼（多檔案）
    ↓
[Vite 編譯 TypeScript]
    ↓
[Parcel 建置和最佳化]
    ↓
dist/index.html + 資源檔
    ↓
[html-inline 內嵌所有資源]
    ↓
bundle.html（單一檔案）
```

---

## 使用的 Prompt 策略

### 1. 明確的觸發條件
SKILL.md 的 description 清楚說明：
- ✅ 使用時機：複雜、多組件 artifacts
- ✅ 需要的功能：狀態管理、路由、shadcn/ui 組件
- ❌ 不使用時機：簡單單檔 HTML/JSX

### 2. 步驟式指引
提供清晰的 5 步驟流程：
1. 初始化
2. 開發
3. 打包
4. 分享
5. 測試（可選）

### 3. 內建最佳實踐提醒
在 SKILL.md 中使用 "VERY IMPORTANT" 強調設計準則，確保 Claude 注意到關鍵資訊。

### 4. 工具參考
提供外部資源連結（shadcn/ui 文檔），讓 Claude 可以查閱最新的組件 API。

---

## 使用的工具（Tools）

此 Skill 主要依賴以下 Claude Code 工具：

### 1. **Bash Tool**
- 執行 `init-artifact.sh` 和 `bundle-artifact.sh`
- 安裝依賴（pnpm install）
- 檔案系統操作

### 2. **Write Tool**
- 建立和編輯 React 組件
- 修改配置檔（tsconfig.json、vite.config.ts 等）
- 撰寫應用邏輯

### 3. **Read Tool**
- 讀取腳本內容（如需除錯或調整）
- 檢查生成的檔案
- 驗證配置

### 4. **Edit Tool**
- 修改現有檔案
- 更新配置
- 調整樣式

---

## 總結

Artifacts Builder 是一個精心設計的 Skill，展示了以下關鍵概念：

### 成功要素
1. **自動化**：減少重複性工作
2. **標準化**：確保配置一致性
3. **可靠性**：版本管理和錯誤處理
4. **易用性**：清晰的步驟和文檔
5. **品質**：內建最佳實踐和設計指引

### 學習重點
對於想要建立類似 Skill 的開發者：
- 識別重複性任務並自動化
- 提供清晰的工作流程
- 處理環境差異（版本、作業系統）
- 包含完整的依賴和資源
- 撰寫清晰的文檔和錯誤訊息

### 適用場景
此 Skill 最適合用於：
- 📊 數據儀表板
- 📝 管理介面
- 🎮 互動式工具
- 📱 原型設計
- 🎨 設計系統展示

---

## 參考資源

- [SKILL.md](./SKILL.md) - 英文原始文檔
- [shadcn/ui 官方文檔](https://ui.shadcn.com/docs/components)
- [Vite 文檔](https://vitejs.dev/)
- [Tailwind CSS 文檔](https://tailwindcss.com/)
- [React 官方文檔](https://react.dev/)

---

**最後更新**：2025-11-14
**文件版本**：1.0.0
**適用於**：Claude Code Skills
