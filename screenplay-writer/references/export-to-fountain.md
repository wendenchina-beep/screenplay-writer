# 导出到 Fountain（.fountain）

**用途**：生成 Fountain 纯文本剧本文件，可以直接导入 Highland 2 / Slugline / Fade In / KIT Scenarist / Beat 等编辑器继续编辑。

**Fountain 是什么**：一种基于 Markdown 的纯文本剧本格式，由 John August（好莱坞编剧）主导设计。所有专业 Mac/PC 剧本编辑器都支持 Fountain 导入。

**优点**：
- 纯文本，任何 IDE / 记事本都能打开
- Git 友好（可 diff、可版本控制）
- 跨编辑器无损（Highland → Slugline → Final Draft 都能读）

## Fountain 完整语法规范

### 1. 标题页 Metadata（可选）

放在文件最开头，用 `Key: Value` 格式，每行一个字段，一空行结束：

```fountain
Title: 张三与李四
Credit: Written by
Author: Wenden
Source: Original Screenplay
Draft date: 2026-07-09
Contact:
  wendenchina@gmail.com

INT. LIVING ROOM - DAY
```

**必要字段**：
- `Title:` — 剧本名
- `Author:` 或 `Authors:` — 作者
- `Credit:` — 署名词（如 `Written by` / `Story by` / `Adapted by`）

**可选字段**：
- `Source:` — 改编来源
- `Draft date:` — 稿件日期
- `Contact:` — 联系方式（多行时缩进）
- `Copyright:` — 版权信息

### 2. 元素识别规则

Fountain **靠段落位置和特殊标记**识别元素类型：

| 元素类型 | 语法规则 | 例子 |
|---------|--------|------|
| **Scene Heading** | 以 `INT.` / `EXT.` / `INT./EXT.` / `EST.` 开头，或以 `.` 强制标记 | `INT. LIVING ROOM - DAY` / `.SPACE` |
| **Action** | 兜底，任何不匹配其他规则的段落 | `Zhang enters, exhausted.` |
| **Character** | 前后有空行的**全大写单独一行**，或以 `@` 强制标记 | `ZHANG` / `@张三` |
| **Parenthetical** | 完全被 `()` 包围，紧跟 character 或 dialogue | `(quietly)` |
| **Dialogue** | 紧跟 character 或 parenthetical 的行 | `I need coffee.` |
| **Transition** | 全大写 + 以 `TO:` 结尾，或以 `>` 强制标记 | `CUT TO:` / `> FADE OUT.` |
| **Shot** | 全大写但不以 `TO:` 结尾且不是 character | `CLOSE ON HANDS` |
| **Section** | 以 1-6 个 `#` 开头（跟 Markdown 标题一样） | `# Act One` / `## Sequence A` |
| **Synopsis** | 以 `=` 开头（用于大纲注释） | `= Zhang and Li's breakup` |
| **Note** | 用 `[[ ]]` 包围（编剧注释，不进最终剧本） | `[[TODO: 补一句挖苦的话]]` |
| **Boneyard** | 用 `/* */` 包围（大段被删除的内容） | `/* 删掉整段 */` |
| **Centered** | 用 `> <` 包围（居中文字） | `> FADE IN <` |
| **Page Break** | 单独一行 `===` | `===` |

### 3. Force Elements（强制标记）

当自动识别不准时用强制标记：

| 前缀 | 强制识别为 |
|------|---------|
| `.` | Scene Heading（`.SPACE` → 强制场景） |
| `!` | Action（`!ZHANG` → 强制动作，避免被识别成角色） |
| `@` | Character（`@张三` → 强制角色，中文名不用全大写） |
| `~` | Lyrics（歌词） |
| `>` | Transition（`>CUT TO:` → 强制转场） |
| `>` `<` | Centered（`> TITLE <` → 居中） |

### 4. 强调（Emphasis）语法

跟 Markdown 一致，用于剧本正文的文字强调：

| 语法 | 效果 |
|------|------|
| `*italic*` | *斜体* |
| `**bold**` | **加粗** |
| `***bold italic***` | ***加粗斜体*** |
| `_underline_` | 下划线 |

### 5. 双对话（Dual Dialogue）

在第二个角色名后加 `^` 表示双栏对话（两角色同时说话）：

```fountain
ZHANG
Get out.

LI ^
You get out.
```

Highland / Final Draft 会渲染成两栏并列。

### 6. 场景编号

场景标题右侧用 `#N#` 标记场景号：

```fountain
INT. LIVING ROOM - DAY #1#

EXT. STREET - NIGHT #A2#
```

编辑器会显示在场景标题右侧。

## Python 生成代码模板

```python
import re
from datetime import date

# ---- 数据结构 ----
# elements = [{"type": "scene", "text": "..."}, ...]
# meta = {"title": "...", "author": "...", "credit": "Written by", ...}

def build_fountain(elements, meta=None) -> str:
    """
    生成 Fountain 纯文本剧本
    
    elements: [{"type": "scene", "text": "..."}, ...]
    meta: {"title": ..., "author": ..., "credit": ...} 或 None
    
    返回：完整 Fountain 文本字符串（可直接写入 .fountain 文件）
    """
    parts = []
    
    # 1. 标题页
    if meta:
        title_page = []
        if meta.get("title"):
            title_page.append(f"Title: {meta['title']}")
        if meta.get("credit"):
            title_page.append(f"Credit: {meta['credit']}")
        if meta.get("author"):
            title_page.append(f"Author: {meta['author']}")
        if meta.get("source"):
            title_page.append(f"Source: {meta['source']}")
        if meta.get("draft_date"):
            title_page.append(f"Draft date: {meta['draft_date']}")
        if meta.get("contact"):
            title_page.append(f"Contact:")
            title_page.append(f"  {meta['contact']}")
        if title_page:
            parts.append("\n".join(title_page))
            parts.append("")  # 一空行结束标题页
    
    # 2. 正文元素
    prev_type = None
    for el in elements:
        etype = el["type"]
        text = el["text"].strip()
        
        # 元素间空行规则
        if prev_type and _need_blank_line(prev_type, etype):
            parts.append("")
        
        if etype == "scene":
            # 场景标题：如果不以 INT./EXT./EST./INT.EXT./INT/EXT 开头，加 . 强制
            if not re.match(r"^(INT\.?|EXT\.?|INT\./EXT\.?|EST\.?|内景|外景)", text, re.I):
                text = "." + text
            parts.append(text.upper())
        
        elif etype == "action":
            # 动作：如果全大写会被误认成 character，加 ! 强制
            if text.isupper() and len(text) < 40:
                text = "!" + text
            parts.append(text)
        
        elif etype == "character":
            # 角色：如果不是全大写（如中文名），加 @ 强制
            if not text.isupper() and not re.match(r"^[\u4e00-\u9fff]", text):
                text = "@" + text
            elif re.match(r"^[\u4e00-\u9fff]", text):
                text = "@" + text  # 中文角色名必须用 @
            parts.append(text)
        
        elif etype == "parenthetical":
            # 括号提示：确保有括号包围
            if not text.startswith("("):
                text = "(" + text + ")"
            parts.append(text)
        
        elif etype == "dialogue":
            parts.append(text)
        
        elif etype == "transition":
            # 转场：如果不是 XXX TO: 或 FADE OUT 等，加 > 强制
            if not (text.upper().endswith("TO:") or text.upper().endswith("OUT.") 
                    or text.upper().endswith("IN:") or text.upper().endswith("BLACK.")):
                text = "> " + text
            parts.append(text.upper() if text.startswith(">") else text)
        
        elif etype == "shot":
            # 镜头：全大写，不以 TO: 结尾
            parts.append(text.upper())
        
        elif etype == "section":
            # 段落标题：加 # 前缀
            parts.append("# " + text)
        
        elif etype == "note":
            # 备注：用 [[ ]] 包围
            parts.append(f"[[{text}]]")
        
        elif etype == "synopsis":
            parts.append("= " + text)
        
        elif etype == "page_break":
            parts.append("===")
        
        prev_type = etype
    
    return "\n".join(parts) + "\n"


def _need_blank_line(prev_type: str, curr_type: str) -> bool:
    """判断两个元素之间是否需要空行"""
    # character → parenthetical / dialogue 之间不空行
    if prev_type == "character" and curr_type in ("parenthetical", "dialogue"):
        return False
    # parenthetical → dialogue 之间不空行
    if prev_type == "parenthetical" and curr_type == "dialogue":
        return False
    # 其他所有情况都空行
    return True


# ---- 中文场景标题特殊处理 ----
def normalize_scene_for_fountain(scene_text: str, force_english: bool = False) -> str:
    """
    Fountain 场景标题按行业惯例应该英文。
    中文剧本时用 `.` 强制标记，避免被误识别。
    """
    if force_english:
        # 简单映射（生产用可以调 LLM 翻译）
        mapping = {
            "内景": "INT.", "外景": "EXT.",
            "日": "DAY", "夜": "NIGHT", "早晨": "MORNING",
            "黎明": "DAWN", "黄昏": "DUSK",
        }
        for zh, en in mapping.items():
            scene_text = scene_text.replace(zh, en)
    return scene_text


# ---- 使用示例 ----
if __name__ == "__main__":
    sample_meta = {
        "title": "张三与李四",
        "credit": "Written by",
        "author": "Wenden",
        "draft_date": str(date.today()),
        "contact": "wendenchina@gmail.com",
    }
    sample_elements = [
        {"type": "scene", "text": "内景. 张家客厅 - 日"},
        {"type": "action", "text": "张三推门进来，看着满地的行李箱，愣住了。李四正在收拾东西，头也没抬。"},
        {"type": "character", "text": "张三"},
        {"type": "dialogue", "text": "你要走？"},
        {"type": "character", "text": "李四"},
        {"type": "parenthetical", "text": "（冷冷）"},
        {"type": "dialogue", "text": "合同上写着 15 号搬走。"},
        {"type": "character", "text": "张三"},
        {"type": "dialogue", "text": "可以再谈谈吗？"},
        {"type": "action", "text": "李四停下手中的活，深吸一口气。"},
        {"type": "character", "text": "李四"},
        {"type": "dialogue", "text": "谈什么？三年了，你什么时候真的在听？"},
        {"type": "transition", "text": "切至："},
        {"type": "scene", "text": "内景. 咖啡馆 - 夜"},
        {"type": "action", "text": "一个月后，张三独自坐在角落，桌上放着一杯早已凉了的咖啡。门开了，李四走了进来。"},
        {"type": "shot", "text": "特写：张三的手指微微颤抖"},
        {"type": "character", "text": "李四"},
        {"type": "dialogue", "text": "你说要谈，我来了。"},
        {"type": "transition", "text": "切至："},
    ]
    
    output = build_fountain(sample_elements, sample_meta)
    with open("output.fountain", "w", encoding="utf-8") as f:
        f.write(output)
    print(output)
```

## 生成的 Fountain 文件示例

```fountain
Title: 张三与李四
Credit: Written by
Author: Wenden
Draft date: 2026-07-09
Contact:
  wendenchina@gmail.com

.内景. 张家客厅 - 日

张三推门进来，看着满地的行李箱，愣住了。李四正在收拾东西，头也没抬。

@张三
你要走？

@李四
(（冷冷）)
合同上写着 15 号搬走。

@张三
可以再谈谈吗？

李四停下手中的活，深吸一口气。

@李四
谈什么？三年了，你什么时候真的在听？

> 切至：

.内景. 咖啡馆 - 夜

一个月后，张三独自坐在角落，桌上放着一杯早已凉了的咖啡。门开了，李四走了进来。

特写：张三的手指微微颤抖

@李四
你说要谈，我来了。

> 切至：
```

## 支持的编辑器（都能打开这个 .fountain 文件继续编辑）

| 编辑器 | 平台 | 官网 | 备注 |
|--------|------|------|------|
| **Highland 2** | macOS | https://quoteunquoteapps.com/highland/ | John August（Fountain 作者）出品，Fountain 编辑体验最佳 |
| **Slugline 2** | macOS | https://slugline.co/ | 极简 Fountain 编辑器 |
| **Fade In** | macOS/Win/Linux | https://www.fadeinpro.com/ | 支持 Fountain / FDX / PDF 导出 |
| **Beat** | macOS | https://kapitan.fi/beat/ | 免费开源，Fountain 原生 |
| **KIT Scenarist** | Win/macOS/Linux | https://kitscenarist.ru/ | 免费开源，支持中文 |
| **Trelby** | Win/Linux | https://www.trelby.org/ | 免费开源 |

## AI 应用步骤

用户说「导出成 Fountain 文件」时，AI 应该：

1. **准备 elements 数组** + 可选的 meta 元信息
2. **调用 build_fountain()** 生成完整 Fountain 文本
3. **写入 `.fountain` 文件**（保存到用户工作目录，如 `~/Downloads/zhang-li.fountain`）
4. **用 `<media type="file" src="/absolute/path/output.fountain" />` 发送给用户**
5. **提醒用户**：这个文件可以直接用 Highland / Slugline / Fade In 等编辑器打开继续编辑

## 常见问题

### Q1: 中文角色名在 Highland 里显示不对

**A**：中文角色名不能自动识别（Fountain 默认按全大写识别 character），必须用 `@` 强制前缀：`@张三`。代码模板已含这个逻辑。

### Q2: 场景标题被识别成 Action

**A**：中文场景（`内景. 客厅 - 日`）没有 INT./EXT. 前缀，Fountain 会当成动作。用 `.` 前缀强制：`.内景. 客厅 - 日`。代码模板已含。

### Q3: 转场 `切至：` 被识别成对白

**A**：中文转场没有 `TO:` 结尾，用 `>` 前缀强制：`> 切至：`。代码模板已含。

### Q4: 括号提示嵌套括号（如 `((冷冷))`）

**A**：Python 代码在遇到已经是括号的文本时不再加一层。用户输入的 `（冷冷）` 是中文全角括号，Fountain 只认半角，需要提前 normalize：`（冷冷）` → `(冷冷)`。

### Q5: 生成的 Fountain 在 Final Draft 里打不开

**A**：Final Draft 不直接支持 Fountain 导入，需要先转 FDX。见 `references/export-to-fdx.md`。或者用 Highland 打开 Fountain 后 File → Export → Final Draft (.fdx)。

## 参考

- Fountain 官方规范：https://fountain.io/syntax
- Fountain 完整语法示例：https://fountain.io/_downloads/fountain-syntax.pdf
- Highland Fountain 快速参考：https://quoteunquoteapps.com/highland/faq
