# Mode 1: 纯格式转换

**用户诉求**：我有剧本，想改成标准的好莱坞/东亚/舞台剧/广播剧格式，或者转成 FDX/Fountain/Markdown。

**Skill 做什么**：只做**排版重排 + 元素归类**，不改任何一个字的剧情。

## 触发关键词

- "改成标准格式"
- "转成 FDX"
- "转成 Fountain"
- "好莱坞标准排版"
- "东亚影视格式"
- "剧本格式化"
- "转成 Final Draft"

## 4 步流程

### Step 1: 识别现有格式

拿到用户文本，先判断当前是哪种：

| 特征 | 判定为 |
|------|--------|
| 含 `<FinalDraft>` XML 标签 | FDX |
| 首行是 `Title:` 且用缩进标识元素 | Fountain |
| 场景标题居中 + 动作斜体 | 舞台剧 |
| Courier 字体 + 大量大写 | 好莱坞 |
| A4 + 中文加粗场景 | 东亚 |
| 纯散文无格式 | 无格式（需要 Skill 完全推断） |

### Step 2: 元素归类

对每一行文本，按 `references/vocab-format-specs.md` §"元素识别规则" 归类：

```python
def classify_line(line: str) -> str:
    line = line.strip()
    if not line: return "empty"
    if is_scene_heading(line): return "scene"
    if is_transition(line): return "transition"
    if is_section(line): return "section"
    if is_shot(line): return "shot"
    if is_parenthetical(line): return "parenthetical"
    if is_character(line): return "character"
    if is_dialogue(line, prev_line_type): return "dialogue"
    if is_note(line): return "note"
    return "action"  # 兜底
```

### Step 3: 按目标格式重排

`hub_read` `references/vocab-format-specs.md` 拿到目标格式的排版参数，对每个元素应用：
- 左缩进 `marginLeft`
- 宽度 `width`
- 对齐方式 `align`
- 大小写 / 加粗 / 斜体

### Step 4: 输出

用户可选输出形式：

**A. Markdown 表格**（便于人类阅读、粘贴到 Notion/飞书文档）：
```markdown
| # | 类型 | 内容 |
|---|------|------|
| 1 | scene | 内景. 客厅 - 日 |
| 2 | action | 张三推门而入，气喘吁吁。 |
| 3 | character | 张三 |
| 4 | parenthetical | （压低声音） |
| 5 | dialogue | 我需要一杯咖啡。 |
```

**B. Fountain 纯文本**（便于粘贴到 Fountain 编辑器）：
```fountain
INT. LIVING ROOM - DAY

Zhang enters, exhausted.

ZHANG
(quietly)
I need coffee.

CUT TO:
```

**C. FDX XML**（便于导入 Final Draft）：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<FinalDraft DocumentType="Script" Template="No" Version="1">
  <TitlePage>
    <Content>
      <Paragraph Type="Title"><Text>标题</Text></Paragraph>
    </Content>
  </TitlePage>
  <Content>
    <Paragraph Type="Scene Heading"><Text>INT. LIVING ROOM - DAY</Text></Paragraph>
    <Paragraph Type="Action"><Text>Zhang enters, exhausted.</Text></Paragraph>
    <Paragraph Type="Character"><Text>ZHANG</Text></Paragraph>
    <Paragraph Type="Parenthetical"><Text>(quietly)</Text></Paragraph>
    <Paragraph Type="Dialogue"><Text>I need coffee.</Text></Paragraph>
    <Paragraph Type="Transition"><Text>CUT TO:</Text></Paragraph>
  </Content>
</FinalDraft>
```

## FDX 结构参考

FDX（Final Draft XML）是行业标准剧本文件格式：

- **根**：`<FinalDraft>` 含 `DocumentType="Script"` 属性
- **标题页**：`<TitlePage><Content><Paragraph Type="Title|Credit|Source">...`
- **正文**：`<Content><Paragraph Type="Scene Heading|Action|Character|Parenthetical|Dialogue|Transition|Shot|General">`
- **XML 转义**：`& → &amp;` / `< → &lt;` / `> → &gt;` / `" → &quot;` / `' → &apos;`

FDX 元素类型 → 内部元素类型映射：

| FDX Type | 内部 ID |
|----------|--------|
| Scene Heading | scene |
| Action | action |
| Character | character |
| Parenthetical | parenthetical |
| Dialogue | dialogue |
| Transition | transition |
| Shot | shot |
| General | action / section / note（按内容判断） |

## Fountain 语法参考

Fountain 是通用的纯文本剧本格式（GitHub / Highland 等编辑器支持）：

| Fountain 语法 | 元素类型 |
|-------------|--------|
| `INT./EXT.` 开头 | scene |
| `>` 结尾 + 大写 | transition（如 `CUT TO:>`） |
| 全大写单独一行 | character |
| `()` 包围 | parenthetical |
| 紧跟 character 的行 | dialogue |
| `#` 开头 | section |
| `[[...]]` | note |
| 其他 | action |

## Review Checkpoint

转换完让用户看 before / after 对照：

- 检查元素归类是否正确（明显错的告诉 Skill 手动改）
- 检查排版参数（缩进、大小写）
- 检查 FDX/Fountain 语法是否合法（用在线校验器）

## 常见问题

- **动作和对白混淆**：模型看到"张三：'我走了'"可能拆成 `dialogue`，但整段有可能是 `action` 引用别人的话。让用户手动确认关键节点。
- **多语言混排**：一段剧本中英混用时，先按主要语言判定，再局部保留另一种语言原文。
- **场景标题误判**：如果内容里出现"内景"两个字但不是场景标题（比如动作描写里写"看内景装修"），可能被误判。规则加严：场景标题必须以 `内景` 开头 + 含时间术语结尾。
