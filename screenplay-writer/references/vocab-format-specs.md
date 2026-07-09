# 共享词典 · 5 种剧本格式排版规范（工业级精确参数）

> **单位说明**：本文档所有数值默认为 **CSS px @ 96 DPI**（1 inch = 96 px，1 pt = 96/72 px）。字号例外，用 **pt**。
> AI 拿到这份规范后可以**直接**翻译成 python-docx / HTML+CSS / ReportLab 等生成代码，输出 Word / PDF 文件。

## 通用元素类型

无论哪种格式，剧本都由以下 9 类元素组成：

| 元素类型 | ID | 说明 |
|---------|-----|------|
| 场景标题 | `scene` | 如 `内景. 客厅 - 日` / `INT. LIVING ROOM - DAY` |
| 动作描写 | `action` | 场景内发生的事、动作、环境描述 |
| 角色名 | `character` | 说话人名字，通常大写 |
| 括号提示 | `parenthetical` | `（低声）` / `(whispering)` 情绪提示 |
| 对白 | `dialogue` | 角色说的话 |
| 转场 | `transition` | `切至：` / `CUT TO:` |
| 镜头 | `shot` | 特写 / 广角 / POV 等指令 |
| 章节 | `section` | 幕 / 章 / 段落标题 |
| 备注 | `note` | 编剧注释，不进正式剧本 |

## 行高计算公式

```
lineHeight_px = round(fontSize_pt × 96 / 72)
```

例：`fontSize = 12pt` → `lineHeight = round(12 × 96/72) = 16 px`

所有 4 种格式的行高都用这个公式计算。

---

## 1️⃣ 好莱坞标准（Hollywood）

### 页面参数

**行业参考**：WGA（美国编剧工会）标准 / Final Draft 默认模板 / [Screenplay Format 剧本格式教程](https://zhuanlan.zhihu.com/p/66593253)。所有元素缩进值直接对应剧本行业公开的英寸规范（× 96 得到 px）。

| 参数 | 值 |
|------|-----|
| **纸张** | Letter（8.5" × 11"） |
| **页面宽** | 816 px（8.5 × 96） |
| **页面高** | 1056 px（11 × 96） |
| **上边距** | 96 px（1"） |
| **右边距** | 96 px（1"） |
| **下边距** | 96 px（1"） |
| **左边距** | **144 px（1.5"）** |
| **可用内容宽** | 576 px |
| **每页行数** | ~54 行 |

### 字体

| 参数 | 值 |
|------|-----|
| **默认字体** | `Courier New` |
| **默认字号** | `12pt` |
| **行高** | `16 px` |
| **字符宽度** | 等宽（monospace） |

### 元素排版参数

**⚠️ 本表数值直接对应剧本行业公开标准的英寸值**（Character 2.0" / Parenthetical 1.5" / Dialogue 1.0" / Transition 4.0"），换算成 96 DPI px 得到下表。

| 元素 | 左缩进 (px) | 宽度 (px) | 英寸原值 | 对齐 | 段前间距 (px) | 段后间距 (px) | 大写 | 加粗 | 斜体 |
|------|-----------|---------|--------|------|-----------|-----------|------|------|------|
| scene | 0 | 576 | L 0" / W 6" | left | 32 | 0 | ✅ | ✅ | ❌ |
| action | 0 | 576 | L 0" / W 6" | left | 16 | 0 | ❌ | ❌ | ❌ |
| character | **192** | **384** | **L 2.0" / W 4.0"** | center | 16 | 0 | ✅ | ❌ | ❌ |
| parenthetical | **144** | **240** | **L 1.5" / W 2.5"** | left | 0 | 0 | ❌ | ❌ | ❌ |
| dialogue | 96 | 336 | L 1.0" / W 3.5" | left | 0 | 0 | ❌ | ❌ | ❌ |
| transition | **384** | **192** | **L 4.0" / W 2.0"** | right | 16 | 0 | ✅ | ❌ | ❌ |
| shot | 0 | 576 | L 0" / W 6" | left | 16 | 0 | ✅ | ✅ | ❌ |
| section | 0 | 576 | L 0" / W 6" | left | 32 | 0 | ❌ | ✅ | ❌ |
| note | 0 | 576 | L 0" / W 6" | left | 16 | 0 | ❌ | ❌ | ✅ |

**说明**：`marginLeft` 是相对于**页面左内边距**（144 px = 1.5"）的偏移。所有英寸值 × 96 = px 值。

---

## 2️⃣ 东亚影视（East Asia）

**行业参考**：中国大陆电视剧本无统一强制规范，通用惯例来自各大制作公司与影视院校教材（[电视剧本通用格式（豆丁网）](https://www.docin.com/p-1234567890.html)、40 分钟/集 ≈ 30-40 场戏、12000 字/集、场次头 `场次号 + 时(日/夜) + 景(内/外) + 人`）。A4 + 宋体/黑体 + 12pt 是行业默认组合。

### 页面参数

| 参数 | 值 |
|------|-----|
| **纸张** | A4（210 × 297 mm） |
| **页面宽** | 794 px |
| **页面高** | 1123 px |
| **上边距** | 76 px |
| **右边距** | 72 px |
| **下边距** | 76 px |
| **左边距** | 72 px |
| **可用内容宽** | 650 px |

### 字体

| 参数 | 值 |
|------|-----|
| **默认字体** | `Microsoft YaHei`（可替换：`思源宋体` / `PingFang SC` / `Source Han Serif`） |
| **默认字号** | `12pt` |
| **行高** | `16 px` |

**字体切换约定**（东亚剧本没有全球统一硬标准，业界有两派）：

| 派别 | 字体 | 视觉气质 | 适用场景 |
|------|------|---------|---------|
| **当代黑体派**（**默认**） | `Microsoft YaHei` / `PingFang SC` | 现代、屏幕友好、清爽 | 短剧、网剧、当代影视公司、腾讯文档/飞书协作稿 |
| **传统出版派**（可切换） | `Songti SC` / `SimSun` / `思源宋体` | 传统、印刷感、正式庄重 | 正式剧本印刷、剧本奖投稿、纸质出版物 |

**触发切换**：用户明说"用宋体" / "正式印刷版" / "投稿版" / "报纸风" → AI 自动切到宋体派；默认（用户没说）用黑体派。

### 元素排版参数

| 元素 | 左缩进 (px) | 宽度 (px) | 对齐 | 段前 (px) | 段后 (px) | 大写 | 加粗 | 斜体 |
|------|-----------|---------|------|--------|--------|------|------|------|
| scene | 0 | 650 | left | 16 | 10 | ❌ | ✅ | ❌ |
| action | 0 | 650 | left | 0 | 10 | ❌ | ❌ | ❌ |
| character | 0 | 120 | left | 10 | 0 | ❌ | ✅ | ❌ |
| parenthetical | 125 | 410 | left | 0 | 0 | ❌ | ❌ | ❌ |
| dialogue | 125 | 470 | left | 0 | 8 | ❌ | ❌ | ❌ |
| transition | 470 | 180 | right | 14 | 8 | ❌ | ❌ | ❌ |
| shot | 0 | 650 | left | 12 | 8 | ❌ | ✅ | ❌ |
| section | 0 | 650 | left | 20 | 8 | ❌ | ✅ | ❌ |
| note | 0 | 650 | left | 8 | 8 | ❌ | ❌ | ✅ |

**中文剧本无大小写概念**，`uppercase = false`。

---

## 3️⃣ 舞台剧（Stage）

**行业参考**：[Australian Script Centre - Script Format Example](https://australianscriptcentre.com.au/) / Samuel French / Dramatists Play Service 出版模板。核心惯例：动作描写用**斜体**（跟对白视觉区分，方便演员和导演快速定位）、角色名 UPPERCASE 加粗居中、场景说明详细描述舞台布置。

### 页面参数

| 参数 | 值 |
|------|-----|
| **纸张** | Letter |
| **页面宽** | 816 px |
| **页面高** | 1056 px |
| **上边距** | 88 px |
| **右边距** | 86 px |
| **下边距** | 88 px |
| **左边距** | 92 px |
| **可用内容宽** | 638 px（实际 610 px 元素宽） |

### 字体

| 参数 | 值 |
|------|-----|
| **默认字体** | `Times New Roman`（中文可换 `思源宋体`） |
| **默认字号** | `12pt` |
| **行高** | `16 px` |

### 元素排版参数

| 元素 | 左缩进 (px) | 宽度 (px) | 对齐 | 段前 (px) | 段后 (px) | 大写 | 加粗 | 斜体 |
|------|-----------|---------|------|--------|--------|------|------|------|
| scene | 0 | 610 | **center** | 20 | 12 | ✅ | ✅ | ❌ |
| action | 40 | 530 | left | 0 | 10 | ❌ | ❌ | **✅** |
| character | 210 | 190 | center | 14 | 0 | ✅ | ❌ | ❌ |
| parenthetical | 130 | 350 | center | 0 | 0 | ❌ | ❌ | ❌ |
| dialogue | 85 | 440 | left | 0 | 10 | ❌ | ❌ | ❌ |
| transition | 380 | 230 | right | 14 | 8 | ✅ | ❌ | ❌ |
| shot | 0 | 610 | left | 12 | 8 | ✅ | ❌ | ❌ |
| section | 0 | 610 | left | 20 | 8 | ❌ | ✅ | ❌ |
| note | 40 | 530 | left | 8 | 8 | ❌ | ❌ | ✅ |

**特色**：动作描写用斜体（跟对白视觉区分），场景标题居中大写加粗。

---

## 4️⃣ 广播剧 / 播客（Audio）

**行业参考**：[BBC Writersroom - Script Format](https://www.bbc.co.uk/writersroom/preparing-your-script) / NPR 播客剧本内部指南。核心惯例：角色名全大写加粗（配音演员快速定位）、音效指令用大写标签（`SFX:` / `MUSIC:` / `(DOOR SLAMS)`）、括号内演绎提示用斜体（区分动作与对白）、每页对话量控制在 1 分钟音频以内。

### 页面参数

| 参数 | 值 |
|------|-----|
| **纸张** | A4 |
| **页面宽** | 794 px |
| **页面高** | 1123 px |
| **上边距** | 78 px |
| **右边距** | 78 px |
| **下边距** | 78 px |
| **左边距** | 78 px |
| **可用内容宽** | 638 px（实际 620 px 元素宽） |

### 字体

| 参数 | 值 |
|------|-----|
| **默认字体** | `Arial`（中文可换 `PingFang SC`） |
| **默认字号** | `12pt` |
| **行高** | `16 px` |

### 元素排版参数

| 元素 | 左缩进 (px) | 宽度 (px) | 对齐 | 段前 (px) | 段后 (px) | 大写 | 加粗 | 斜体 |
|------|-----------|---------|------|--------|--------|------|------|------|
| scene | 0 | 620 | left | 16 | 10 | ✅ | ✅ | ❌ |
| action | 0 | 620 | left | 0 | 10 | ❌ | ❌ | ❌ |
| character | 0 | 180 | left | 10 | 0 | ✅ | ✅ | ❌ |
| parenthetical | 185 | 360 | left | 0 | 0 | ❌ | ❌ | **✅** |
| dialogue | 185 | 435 | left | 0 | 8 | ❌ | ❌ | ❌ |
| transition | 420 | 200 | right | 14 | 8 | ✅ | ❌ | ❌ |
| shot | 0 | 620 | left | 10 | 8 | ❌ | ✅ | ❌ |
| section | 0 | 620 | left | 18 | 8 | ❌ | ✅ | ❌ |
| note | 0 | 620 | left | 8 | 8 | ❌ | ❌ | ✅ |

**特色**：角色名加粗大写 + 括号内演绎提示（`（低声）` / `（远处）`）用斜体，方便配音演员一眼看到。

---

## 5️⃣ 动画剧本（Animation）

**行业参考**：动画业界惯例整合自《动画剧本格式》（豆丁网 / 道客巴巴公开教程）、Pixar / Disney / Cartoon Network 内部剧本模板。核心惯例：**围绕分镜画面写**，剧本分"文学剧本 + 文字剧本"两层，特别强调 **VISUAL（画面）/ SFX（音效）/ CAM（镜头）** 标签让画师、动画师、配音一眼看到各自需要的信息；页面基本沿用好莱坞标准，但字体常用非等宽以便中英文混排。

### 页面参数

| 参数 | 值 |
|------|-----|
| **纸张** | Letter（8.5 × 11 in） |
| **页面宽** | 816 px |
| **页面高** | 1056 px |
| **上边距** | 96 px |
| **右边距** | 96 px |
| **下边距** | 96 px |
| **左边距** | 144 px（1.5" 装订侧） |
| **可用内容宽** | 576 px |

### 字体

| 参数 | 值 |
|------|-----|
| **默认字体** | `Courier New`（英文主字体，等宽）+ `Songti SC` / `SimSun`（中文降级，衬线宋体） |
| **默认字号** | `12pt` |
| **行高** | `16 px` |

### 元素排版参数

| 元素 | 左缩进 (px) | 宽度 (px) | 对齐 | 段前 (px) | 段后 (px) | 大写 | 加粗 | 斜体 |
|------|-----------|---------|------|--------|--------|------|------|------|
| scene | 0 | 576 | left | 20 | 12 | ✅ | ✅ | ❌ |
| action | 0 | 576 | left | 0 | 10 | ❌ | ❌ | ❌ |
| character | 192 | 384 | left | 12 | 0 | ✅ | ✅ | ❌ |
| parenthetical | 144 | 240 | left | 0 | 0 | ❌ | ❌ | **✅** |
| dialogue | 96 | 336 | left | 0 | 10 | ❌ | ❌ | ❌ |
| transition | 384 | 192 | right | 14 | 8 | ✅ | ❌ | ❌ |
| shot | 0 | 576 | left | 12 | 8 | ✅ | ✅ | ❌ |
| section | 0 | 576 | left | 24 | 10 | ❌ | ✅ | ❌ |
| note | 0 | 576 | left | 8 | 8 | ❌ | ❌ | ✅ |
| **visual** | 0 | 576 | left | 12 | 8 | ✅ | ✅ | ❌ |
| **sfx** | 0 | 576 | left | 8 | 8 | ✅ | ✅ | ❌ |

**特色**：
- 新增 **`visual`** 元素（`VISUAL:` / `画面：` 前缀）— 描述关键视觉画面，供分镜师参考
- 新增 **`sfx`** 元素（`SFX:` / `音效：` 前缀）— 音效指令，供音效师参考
- `shot` 元素改为**大写加粗**，比真人电影更醒目（动画每个镜头都要单独渲染）
- 场景描述允许**超现实描写**（"人物变成一朵云"），不用像真人剧本考虑实拍成本

---

## 元素识别规则（AI 用来判定每行属于哪类元素）

按优先级从上往下判断：

1. **scene** — 以 `内景/外景/INT./EXT./內景/外景/室内/室外` 开头 + 含时间术语结尾
2. **transition** — 以 `切至/CUT TO/淡入/FADE IN/叠化/DISSOLVE` 等转场词开头（可含冒号）
3. **section** — 以 `第X幕/Act X/幕/Chapter` 开头
4. **shot** — 以镜头术语开头（`特写/CU/POV/SHOT/CLOSE ON`）
5. **parenthetical** — 完全被 `()` / `（）` 包围
6. **character** — 短行（≤ 20 字），无标点，全大写或中文名（3-5 字）+ 冒号
7. **dialogue** — 紧跟 `character` 之后的行
8. **note** — 以 `//` / `[[` / `TODO:` 开头
9. **action** — 兜底：其他所有情况

---

## 页码 (Page Number)

- 位置：**页面右上角**
- 距离顶端：36 px
- 距离右边距：右对齐到 `pageWidth - marginRight`
- 字号：`9pt`
- 字体：跟正文字体一致
- 颜色：`#6b7280`（灰色，不抢戏）

---

## 分页规则

**核心公式**（伪代码）：

```python
def paginate(elements, format):
    pages = []
    current_page = []
    cursor = format.page.marginTop
    max_y = format.page.height - format.page.marginBottom
    
    for element in elements:
        layout = format.elements[element.type]
        is_first = len(current_page) == 0
        height = estimate_height(element, layout, format.fontSize, include_before=not is_first)
        
        if not is_first and cursor + height > max_y:
            pages.append(current_page)
            current_page = []
            cursor = format.page.marginTop
        
        current_page.append(element)
        cursor += height
    
    if current_page:
        pages.append(current_page)
    return pages
```

**元素高度估算**：

```python
def estimate_height(element, layout, fontSize, include_before=True):
    lineHeight = round(fontSize * 96 / 72)
    text_lines = wrap_lines(element.text, layout.width, fontSize)
    return (layout.before if include_before else 0) + len(text_lines) * lineHeight + layout.after
```

**换行规则**：
- 中/日/韩字符宽度 ≈ `fontSize × 96/72`（方块字）
- 拉丁字符宽度 ≈ `fontSize × 0.8`（等宽字体）
- 按空格分词 tokenize；超长行按字符断行

---

## Fountain 语法对照

Fountain 是通用的纯文本剧本格式，AI 可以拿 Fountain 作为**内部中间态**再转成 Word / PDF：

```fountain
INT. LIVING ROOM - DAY                    → scene

Zhang enters, exhausted.                  → action

ZHANG                                     → character
(quietly)                                 → parenthetical
I need coffee.                            → dialogue

CUT TO:                                   → transition
```

## 输出格式选择

Skill 支持 4 种输出形态：

| 形态 | 何时用 | 精度 |
|------|-------|------|
| **`.docx` Word** | 用户要提交/发行/打印/给制作人看 | ⭐⭐⭐⭐⭐ 完整保留所有排版 |
| **`.pdf` PDF** | 用户要打印/存档/发布 | ⭐⭐⭐⭐⭐ 完整保留所有排版 |
| **`.fountain` 纯文本** | 用户要粘贴到 Highland / Slugline 等 Fountain 编辑器 | ⭐⭐⭐ 语义完整，样式重建 |
| **`.fdx` FDX** | 用户要导入 Final Draft | ⭐⭐⭐⭐ Final Draft 内部会按自己规则重排 |

**默认输出**：Word（.docx）。如果用户明确要 PDF 或 Fountain 则按需切换。

导出流程详见：
- `references/export-to-docx.md` — Word 导出（python-docx）
- `references/export-to-pdf.md` — PDF 导出（HTML+CSS）
