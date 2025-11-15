# Changelog Generator - 正體中文說明文件

## Skill 概述

### 功能
Changelog Generator skill 自動從 git 提交記錄中創建用戶友善的變更日誌。它能分析提交歷史、分類變更，並將技術性的提交訊息轉換為清晰、客戶友善的發布說明。這個工具將數小時的手動變更日誌撰寫工作縮短為幾分鐘的自動生成過程。

### 解決的問題
開發團隊在準備發布說明時常面臨以下挑戰：

1. **耗時的手動工作**：逐一檢查 git 提交並撰寫變更日誌非常費時
2. **技術用語難懂**：開發者的提交訊息對客戶來說太技術性
3. **分類不一致**：難以系統化地分類功能、修復和改進
4. **遺漏重要變更**：手動過程容易遺漏重要的更新
5. **格式不統一**：每次發布的變更日誌格式可能不一致
6. **雜訊過多**：內部重構和測試提交不應該出現在客戶面向的變更日誌中

這個 skill 自動化整個流程，從 git 歷史中提取資訊，過濾雜訊，並生成專業、易讀的變更日誌。

---

## 目錄結構

```
changelog-generator/
├── SKILL.md          # Skill 主要說明文件（英文）
└── README.zh-TW.md   # 正體中文說明文件（本文件）
```

---

## 核心功能分析

### 1. Git 歷史掃描

**時間範圍選擇**：
- 特定時間段（如過去 7 天）
- 版本間的變更（如 v2.4.0 到 v2.5.0）
- 自上次發布以來的所有提交
- 按季度或月份分組

**提交分析**：
- 讀取提交訊息
- 識別提交類型
- 提取相關資訊
- 理解變更脈絡

### 2. 變更分類

自動將提交分組到邏輯類別：

**新功能（New Features）**：
- 全新的功能或能力
- 用戶可見的新增項
- 使用 ✨ 圖示

**改進（Improvements）**：
- 現有功能的增強
- 效能提升
- 使用者體驗改善
- 使用 🔧 圖示

**錯誤修復（Bug Fixes）**：
- 問題解決
- 錯誤更正
- 使用 🐛 圖示

**重大變更（Breaking Changes）**：
- API 變更
- 不向後相容的更新
- 需要用戶採取行動
- 使用 ⚠️ 圖示

**安全性（Security）**：
- 安全性修復
- 漏洞修補
- 使用 🔒 圖示

### 3. 技術翻譯為用戶語言

**技術提交範例**：
```
feat: implement workspace feature with user invite system
```

**轉換為用戶友善**：
```
✨ New Features
- **Team Workspaces**: Create separate workspaces for different
  projects. Invite team members and keep everything organized.
```

**轉換原則**：
- 使用第二人稱（你、您）
- 專注於好處而非實作
- 使用清晰、簡單的語言
- 避免技術術語
- 說明「為什麼重要」

### 4. 雜訊過濾

**排除的提交類型**：
- 重構（refactor）
- 測試（test）
- 文件更新（docs）
- 建構配置（build/ci）
- 內部工具變更
- 依賴更新（除非影響用戶）

**保留的提交**：
- 用戶可見的變更
- 功能和修復
- 效能改進
- 安全性更新

### 5. 專業格式化

**結構化輸出**：
```markdown
# Updates - Week of March 10, 2024

## ✨ New Features
[按重要性排序的功能清單]

## 🔧 Improvements
[改進項目清單]

## 🐛 Fixes
[錯誤修復清單]
```

**格式特點**：
- 清晰的標題和分類
- 視覺圖示提高可讀性
- 簡潔的描述
- 重點突出
- 易於掃描

### 6. 品牌聲音應用

**遵循指南**：
- 如果專案有 `CHANGELOG_STYLE.md`，遵循其格式
- 保持品牌語氣和風格
- 使用一致的術語
- 符合公司溝通標準

---

## Claude Code 使用方式

### 觸發時機

Claude Code 會在以下情況使用此 Skill：

1. **準備發布說明時**
   ```
   為版本 2.5.0 創建變更日誌
   ```

2. **定期更新摘要**
   ```
   生成過去一週的所有提交的變更日誌
   ```

3. **版本間比較**
   ```
   創建從 v2.4.0 到現在的變更日誌
   ```

4. **特定時間範圍**
   ```
   為 3 月 1 日至 3 月 15 日之間的所有提交創建變更日誌
   ```

### 典型使用場景

#### 場景 1：週更新摘要

```
創建過去 7 天的提交變更日誌
```

**Claude 會**：
1. 掃描過去 7 天的 git 提交
2. 過濾掉內部/測試提交
3. 分類為功能、改進、修復
4. 轉換為用戶友善語言
5. 輸出格式化的變更日誌

**輸出範例**：
```markdown
# Updates - Week of March 10, 2024

## ✨ New Features

- **Team Workspaces**: Create separate workspaces for different
  projects. Invite team members and keep everything organized.

- **Keyboard Shortcuts**: Press ? to see all available shortcuts.
  Navigate faster without touching your mouse.

## 🔧 Improvements

- **Faster Sync**: Files now sync 2x faster across devices
- **Better Search**: Search now includes file contents, not just titles

## 🐛 Fixes

- Fixed issue where large images wouldn't upload
- Resolved timezone confusion in scheduled posts
- Corrected notification badge count
```

#### 場景 2：版本發布說明

```
創建版本 2.5.0 的發布說明
```

**Claude 會**：
1. 識別自上一個版本標籤以來的所有提交
2. 生成完整的發布說明
3. 突出重大變更
4. 包含升級指南（如需要）

#### 場景 3：使用自訂風格指南

```
使用 CHANGELOG_STYLE.md 中的指南，為自 v2.4.0 以來的提交創建變更日誌
```

**Claude 會**：
1. 讀取專案的變更日誌風格指南
2. 應用自訂格式和語氣
3. 使用專案特定的術語
4. 遵循品牌聲音

---

## 使用的 Prompt 策略

### 1. 明確的使用場景

SKILL.md 清楚列出使用時機：
- 準備發布說明
- 週/月產品更新摘要
- 為客戶記錄變更
- App store 提交
- 產品更新通知
- 內部發布文件
- 公開變更日誌頁面

### 2. 步驟化處理流程

明確的六步驟流程：
1. 掃描 Git 歷史
2. 分類變更
3. 技術 → 用戶友善翻譯
4. 格式化
5. 過濾雜訊
6. 應用最佳實踐

### 3. 具體範例輸出

提供完整的輸出範例，展示：
- 結構和格式
- 分類方式
- 描述風格
- 視覺元素（圖示）

### 4. 多種使用模式

支援不同的使用方式：
- 基本使用（簡單請求）
- 特定日期範圍
- 使用自訂指南
- 版本標籤間比較

---

## 使用的工具（Tools）

此 Skill 主要依賴以下 Claude Code 工具：

### 1. **Bash Tool**
- 執行 git 命令
- 讀取提交歷史
- 識別版本標籤
- 過濾提交

**常用命令**：
```bash
# 獲取提交歷史
git log --since="7 days ago" --pretty=format:"%h - %s"

# 版本間的變更
git log v2.4.0..v2.5.0

# 提交詳情
git show [commit-hash]
```

### 2. **Read Tool**
- 讀取 CHANGELOG_STYLE.md（如果存在）
- 理解專案的格式偏好
- 讀取現有 CHANGELOG.md 以了解風格

### 3. **Write Tool**
- 生成新的變更日誌條目
- 更新 CHANGELOG.md 檔案
- 創建發布說明文件

### 4. **Grep Tool**
- 搜尋特定類型的提交
- 過濾提交訊息
- 識別模式

---

## 最佳實踐

### 使用技巧

1. **從專案根目錄執行**：確保在 git 倉庫根目錄運行
2. **指定明確的時間範圍**：提供具體日期或版本標籤
3. **使用風格指南**：創建 `CHANGELOG_STYLE.md` 以保持一致性
4. **審查後再發布**：在發布前檢查和調整生成的變更日誌
5. **直接保存到檔案**：要求將輸出保存到 `CHANGELOG.md`

### 提交訊息最佳實踐

為了獲得更好的變更日誌：

**使用約定式提交（Conventional Commits）**：
```
feat: add user authentication
fix: resolve login timeout issue
improvement: optimize database queries
breaking: change API response format
```

**撰寫清晰的描述**：
- 使用現在式動詞
- 說明「什麼」和「為什麼」
- 保持簡潔但具描述性

### 變更日誌格式建議

**標題**：
- 包含日期或版本號
- 清晰標示更新類型

**分類順序**：
1. 重大變更（如有）
2. 新功能
3. 改進
4. 錯誤修復
5. 安全性更新

**描述撰寫**：
- 以動詞開頭
- 專注於用戶價值
- 保持簡短（1-2 句）
- 使用粗體突出功能名稱

---

## 相關使用案例

### GitHub 發布說明
```
創建 GitHub 發布說明的變更日誌
```

### App Store 更新描述
```
生成適合 App Store 提交的更新描述
```

### 電子郵件用戶更新
```
創建可以發送給用戶的電子郵件更新內容
```

### 社群媒體公告
```
將變更日誌改寫為社群媒體公告貼文
```

---

## 總結

### 成功要素

1. **自動化**：節省大量手動工作時間
2. **用戶友善**：將技術語言轉換為客戶可理解的內容
3. **一致性**：每次都使用相同的格式和結構
4. **完整性**：不會遺漏重要的變更
5. **可自訂**：支援專案特定的風格指南

### 時間節省

**手動方式**：
- 檢查所有提交：30-60 分鐘
- 分類和組織：30 分鐘
- 撰寫用戶友善描述：60 分鐘
- **總計：2-3 小時**

**使用此 Skill**：
- 生成變更日誌：2-5 分鐘
- 審查和調整：10-15 分鐘
- **總計：15-20 分鐘**

### 適用場景

- 軟體發布
- 週/月產品更新
- App store 提交
- GitHub 發布頁面
- 產品變更通知
- 內部發布文件
- 客戶溝通

### 靈感來源

此 skill 受到 Manik Aggarwal 在 Lenny's Newsletter 中的使用案例啟發。

---

**最後更新**：2025-11-14
**文件版本**：1.0.0
**適用於**：Claude Code Skills
