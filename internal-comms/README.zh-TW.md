# Internal Comms - 內部溝通正體中文說明文件

## 📋 目錄
- [Skill 概述](#skill-概述)
- [目錄結構](#目錄結構)
- [檔案說明](#檔案說明)
- [範例指南深度分析](#範例指南深度分析)
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
Internal Comms 是一個專門用於撰寫公司內部溝通文件的資源套件。它提供了多種標準化的溝通格式範本和撰寫指南，讓 Claude 能夠按照公司慣用的風格和格式建立各種內部溝通文件，包括：
- 3P 更新（Progress, Plans, Problems）
- 公司電子報
- FAQ 問答集
- 狀態報告
- 領導層更新
- 專案更新
- 事故報告

### 解決的問題
在企業環境中，內部溝通經常遇到以下挑戰：

1. **格式不一致**：不同團隊成員使用不同的格式，導致溝通效率低落
2. **資訊過載**：內部更新過於冗長，主管和同事沒時間閱讀
3. **缺乏結構**：溝通文件結構鬆散，重點不明確
4. **語氣不當**：過於正式或過於隨意，無法符合企業文化
5. **資訊蒐集困難**：不知道從哪裡獲取相關資訊來撰寫更新
6. **重複性工作**：每週都要撰寫相似格式的文件，耗費時間

Internal Comms 透過提供標準化模板和詳細的撰寫指南，讓 Claude 能夠快速產生格式一致、簡潔有力的內部溝通文件，大幅提升溝通效率。

---

## 目錄結構

```
internal-comms/
├── SKILL.md                       # Skill 主要說明文件（英文）
├── LICENSE.txt                    # 授權條款
├── README.zh-TW.md               # 正體中文說明文件（本文件）
└── examples/                      # 溝通格式範例目錄
    ├── 3p-updates.md             # 3P 更新格式指南
    ├── company-newsletter.md     # 公司電子報格式指南
    ├── faq-answers.md            # FAQ 問答格式指南
    └── general-comms.md          # 一般溝通格式指南
```

---

## 檔案說明

### 核心檔案

#### 1. `SKILL.md`
- **用途**：Skill 的主要說明文件
- **內容**：
  - 使用時機說明
  - 如何使用此 Skill
  - 溝通類型與對應的指南檔案
  - 關鍵字列表
- **特點**：簡潔明瞭，提供清晰的決策樹

#### 2. `LICENSE.txt`
- **用途**：定義 Skill 的使用授權條款
- **重要性**：確保使用者了解使用限制和權利

#### 3. `README.zh-TW.md`（本文件）
- **用途**：為繁體中文使用者提供詳細的說明和教學
- **內容**：完整的架構分析、使用說明和設計理念

### Examples 目錄

#### 1. `3p-updates.md`（47 行）
**功能**：提供 3P 更新（Progress, Plans, Problems）的撰寫指南

**核心內容**：
- **目標受眾**：主管、領導層、團隊成員
- **閱讀時間**：30-60 秒
- **時間範圍**：通常為一週
- **三個部分**：
  1. Progress（進展）：過去一週完成的事項
  2. Plans（計畫）：下週計畫執行的事項
  3. Problems（問題）：阻礙團隊進展的事項

**格式規範**：
```
[emoji] [團隊名稱] (涵蓋日期)
Progress: [1-3 句簡潔說明]
Plans: [1-3 句簡潔說明]
Problems: [1-3 句簡潔說明]
```

**關鍵特點**：
- 嚴格的格式要求
- 以數據為導向
- 簡潔扼要（每部分不超過 3 句話）
- 選擇合適的 emoji 來表達團隊氛圍

**工作流程**：
1. 確認範圍（團隊名稱和時間範圍）
2. 蒐集資訊（使用可用工具或詢問使用者）
3. 草擬更新（遵循嚴格格式）
4. 審查（確保簡潔且以數據為導向）

**資訊來源**：
- Slack：團隊成員的發文（特別是有大量反應的貼文）
- Google Drive：重要成員撰寫的文件
- Email：相關往來郵件
- Calendar：重要的非例行會議

#### 2. `company-newsletter.md`（66 行）
**功能**：提供公司全員電子報的撰寫指南

**核心內容**：
- **目標受眾**：全體員工（1000+ 人）
- **長度**：約 20-25 個要點
- **發送管道**：Slack 和 Email
- **時間範圍**：過去一週或一個月

**關鍵特點**：
- 大量連結：Google Drive 文件、Slack 訊息、Email
- 簡短扼要：每個要點 1-2 句話
- 使用「我們」的語氣（展現團隊精神）

**分類策略**：
將公司動態分成不同區塊，例如：
- 產品開發、市場拓展、財務
- 招聘、執行、願景
- 外部新聞、內部新聞

**優先順序**：
關注：
- 公司層級的影響（非團隊細節）
- 領導層公告
- 重要里程碑和成就
- 影響大多數員工的資訊
- 外部認可或媒體報導

避免：
- 過於細節的團隊更新
- 僅與小群體相關的資訊
- 重複已溝通的資訊

**範例格式**：
```
:megaphone: 公司公告
- 公告 1
- 公告 2

:dart: 優先事項進展
- 領域 1
  - 子領域 1
  - 子領域 2
- 領域 2
  - 子領域 1

:pillar: 領導層更新
- 更新 1
- 更新 2
```

**資訊來源**：
- Slack：大型頻道中有大量反應的訊息
- Email：主管的公司公告
- Calendar：大型會議（如全員會議）
- Documents：受到關注的新文件（願景文件、季度計畫等）
- 外部媒體：公司相關的新聞報導

#### 3. `faq-answers.md`（30 行）
**功能**：提供 FAQ 問答集的撰寫指南

**核心目標**：
- 找出員工普遍困惑的問題
- 提供簡潔的答案以減少混淆

**適合主題**：
- 近期企業事件（募資、新主管等）
- 即將推出的產品
- 招聘進度
- 願景或重點變更

**格式規範**：
```
- *Question*: [插入問題 - 1 句話]
- *Answer*: [插入答案 - 1-2 句話]
```

**指導原則**：
- 全面性：涵蓋整個公司，不只專注於特定團隊
- 閱讀所有可用工具
- 產生與全公司相關的回應

**答案準則**：
- 盡可能基於官方公司溝通
- 如果資訊不確定，明確指出
- 連結到權威來源（文件、公告、郵件）
- 保持專業但平易近人的語氣
- 標記需要主管或官方回應的問題

**資訊來源**：
- Slack：跨公司的問題（有大量反應或支持的問題）
- Email：直接寫入 FAQ 的郵件
- Documents：Google Drive 文件、會議文件等

#### 4. `general-comms.md`（16 行）
**功能**：提供一般內部溝通的撰寫指南

**適用場景**：
不符合 3P 更新、電子報或 FAQ 的溝通需求

**前置步驟**：
1. 詢問使用者目標受眾
2. 了解溝通目的
3. 確認期望的語氣（正式、隨意、緊急、資訊性）
4. 確認特定格式要求

**一般原則**：
- 清晰簡潔
- 使用主動語態
- 將最重要的資訊放在前面
- 包含相關連結和參考資料
- 符合公司的溝通風格

---

## 範例指南深度分析

### 3P 更新的設計哲學

#### 1. 為何使用 3P 格式？

**問題**：傳統的狀態更新往往：
- 過於冗長（需要 5-10 分鐘閱讀）
- 缺乏結構（資訊散亂）
- 難以快速掌握重點

**解決方案**：3P 格式強制簡潔性
```
嚴格限制：
- 每個部分 1-3 句話
- 30-60 秒閱讀時間
- 資料導向（包含指標）
```

#### 2. 團隊規模的差異化處理

**小型團隊**（例如：Mobile Team）：
```
Progress: Shipped push notification feature to 50% of users
Plans: Complete A/B test analysis and expand to 100% by Friday
Problems: iOS crash rate increased to 2.3%, investigating root cause
```
- 具體任務和功能
- 明確的數字和百分比
- 技術細節適當

**大型團隊**（例如：整個公司）：
```
Progress: Hired 20 new engineers, closed $10M Series B funding
Plans: Launch new product line in Q2, expand to 3 new markets
Problems: Delay in regulatory approval affecting EU launch timeline
```
- 高層次的成就
- 業務影響
- 策略層級的資訊

#### 3. 數據導向的重要性

**弱範例**：
```
Progress: We made good progress on the project
Plans: We will continue working hard
Problems: Some challenges remain
```

**強範例**：
```
Progress: Reduced API latency by 40%, from 200ms to 120ms
Plans: Deploy caching layer to 5 regions, targeting 80ms by EOW
Problems: Database migration blocked by vendor, 2-week delay expected
```

**差異**：
- 具體數字 vs 模糊描述
- 可衡量目標 vs 抽象承諾
- 明確問題 vs 含糊其辭

### 公司電子報的策略

#### 1. 20-25 要點的黃金數量

**太少**（<10 要點）：
- 無法涵蓋所有重要事項
- 員工感覺資訊不足

**太多**（>30 要點）：
- 資訊過載
- 降低閱讀完成率

**剛好**（20-25 要點）：
- 5-7 分鐘閱讀時間
- 涵蓋主要動態
- 保持讀者參與度

#### 2. 分區策略

**為何需要分區**：
```
未分區：
- 招聘更新
- 產品發布
- 財務報告
- 新員工入職
- 產品 bug 修復
- 市場活動
- ...混亂

已分區：
:rocket: 產品與工程
- 產品發布
- 技術突破

:moneybag: 業務與財務
- 財務報告
- 新客戶

:busts_in_silhouette: 人才與文化
- 招聘更新
- 新員工入職
```

**優點**：
- 提高可讀性
- 幫助員工快速找到相關資訊
- 展現公司的組織性

#### 3. 連結策略

**為何大量連結很重要**：
```
無連結：
"We published our Q2 roadmap"

有連結：
"We published our [Q2 roadmap](link-to-doc) outlining 3 major initiatives"
```

**優點**：
- 提供更深入的資訊
- 增加可信度
- 方便員工查找細節
- 提高參與度

### FAQ 的資訊架構

#### 1. 問題識別策略

**好問題**：
```
影響範圍：大部分員工
明確性：具體且可回答
時效性：當前相關
```

範例：
- "What's our timeline for the office reopening?"
- "How will the new performance review system work?"
- "What does the recent funding round mean for hiring?"

**壞問題**：
```
影響範圍：僅特定個人
模糊性：難以回答
時效性：已過時
```

範例：
- "Why is my desk chair broken?"（個人問題）
- "What's the meaning of life?"（過於模糊）
- "When will we hire more people?" → 如果已經公布招聘計畫

#### 2. 答案品質控制

**清楚的答案結構**：
```
- *Question*: Will we be returning to the office?
- *Answer*: We're planning a hybrid model starting Q3 2024, with teams
  coming in 2-3 days/week. Full details in [this doc](link).
```

**包含的元素**：
1. 直接回答
2. 關鍵細節（時間、數字、政策）
3. 連結到詳細資訊
4. 如果不確定，明確說明

**避免**：
```
- *Question*: Will we be returning to the office?
- *Answer*: Maybe, we're still figuring it out.
```

---

## 設計策略

### 1. 模板導向設計

**核心理念**：標準化格式提高效率

**實作方式**：
- 每種溝通類型有獨立的指南檔案
- 提供明確的格式規範
- 包含範例和反範例

**優點**：
- 一致性：所有更新使用相同格式
- 效率：不需每次重新思考結構
- 品質：內建最佳實踐

### 2. 資訊來源指引

**為何重要**：
Claude 需要知道去哪裡找資訊

**策略**：
每個指南都明確列出工具：
- Slack（哪些頻道、什麼類型的訊息）
- Email（誰發送的、什麼主題）
- Calendar（什麼類型的會議）
- Documents（什麼類型的文件）

**範例**（從 3p-updates.md）：
```markdown
## Tools Available
- Slack: posts from team members with their updates
- Google Drive: docs written from critical team members
- Email: emails with lots of responses
- Calendar: non-recurring meetings with importance
```

### 3. 工作流程指導

**明確的步驟**：
每個指南提供清晰的工作流程

範例（3P 更新）：
```
1. Clarify scope（確認範圍）
2. Gather information（蒐集資訊）
3. Draft the update（草擬更新）
4. Review（審查）
```

**優點**：
- 減少不確定性
- 提高輸出品質
- 確保不遺漏關鍵步驟

### 4. 語氣和風格指引

**3P 更新**：
```
語氣：事實陳述、不過度修飾
風格：簡潔、數據導向
範例："Reduced latency by 40%"
非範例："We're excited to share amazing progress"
```

**公司電子報**：
```
語氣：團隊精神（使用「我們」）
風格：簡短、包含連結
範例："We shipped feature X to 100K users"
```

**FAQ**：
```
語氣：專業但平易近人
風格：清晰、基於官方資訊
範例：提供確定的答案或明確說明不確定性
```

### 5. 漸進式詳細度

**分層設計**：

**第一層**：SKILL.md
- 簡單說明使用時機
- 指向對應的範例檔案

**第二層**：範例檔案
- 詳細格式說明
- 工作流程
- 資訊來源
- 最佳實踐

**優點**：
- 快速載入（只載入需要的檔案）
- 保持 context 整潔
- 靈活性（可擴展新的溝通類型）

---

## Claude Code 使用方式

### 觸發時機

Claude Code 會在以下情況使用此 Skill：

1. **需要撰寫 3P 更新**
   - 範例：「幫我寫本週的團隊更新」
   - 範例：「產生工程團隊的 3P」
   - 範例：「寫一個進展、計畫和問題的摘要」

2. **需要撰寫公司電子報**
   - 範例：「撰寫本週的公司電子報」
   - 範例：「整理公司動態成一份更新」
   - 範例：「產生全員郵件更新」

3. **需要撰寫 FAQ**
   - 範例：「整理本週的常見問題」
   - 範例：「建立 FAQ 文件」
   - 範例：「回答員工的常見問題」

4. **其他內部溝通**
   - 範例：「寫一份事故報告」
   - 範例：「撰寫專案狀態更新」
   - 範例：「建立領導層簡報」

### 典型工作流程

#### 工作流程 1：撰寫 3P 更新

```
步驟 1：識別溝通類型
使用者：「幫我寫本週的團隊更新」
Claude：識別為 3P 更新

步驟 2：載入指南
Claude 讀取：examples/3p-updates.md

步驟 3：確認範圍
Claude：「請問是哪個團隊的更新？時間範圍是過去一週嗎？」

步驟 4：蒐集資訊
如果有工具存取權：
  - 搜尋 Slack 頻道的團隊更新
  - 檢查 Google Drive 的相關文件
  - 查看 Calendar 的重要會議
否則：
  - 詢問使用者提供資訊

步驟 5：草擬更新
按照格式撰寫：
🚀 Engineering Team (Nov 8-15)
Progress: [從資訊中提取的成就]
Plans: [從資訊中提取的計畫]
Problems: [從資訊中提取的問題]

步驟 6：審查和調整
- 確認簡潔性（30-60 秒閱讀）
- 驗證數據準確性
- 檢查格式正確性
```

#### 工作流程 2：撰寫公司電子報

```
步驟 1：載入指南
Claude 讀取：examples/company-newsletter.md

步驟 2：確認時間範圍
Claude：「這是本週還是本月的更新？」

步驟 3：跨工具蒐集資訊
Slack：
  - 檢查 #announcements, #general 等大型頻道
  - 尋找高反應數的訊息
  - 識別主管的重要發文

Email：
  - 尋找公司公告
  - 識別全員郵件

Calendar：
  - 檢查 All-Hands 會議
  - 查看大型活動

Documents：
  - 尋找新發布的重要文件
  - 識別高瀏覽量的文件

步驟 4：分類和組織
將蒐集的資訊分成區塊：
- 產品與工程
- 業務與財務
- 人才與文化
- 外部認可

步驟 5：撰寫電子報
按照格式撰寫 20-25 個要點
每個要點包含：
- 簡短描述（1-2 句）
- 相關連結
- 使用「我們」的語氣

步驟 6：審查
- 確認涵蓋所有重要領域
- 驗證連結有效性
- 檢查長度適中（20-25 要點）
```

#### 工作流程 3：撰寫 FAQ

```
步驟 1：載入指南
Claude 讀取：examples/faq-answers.md

步驟 2：識別問題
從可用工具尋找：
Slack：
  - 多人詢問的問題
  - 高反應數的問題貼文
  - 表達困惑的訊息

Email：
  - 包含 FAQ 的郵件
  - 回覆中的常見問題

Documents：
  - 文件中列出的 FAQ
  - 從內容推斷的問題

步驟 3：優先排序
選擇：
  - 影響大多數員工的問題
  - 當前相關的問題
  - 具體且可回答的問題

步驟 4：研究答案
從官方來源尋找答案：
  - 公司公告
  - 政策文件
  - 主管聲明

步驟 5：撰寫 FAQ
格式：
- *Question*: [問題]
- *Answer*: [答案 + 連結]

步驟 6：品質檢查
- 答案是否基於官方資訊？
- 是否包含連結？
- 不確定的地方是否明確標示？
```

### 不使用此 Skill 的情況

- 外部溝通（客戶郵件、行銷內容）
- 技術文檔（API 文檔、系統設計）
- 程式碼註解
- 個人訊息（非正式溝通）

---

## 技術架構

### 資源類型

```
純文檔型 Skill
├── 格式模板（Markdown）
├── 撰寫指南
└── 範例和反範例
```

**無程式碼**：
- 不包含可執行腳本
- 純粹的文檔和指南
- 依賴 Claude 的自然語言理解

### 資訊流

```
使用者請求
    ↓
Claude 識別溝通類型
    ↓
載入對應的範例指南
    ↓
[可選] 存取公司工具
    ↓
蒐集資訊
    ↓
按照格式撰寫
    ↓
輸出標準化的溝通文件
```

### 整合點

**Slack 整合**：
```
能力：
- 讀取頻道訊息
- 識別高參與度貼文
- 提取團隊更新

使用場景：
- 蒐集 3P 資訊
- 識別電子報內容
- 發現 FAQ 問題
```

**Google Drive 整合**：
```
能力：
- 搜尋文件
- 讀取內容
- 識別重要文件

使用場景：
- 提取詳細資訊
- 連結到官方文檔
- 驗證答案準確性
```

**Email 整合**：
```
能力：
- 讀取郵件
- 識別公司公告
- 提取關鍵資訊

使用場景：
- 蒐集官方溝通
- 識別重要更新
- 找出 FAQ 主題
```

**Calendar 整合**：
```
能力：
- 讀取會議資訊
- 識別重要事件
- 提取會議文件

使用場景：
- 發現重大事件
- 連結到會議記錄
- 了解優先事項
```

---

## 使用的 Prompt 策略

### 1. 明確的觸發關鍵字

**SKILL.md 包含豐富的關鍵字**：
```
Keywords: 3P updates, company newsletter, company comms,
weekly update, faqs, common questions, updates, internal comms
```

**效果**：
- 提高觸發準確性
- 涵蓋各種表達方式
- 減少誤觸發

### 2. 決策樹導向

**SKILL.md 提供清晰的分支**：
```
溝通類型？
├─ 3P 更新 → examples/3p-updates.md
├─ 公司電子報 → examples/company-newsletter.md
├─ FAQ → examples/faq-answers.md
└─ 其他 → examples/general-comms.md
```

**優點**：
- 快速定位正確的指南
- 避免混淆不同格式
- 確保使用正確的模板

### 3. 範例導向學習

**每個指南都包含**：
- 格式範例
- 好的範例（隱含）
- 明確的反範例（避免事項）

**3P 更新範例**：
```markdown
## Formatting
[emoji] [Team Name] (Dates)
Progress: [1-3 sentences]
Plans: [1-3 sentences]
Problems: [1-3 sentences]
```

### 4. 工作流程嵌入

**每個指南都有工作流程章節**：
```markdown
## Workflow
1. Clarify scope
2. Gather information
3. Draft the update
4. Review
```

**優點**：
- 引導 Claude 遵循最佳實踐
- 確保不遺漏關鍵步驟
- 提高輸出品質

### 5. 語氣和風格指引

**明確的語氣要求**：

3P 更新：
```markdown
The tone should be very matter-of-fact, not super prose-heavy.
```

公司電子報：
```markdown
Use the "we" tense, as you are part of the company.
```

FAQ：
```markdown
Keep tone professional but approachable.
```

### 6. 限制性指引

**明確的「避免」清單**：

公司電子報：
```markdown
Avoid:
- Overly granular team updates
- Information only relevant to small groups
- Duplicate information
```

**效果**：
- 防止常見錯誤
- 提高內容相關性
- 確保品質

---

## 使用的工具（Tools）

此 Skill 主要依賴以下 Claude Code 工具：

### 1. **Read Tool**
- 讀取 SKILL.md（識別溝通類型）
- 讀取對應的範例指南檔案
- 讀取 Slack 訊息（蒐集資訊）
- 讀取 Google Drive 文件（驗證資訊）
- 讀取郵件（蒐集官方溝通）

### 2. **Write Tool**
- 建立草稿文件
- 產生最終的溝通文件
- 儲存範本以供重複使用

### 3. **可能的 MCP 工具**（如果可用）
- **Slack MCP**：
  - 讀取頻道訊息
  - 搜尋特定主題
  - 識別高參與度貼文

- **Google Drive MCP**：
  - 搜尋文件
  - 讀取文件內容
  - 提取連結

- **Gmail MCP**：
  - 讀取郵件
  - 搜尋特定主題
  - 識別公司公告

### 4. **WebFetch Tool**（如果需要）
- 讀取公司內部網站
- 提取公開資訊
- 驗證外部連結

---

## 最佳實踐

### 1. 3P 更新最佳實踐

**✅ 建議做法**：

**具體且可衡量**：
```
Progress: Reduced API response time from 200ms to 120ms (40% improvement)
Plans: Deploy caching to 5 regions by Friday, targeting <100ms latency
Problems: Database migration delayed 2 weeks due to vendor issues
```

**選擇合適的 emoji**：
```
🚀 - 工程團隊（創新、快速）
📊 - 數據團隊（分析、洞察）
💰 - 銷售團隊（收益、成長）
🎨 - 設計團隊（創意、美學）
```

**針對團隊規模調整粒度**：
```
小團隊：具體任務和功能
大團隊/公司：高層次成就和策略
```

**❌ 避免做法**：

**過於模糊**：
```
Progress: We made good progress
Plans: We will continue working
Problems: Some challenges exist
```

**過於冗長**：
```
Progress: We spent a lot of time this week working on improving
the performance of our API. We did extensive profiling, identified
several bottlenecks, implemented caching strategies, optimized
database queries, and conducted thorough testing. The results
were very positive and we're excited about the improvements...
```

**缺乏數據**：
```
Progress: Improved performance
Plans: Work on new features
Problems: Need more resources
```

### 2. 公司電子報最佳實踐

**✅ 建議做法**：

**豐富的連結**：
```
✅ We published our [Q2 Roadmap](link) outlining 3 major product launches
✅ Sarah's [vision doc](link) on AI integration got 500+ views
✅ Check out the [All-Hands recording](link) from Monday
```

**使用「我們」的語氣**：
```
✅ We shipped feature X to 100K users
✅ We closed $10M in new deals this month
✅ We welcomed 15 new team members
```

**適當的分區**：
```
:rocket: 產品與工程
- [3-5 個相關更新]

:moneybag: 業務與銷售
- [3-5 個相關更新]

:busts_in_silhouette: 人才與文化
- [3-5 個相關更新]
```

**❌ 避免做法**：

**缺乏連結**：
```
❌ We published the Q2 roadmap
❌ Sarah wrote a vision doc
❌ The All-Hands happened Monday
```

**過於個人化**：
```
❌ I shipped feature X
❌ Our team closed deals
```

**缺乏組織**：
```
❌ 25 個要點全部混在一起，沒有分區
```

**過於細節**：
```
❌ The mobile team fixed a bug in the login flow that was affecting
0.01% of users on Android 11 devices with specific screen sizes...
```

### 3. FAQ 最佳實踐

**✅ 建議做法**：

**清晰的問答結構**：
```
- *Question*: When are we returning to the office?
- *Answer*: We're implementing a hybrid model starting Q3 2024,
  with 2-3 days in-office per week. Full policy details are in
  [this document](link).
```

**基於官方資訊**：
```
✅ According to the [CEO's email](link), we plan to...
✅ The [HR policy doc](link) states that...
✅ As announced in [last week's All-Hands](link)...
```

**明確標示不確定性**：
```
✅ The exact timeline hasn't been finalized yet, but we expect
   to share more details by end of month.
✅ This is still being discussed by leadership. We'll update
   when a decision is made.
```

**❌ 避免做法**：

**模糊的答案**：
```
❌ Question: When are we returning to office?
❌ Answer: Soon, we're working on it.
```

**缺乏來源**：
```
❌ Answer: We're implementing a hybrid model.
   （沒有連結或參考）
```

**個人意見**：
```
❌ Answer: I think we should return to office because...
```

**僅針對特定個人**：
```
❌ Question: Why is my laptop slow?
❌ Question: Can I get a new desk chair?
```

### 4. 一般最佳實踐

**資訊蒐集**：
```
1. 優先使用可用工具（Slack, Drive, Email）
2. 尋找高參與度的內容（反應、回覆、瀏覽）
3. 專注於相關時間範圍
4. 驗證資訊準確性
```

**撰寫流程**：
```
1. 蒐集所有相關資訊
2. 按照指南組織內容
3. 遵循格式要求
4. 添加連結和參考
5. 審查簡潔性和清晰度
6. 驗證語氣和風格
```

**品質檢查清單**：
```
□ 格式正確？
□ 長度適當？
□ 包含數據/指標？
□ 有相關連結？
□ 語氣符合指南？
□ 資訊準確？
□ 受眾適當？
```

---

## 總結

### 成功要素

1. **標準化格式**：確保一致性和專業性
2. **清晰的指南**：每種溝通類型都有詳細說明
3. **工具整合**：利用 Slack、Drive、Email 等蒐集資訊
4. **簡潔性強調**：所有格式都優先考慮簡潔
5. **範例導向**：提供清晰的格式和風格範例

### 學習重點

對於想要理解此 Skill 的使用者：
- 內部溝通的重要性和最佳實踐
- 如何撰寫簡潔有效的更新
- 不同溝通類型的適當格式
- 如何從多個來源蒐集資訊
- 語氣和風格在企業溝通中的作用

### 適用場景

此 Skill 最適合用於：
- 📊 定期團隊更新（3P）
- 📰 公司全員溝通（電子報）
- ❓ 常見問題整理（FAQ）
- 📢 領導層公告
- 📝 專案狀態報告
- 🚨 事故報告
- 📈 進度更新

### 技術亮點

- **純文檔設計**：無需程式碼，純靠結構化指南
- **模板導向**：標準化格式提高效率
- **工具整合友好**：設計用於與 Slack、Drive、Email 等整合
- **可擴展性**：容易添加新的溝通類型
- **語氣控制**：明確的風格指引確保適當的語氣

### 企業價值

**時間節省**：
- 減少撰寫時間（從 30 分鐘到 5 分鐘）
- 避免重複格式化工作
- 自動化資訊蒐集

**品質提升**：
- 一致的格式和風格
- 內建最佳實踐
- 減少錯誤和疏漏

**溝通效率**：
- 簡潔的更新提高閱讀率
- 標準化格式降低理解成本
- 清晰的結構便於快速掃描

---

## 參考資源

- [SKILL.md](./SKILL.md) - 英文原始文檔
- [examples/3p-updates.md](./examples/3p-updates.md) - 3P 更新指南
- [examples/company-newsletter.md](./examples/company-newsletter.md) - 公司電子報指南
- [examples/faq-answers.md](./examples/faq-answers.md) - FAQ 指南
- [examples/general-comms.md](./examples/general-comms.md) - 一般溝通指南

---

**最後更新**：2025-11-15
**文件版本**：1.0.0
**適用於**：Claude Code Skills
