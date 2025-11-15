# Skill Share - 正體中文說明文件

## Skill 概述

### 功能
Skill Share skill 用於創建新的 Claude skills 並自動透過 Rube 在 Slack 上分享，實現無縫的團隊協作和 skill 發現。

### 解決的問題
團隊在建立和分享 Claude skills 時常面臨以下挑戰：

1. **創建過程繁瑣**：手動建立 skill 結構和元數據
2. **格式不一致**：團隊成員使用不同的 skill 格式
3. **分享困難**：難以讓團隊知道新 skill 的存在
4. **文件缺失**：Skill 缺乏適當的文件和元數據
5. **發現問題**：團隊成員不知道有哪些 skills 可用
6. **版本混亂**：難以追蹤 skill 的版本和更新

這個 skill 自動化了 skill 創建和團隊通知過程，確保一致性和可見性。

---

## 目錄結構

```
skill-share/
├── SKILL.md          # Skill 主要說明文件（英文）
├── LICENSE.txt       # 授權條款
└── README.zh-TW.md   # 正體中文說明文件（本文件）
```

---

## 核心功能分析

### 1. Skill 創建

**建立結構**：
- 創建正確結構的 skill 目錄
- 生成帶有 YAML frontmatter 的 SKILL.md
- 創建標準化的子目錄：
  - `scripts/`：腳本檔案
  - `references/`：參考資料
  - `assets/`：資源檔案

**自動生成元數據**：
- 自動產生 YAML frontmatter
- 包含必要欄位：
  - name（名稱）
  - description（描述）
  - license（授權）

**命名規範**：
- 強制使用連字號命名（hyphen-case）
- 範例：`skill-pdf-analyzer`

### 2. Skill 驗證

**檢查項目**：
- **SKILL.md 格式**：驗證結構正確
- **必要欄位**：確保元數據完整
- **命名規範**：檢查命名是否符合標準
- **元數據完整性**：確保所有必要資訊都存在

**驗證時機**：
- 創建後自動驗證
- 打包前驗證
- 分享前驗證

### 3. Skill 打包

**打包功能**：
- 創建可分發的 zip 檔案
- 包含所有 skill 資源和文件
- 在打包前自動運行驗證

**打包內容**：
- SKILL.md
- 所有腳本和資源
- 參考文件
- 授權檔案

### 4. Slack 整合（透過 Rube）

**自動通知**：
- 將創建的 skill 資訊發送到指定的 Slack 頻道
- 分享 skill 元數據（名稱、描述、連結）
- 發布 skill 摘要供團隊發現
- 提供直接連結到 skill 檔案

**Rube 功能使用**：
- **SLACK_SEND_MESSAGE**：發布 skill 資訊到團隊頻道
- **SLACK_POST_MESSAGE_WITH_BLOCKS**：分享格式化的 skill 元數據
- **SLACK_FIND_CHANNELS**：發現 skill 公告的目標頻道

### 5. 工作流程

**完整流程**：
1. **初始化**：提供 skill 名稱和描述
2. **創建**：建立正確結構的 skill 目錄
3. **驗證**：驗證 skill 元數據的正確性
4. **打包**：將 skill 打包成可分發格式
5. **Slack 通知**：將 skill 詳情發布到團隊的 Slack 頻道

---

## Claude Code 使用方式

### 觸發時機

Claude Code 會在以下情況使用此 Skill：

1. **明確的 skill 創建請求**
   ```
   創建一個名為 "pdf-analyzer" 的新 skill
   ```

2. **用戶表示想創建/分享 skill**
   ```
   我想創建並分享一個新的 skill
   ```

3. **從描述創建 skill**
   ```
   幫我創建一個用於分析 PDF 文件的 skill
   ```

### 典型使用場景

#### 場景 1：創建並分享新 Skill

**使用者**：「創建一個名為 'pdf-analyzer' 的 skill」

**處理流程**：
1. 創建 `/skill-pdf-analyzer/` 目錄與 SKILL.md 模板
2. 生成結構化目錄（scripts/、references/、assets/）
3. 驗證 skill 結構
4. 將 skill 打包為 zip 檔案
5. 發布到 Slack：「新 Skill 已創建：pdf-analyzer - 進階 PDF 分析和提取功能」

**Slack 通知範例**：
```
🎉 新 Skill 已創建！

📦 Skill 名稱：pdf-analyzer
📝 描述：進階 PDF 分析和提取功能
🔗 位置：/skills/pdf-analyzer/
📄 文件：[連結到 SKILL.md]

由 @username 創建
```

#### 場景 2：驗證現有 Skill

```
驗證 my-custom-skill 的結構
```

**Claude 會**：
1. 檢查 SKILL.md 是否存在
2. 驗證 YAML frontmatter
3. 檢查必要欄位
4. 確認命名規範
5. 報告任何問題

---

## 使用的 Prompt 策略

### 1. 明確的使用時機

SKILL.md 列出何時使用：
- 創建具有適當結構和元數據的新 Claude skills
- 生成準備分發的 skill 套件
- 自動在 Slack 頻道上分享創建的 skills 以提高團隊可見性
- 在分享前驗證 skill 結構
- 為團隊打包和分發 skills

### 2. 關鍵功能說明

四大核心功能：
1. Skill 創建
2. Skill 驗證
3. Skill 打包
4. Slack 整合

### 3. 工作流程步驟

清晰的 5 步驟流程：
1. 初始化
2. 創建
3. 驗證
4. 打包
5. Slack 通知

---

## 使用的工具（Tools）

此 Skill 主要依賴以下 Claude Code 工具：

### 1. **Write Tool**
- 創建 SKILL.md 檔案
- 生成 YAML frontmatter
- 寫入模板內容

### 2. **Bash Tool**
- 創建目錄結構
- 打包 skill 為 zip
- 組織檔案

### 3. **Rube Integration**（Slack 工具）
- **SLACK_SEND_MESSAGE**：發送簡單通知
- **SLACK_POST_MESSAGE_WITH_BLOCKS**：發送格式化訊息
- **SLACK_FIND_CHANNELS**：找出目標頻道

### 4. **Read Tool**
- 讀取現有 skill 以驗證
- 檢查 SKILL.md 格式
- 驗證元數據

---

## 最佳實踐

### Skill 命名

**使用連字號命名**：
- ✓ `pdf-analyzer`
- ✓ `image-processor`
- ✓ `data-validator`
- ✗ `pdfAnalyzer`
- ✗ `image_processor`
- ✗ `DataValidator`

### 描述撰寫

**清晰且描述性**：
- 說明 skill 做什麼
- 說明何時使用
- 保持簡潔（1-2 句）

**範例**：
```
description: Advanced PDF analysis and extraction capabilities.
Use when the user asks to analyze, extract, or process PDF documents.
```

### Slack 頻道設定

**選擇適當頻道**：
- #skills：skill 公告專用頻道
- #engineering：技術團隊更新
- #general：全公司可見性

---

## 需求

**系統需求**：
- 透過 Rube 連接 Slack workspace
- Skill 創建目錄的寫入權限
- Python 3.7+（用於 skill 創建腳本）
- Skill 通知的目標 Slack 頻道

**Rube 配置**：
- 已安裝並配置 Rube
- Slack workspace 已連接
- 適當的頻道權限

---

## 總結

### 成功要素

1. **標準化**：一致的 skill 結構和格式
2. **自動化**：減少手動創建和分享步驟
3. **驗證**：確保 skill 品質和完整性
4. **可見性**：團隊自動知道新 skills
5. **協作**：便於團隊共享和發現

### 理想用途

**適合用於**：
- 作為團隊工作流程的一部分創建 skills
- 建立需要 skill 創建 + 團隊通知的內部工具
- 自動化 skill 開發流程
- 協作 skill 創建與團隊通知

**適用場景**：
- 團隊 skill 開發
- 內部工具創建
- Skill 庫管理
- 團隊協作增強

---

**最後更新**：2025-11-14
**文件版本**：1.0.0
**適用於**：Claude Code Skills
