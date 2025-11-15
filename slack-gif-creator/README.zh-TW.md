# Slack GIF Creator - 正體中文說明文件

## 📋 目錄
- [Skill 概述](#skill-概述)
- [目錄結構](#目錄結構)
- [檔案說明](#檔案說明)
- [關鍵模組分析](#關鍵模組分析)
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
Slack GIF Creator 是一個專門用於建立符合 Slack 規範的動畫 GIF 工具套件。它提供完整的動畫基礎組件(primitives)、驗證器和視覺效果,讓 Claude 能夠建立專業品質的動畫 GIF,包括訊息 GIF 和 emoji GIF。

主要功能包括:
- Slack 限制驗證器(檔案大小、尺寸、顏色等)
- 13 種可組合的動畫基礎組件(shake、bounce、spin、pulse、fade等)
- 完整的視覺效果系統(粒子、閃光、震波等)
- 文字渲染與排版工具
- 色彩管理和調色盤系統
- GIF 建構和優化工具
- 緩動函數(easing functions)支援平滑動畫

### 解決的問題
在為 Slack 建立動畫 GIF 時,經常遇到以下挑戰:

1. **嚴格的檔案大小限制**:Emoji GIF 僅 64KB,訊息 GIF 約 2MB
2. **優化困難**:需要平衡動畫品質和檔案大小
3. **重複編寫動畫程式碼**:每次都要重新實作 shake、bounce 等效果
4. **缺乏視覺效果**:手動建立粒子、閃光等效果耗時
5. **文字可讀性問題**:小尺寸 GIF 的文字難以閱讀
6. **驗證麻煩**:不知道 GIF 是否符合 Slack 要求

Slack GIF Creator 提供完整的工具套件,讓建立專業 GIF 變得系統化和高效。

---

## 目錄結構

```
slack-gif-creator/
├── SKILL.md                    # Skill 主要說明文件(英文)
├── LICENSE.txt                 # 授權條款
├── README.zh-TW.md            # 正體中文說明文件(本文件)
├── requirements.txt            # Python 依賴套件
├── core/                       # 核心工具模組
│   ├── color_palettes.py      # 色彩管理和調色盤
│   ├── easing.py              # 緩動函數
│   ├── frame_composer.py      # 畫面組合工具
│   ├── gif_builder.py         # GIF 建構和優化
│   ├── typography.py          # 文字渲染
│   ├── validators.py          # Slack 規範驗證器
│   └── visual_effects.py      # 視覺效果系統
└── templates/                  # 動畫範本庫
    ├── bounce.py              # 彈跳動畫
    ├── explode.py             # 爆炸/碎裂動畫
    ├── fade.py                # 淡入淡出動畫
    ├── flip.py                # 翻轉動畫
    ├── kaleidoscope.py        # 萬花筒效果
    ├── morph.py               # 變形動畫
    ├── move.py                # 移動動畫
    ├── pulse.py               # 脈動/心跳動畫
    ├── shake.py               # 晃動動畫
    ├── slide.py               # 滑動動畫
    ├── spin.py                # 旋轉動畫
    ├── wiggle.py              # 擺動動畫
    └── zoom.py                # 縮放動畫
```

---

## 檔案說明

### 核心檔案

#### 1. `SKILL.md`(647 行)
- **用途**:完整的 GIF 建立指南
- **內容**:
  - Slack 要求說明(訊息 GIF vs Emoji GIF)
  - 工具套件結構(驗證器、動畫基礎組件、輔助工具)
  - 13 種動畫範本詳細說明
  - 優化策略
  - 組合模式範例
- **特點**:強調創意自由和可組合性

#### 2. `requirements.txt`
**內容**:
```
pillow
imageio
numpy
```
- **pillow**:圖像處理(繪製、合成)
- **imageio**:GIF 讀寫
- **numpy**:數值計算(變換、效果)

### Core 目錄(核心模組)

#### 1. `gif_builder.py`(約 239 行)
**功能**:GIF 組裝和優化

**主要類別**:`GIFBuilder`
- 管理畫面(frames)集合
- 自動顏色量化(color quantization)
- 移除重複畫面
- 優化檔案大小
- Emoji 模式(激進優化)

**關鍵方法**:
```python
builder = GIFBuilder(width=128, height=128, fps=10)
builder.add_frame(frame)  # 添加畫面
builder.save('output.gif', num_colors=48, optimize_for_emoji=True)
```

#### 2. `validators.py`(約 206 行)
**功能**:驗證 GIF 是否符合 Slack 規範

**Slack 限制**:
- **訊息 GIF**:約 2MB,480x480,15-20 FPS,128-256 色,2-5秒
- **Emoji GIF**:64KB(嚴格),128x128,10-12 FPS,32-48 色,1-2秒

**驗證函數**:
```python
check_slack_size(path, is_emoji=True)  # 檔案大小
validate_dimensions(width, height, is_emoji=True)  # 尺寸
validate_gif(path, is_emoji=True)  # 完整驗證
is_slack_ready(path, is_emoji=True)  # 快速檢查
```

#### 3. `easing.py`(約 157 行)
**功能**:緩動函數,讓動畫更平滑自然

**可用緩動**:
- `linear`:線性
- `ease_in`:加速
- `ease_out`:減速
- `ease_in_out`:兩端平滑
- `bounce_out`:彈跳
- `elastic_out`:彈性
- `back_out`:超調(overshoot)

**使用方式**:
```python
from core.easing import interpolate

# 物體下落(加速)
y = interpolate(start=0, end=400, t=progress, easing='ease_in')

# 物體著陸(減速)
y = interpolate(start=0, end=400, t=progress, easing='ease_out')

# 彈跳
y = interpolate(start=0, end=400, t=progress, easing='bounce_out')
```

#### 4. `typography.py`(約 269 行)
**功能**:文字渲染和排版

**關鍵功能**:
- `draw_text_with_outline()`:帶外框的文字(提高可讀性)
- `TYPOGRAPHY_SCALE`:預設字體大小(h1=60, h2=48, h3=36等)
- 自動置中和對齊

**範例**:
```python
from core.typography import draw_text_with_outline

draw_text_with_outline(
    frame, "GOAL!",
    position=(240, 100),
    font_size=60,
    text_color=(255, 68, 68),
    outline_color=(0, 0, 0),
    outline_width=4,
    centered=True
)
```

#### 5. `color_palettes.py`(約 218 行)
**功能**:色彩管理和預設調色盤

**預設調色盤**:
- `vibrant`:鮮豔色彩
- `pastel`:粉彩色調
- `dark`:深色主題
- `neon`:霓虹色
- `professional`:專業配色

**使用方式**:
```python
from core.color_palettes import get_palette

palette = get_palette('vibrant')
bg_color = palette['background']
text_color = palette['primary']
accent_color = palette['accent']
```

#### 6. `frame_composer.py`(約 362 行)
**功能**:畫面組合和繪圖工具

**工具函數**:
- `create_gradient_background()`:漸層背景
- `draw_emoji_enhanced()`:帶陰影的 emoji
- `draw_circle_with_shadow()`:帶陰影的形狀
- `draw_star()`:五角星

#### 7. `visual_effects.py`(約 371 行)
**功能**:視覺效果系統

**主要效果**:
- `ParticleSystem`:粒子系統(sparkles、confetti)
- `create_impact_flash()`:衝擊閃光
- `create_shockwave_rings()`:震波環
- 粒子物理(重力、速度、生命週期)

**使用方式**:
```python
from core.visual_effects import ParticleSystem, create_impact_flash

particles = ParticleSystem()
particles.emit_sparkles(x=240, y=200, count=15)
particles.update()
particles.render(frame)

# 閃光效果
frame = create_impact_flash(frame, position=(240, 200), radius=100)
```

### Templates 目錄(動畫範本)

13 種動畫基礎組件,可自由組合:

#### 1. `shake.py`
**功能**:晃動動畫
```python
create_shake_animation(
    object_type='emoji',
    object_data={'emoji': '😱', 'size': 80},
    num_frames=20,
    shake_intensity=15,
    direction='both'  # 'horizontal', 'vertical', 'both'
)
```

#### 2. `bounce.py`
**功能**:彈跳動畫
```python
create_bounce_animation(
    object_type='circle',
    object_data={'radius': 40, 'color': (255, 100, 100)},
    num_frames=30,
    bounce_height=150
)
```

#### 3. `spin.py`
**功能**:旋轉動畫
```python
create_spin_animation(
    object_type='emoji',
    object_data={'emoji': '🔄', 'size': 100},
    rotation_type='clockwise',  # 'clockwise', 'counter', 'wobble'
    full_rotations=2
)
```

#### 4. `pulse.py`
**功能**:脈動/心跳動畫
```python
create_pulse_animation(
    object_data={'emoji': '❤️', 'size': 100},
    pulse_type='smooth',  # 'smooth', 'heartbeat', 'snap'
    scale_range=(0.8, 1.2)
)
```

#### 5. `fade.py`
**功能**:淡入淡出動畫
```python
create_fade_animation(fade_type='in')  # 'in', 'out'
create_crossfade(object1_data, object2_data)  # 交叉淡入淡出
```

#### 6. `zoom.py`
**功能**:縮放動畫
```python
create_zoom_animation(
    zoom_type='in',  # 'in', 'out'
    scale_range=(0.1, 2.0),
    add_motion_blur=True
)
```

#### 7. `explode.py`
**功能**:爆炸/碎裂動畫
```python
create_explode_animation(
    explode_type='burst',  # 'burst', 'shatter', 'dissolve'
    num_pieces=25
)
```

#### 8. `wiggle.py`
**功能**:擺動/抖動動畫
```python
create_wiggle_animation(
    wiggle_type='jello',  # 'jello', 'wave', 'rubber'
    intensity=1.0,
    cycles=2
)
```

#### 9. `slide.py`
**功能**:滑動動畫
```python
create_slide_animation(
    direction='left',  # 'left', 'right', 'up', 'down'
    slide_type='in',  # 'in', 'out', 'across'
    overshoot=True
)
```

#### 10. `flip.py`
**功能**:翻轉動畫
```python
create_flip_animation(
    object1_data={'emoji': '😊', 'size': 120},
    object2_data={'emoji': '😂', 'size': 120},
    flip_axis='horizontal'  # 'horizontal', 'vertical'
)
```

#### 11. `morph.py`
**功能**:變形動畫
```python
create_morph_animation(
    object1_data={'emoji': '😊'},
    object2_data={'emoji': '😂'},
    morph_type='crossfade'  # 'crossfade', 'scale', 'spin_morph'
)
```

#### 12. `move.py`
**功能**:移動動畫
```python
create_move_animation(
    motion_type='arc',  # 'linear', 'arc', 'circle', 'wave'
    start_pos=(50, 240),
    end_pos=(430, 240),
    easing='ease_out'
)
```

#### 13. `kaleidoscope.py`
**功能**:萬花筒效果
```python
apply_kaleidoscope(frame, segments=8)
create_kaleidoscope_animation(base_frame, num_frames=30)
```

---

## 關鍵模組分析

### gif_builder.py 深度分析

#### 顏色量化策略
```python
def save(self, filename, num_colors=256, optimize_for_emoji=False):
    if optimize_for_emoji:
        # Emoji 模式:激進優化
        num_colors = min(num_colors, 48)
        # 移除重複畫面
        # 減少調色盤大小

    # PIL 顏色量化
    quantized = frame.quantize(colors=num_colors, method=2)
```

**設計理念**:
- 平衡視覺品質和檔案大小
- Emoji 模式自動限制顏色
- 使用 PIL 的最佳量化演算法

#### 重複畫面移除
```python
# 檢測連續相同畫面
if previous_frame and frames_are_identical(frame, previous_frame):
    duplicate_count += 1
else:
    if duplicate_count > 0:
        # 只保留一個副本
        optimized_frames.append(previous_frame)
```

**優點**:
- 大幅減少檔案大小
- 不影響視覺效果
- 自動化處理

### validators.py 深度分析

#### Slack 限制的嚴格檢查
```python
def check_slack_size(filepath, is_emoji=True):
    size_kb = os.path.getsize(filepath) / 1024

    if is_emoji:
        limit_kb = 64
        passes = size_kb <= limit_kb
    else:
        limit_mb = 2
        passes = (size_kb / 1024) <= limit_mb

    return passes, {
        'size_kb': size_kb,
        'size_mb': size_kb / 1024,
        'limit': f"{limit_kb}KB" if is_emoji else f"{limit_mb}MB",
        'passes': passes,
        'over_by': size_kb - limit_kb if not passes else 0
    }
```

**設計亮點**:
- 清晰的 pass/fail 回傳
- 詳細的大小資訊
- 計算超出量(便於優化)

### easing.py 深度分析

#### 緩動函數實作
```python
def ease_out(t):
    """減速緩動(物體著陸)"""
    return 1 - (1 - t) ** 2

def bounce_out(t):
    """彈跳緩動"""
    if t < 1/2.75:
        return 7.5625 * t * t
    elif t < 2/2.75:
        t -= 1.5/2.75
        return 7.5625 * t * t + 0.75
    elif t < 2.5/2.75:
        t -= 2.25/2.75
        return 7.5625 * t * t + 0.9375
    else:
        t -= 2.625/2.75
        return 7.5625 * t * t + 0.984375
```

**應用場景**:
- `ease_in`:物體下落、加速
- `ease_out`:物體著陸、減速
- `bounce_out`:彈跳效果
- `elastic_out`:彈性效果(Q彈)

---

## 設計策略

### 1. 可組合性設計(Composability)

**動畫基礎組件可自由組合**:
```python
# 範例:彈跳 + 震動
for i in range(num_frames):
    # 彈跳運動
    t = i / (num_frames - 1)
    y = interpolate(start_y, ground_y, t, 'bounce_out')

    # 著地時加入震動
    if y >= ground_y - 5:
        shake_x = math.sin(i * 2) * 10
        x = center_x + shake_x
    else:
        x = center_x

    draw_emoji(frame, '⚽', (x, y))
```

**優點**:
- 無限創意可能
- 不受範本限制
- 鼓勵實驗

### 2. 分層優化策略

**優化層級**:
1. **畫面層級**:減少畫面數量(降低 FPS 或縮短時長)
2. **顏色層級**:減少顏色數量(256 → 128 → 64 → 32)
3. **尺寸層級**:減少尺寸(480x480 → 320x320)
4. **內容層級**:簡化設計(避免漸層、減少元素)

**針對 Emoji GIF 的激進優化**:
```
目標:64KB
策略:10-12 畫面,32-40 色,純色設計,極簡元素
```

### 3. 驗證優先哲學

**建立流程**:
```
建立動畫
    ↓
儲存 GIF
    ↓
驗證大小
    ↓
是否通過?
├─ 是 → 完成
└─ 否 → 優化 → 重新儲存 → 重新驗證
```

**自動警告**:
```python
info = builder.save('emoji.gif', num_colors=48, optimize_for_emoji=True)
# 自動檢查並警告如果超出限制
```

### 4. 創意自由 vs 技術限制

**哲學**:
- 提供工具,不限制創意
- 驗證器確保符合規範
- 範本作為起點,非終點
- 鼓勵自訂和實驗

**SKILL.md 強調**:
> "Complete creative freedom is available in how these tools are applied."

---

## Claude Code 使用方式

### 觸發時機

Claude Code 會在以下情況使用此 Skill:

1. **需要建立 Slack GIF**
   - 範例:「建立一個踢足球的 GIF 給 Slack」
   - 範例:「做一個震動的 emoji GIF」
   - 範例:「建立一個慶祝動畫」

2. **需要優化現有 GIF**
   - 範例:「這個 GIF 太大了,優化到 64KB」
   - 範例:「減少這個 GIF 的檔案大小」

3. **需要組合多種動畫效果**
   - 範例:「建立一個彈跳然後爆炸的動畫」
   - 範例:「做一個旋轉淡入的效果」

### 典型工作流程

#### 工作流程 1:建立簡單 Emoji GIF

```python
from core.gif_builder import GIFBuilder
from core.validators import check_slack_size
from templates.pulse import create_attention_pulse

# 步驟 1:建立動畫
frames = create_attention_pulse(emoji='⚠️', num_frames=20)

# 步驟 2:組裝 GIF
builder = GIFBuilder(128, 128, 10)
for frame in frames:
    builder.add_frame(frame)

# 步驟 3:儲存並優化
info = builder.save('warning.gif', num_colors=40, optimize_for_emoji=True)

# 步驟 4:驗證
passes, details = check_slack_size('warning.gif', is_emoji=True)
if not passes:
    print(f"檔案太大,超出 {details['over_by']:.1f}KB")
    # 重新儲存,減少顏色或畫面數
```

#### 工作流程 2:建立複雜組合動畫

```python
from core.gif_builder import GIFBuilder
from core.easing import interpolate
from core.visual_effects import create_impact_flash
from core.frame_composer import create_gradient_background, draw_emoji_enhanced
from core.typography import draw_text_with_outline

builder = GIFBuilder(480, 480, 20)

# 階段 1:物體下落
for i in range(15):
    frame = create_gradient_background(480, 480, (240, 248, 255), (200, 230, 255))
    t = i / 14
    y = interpolate(0, 350, t, 'ease_in')
    draw_emoji_enhanced(frame, '⚽', position=(220, int(y)), size=80)
    builder.add_frame(frame)

# 階段 2:衝擊
for i in range(8):
    frame = create_gradient_background(480, 480, (240, 248, 255), (200, 230, 255))

    # 前幾個畫面加入閃光
    if i < 3:
        frame = create_impact_flash(frame, (240, 350), radius=120, intensity=0.6)

    draw_emoji_enhanced(frame, '⚽', position=(220, 350), size=80)

    # 顯示文字
    if i > 2:
        draw_text_with_outline(frame, "GOAL!", position=(240, 150),
                              font_size=60, text_color=(255, 68, 68),
                              outline_color=(0, 0, 0), outline_width=4, centered=True)

    builder.add_frame(frame)

# 儲存
builder.save('goal.gif', num_colors=128)
```

#### 工作流程 3:優化過大的 GIF

```python
from core.validators import check_slack_size
from core.gif_builder import GIFBuilder

# 檢查當前大小
passes, info = check_slack_size('emoji.gif', is_emoji=True)
print(f"當前大小: {info['size_kb']:.1f}KB, 限制: 64KB")

if not passes:
    # 策略 1:減少顏色
    builder.save('emoji.gif', num_colors=32, optimize_for_emoji=True)

    # 再次檢查
    passes, info = check_slack_size('emoji.gif', is_emoji=True)

    if not passes:
        # 策略 2:減少畫面數
        # 重新建立,使用更少的畫面(10-12 而非 20)
        ...
```

### 不使用此 Skill 的情況

- 建立一般圖片(非動畫)
- 建立影片檔案
- 建立非 Slack 用途的 GIF(無大小限制)

---

## 技術架構

### 技術堆疊

**Python 生態系**:
```
Pillow (PIL)       # 圖像處理和繪製
imageio            # GIF 讀寫
numpy              # 數值計算和陣列操作
```

### 模組依賴圖

```
templates/ (動畫範本)
├── 使用 core/gif_builder.py
├── 使用 core/easing.py
├── 使用 core/frame_composer.py
└── 使用 PIL (Image, ImageDraw)

core/gif_builder.py
├── 使用 PIL.Image
├── 使用 imageio
└── 使用 core/validators.py

core/visual_effects.py
├── 使用 PIL
├── 使用 numpy
└── 使用 core/easing.py

core/typography.py
├── 使用 PIL.ImageDraw
└── 使用 PIL.ImageFont
```

### GIF 建立流程

```
建立畫面(frames)
    ↓
[使用 templates/ 或自訂程式碼]
    ├─ 套用緩動函數(easing)
    ├─ 添加視覺效果(visual_effects)
    ├─ 繪製文字(typography)
    └─ 組合元素(frame_composer)
    ↓
[GIFBuilder 組裝]
    ├─ 添加畫面
    ├─ 顏色量化
    ├─ 移除重複畫面
    └─ 儲存
    ↓
[驗證]
    ├─ 檢查檔案大小
    ├─ 檢查尺寸
    └─ 完整驗證
    ↓
[如需要] 優化
    ├─ 減少顏色
    ├─ 減少畫面
    └─ 簡化設計
```

---

## 使用的 Prompt 策略

### 1. 工具套件導向

**SKILL.md 結構**:
```markdown
## Toolkit Structure
1. **Validators** - Check requirements
2. **Animation Primitives** - Building blocks
3. **Helper Utilities** - Common needs
```

**優點**:
- 清晰的分類
- 便於選擇適當工具
- 鼓勵模組化思考

### 2. 範例驅動

**每個範本都包含完整範例**:
```python
# Shake
create_shake_animation(
    object_type='emoji',
    object_data={'emoji': '😱', 'size': 80},
    num_frames=20,
    shake_intensity=15,
    direction='both'
)
```

**優點**:
- 具體展示用法
- 減少猜測
- 加速實作

### 3. 組合模式教學

**SKILL.md 包含組合範例**:
```python
## Example Composition Patterns
### Combining Primitives (Move + Shake)
...
```

**設計理念**:
- 教導如何組合
- 激發創意
- 提供模式

### 4. 優化策略指引

**明確的優化建議**:
```markdown
**For Emoji GIFs (>64KB) - be aggressive:**
1. Limit to 10-12 frames total
2. Use 32-40 colors maximum
3. Avoid gradients
4. Simplify design
5. Use `optimize_for_emoji=True`
```

---

## 使用的工具(Tools)

此 Skill 主要依賴以下 Claude Code 工具:

### 1. **Write Tool**
- 建立 Python 腳本(GIF 建立程式碼)
- 建立動畫程式碼

### 2. **Bash Tool**
- 執行 Python 腳本
- 安裝依賴:`pip install pillow imageio numpy`
- 執行驗證

### 3. **Read Tool**
- 讀取 core/ 模組
- 讀取 templates/ 範本
- 檢查 GIF 檔案資訊

---

## 最佳實踐

### 1. Emoji GIF 最佳實踐

**嚴格控制大小**:
```python
# ✅ 好的做法
- 10-12 畫面
- 32-40 顏色
- 純色設計(避免漸層)
- 極簡元素
- optimize_for_emoji=True

# ❌ 避免
- 20+ 畫面
- 128+ 顏色
- 複雜漸層
- 過多元素
```

### 2. 動畫設計最佳實踐

**使用緩動函數**:
```python
# ✅ 自然的動畫
y = interpolate(0, 400, t, 'ease_out')

# ❌ 機械的動畫
y = start + (end - start) * t  # 線性,不自然
```

**分階段設計**:
```python
# ✅ 清晰的階段
# 階段 1:預備動作(anticipation)
# 階段 2:主要動作(action)
# 階段 3:反應(reaction)

# ❌ 混在一起
# 所有動作同時發生
```

### 3. 檔案大小優化最佳實踐

**漸進式優化**:
```
1. 檢查大小
2. 減少顏色(128 → 64 → 32)
3. 如仍太大,減少畫面數
4. 如仍太大,簡化設計
5. 如仍太大,減少尺寸
```

---

## 總結

### 成功要素

1. **完整的工具套件**:涵蓋驗證、動畫、效果
2. **可組合性**:13 種動畫基礎組件可自由組合
3. **Slack 優化**:內建 Slack 限制的驗證和優化
4. **創意自由**:工具而非限制
5. **專業品質**:視覺效果、緩動函數、排版

### 技術亮點

- **13 種動畫範本**:涵蓋常見動畫需求
- **自動優化**:顏色量化、重複畫面移除
- **嚴格驗證**:確保符合 Slack 規範
- **視覺效果系統**:粒子、閃光、震波
- **緩動函數**:10+ 種平滑動畫曲線

### 適用場景

此 Skill 最適合用於:
- Slack Emoji GIF
- Slack 訊息 GIF
- 反應動畫
- 慶祝動畫
- 警告動畫
- 品牌動畫

---

## 參考資源

- [SKILL.md](./SKILL.md) - 英文原始文檔
- [core/](./core/) - 核心模組
- [templates/](./templates/) - 動畫範本

---

**最後更新**:2025-11-15
**文件版本**:1.0.0
**適用於**:Claude Code Skills
