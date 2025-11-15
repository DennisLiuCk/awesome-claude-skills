# Web Application Testing - 正體中文說明文件

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
Web Application Testing 是一個專門用於測試本地網頁應用程式的工具套件,使用 Playwright 自動化瀏覽器操作。它提供伺服器生命週期管理、DOM 檢查、螢幕截圖、控制台日誌捕獲等功能,讓 Claude 能夠驗證前端功能、除錯 UI 行為和自動化測試流程。

主要功能包括:
- 自動化伺服器啟動和清理(支援多伺服器)
- Playwright 自動化腳本範例
- DOM 元素探索和選擇器識別
- 螢幕截圖捕獲
- 控制台日誌監控
- 支援靜態 HTML 和動態網頁應用

### 解決的問題
在測試網頁應用時,經常遇到以下挑戰:

1. **伺服器管理困難**:需要手動啟動/停止開發伺服器
2. **多伺服器協調**:前後端分離時需同時運行多個伺服器
3. **選擇器識別困難**:不知道如何找到正確的 DOM 元素
4. **動態內容等待**:JavaScript 渲染的內容需要等待載入
5. **除錯困難**:無法看到瀏覽器狀態和錯誤
6. **重複性手動測試**:每次都要手動點擊和驗證

Web Application Testing 提供完整的自動化解決方案,讓測試變得可靠和高效。

---

## 目錄結構

```
webapp-testing/
├── SKILL.md                        # Skill 主要說明文件(英文)
├── LICENSE.txt                     # 授權條款
├── README.zh-TW.md                # 正體中文說明文件(本文件)
├── scripts/                        # 輔助腳本目錄
│   └── with_server.py             # 伺服器生命週期管理
└── examples/                       # 範例腳本目錄
    ├── element_discovery.py       # 元素探索範例
    ├── static_html_automation.py  # 靜態 HTML 自動化
    └── console_logging.py         # 控制台日誌捕獲
```

---

## 檔案說明

### 核心檔案

#### 1. `SKILL.md`(96 行)
- **用途**:網頁應用測試指南
- **內容**:
  - 決策樹(選擇測試方法)
  - with_server.py 使用說明
  - 偵察-行動模式(Reconnaissance-Then-Action)
  - 常見陷阱和最佳實踐
  - 參考檔案說明
- **特點**:強調「黑箱」使用腳本,避免污染 context

#### 2. `LICENSE.txt`
- **用途**:定義 Skill 的使用授權條款
- **重要性**:確保使用者了解使用限制和權利

#### 3. `README.zh-TW.md`(本文件)
- **用途**:為繁體中文使用者提供詳細說明
- **內容**:完整的架構分析和使用指南

### Scripts 目錄

#### 1. `with_server.py`(106 行)
**功能**:管理伺服器生命週期並執行測試腳本

**主要功能**:
- 啟動單一或多個伺服器
- 等待伺服器就緒(port 檢查)
- 執行測試命令
- 自動清理伺服器行程

**核心機制**:

**伺服器就緒檢查**:
```python
def is_server_ready(port, timeout=30):
    """輪詢 port 直到連線成功"""
    start_time = time.time()
    while time.time() - start_time < timeout:
        try:
            with socket.create_connection(('localhost', port), timeout=1):
                return True  # 連線成功,伺服器就緒
        except (socket.error, ConnectionRefusedError):
            time.sleep(0.5)  # 等待 0.5 秒後重試
    return False  # 超時
```

**生命週期管理**:
```python
try:
    # 1. 啟動所有伺服器
    for server in servers:
        process = subprocess.Popen(server['cmd'], shell=True, ...)
        server_processes.append(process)
        wait_for_ready(server['port'])

    # 2. 執行測試命令
    result = subprocess.run(command)

finally:
    # 3. 清理所有伺服器(即使測試失敗)
    for process in server_processes:
        process.terminate()
        process.wait(timeout=5)
```

**命令列介面**:

**單一伺服器**:
```bash
python scripts/with_server.py \
  --server "npm run dev" \
  --port 5173 \
  -- python automation.py
```

**多伺服器(前後端)**:
```bash
python scripts/with_server.py \
  --server "cd backend && python server.py" --port 3000 \
  --server "cd frontend && npm run dev" --port 5173 \
  -- python test.py
```

**設計亮點**:
- **可靠清理**:使用 finally 確保伺服器停止
- **超時保護**:防止無限等待
- **支援多伺服器**:可重複 `--server` 和 `--port`
- **Shell 支援**:可執行 `cd && command` 等複雜命令

### Examples 目錄

#### 1. `element_discovery.py`(40 行)
**功能**:探索頁面上的所有元素

**主要操作**:
```python
# 1. 導航並等待載入
page.goto('http://localhost:5173')
page.wait_for_load_state('networkidle')

# 2. 探索按鈕
buttons = page.locator('button').all()
for button in buttons:
    text = button.inner_text()
    print(f"Button: {text}")

# 3. 探索連結
links = page.locator('a[href]').all()
for link in links:
    text = link.inner_text()
    href = link.get_attribute('href')
    print(f"Link: {text} -> {href}")

# 4. 探索輸入欄位
inputs = page.locator('input, textarea, select').all()
for input_elem in inputs:
    name = input_elem.get_attribute('name')
    type = input_elem.get_attribute('type')
    print(f"Input: {name} ({type})")

# 5. 截圖
page.screenshot(path='/tmp/page_discovery.png', full_page=True)
```

**使用場景**:
- 不熟悉頁面結構時
- 識別可用的互動元素
- 找出正確的選擇器

#### 2. `static_html_automation.py`
**功能**:測試本地 HTML 檔案

**關鍵特點**:
- 使用 `file://` 協議
- 不需要伺服器
- 直接讀取 HTML 檔案找選擇器

**使用場景**:
- 測試純靜態 HTML
- 快速原型驗證
- 無後端的頁面

#### 3. `console_logging.py`
**功能**:捕獲瀏覽器控制台日誌

**實作方式**:
```python
# 監聽控制台訊息
page.on('console', lambda msg: print(f"[{msg.type}] {msg.text}"))

# 執行操作
page.goto('http://localhost:5173')
page.click('button#submit')

# 所有 console.log, console.error 都會被捕獲
```

**使用場景**:
- 除錯 JavaScript 錯誤
- 監控應用日誌
- 驗證特定日誌輸出

---

## 關鍵腳本分析

### with_server.py 深度分析

#### 1. Port 檢查機制

**為何使用 Socket 而非 HTTP 請求**:
```python
# ✅ Socket 連線(快速、可靠)
with socket.create_connection(('localhost', port), timeout=1):
    return True

# ❌ HTTP 請求(較慢、可能 404)
response = requests.get(f'http://localhost:{port}')
```

**優點**:
- **快速**:只檢查 port 是否監聽
- **可靠**:不依賴特定路徑
- **通用**:適用於任何 HTTP/非 HTTP 伺服器

#### 2. 多伺服器支援設計

**參數解析**:
```python
parser.add_argument('--server', action='append', dest='servers')
parser.add_argument('--port', action='append', dest='ports')

# 使用者執行:
# --server "cmd1" --port 3000 --server "cmd2" --port 5173
# 結果:
# servers = ["cmd1", "cmd2"]
# ports = [3000, 5173]
```

**順序啟動策略**:
```python
# 順序啟動,等待每個伺服器就緒
for server in servers:
    start_server(server)
    wait_for_ready(server.port)  # 確保就緒才啟動下一個
```

**原因**:
- 避免 port 衝突
- 確保依賴順序(如後端先啟動)
- 提供清晰的錯誤訊息

#### 3. 清理保證機制

**finally 塊**:
```python
try:
    # 啟動伺服器和執行測試
    ...
finally:
    # 無論如何都會執行
    for process in server_processes:
        process.terminate()
        process.wait(timeout=5)
        if process.is_alive():
            process.kill()  # 強制終止
```

**保證**:
- 即使測試失敗,伺服器也會停止
- 即使 Python 例外,清理仍會執行
- 避免殘留行程

### Playwright 模式分析

#### 偵察-行動模式(Reconnaissance-Then-Action)

**步驟 1:偵察(Reconnaissance)**
```python
page.goto('http://localhost:5173')
page.wait_for_load_state('networkidle')  # 關鍵!

# 偵察 DOM
content = page.content()  # 取得完整 HTML
buttons = page.locator('button').all()  # 找所有按鈕
page.screenshot(path='/tmp/inspect.png')  # 視覺檢查
```

**步驟 2:識別選擇器**
```python
# 從偵察結果識別:
# - 按鈕文字: "Submit"
# - 按鈕 ID: "submit-btn"
# - 輸入欄位名稱: "email"
```

**步驟 3:行動(Action)**
```python
# 使用發現的選擇器執行操作
page.fill('input[name="email"]', 'test@example.com')
page.click('button#submit-btn')
page.wait_for_selector('.success-message')
```

**為何這樣設計**:
- **動態內容**:JavaScript 渲染的元素需要時間載入
- **避免競態條件**:確保元素存在才操作
- **除錯友善**:截圖幫助理解頁面狀態

---

## 設計策略

### 1. 決策樹導向

**SKILL.md 提供清晰的決策路徑**:
```
用戶任務 → 是靜態 HTML 嗎?
    ├─ 是 → 直接讀取 HTML 找選擇器
    │       ├─ 成功 → 寫 Playwright 腳本
    │       └─ 失敗 → 當作動態處理
    │
    └─ 否(動態應用) → 伺服器已運行?
        ├─ 否 → 使用 with_server.py
        └─ 是 → 使用偵察-行動模式
```

**優點**:
- 快速決策
- 避免不必要的複雜性
- 適應不同場景

### 2. 黑箱使用策略

**SKILL.md 強調**:
> "Always run scripts with `--help` first. DO NOT read the source until you try running the script first..."

**設計理念**:
- **保護 context**:避免載入大型腳本
- **抽象化**:使用者只需知道介面
- **可靠性**:經過測試的黑箱工具

**實踐**:
```bash
# ✅ 好的做法
python scripts/with_server.py --help
# 了解用法後直接使用

# ❌ 避免
# 讀取 with_server.py 原始碼理解實作
```

### 3. 範例驅動學習

**提供三類範例**:
1. **元素探索**(element_discovery.py):學習如何找元素
2. **靜態自動化**(static_html_automation.py):學習簡單場景
3. **日誌捕獲**(console_logging.py):學習除錯技巧

**優點**:
- 具體展示模式
- 可直接複製修改
- 涵蓋常見需求

### 4. 關鍵等待強調

**SKILL.md 標記為 CRITICAL**:
```python
page.wait_for_load_state('networkidle')  # CRITICAL: Wait for JS to execute
```

**常見陷阱**:
```python
# ❌ 錯誤:不等待就檢查
page.goto('http://localhost:5173')
content = page.content()  # 可能只有骨架,無動態內容

# ✅ 正確:等待 JavaScript 執行完成
page.goto('http://localhost:5173')
page.wait_for_load_state('networkidle')
content = page.content()  # 完整渲染的內容
```

---

## Claude Code 使用方式

### 觸發時機

Claude Code 會在以下情況使用此 Skill:

1. **需要測試網頁應用**
   - 範例:「測試這個 React 應用的登入功能」
   - 範例:「驗證表單提交是否正常工作」
   - 範例:「檢查按鈕點擊後的行為」

2. **需要自動化瀏覽器操作**
   - 範例:「自動填寫這個表單並提交」
   - 範例:「截圖這個頁面的所有狀態」
   - 範例:「檢查這個網站的所有連結」

3. **需要除錯前端問題**
   - 範例:「為何這個按鈕點不到?」
   - 範例:「查看控制台有什麼錯誤」
   - 範例:「截圖看看頁面實際長什麼樣」

### 典型工作流程

#### 工作流程 1:測試動態網頁應用

```bash
步驟 1:確認伺服器狀態
使用者:「測試我的 React 應用」

Claude:「伺服器目前運行嗎?」
使用者:「沒有,需要 npm run dev」

步驟 2:使用 with_server.py
Claude 執行:
python scripts/with_server.py --help
# 了解用法

步驟 3:建立測試腳本
Claude 建立 test_app.py:
```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()

    # 導航並等待
    page.goto('http://localhost:5173')
    page.wait_for_load_state('networkidle')

    # 偵察:找出所有元素
    page.screenshot(path='/tmp/inspect.png', full_page=True)
    buttons = page.locator('button').all()
    print(f"Found {len(buttons)} buttons")

    # 執行測試
    page.fill('input[name="email"]', 'test@example.com')
    page.fill('input[name="password"]', 'password123')
    page.click('button[type="submit"]')

    # 驗證
    page.wait_for_selector('.success-message')
    print("Login successful!")

    browser.close()
```

步驟 4:執行測試
```bash
python scripts/with_server.py \
  --server "npm run dev" \
  --port 5173 \
  -- python test_app.py
```

輸出:
```
Starting server 1/1: npm run dev
Waiting for server on port 5173...
Server ready on port 5173

All 1 server(s) ready
Running: python test_app.py

Found 3 buttons
Login successful!

Stopping 1 server(s)...
Server 1 stopped
```
```

#### 工作流程 2:測試靜態 HTML

```
使用者:「測試這個 index.html 檔案」

Claude:
步驟 1:讀取 HTML 找選擇器
# 讀取 index.html
# 識別: <button id="submit">Submit</button>

步驟 2:建立測試腳本
```python
from playwright.sync_api import sync_playwright
from pathlib import Path

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()

    # 使用 file:// 協議
    html_path = Path('index.html').absolute()
    page.goto(f'file://{html_path}')

    # 靜態 HTML 通常不需要等待 networkidle
    # 但等待也無妨
    page.wait_for_load_state('load')

    # 執行測試
    page.fill('#email', 'test@example.com')
    page.click('#submit')

    # 截圖
    page.screenshot(path='/tmp/result.png')

    browser.close()
```

步驟 3:執行(不需要伺服器)
```bash
python test_static.py
```
```

#### 工作流程 3:除錯前端問題

```
使用者:「為何登入按鈕點不到?」

Claude:
步驟 1:偵察頁面狀態
```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()

    # 導航
    page.goto('http://localhost:5173')
    page.wait_for_load_state('networkidle')

    # 截圖看實際狀態
    page.screenshot(path='/tmp/debug.png', full_page=True)

    # 找所有按鈕
    buttons = page.locator('button').all()
    print("All buttons:")
    for i, btn in enumerate(buttons):
        visible = btn.is_visible()
        enabled = btn.is_enabled()
        text = btn.inner_text() if visible else "[hidden]"
        print(f"  [{i}] {text} - visible:{visible}, enabled:{enabled}")

    # 嘗試點擊
    try:
        page.click('button#login', timeout=5000)
        print("Click successful")
    except Exception as e:
        print(f"Click failed: {e}")

    browser.close()
```

輸出分析:
```
All buttons:
  [0] Login - visible:False, enabled:True
  [1] Cancel - visible:True, enabled:True

Click failed: Timeout 5000ms exceeded.
Element is not visible
```

診斷:登入按鈕存在但不可見(可能被 CSS 隱藏或在 modal 內)
```
```

### 不使用此 Skill 的情況

- 測試後端 API(使用 HTTP 客戶端)
- 測試行動應用(使用 Appium)
- 單元測試(使用 pytest/jest)

---

## 技術架構

### 技術堆疊

**Python 生態系**:
```
Playwright          # 瀏覽器自動化
subprocess          # 行程管理
socket              # Port 檢查
```

**支援的瀏覽器**:
```
Chromium (推薦)
Firefox
WebKit
```

### 依賴關係圖

```
with_server.py
├── subprocess (啟動伺服器)
├── socket (檢查 port)
└── time (等待和超時)

範例腳本
├── playwright.sync_api
├── chromium.launch()
└── page 操作
```

### 測試流程架構

```
[with_server.py 管理]
    ↓
啟動伺服器
    ↓
等待就緒(port 檢查)
    ↓
[Playwright 腳本執行]
    ├─ 啟動瀏覽器(headless)
    ├─ 導航到頁面
    ├─ 等待載入(networkidle)
    ├─ 偵察(screenshot, locator)
    ├─ 執行操作(fill, click)
    ├─ 驗證結果(wait_for_selector)
    └─ 關閉瀏覽器
    ↓
[with_server.py 清理]
    ↓
停止伺服器
```

---

## 使用的 Prompt 策略

### 1. 黑箱使用強調

**SKILL.md 明確指示**:
```markdown
**Always run scripts with `--help` first**
DO NOT read the source until you try running the script first...
```

**設計理念**:
- 避免 context 污染
- 鼓勵工具化使用
- 提供清晰介面

### 2. 決策樹導向

**視覺化決策流程**:
```
User task → Is it static HTML?
    ├─ Yes → ...
    └─ No → Is server running?
        ├─ No → ...
        └─ Yes → ...
```

**優點**:
- 快速定位方法
- 避免過度複雜化
- 適應不同場景

### 3. 關鍵步驟標記

**使用 CRITICAL 標記**:
```python
page.wait_for_load_state('networkidle')  # CRITICAL: Wait for JS to execute
```

**效果**:
- 引起注意
- 防止常見錯誤
- 強調重要性

### 4. 範例驅動

**提供三個範例**:
- element_discovery.py
- static_html_automation.py
- console_logging.py

**優點**:
- 具體展示模式
- 可直接複製修改
- 涵蓋常見需求

---

## 使用的工具(Tools)

此 Skill 主要依賴以下 Claude Code 工具:

### 1. **Write Tool**
- 建立 Playwright 測試腳本
- 建立自動化腳本

### 2. **Bash Tool**
- 執行 with_server.py
- 執行 Playwright 腳本
- 安裝依賴:`pip install playwright && playwright install chromium`

### 3. **Read Tool**
- 讀取 HTML 檔案(靜態頁面)
- 讀取範例腳本
- 檢視截圖

---

## 最佳實踐

### 1. 等待策略最佳實踐

**總是等待 networkidle**:
```python
# ✅ 動態應用
page.goto('http://localhost:5173')
page.wait_for_load_state('networkidle')  # 等待 JavaScript

# ✅ 靜態 HTML(可選但建議)
page.goto(f'file://{path}')
page.wait_for_load_state('load')

# ❌ 不等待(常見錯誤)
page.goto('http://localhost:5173')
page.click('button')  # 可能元素還未渲染
```

### 2. 選擇器策略最佳實踐

**選擇器優先順序**:
```python
# 1. 文字內容(最穩定)
page.click('text=Submit')

# 2. Role(語義化)
page.click('role=button[name="Submit"]')

# 3. ID(如果存在)
page.click('#submit-btn')

# 4. CSS 選擇器(最後選擇)
page.click('button.btn-primary')

# ❌ 避免:脆弱的選擇器
page.click('div > div > button:nth-child(3)')
```

### 3. 伺服器管理最佳實踐

**使用 with_server.py**:
```bash
# ✅ 自動管理
python scripts/with_server.py \
  --server "npm run dev" --port 5173 \
  -- python test.py

# ❌ 手動管理(容易忘記清理)
# Terminal 1: npm run dev
# Terminal 2: python test.py
# Terminal 1: Ctrl+C (可能忘記)
```

### 4. 除錯最佳實踐

**截圖 + 元素探索**:
```python
# 偵察階段
page.screenshot(path='/tmp/before.png', full_page=True)
buttons = page.locator('button').all()
for btn in buttons:
    print(f"{btn.inner_text()} - visible:{btn.is_visible()}")

# 操作階段
page.click('button#submit')

# 驗證階段
page.screenshot(path='/tmp/after.png', full_page=True)
```

### 5. 多伺服器最佳實踐

**順序很重要**:
```bash
# ✅ 後端先啟動
python scripts/with_server.py \
  --server "cd backend && python server.py" --port 3000 \
  --server "cd frontend && npm run dev" --port 5173 \
  -- python test.py

# 順序:後端(3000) → 前端(5173) → 測試
# 前端可能依賴後端 API
```

---

## 總結

### 成功要素

1. **自動化伺服器管理**:with_server.py 處理生命週期
2. **清晰的決策樹**:快速選擇正確方法
3. **偵察-行動模式**:可靠的動態頁面處理
4. **範例驅動**:涵蓋常見場景
5. **黑箱策略**:保護 context window

### 學習重點

對於想要理解此 Skill 的使用者:
- Playwright 基本用法
- 等待策略的重要性
- 選擇器識別技巧
- 伺服器生命週期管理
- 除錯方法(截圖、日誌)

### 適用場景

此 Skill 最適合用於:
- 前端功能測試
- UI 行為驗證
- 自動化截圖
- 表單提交測試
- 連結檢查
- 控制台錯誤監控

### 技術亮點

- **自動伺服器管理**:可靠的啟動和清理
- **多伺服器支援**:前後端分離場景
- **偵察-行動模式**:處理動態內容
- **Headless 模式**:快速執行
- **範例導向**:快速上手

---

## 參考資源

- [SKILL.md](./SKILL.md) - 英文原始文檔
- [scripts/with_server.py](./scripts/with_server.py) - 伺服器管理腳本
- [examples/](./examples/) - 範例腳本目錄
- [Playwright 官方文檔](https://playwright.dev/python/)

---

**最後更新**:2025-11-15
**文件版本**:1.0.0
**適用於**:Claude Code Skills
