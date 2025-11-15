# MCP Builder - 正體中文說明文件

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
MCP Builder 是一個全方位的 MCP (Model Context Protocol) 伺服器開發指南套件。它提供完整的架構、最佳實踐和評估工具,讓 Claude 能夠建立高品質的 MCP 伺服器,使 LLM 能夠透過精心設計的工具與外部服務互動。

主要功能包括:
- 提供結構化的四階段開發流程(研究、實作、審查、評估)
- 包含 Python (FastMCP) 和 Node/TypeScript (MCP SDK) 兩種實作指南
- 提供自動化評估框架,測試 MCP 伺服器的效能
- 內建最佳實踐指南,涵蓋命名規範、錯誤處理、安全性等
- 支援多種傳輸協定(stdio, HTTP, SSE)

### 解決的問題
在建立 MCP 伺服器時,開發者經常遇到以下挑戰:

1. **設計不良**:只是簡單地包裝 API 端點,未考慮 AI Agent 的實際需求
2. **缺乏一致性**:工具命名、回應格式、錯誤處理缺乏標準化
3. **無效率的工具**:回傳過多資料導致 context 浪費,或回傳太少資訊無法完成任務
4. **測試困難**:缺乏系統化的測試方法,無法驗證 MCP 伺服器是否真正有用
5. **安全性問題**:未妥善處理輸入驗證、認證授權等安全議題
6. **文檔不足**:缺乏清晰的實作範例和設計原則

MCP Builder 提供完整的開發框架和自動化工具,解決了這些問題,讓開發者能夠建立真正實用、安全且高效的 MCP 伺服器。

---

## 目錄結構

```
mcp-builder/
├── SKILL.md                          # Skill 主要說明文件(英文)
├── LICENSE.txt                       # 授權條款
├── README.zh-TW.md                  # 正體中文說明文件(本文件)
├── scripts/                          # 評估和連線腳本目錄
│   ├── evaluation.py                # MCP 伺服器評估框架
│   ├── connections.py               # MCP 連線處理模組
│   ├── example_evaluation.xml       # 評估範例檔案
│   └── requirements.txt             # Python 依賴套件
└── reference/                        # 參考文檔目錄
    ├── mcp_best_practices.md        # MCP 最佳實踐指南
    ├── python_mcp_server.md         # Python/FastMCP 實作指南
    ├── node_mcp_server.md           # Node/TypeScript 實作指南
    └── evaluation.md                # 評估創建指南
```

---

## 檔案說明

### 核心檔案

#### 1. `SKILL.md`(329 行)
- **用途**:MCP 伺服器開發的主要指南
- **內容**:
  - 四階段開發流程(研究、實作、審查、評估)
  - Agent 導向設計原則
  - 工具開發指導方針
  - 測試和建置策略
  - 參考資源連結
- **特點**:結構化的步驟式指引,確保開發品質

#### 2. `LICENSE.txt`
- **用途**:定義 Skill 的使用授權條款
- **重要性**:確保使用者了解使用限制和權利

#### 3. `README.zh-TW.md`(本文件)
- **用途**:為繁體中文使用者提供詳細的說明和教學
- **內容**:完整的架構分析、使用說明和設計理念

### Scripts 目錄

#### 1. `evaluation.py`(374 行)
**功能**:MCP 伺服器評估框架

**主要組件**:

**`parse_evaluation_file()` 函數**:
- 解析 XML 格式的評估檔案
- 提取問答對(question/answer pairs)
- 回傳評估任務清單

**`agent_loop()` 函數**:
- 執行 Agent 迴圈,使用 MCP 工具回答問題
- 管理 Claude API 呼叫
- 處理工具使用流程(tool use → tool result → response)
- 追蹤工具呼叫次數和執行時間
- 提取回應、摘要和回饋

**`evaluate_single_task()` 函數**:
- 評估單一問答對
- 比較預期答案和實際答案
- 計算分數(精確匹配)
- 收集效能指標

**`run_evaluation()` 函數**:
- 執行完整的評估流程
- 載入 MCP 伺服器工具
- 對所有問答對執行評估
- 生成詳細的評估報告
- 包含準確率、平均執行時間、工具呼叫統計

**命令列介面**:
```bash
# 評估本地 stdio MCP 伺服器
python evaluation.py -t stdio -c python -a my_server.py eval.xml

# 評估 SSE MCP 伺服器
python evaluation.py -t sse -u https://example.com/mcp -H "Authorization: Bearer token" eval.xml

# 評估 HTTP MCP 伺服器並指定模型
python evaluation.py -t http -u https://example.com/mcp -m claude-3-5-sonnet-20241022 eval.xml
```

**評估提示詞**(EVALUATION_PROMPT):
- 要求 Agent 使用可用工具完成任務
- 要求提供步驟摘要(summary)
- 要求提供工具回饋(feedback)
- 要求提供最終回應(response)
- 格式:使用 XML 標籤包裝各部分

**報告格式**:
- 總體摘要(準確率、平均時長、工具呼叫次數)
- 每個任務的詳細結果
- 包含問題、預期答案、實際答案、正確性、執行時間
- 包含 Agent 的步驟摘要和工具回饋

#### 2. `connections.py`(152 行)
**功能**:輕量級 MCP 伺服器連線處理模組

**主要類別**:

**`MCPConnection` 抽象基類**:
- 管理 MCP 伺服器連線生命週期
- 實作 async context manager (`__aenter__`, `__aexit__`)
- 提供 `list_tools()` 方法:列出可用工具
- 提供 `call_tool()` 方法:呼叫工具並回傳結果

**`MCPConnectionStdio` 類別**:
- 使用標準輸入/輸出連線
- 適用於本地 MCP 伺服器(子行程)
- 接受命令、參數和環境變數

**`MCPConnectionSSE` 類別**:
- 使用 Server-Sent Events 連線
- 適用於遠端 MCP 伺服器
- 支援即時更新和推送通知

**`MCPConnectionHTTP` 類別**:
- 使用 Streamable HTTP 連線
- 適用於遠端 MCP 伺服器
- 支援多客戶端存取

**`create_connection()` 工廠函數**:
- 根據傳輸類型建立適當的連線物件
- 支援 stdio, SSE, HTTP 三種傳輸方式
- 驗證必要參數

**設計亮點**:
- 統一的介面,隱藏傳輸細節
- 自動資源管理(使用 AsyncExitStack)
- 錯誤處理和驗證
- 支援多種傳輸協定

#### 3. `example_evaluation.xml`
**功能**:評估檔案範例

**格式**:
```xml
<evaluation>
  <qa_pair>
    <question>問題文字</question>
    <answer>答案文字</answer>
  </qa_pair>
  <!-- 更多 qa_pair... -->
</evaluation>
```

**用途**:
- 提供評估檔案格式範例
- 展示如何設計評估問題
- 作為建立自訂評估的模板

#### 4. `requirements.txt`
**內容**:
```
anthropic
mcp
```

**用途**:
- 列出 Python 腳本所需的依賴套件
- `anthropic`:Claude API 客戶端
- `mcp`:MCP Python SDK

### Reference 目錄

#### 1. `mcp_best_practices.md`(約 916 行)
**功能**:MCP 伺服器開發的綜合最佳實踐指南

**主要章節**:

**命名規範**:
- 伺服器命名:Python 使用 `{service}_mcp`,Node/TypeScript 使用 `{service}-mcp-server`
- 工具命名:使用 snake_case,包含服務前綴,例如 `slack_send_message`

**回應格式指南**:
- JSON 格式:機器可讀的結構化資料
- Markdown 格式:人類可讀的格式化文字
- 支援兩種格式,根據用途選擇

**分頁最佳實踐**:
- 始終遵守 `limit` 參數
- 實作分頁(offset 或 cursor-based)
- 回傳分頁元資料(`has_more`, `next_offset`, `total_count`)
- 預設合理的限制(20-50 項)

**字元限制和截斷**:
- 定義 CHARACTER_LIMIT 常數(通常 25,000)
- 檢查回應大小
- 優雅地截斷並提供指引

**工具開發最佳實踐**:
- 描述性的名稱
- 使用參數驗證(Pydantic/Zod)
- 包含範例
- 適當的錯誤處理
- 進度報告(長時間操作)
- 工具註釋(readOnlyHint, destructiveHint 等)

**安全性考量**:
- 輸入驗證(防止路徑遍歷、命令注入)
- 存取控制(認證、授權、審計)
- 錯誤處理(不暴露內部錯誤)
- OAuth 2.1 實作
- API 金鑰管理

**傳輸選項**(stdio, HTTP, SSE):
- 各自的適用場景
- 選擇標準
- 安全性最佳實踐

**測試要求**:
- 功能測試
- 整合測試
- 安全性測試
- 效能測試
- 錯誤處理測試

#### 2. `python_mcp_server.md`(約 673 行)
**功能**:Python/FastMCP 完整實作指南

**主要內容**:
- 伺服器初始化模式
- Pydantic 模型範例
- 使用 `@mcp.tool` 註冊工具
- 完整的工作範例
- 品質檢查清單

**關鍵特性**:
- 使用 MCP Python SDK
- Pydantic v2 模型與 `model_config`
- 完整的類型提示
- Async/await 用於所有 I/O 操作
- 模組級常數(CHARACTER_LIMIT, API_BASE_URL)

#### 3. `node_mcp_server.md`(約 671 行)
**功能**:Node/TypeScript 完整實作指南

**主要內容**:
- 專案結構
- Zod schema 模式
- 使用 `server.registerTool` 註冊工具
- 完整的工作範例
- 品質檢查清單

**關鍵特性**:
- 使用 MCP TypeScript SDK
- Zod schemas 與 `.strict()`
- TypeScript 嚴格模式
- 無 `any` 類型 - 使用適當的類型
- 明確的 Promise<T> 回傳類型
- 建置流程配置

#### 4. `evaluation.md`(約 543 行)
**功能**:評估創建完整指南

**主要內容**:
- 問題創建指南
- 答案驗證策略
- XML 格式規格
- 範例問題和答案
- 執行評估的腳本使用說明

**評估要求**:
- 獨立性:問題互不依賴
- 唯讀性:僅需非破壞性操作
- 複雜性:需要多次工具呼叫和深入探索
- 現實性:基於真實使用案例
- 可驗證性:單一明確答案(字串比較)
- 穩定性:答案不會隨時間改變

---

## 關鍵腳本分析

### evaluation.py 深度分析

#### 1. Agent 迴圈設計

**核心機制**:
```python
async def agent_loop(client, model, question, tools, connection):
    messages = [{"role": "user", "content": question}]

    # 第一次 API 呼叫
    response = await asyncio.to_thread(
        client.messages.create,
        model=model,
        max_tokens=4096,
        system=EVALUATION_PROMPT,
        messages=messages,
        tools=tools,
    )

    # 迴圈處理工具使用
    while response.stop_reason == "tool_use":
        tool_use = next(block for block in response.content if block.type == "tool_use")
        tool_name = tool_use.name
        tool_input = tool_use.input

        # 執行工具並測量時間
        tool_start_ts = time.time()
        tool_result = await connection.call_tool(tool_name, tool_input)
        tool_duration = time.time() - tool_start_ts

        # 追蹤工具指標
        if tool_name not in tool_metrics:
            tool_metrics[tool_name] = {"count": 0, "durations": []}
        tool_metrics[tool_name]["count"] += 1
        tool_metrics[tool_name]["durations"].append(tool_duration)

        # 將工具結果加入訊息
        messages.append({
            "role": "user",
            "content": [{"type": "tool_result", "tool_use_id": tool_use.id, "content": tool_response}]
        })

        # 繼續對話
        response = await asyncio.to_thread(...)
        messages.append({"role": "assistant", "content": response.content})

    return response_text, tool_metrics
```

**設計亮點**:
- 完整的工具使用循環
- 自動追蹤工具呼叫次數和執行時間
- 錯誤處理:工具執行失敗時提供 traceback
- 保留完整的對話歷史

#### 2. 評估提示詞策略

**EVALUATION_PROMPT 的設計**:
```python
EVALUATION_PROMPT = """You are an AI assistant with access to tools.

When given a task, you MUST:
1. Use the available tools to complete the task
2. Provide summary of each step in your approach, wrapped in <summary> tags
3. Provide feedback on the tools provided, wrapped in <feedback> tags
4. Provide your final response, wrapped in <response> tags

Summary Requirements:
- The steps you took to complete the task
- Which tools you used, in what order, and why
- The inputs you provided to each tool
- The outputs you received from each tool
- A summary for how you arrived at the response

Feedback Requirements:
- Comment on tool names: Are they clear and descriptive?
- Comment on input parameters: Are they well-documented?
- Comment on descriptions: Do they accurately describe what the tool does?
- Comment on any errors encountered during tool usage
- Identify specific areas for improvement and explain WHY
- Be specific and actionable in your suggestions

Response Requirements:
- Your response should be concise and directly address what was asked
- Always wrap your final response in <response> tags
- If you cannot solve the task return <response>NOT_FOUND</response>
- For numeric responses, provide just the number
- For IDs, provide just the ID
"""
```

**設計理念**:
- **結構化輸出**:使用 XML 標籤確保可解析性
- **強制工具使用**:明確要求使用工具
- **收集回饋**:獲取工具改進建議
- **步驟追蹤**:了解 Agent 的思考過程
- **標準化答案**:簡潔的回應格式,便於比較

#### 3. 報告生成機制

**報告結構**:
```python
REPORT_HEADER = """
# Evaluation Report

## Summary

- **Accuracy**: {correct}/{total} ({accuracy:.1f}%)
- **Average Task Duration**: {average_duration_s:.2f}s
- **Average Tool Calls per Task**: {average_tool_calls:.2f}
- **Total Tool Calls**: {total_tool_calls}

---
"""

TASK_TEMPLATE = """
### Task {task_num}

**Question**: {question}
**Ground Truth Answer**: `{expected_answer}`
**Actual Answer**: `{actual_answer}`
**Correct**: {correct_indicator}
**Duration**: {total_duration:.2f}s
**Tool Calls**: {tool_calls}

**Summary**
{summary}

**Feedback**
{feedback}

---
"""
```

**設計優點**:
- **全面性**:包含所有關鍵指標
- **可讀性**:Markdown 格式,易於檢視
- **可操作性**:Agent 回饋指出改進方向
- **可追蹤性**:每個任務的詳細步驟

#### 4. 傳輸抽象層

**connections.py 的設計模式**:
```python
class MCPConnection(ABC):
    """抽象基類,定義統一介面"""

    @abstractmethod
    def _create_context(self):
        """子類別實作特定傳輸協定"""

    async def __aenter__(self):
        """初始化連線"""
        self._stack = AsyncExitStack()
        ctx = self._create_context()
        result = await self._stack.enter_async_context(ctx)
        # 處理不同回傳值(2 或 3 元組)
        if len(result) == 2:
            read, write = result
        elif len(result) == 3:
            read, write, _ = result
        # 建立 session
        session_ctx = ClientSession(read, write)
        self.session = await self._stack.enter_async_context(session_ctx)
        await self.session.initialize()
        return self

    async def list_tools(self):
        """列出工具"""
        response = await self.session.list_tools()
        return [{"name": tool.name, "description": tool.description, ...}
                for tool in response.tools]

    async def call_tool(self, tool_name, arguments):
        """呼叫工具"""
        result = await self.session.call_tool(tool_name, arguments=arguments)
        return result.content
```

**優點**:
- **統一介面**:不論傳輸協定,使用方式相同
- **自動資源管理**:使用 AsyncExitStack 確保清理
- **錯誤處理**:處理不同回傳格式
- **可擴展性**:新增傳輸協定只需新增子類別

### mcp_best_practices.md 核心概念

#### 1. 命名規範的重要性

**為何需要服務前綴**:
```python
# ❌ 不好 - 可能與其他伺服器衝突
def send_message(channel, text):
    ...

# ✅ 好 - 明確識別服務
def slack_send_message(channel, text):
    ...
```

**原因**:
- MCP 客戶端可能連接多個伺服器
- 避免工具名稱衝突
- 提高工具可發現性
- 明確工具的功能範圍

#### 2. 回應格式策略

**JSON vs Markdown 的使用場景**:
```python
# JSON 格式 - 用於進一步處理
{
    "users": [
        {
            "id": "U123456",
            "name": "John Doe",
            "email": "john@example.com",
            "created_at": 1705320000
        }
    ],
    "total": 150,
    "has_more": true
}

# Markdown 格式 - 用於呈現給使用者
"""
## Users (150 total, showing 1-20)

- **John Doe** (@john.doe, U123456)
  - Email: john@example.com
  - Created: 2024-01-15 10:30:00 UTC

[More results available. Use offset=20 to see more.]
"""
```

**設計原則**:
- JSON:保留完整資訊,便於程式處理
- Markdown:人類可讀,減少視覺雜訊
- 提供選項:讓 Agent 根據需求選擇

#### 3. 分頁設計哲學

**為何不能載入所有結果**:
```python
# ❌ 錯誤 - 忽略 limit
async def list_items(limit: int = 20):
    all_items = await api.get_all_items()  # 可能有 10,000+ 項
    return all_items  # 忽略 limit,回傳所有資料

# ✅ 正確 - 遵守 limit
async def list_items(limit: int = 20, offset: int = 0):
    items = await api.get_items(limit=limit, offset=offset)
    total = await api.count_items()
    return {
        "items": items,
        "total": total,
        "count": len(items),
        "offset": offset,
        "has_more": offset + len(items) < total,
        "next_offset": offset + len(items) if offset + len(items) < total else None
    }
```

**原因**:
- Context window 有限
- 網路頻寬考量
- API 速率限制
- 使用者體驗(回應速度)

#### 4. 字元限制策略

**截斷處理**:
```python
CHARACTER_LIMIT = 25000

def format_response(data):
    result = format_data(data)

    if len(result) > CHARACTER_LIMIT:
        # 減少資料量
        truncated_data = data[:max(1, len(data) // 2)]
        result = format_data(truncated_data)

        # 添加截斷訊息
        result += f"\n\n⚠️ Response truncated from {len(data)} to {len(truncated_data)} items."
        result += f"\n💡 Tip: Use filters (status='active', type='bug') or smaller limit to reduce results."

        # 提供下一步指引
        if has_filters_available:
            result += f"\n📋 Available filters: status, type, priority, assignee"

    return result
```

**設計理念**:
- **防止 context 爆炸**:避免浪費 token
- **提供指引**:告訴 Agent 如何獲取更多資料
- **保持可用性**:截斷後仍提供有用資訊

---

## 設計策略

### 1. 四階段開發流程

**階段設計理念**:

**Phase 1: 深度研究和規劃**
- **目標**:完全理解問題域和最佳實踐
- **活動**:
  - 學習 Agent 導向設計原則
  - 研究 MCP 協定文檔
  - 深入研究 API 文檔
  - 建立綜合實作計畫
- **輸出**:詳細的工具清單、共用工具設計、錯誤處理策略

**Phase 2: 實作**
- **目標**:按照計畫建立高品質的程式碼
- **活動**:
  - 設定專案結構
  - 實作核心基礎設施(共用工具)
  - 系統化地實作每個工具
  - 遵循語言特定最佳實踐
- **輸出**:完整的 MCP 伺服器程式碼

**Phase 3: 審查和改進**
- **目標**:確保程式碼品質和正確性
- **活動**:
  - 程式碼品質審查(DRY, 一致性, 錯誤處理)
  - 測試和建置
  - 使用品質檢查清單
- **輸出**:經過審查和測試的程式碼

**Phase 4: 建立評估**
- **目標**:驗證 MCP 伺服器的實用性
- **活動**:
  - 建立 10 個複雜的評估問題
  - 執行評估
  - 分析結果和回饋
  - 迭代改進
- **輸出**:評估結果報告和改進建議

**優點**:
- 結構化:每個階段有明確目標
- 可追蹤:容易確認進度
- 品質導向:多次審查和驗證
- 迭代改進:評估驅動優化

### 2. Agent 導向設計原則

**為工作流程建構,而非僅為 API 端點**:
```python
# ❌ API 導向 - 直接包裝端點
def check_availability(user_id, start_time, end_time):
    """檢查使用者可用性"""
    return calendar_api.check_availability(user_id, start_time, end_time)

def create_event(user_id, title, start_time, end_time):
    """建立日曆事件"""
    return calendar_api.create_event(user_id, title, start_time, end_time)

# Agent 需要呼叫兩個工具:
# 1. check_availability
# 2. create_event

# ✅ 工作流程導向 - 整合相關操作
def schedule_event(user_id, title, start_time, end_time):
    """安排事件(自動檢查可用性並建立)"""
    # 檢查可用性
    if not calendar_api.check_availability(user_id, start_time, end_time):
        return {"success": False, "reason": "Time slot not available",
                "suggestion": "Try using find_available_slots tool"}

    # 建立事件
    event = calendar_api.create_event(user_id, title, start_time, end_time)
    return {"success": True, "event": event}

# Agent 只需呼叫一個工具:schedule_event
```

**優點**:
- 減少工具呼叫次數
- 更符合人類思考方式
- 提高成功率

**優化有限 Context**:
```python
# ❌ 資訊過載
def get_user_info(user_id):
    return {
        "id": "U123456",
        "name": "John Doe",
        "email": "john@example.com",
        "phone": "+1-555-0123",
        "address": {"street": "123 Main St", "city": "...", ...},
        "preferences": {...},  # 100+ 欄位
        "metadata": {...},  # 各種時間戳、ID
        "profile_images": {  # 所有尺寸的圖片 URL
            "24": "https://...", "32": "https://...",
            "48": "https://...", "64": "https://...",
            # ... 10+ 種尺寸
        }
    }

# ✅ 高信號資訊
def get_user_info(user_id, detail_level="concise"):
    if detail_level == "concise":
        return {
            "name": "John Doe",
            "email": "john@example.com",
            "id": "U123456",
            "profile_image": "https://...64.jpg"  # 僅一個 URL
        }
    else:  # detail_level == "detailed"
        return {
            "id": "U123456",
            "name": "John Doe",
            "email": "john@example.com",
            "phone": "+1-555-0123",
            "created": "2024-01-15 10:30:00 UTC",  # 人類可讀
            "profile_image": "https://...64.jpg",
            "status": "active"
        }
```

**設計可操作的錯誤訊息**:
```python
# ❌ 不明確的錯誤
"Error: Too many results"

# ✅ 可操作的錯誤
"""
Error: Query returned 5,000 results, exceeding the limit.

💡 Suggestions:
- Use the 'status' filter to narrow results (e.g., status='active')
- Add a date range filter (e.g., created_after='2024-01-01')
- Increase pagination limit (current: 20, max: 100)

📋 Available filters: status, type, priority, assignee, created_after, created_before
"""
```

### 3. 評估驅動開發

**建立評估的時機**:
```
傳統開發流程:
設計 → 實作 → 測試 → 部署 → (發現問題) → 修改

評估驅動開發:
設計 → 建立評估問題 → 實作 → 執行評估 → 分析回饋 → 迭代改進 → 部署
```

**優點**:
- **早期發現問題**:在完成大部分實作前發現設計缺陷
- **客觀指標**:準確率、工具呼叫次數等可量化指標
- **Agent 回饋**:獲取真實 Agent 對工具的意見
- **持續改進**:評估結果指導優化方向

### 4. 語言抽象設計

**統一概念,不同實作**:

**Python 模式**:
```python
from mcp import FastMCP
from pydantic import BaseModel, Field

class UserInput(BaseModel):
    user_id: str = Field(..., min_length=1, description="User ID")
    detail_level: str = Field("concise", pattern="^(concise|detailed)$")

mcp = FastMCP("example-server")

@mcp.tool(
    annotations={
        "readOnlyHint": True,
        "openWorldHint": True
    }
)
async def get_user(user_id: str, detail_level: str = "concise") -> str:
    """Get user information.

    Args:
        user_id: The user's unique identifier
        detail_level: Level of detail (concise or detailed)
    """
    # 實作...
```

**TypeScript 模式**:
```typescript
import { z } from 'zod';

const UserInputSchema = z.object({
  user_id: z.string().min(1).describe("User ID"),
  detail_level: z.enum(["concise", "detailed"]).default("concise")
}).strict();

server.registerTool({
  name: "get_user",
  description: "Get user information",
  inputSchema: zodToJsonSchema(UserInputSchema),
  annotations: {
    readOnlyHint: true,
    openWorldHint: true
  },
  handler: async ({ user_id, detail_level }: z.infer<typeof UserInputSchema>): Promise<string> => {
    // 實作...
  }
});
```

**統一概念**:
- 輸入驗證(Pydantic vs Zod)
- 類型安全
- 工具註釋
- 文檔字串/描述

**不同實作**:
- 語言特定語法
- 框架特定模式
- 建置流程

### 5. 傳輸協定抽象

**設計理念:選擇合適的傳輸**

| 傳輸協定 | 適用場景 | 優點 | 缺點 |
|---------|---------|------|------|
| **stdio** | 本地工具、CLI | 簡單、無需網路配置 | 單一客戶端 |
| **HTTP** | 遠端服務、多客戶端 | 可擴展、標準協定 | 需要網路配置 |
| **SSE** | 即時更新、推送通知 | 即時性、持久連線 | 複雜度較高 |

**實作抽象**:
- connections.py 提供統一介面
- evaluation.py 支援所有傳輸協定
- 使用者只需選擇傳輸類型,無需修改程式碼

---

## Claude Code 使用方式

### 觸發時機

Claude Code 會在以下情況使用此 Skill:

1. **需要建立 MCP 伺服器**
   - 範例:「建立一個 Slack MCP 伺服器」
   - 範例:「為 GitHub API 建立 MCP 整合」
   - 範例:「開發一個 Notion MCP 伺服器」

2. **需要整合外部服務**
   - 範例:「讓 Claude 能夠存取 Jira」
   - 範例:「整合 Google Calendar API」
   - 範例:「建立 Stripe 支付整合」

3. **需要評估 MCP 伺服器**
   - 範例:「測試這個 MCP 伺服器」
   - 範例:「建立 MCP 伺服器評估問題」
   - 範例:「評估工具的效能」

### 典型工作流程

#### 工作流程 1:建立 Python MCP 伺服器

```
步驟 1:研究和規劃(Phase 1)
Claude:
1. 載入 SKILL.md 了解整體流程
2. 使用 WebFetch 載入 MCP 協定文檔
3. 載入 reference/mcp_best_practices.md
4. 載入 reference/python_mcp_server.md
5. 使用 WebFetch 和 WebSearch 研究目標 API(如 Slack API)
6. 建立工具清單和實作計畫

步驟 2:實作(Phase 2)
Claude:
1. 建立專案結構(單一 .py 檔案或模組)
2. 實作共用工具(API 請求、錯誤處理、格式化)
3. 使用 Pydantic 定義輸入 schema
4. 實作每個工具,包含:
   - 完整的 docstring
   - 輸入驗證
   - 錯誤處理
   - 回應格式化(JSON/Markdown)
   - 分頁支援
   - 字元限制檢查
5. 添加工具註釋

步驟 3:審查和改進(Phase 3)
Claude:
1. 程式碼品質審查:
   - 檢查 DRY 原則
   - 驗證一致性
   - 確認錯誤處理
   - 檢查類型安全
2. 測試:
   - python -m py_compile server.py
   - 檢查 import
3. 使用品質檢查清單驗證

步驟 4:建立評估(Phase 4)
Claude:
1. 載入 reference/evaluation.md
2. 使用 READ-ONLY 工具探索可用資料
3. 建立 10 個複雜問題
4. 自己解決問題以驗證答案
5. 建立 evaluation.xml 檔案
6. 執行評估:
   python scripts/evaluation.py -t stdio -c python -a server.py evaluation.xml
7. 分析結果和回饋
8. 根據回饋改進工具
```

#### 工作流程 2:建立 TypeScript MCP 伺服器

```
步驟 1:研究和規劃(Phase 1)
Claude:
1. 載入 SKILL.md
2. 使用 WebFetch 載入 MCP 協定文檔
3. 載入 reference/mcp_best_practices.md
4. 載入 reference/node_mcp_server.md
5. 研究目標 API
6. 建立實作計畫

步驟 2:實作(Phase 2)
Claude:
1. 建立專案結構:
   - package.json
   - tsconfig.json
   - src/index.ts
2. 安裝依賴:
   npm install @modelcontextprotocol/sdk zod
3. 實作共用工具
4. 使用 Zod 定義 schema
5. 使用 server.registerTool 註冊工具
6. 實作每個工具

步驟 3:審查和改進(Phase 3)
Claude:
1. 程式碼品質審查
2. 建置測試:
   npm run build
3. 驗證 dist/index.js 生成
4. 使用品質檢查清單

步驟 4:建立評估(Phase 4)
Claude:
1. 建立評估問題
2. 執行評估:
   python scripts/evaluation.py -t stdio -c node -a dist/index.js evaluation.xml
3. 分析和改進
```

#### 工作流程 3:評估現有 MCP 伺服器

```
步驟 1:準備
Claude:
1. 載入 reference/evaluation.md
2. 連接到 MCP 伺服器
3. 列出可用工具

步驟 2:探索
Claude:
1. 使用 READ-ONLY 工具探索資料
2. 了解資料結構和可用資訊

步驟 3:建立問題
Claude:
1. 基於實際資料建立 10 個問題
2. 確保問題:
   - 獨立
   - 唯讀
   - 複雜(多次工具呼叫)
   - 現實
   - 可驗證
   - 穩定

步驟 4:驗證答案
Claude:
1. 自己使用工具解決每個問題
2. 記錄答案
3. 建立 evaluation.xml

步驟 5:執行評估
Claude:
1. 執行 evaluation.py
2. 檢查結果報告
3. 分析 Agent 回饋
4. 識別改進機會
```

### 不使用此 Skill 的情況

- 建立簡單的腳本或工具(非 MCP)
- 直接呼叫 API(不需要 LLM 整合)
- 建立非 LLM 的應用程式
- 簡單的資料處理任務

---

## 技術架構

### 技術堆疊

**Python 生態系**:
```
MCP Python SDK     # MCP 伺服器框架
FastMCP            # 快速 MCP 開發
Pydantic v2        # 資料驗證和 schema
anthropic          # Claude API 客戶端
asyncio            # 非同步 I/O
defusedxml         # 安全的 XML 解析(評估腳本)
```

**Node/TypeScript 生態系**:
```
@modelcontextprotocol/sdk   # MCP SDK
zod                         # Schema 驗證
TypeScript 5.x              # 類型安全
```

**評估框架**:
```
Python 3.8+        # 執行環境
anthropic          # Claude API
mcp                # MCP Python SDK
xml.etree          # XML 解析
```

### 依賴關係圖

```
MCP Builder Skill
├── SKILL.md (主要指南)
│   ├── Phase 1: 研究
│   │   ├── WebFetch → MCP 協定文檔
│   │   ├── WebSearch → API 文檔
│   │   └── reference/ → 最佳實踐
│   ├── Phase 2: 實作
│   │   ├── reference/python_mcp_server.md
│   │   ├── reference/node_mcp_server.md
│   │   └── reference/mcp_best_practices.md
│   ├── Phase 3: 審查
│   │   └── 品質檢查清單
│   └── Phase 4: 評估
│       ├── reference/evaluation.md
│       └── scripts/evaluation.py
├── scripts/
│   ├── evaluation.py
│   │   ├── anthropic (Claude API)
│   │   ├── mcp (MCP SDK)
│   │   ├── connections.py
│   │   └── xml.etree
│   ├── connections.py
│   │   └── mcp (MCP SDK)
│   └── requirements.txt
└── reference/
    ├── mcp_best_practices.md (通用指南)
    ├── python_mcp_server.md (Python 特定)
    ├── node_mcp_server.md (TypeScript 特定)
    └── evaluation.md (評估指南)
```

### 評估流程架構

```
評估流程
    ↓
[載入 evaluation.xml]
    ↓
[解析問答對]
    ↓
[建立 MCP 連線] (stdio/HTTP/SSE)
    ↓
[列出可用工具]
    ↓
For 每個問題:
    ↓
[Agent 迴圈]
    ├─ 呼叫 Claude API (傳入工具)
    ├─ Claude 決定使用工具
    ├─ 執行工具 (透過 MCP 連線)
    ├─ 回傳結果給 Claude
    ├─ 重複直到獲得答案
    └─ 提取 <response>, <summary>, <feedback>
    ↓
[比較預期 vs 實際答案]
    ↓
[計算指標]
    ↓
[生成報告]
```

### MCP 連線架構

```
MCPConnection (抽象基類)
    ├── list_tools()
    ├── call_tool()
    └── 子類別:
        ├── MCPConnectionStdio
        │   └── stdio_client → 本地子行程
        ├── MCPConnectionSSE
        │   └── sse_client → 遠端伺服器 (SSE)
        └── MCPConnectionHTTP
            └── streamablehttp_client → 遠端伺服器 (HTTP)
```

---

## 使用的 Prompt 策略

### 1. 階段式指導

**SKILL.md 的結構**:
```markdown
## 🚀 High-Level Workflow

### Phase 1: Deep Research and Planning
#### 1.1 Understand Agent-Centric Design Principles
#### 1.2 Study MCP Protocol Documentation
#### 1.3 Study Framework Documentation
...

### Phase 2: Implementation
#### 2.1 Set Up Project Structure
#### 2.2 Implement Core Infrastructure First
...

### Phase 3: Review and Refine
...

### Phase 4: Create Evaluations
...
```

**優點**:
- 清晰的進度指示
- 防止跳過重要步驟
- 確保完整的開發週期

### 2. 強制完整讀取策略

**關鍵文件的載入指示**:
```markdown
**Load and read the following reference files:**
- 📋 View Best Practices
- Use WebFetch to load `https://modelcontextprotocol.io/llms-full.txt`
```

**設計理念**:
- 確保 Claude 了解完整的規範
- 避免基於部分資訊做決策
- 提供最新的協定文檔

### 3. 條件式文檔載入

**根據語言載入對應指南**:
```markdown
**For Python implementations, also load:**
- Python SDK Documentation
- 🐍 Python Implementation Guide

**For Node/TypeScript implementations, also load:**
- TypeScript SDK Documentation
- ⚡ TypeScript Implementation Guide
```

**優點**:
- 減少不相關資訊
- 保持 context 整潔
- 提供語言特定的最佳實踐

### 4. 範例驅動學習

**每個指南都包含完整範例**:
```markdown
## Example

Here's a complete example of a Python MCP server:

```python
# 完整可執行的程式碼
from mcp import FastMCP
...
```
```

**優點**:
- 具體展示正確模式
- 減少歧義
- 可直接參考或修改

### 5. 品質檢查清單

**Python 品質檢查清單**(從 python_mcp_server.md):
```markdown
## Quality Checklist

Before finalizing your MCP server:

### Code Quality
- [ ] All tools have clear, descriptive names with service prefix
- [ ] Input schemas use Pydantic v2 models with proper constraints
- [ ] All functions have comprehensive docstrings
- [ ] Type hints are used throughout
- [ ] No code duplication between tools
...

### Testing
- [ ] Server syntax is valid: `python -m py_compile your_server.py`
- [ ] All imports are available
...
```

**優點**:
- 系統化的品質驗證
- 防止常見錯誤
- 確保一致性

### 6. 評估提示詞設計

**結構化輸出要求**:
```python
EVALUATION_PROMPT = """
When given a task, you MUST:
1. Use the available tools to complete the task
2. Provide summary of each step in your approach, wrapped in <summary> tags
3. Provide feedback on the tools provided, wrapped in <feedback> tags
4. Provide your final response, wrapped in <response> tags
"""
```

**設計理念**:
- **可解析性**:XML 標籤確保能夠提取資訊
- **完整性**:要求步驟、回饋和答案
- **可操作性**:回饋指導改進
- **標準化**:一致的格式便於自動化處理

### 7. 錯誤預防提示

**測試警告**:
```markdown
**Important:** MCP servers are long-running processes that wait for requests.
Running them directly will cause your process to hang indefinitely.

**Safe ways to test:**
- Use the evaluation harness (recommended)
- Run the server in tmux
- Use a timeout: `timeout 5s python server.py`
```

**優點**:
- 預防常見錯誤
- 提供替代方案
- 解釋原因

---

## 使用的工具(Tools)

此 Skill 主要依賴以下 Claude Code 工具:

### 1. **Read Tool**
- 讀取 SKILL.md(主要流程)
- 讀取 reference/ 目錄的所有指南
- 讀取 scripts/ 目錄的腳本
- 檢查生成的程式碼

### 2. **Write Tool**
- 建立 MCP 伺服器程式碼(.py 或 .ts)
- 建立 package.json, tsconfig.json(TypeScript)
- 建立 evaluation.xml
- 建立共用工具模組

### 3. **WebFetch Tool**
- 載入 MCP 協定文檔
- 載入 Python SDK README
- 載入 TypeScript SDK README
- 讀取 API 文檔

### 4. **WebSearch Tool**
- 搜尋 API 文檔
- 搜尋最佳實踐
- 搜尋程式庫使用範例
- 研究認證方法

### 5. **Bash Tool**
- 執行 Python 語法檢查:`python -m py_compile`
- 執行 TypeScript 建置:`npm run build`
- 執行評估:`python scripts/evaluation.py`
- 安裝依賴:`pip install` 或 `npm install`

### 6. **Edit Tool**
- 修改現有 MCP 伺服器程式碼
- 調整工具實作
- 更新配置檔

---

## 最佳實踐

### 1. MCP 伺服器開發最佳實踐

**工具命名**:
```python
# ✅ 好的命名
slack_send_message
github_create_issue
notion_create_page
stripe_create_payment

# ❌ 不好的命名
send_message        # 缺少服務前綴
createIssue         # 不是 snake_case
gh_issue            # 過於簡短/不清楚
```

**輸入驗證**:
```python
# ✅ 完整的驗證
class SendMessageInput(BaseModel):
    channel_id: str = Field(
        ...,
        min_length=1,
        max_length=100,
        pattern=r"^C[A-Z0-9]{10}$",
        description="Slack channel ID (e.g., C1234567890)",
        examples=["C1234567890", "CABCDEF123"]
    )
    text: str = Field(
        ...,
        min_length=1,
        max_length=4000,
        description="Message text to send"
    )
    thread_ts: str | None = Field(
        None,
        pattern=r"^\d+\.\d+$",
        description="Thread timestamp for replies (optional)"
    )

# ❌ 不足的驗證
class SendMessageInput(BaseModel):
    channel_id: str
    text: str
```

**錯誤處理**:
```python
# ✅ 詳細且可操作的錯誤
async def slack_send_message(channel_id: str, text: str) -> str:
    try:
        result = await slack_api.send_message(channel_id, text)
        return f"✅ Message sent to channel {channel_id}"
    except ChannelNotFoundError:
        return """
❌ Error: Channel not found

Channel ID '{channel_id}' does not exist or the bot doesn't have access.

💡 Suggestions:
- Use slack_list_channels to find available channels
- Check if the bot has been added to this channel
- Verify the channel ID format (should start with 'C')
        """
    except RateLimitError as e:
        return f"""
❌ Error: Rate limit exceeded

You can retry after {e.retry_after} seconds.

💡 Suggestion: Use a delay between messages to avoid rate limits.
        """
    except Exception as e:
        return f"❌ Error: {str(e)}"

# ❌ 不明確的錯誤
async def slack_send_message(channel_id: str, text: str) -> str:
    result = await slack_api.send_message(channel_id, text)
    return str(result)
```

**回應格式化**:
```python
# ✅ 支援兩種格式
async def slack_list_messages(
    channel_id: str,
    limit: int = 20,
    response_format: str = "markdown"
) -> str:
    messages = await slack_api.get_messages(channel_id, limit)

    if response_format == "json":
        return json.dumps({
            "messages": [
                {
                    "ts": m.ts,
                    "user": m.user,
                    "text": m.text,
                    "timestamp": m.timestamp
                }
                for m in messages
            ],
            "count": len(messages)
        })
    else:  # markdown
        result = f"## Messages in {channel_id} ({len(messages)} messages)\n\n"
        for m in messages:
            user_name = await get_user_name(m.user)
            time_str = format_timestamp(m.timestamp)
            result += f"**{user_name}** ({time_str})\n{m.text}\n\n"
        return result

# ❌ 僅單一格式
async def slack_list_messages(channel_id: str, limit: int = 20) -> dict:
    return await slack_api.get_messages(channel_id, limit)
```

### 2. 評估創建最佳實踐

**好的評估問題**:
```xml
<!-- ✅ 好的問題 -->
<qa_pair>
  <question>Find the Slack channel where the most recent discussion about the Q4 roadmap took place. What was the channel name?</question>
  <answer>product-planning</answer>
</qa_pair>

特點:
- 複雜:需要搜尋多個頻道,比較時間戳
- 現實:實際工作場景
- 可驗證:單一明確答案
- 穩定:頻道名稱不會改變
- 唯讀:只需查詢操作

<!-- ❌ 不好的問題 -->
<qa_pair>
  <question>What channels exist?</question>
  <answer>...</answer>
</qa_pair>

問題:
- 太簡單:只需一次工具呼叫
- 不穩定:頻道清單可能改變
- 答案太長:難以驗證
```

**答案驗證**:
```python
# ✅ 自己解決問題驗證答案
問題: Find the user who posted the most messages in #general last week. What is their user ID?

驗證步驟:
1. 使用 slack_list_messages(channel="general", ...) 獲取訊息
2. 篩選上週的訊息
3. 統計每個使用者的訊息數
4. 找出最多的使用者
5. 記錄 user_id: U1234567890

答案: U1234567890

# ❌ 未驗證就猜測答案
答案: U1234567890  # 未實際執行查詢
```

### 3. 程式碼品質最佳實踐

**DRY 原則**:
```python
# ✅ 提取共用邏輯
async def _make_api_request(endpoint: str, method: str = "GET", data: dict = None) -> dict:
    """共用的 API 請求函數"""
    try:
        response = await api_client.request(method, endpoint, json=data)
        response.raise_for_status()
        return response.json()
    except HTTPError as e:
        if e.status_code == 429:
            raise RateLimitError(e.headers.get("Retry-After"))
        elif e.status_code == 404:
            raise NotFoundError(f"{endpoint} not found")
        else:
            raise

async def slack_get_channel(channel_id: str) -> str:
    data = await _make_api_request(f"/channels/{channel_id}")
    return format_channel(data)

async def slack_get_user(user_id: str) -> str:
    data = await _make_api_request(f"/users/{user_id}")
    return format_user(data)

# ❌ 重複的程式碼
async def slack_get_channel(channel_id: str) -> str:
    try:
        response = await api_client.request("GET", f"/channels/{channel_id}")
        response.raise_for_status()
        data = response.json()
        return format_channel(data)
    except HTTPError as e:
        if e.status_code == 429:
            raise RateLimitError(...)
        # 重複的錯誤處理

async def slack_get_user(user_id: str) -> str:
    try:
        response = await api_client.request("GET", f"/users/{user_id}")
        response.raise_for_status()
        data = response.json()
        return format_user(data)
    except HTTPError as e:
        if e.status_code == 429:
            raise RateLimitError(...)
        # 重複的錯誤處理
```

**一致性**:
```python
# ✅ 一致的命名和模式
async def slack_list_channels(...) -> str:
    # 列出頻道

async def slack_list_users(...) -> str:
    # 列出使用者

async def slack_list_messages(...) -> str:
    # 列出訊息

# 所有 list_ 函數都:
# - 接受 limit 和 offset 參數
# - 支援 response_format 參數
# - 回傳分頁元資料
# - 使用相同的錯誤處理

# ❌ 不一致
async def slack_list_channels(...) -> str:
    # 使用 limit/offset

async def slack_get_all_users(...) -> list:
    # 不支援分頁,不同的回傳類型

async def slack_messages(...) -> dict:
    # 不同的命名模式,不同的回傳類型
```

### 4. 安全性最佳實踐

**輸入驗證**:
```python
# ✅ 防止路徑遍歷
def read_file(file_path: str) -> str:
    # 驗證路徑
    safe_path = Path(file_path).resolve()
    allowed_dir = Path("/allowed/directory").resolve()

    if not safe_path.is_relative_to(allowed_dir):
        raise ValueError(f"Access denied: {file_path} is outside allowed directory")

    if not safe_path.exists():
        raise FileNotFoundError(f"File not found: {file_path}")

    return safe_path.read_text()

# ❌ 不安全
def read_file(file_path: str) -> str:
    return Path(file_path).read_text()  # 可能讀取任意檔案
```

**API 金鑰管理**:
```python
# ✅ 從環境變數讀取
import os

API_KEY = os.getenv("SLACK_API_KEY")
if not API_KEY:
    raise ValueError("SLACK_API_KEY environment variable is required")

# ❌ 硬編碼
API_KEY = "xoxb-1234567890-abcdefghij"  # 永遠不要這樣做!
```

### 5. 評估執行最佳實踐

**測試 MCP 伺服器**:
```bash
# ✅ 使用評估框架(推薦)
python scripts/evaluation.py -t stdio -c python -a my_server.py eval.xml -o report.md

# ✅ 使用 tmux(手動測試)
# Terminal 1
tmux
python my_server.py

# Terminal 2
python test_client.py

# ✅ 使用 timeout(快速檢查)
timeout 5s python my_server.py
# 應該會 timeout,表示伺服器正在執行

# ❌ 直接執行(會卡住)
python my_server.py  # 主行程會無限等待
```

---

## 總結

### 成功要素

1. **結構化流程**:四階段開發確保品質和完整性
2. **Agent 導向設計**:工具設計考慮 AI Agent 的實際需求
3. **評估驅動**:自動化評估提供客觀反饋
4. **最佳實踐內建**:涵蓋命名、格式、安全性等各方面
5. **語言抽象**:支援 Python 和 TypeScript,統一概念
6. **完整文檔**:詳盡的指南和範例

### 學習重點

對於想要建立 MCP 伺服器的開發者:
- MCP 協定的運作方式
- Agent 導向設計 vs API 導向設計的差異
- 如何設計有效的工具(命名、參數、回應)
- 評估的重要性和創建方法
- 安全性和最佳實踐
- Python 和 TypeScript 的實作模式

### 適用場景

此 Skill 最適合用於:
- 建立 Slack MCP 伺服器
- 建立 GitHub MCP 伺服器
- 建立 Notion MCP 伺服器
- 建立 Google Calendar MCP 伺服器
- 建立 Jira MCP 伺服器
- 建立任何 SaaS API 的 MCP 整合
- 評估和改進現有 MCP 伺服器

### 技術亮點

- **評估框架**:自動化測試 MCP 伺服器效能
- **連線抽象**:統一介面支援多種傳輸協定
- **Agent 回饋**:從真實 Agent 獲取改進建議
- **四階段流程**:系統化的開發方法論
- **雙語言支援**:Python 和 TypeScript 統一概念
- **完整文檔**:涵蓋所有面向的詳盡指南

### 企業價值

**提高開發效率**:
- 結構化流程減少試錯時間
- 內建最佳實踐避免常見錯誤
- 完整範例加速學習

**確保品質**:
- 評估框架提供客觀指標
- 品質檢查清單防止疏漏
- Agent 回饋指導改進

**降低風險**:
- 安全性最佳實踐內建
- 完整的錯誤處理指導
- 輸入驗證範例

---

## 參考資源

- [SKILL.md](./SKILL.md) - 英文原始文檔
- [reference/mcp_best_practices.md](./reference/mcp_best_practices.md) - MCP 最佳實踐
- [reference/python_mcp_server.md](./reference/python_mcp_server.md) - Python 實作指南
- [reference/node_mcp_server.md](./reference/node_mcp_server.md) - Node/TypeScript 實作指南
- [reference/evaluation.md](./reference/evaluation.md) - 評估創建指南
- [MCP 協定文檔](https://modelcontextprotocol.io/llms-full.txt)
- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)
- [MCP TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)

---

**最後更新**:2025-11-15
**文件版本**:1.0.0
**適用於**:Claude Code Skills
