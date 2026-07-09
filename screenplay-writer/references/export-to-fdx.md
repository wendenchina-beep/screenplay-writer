# 导出到 FDX（.fdx）—— Final Draft XML

**用途**：生成 FDX 剧本文件（Final Draft XML），可以直接导入 **Final Draft**（好莱坞事实标准剧本软件）继续编辑。

**FDX 是什么**：Final Draft 使用的 XML 格式，也是好莱坞剧本行业**事实标准的交换格式**。几乎所有专业剧本软件都能读写 FDX（Final Draft / Movie Magic Screenwriter / Fade In / KIT Scenarist / Highland 等）。

**用途场景**：
- 用户是 Final Draft 用户，要拿去继续在 FD 里编辑
- 用户要给制片公司/经纪人提交剧本（好莱坞行业几乎都要 FDX 或 PDF）
- 跨编辑器无损交换（Fountain → FDX → Movie Magic 等）

## FDX 完整结构规范

### 顶层结构

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<FinalDraft DocumentType="Script" Template="No" Version="1">
  <TitlePage>
    <Content>
      <Paragraph Type="Title">
        <Text>剧本名</Text>
      </Paragraph>
      <Paragraph Type="Credit">
        <Text>Written by</Text>
      </Paragraph>
      <Paragraph Type="Author">
        <Text>作者</Text>
      </Paragraph>
    </Content>
  </TitlePage>
  <Content>
    <Paragraph Type="Scene Heading"><Text>INT. LIVING ROOM - DAY</Text></Paragraph>
    <Paragraph Type="Action"><Text>...</Text></Paragraph>
    <!-- 更多 Paragraph -->
  </Content>
</FinalDraft>
```

### 段落类型（Paragraph Type 属性）

| FDX Type | 内部元素 | 说明 |
|----------|--------|------|
| `Scene Heading` | scene | 场景标题（INT./EXT.） |
| `Action` | action | 动作/描述 |
| `Character` | character | 角色名（Final Draft 默认全大写） |
| `Parenthetical` | parenthetical | 括号提示（含 `()`） |
| `Dialogue` | dialogue | 对白 |
| `Transition` | transition | 转场（CUT TO: 等） |
| `Shot` | shot | 镜头指令 |
| `General` | section / note / 其他 | 通用段落（无固定样式） |

### 场景编号（可选）

场景标题带编号时：

```xml
<Paragraph Type="Scene Heading" Number="1">
  <SceneProperties Length="1/8" Page="1" Title="">
    <SceneArcBeats/>
  </SceneProperties>
  <Text>INT. LIVING ROOM - DAY</Text>
</Paragraph>
```

`Number` 属性可以是数字（`1`）或带前缀（`A1`, `1A`）。

### 双对话（Dual Dialogue）

Final Draft 用嵌套 `<DualDialogue>` 标签：

```xml
<Paragraph>
  <DualDialogue>
    <Paragraph Type="Character"><Text>ZHANG</Text></Paragraph>
    <Paragraph Type="Dialogue"><Text>Get out.</Text></Paragraph>
    <Paragraph Type="Character"><Text>LI</Text></Paragraph>
    <Paragraph Type="Dialogue"><Text>You get out.</Text></Paragraph>
  </DualDialogue>
</Paragraph>
```

### 修订标记（Revisions）

```xml
<Paragraph Type="Action">
  <Text RevisionID="1">修改后的文字</Text>
</Paragraph>
```

`RevisionID` 对应 `<Revisions>` 段里定义的修订色（蓝/粉/黄/绿等）。

### 强调（Emphasis）

在 `<Text>` 元素上加属性：

```xml
<Text Style="Italic">斜体</Text>
<Text Style="Bold">加粗</Text>
<Text Style="Underline">下划线</Text>
<Text Style="Italic+Bold">加粗斜体</Text>
```

### 注释和备注

```xml
<Paragraph Type="Action">
  <Text>正文</Text>
  <ScriptNote>
    <Paragraph>
      <Text>这是给自己看的注释</Text>
    </Paragraph>
  </ScriptNote>
</Paragraph>
```

### XML 转义

FDX 是标准 XML，特殊字符必须转义：

| 原字符 | 转义 |
|--------|------|
| `&` | `&amp;` |
| `<` | `&lt;` |
| `>` | `&gt;` |
| `"` | `&quot;` |
| `'` | `&apos;` |

## Python 生成代码模板

```python
from xml.sax.saxutils import escape as xml_escape

# ---- 数据结构 ----
# elements = [{"type": "scene", "text": "..."}, ...]
# meta = {"title": "...", "author": "...", "credit": "Written by"}

ELEMENT_TO_FDX = {
    "scene": "Scene Heading",
    "action": "Action",
    "character": "Character",
    "parenthetical": "Parenthetical",
    "dialogue": "Dialogue",
    "transition": "Transition",
    "shot": "Shot",
    "section": "General",
    "note": "General",
}


def build_fdx(elements, meta=None) -> str:
    """
    生成 Final Draft XML (.fdx) 剧本文件
    
    elements: [{"type": "scene", "text": "..."}, ...]
    meta: {"title": ..., "author": ..., "credit": ...}
    
    返回：完整 FDX XML 字符串
    """
    lines = ['<?xml version="1.0" encoding="UTF-8" standalone="no" ?>']
    lines.append('<FinalDraft DocumentType="Script" Template="No" Version="1">')
    
    # 标题页
    lines.append("  <TitlePage>")
    lines.append("    <Content>")
    if meta:
        if meta.get("title"):
            lines.append(f'      <Paragraph Type="Title"><Text>{xml_escape(meta["title"])}</Text></Paragraph>')
        if meta.get("credit"):
            lines.append(f'      <Paragraph Type="Credit"><Text>{xml_escape(meta["credit"])}</Text></Paragraph>')
        if meta.get("author"):
            lines.append(f'      <Paragraph Type="Author"><Text>{xml_escape(meta["author"])}</Text></Paragraph>')
        if meta.get("source"):
            lines.append(f'      <Paragraph Type="Source"><Text>{xml_escape(meta["source"])}</Text></Paragraph>')
        if meta.get("draft_date"):
            lines.append(f'      <Paragraph Type="Draft Date"><Text>{xml_escape(meta["draft_date"])}</Text></Paragraph>')
        if meta.get("contact"):
            lines.append(f'      <Paragraph Type="Contact"><Text>{xml_escape(meta["contact"])}</Text></Paragraph>')
    lines.append("    </Content>")
    lines.append("  </TitlePage>")
    
    # 正文
    lines.append("  <Content>")
    for el in elements:
        etype = el["type"]
        text = el["text"]
        fdx_type = ELEMENT_TO_FDX.get(etype, "Action")
        
        # 括号提示确保有括号
        if etype == "parenthetical" and not text.startswith("("):
            text = "(" + text + ")"
        # 角色名 Final Draft 会自动大写显示，但内部存原文
        # 转场词也是内部存原文
        
        lines.append(f'    <Paragraph Type="{fdx_type}"><Text>{xml_escape(text)}</Text></Paragraph>')
    
    lines.append("  </Content>")
    lines.append("</FinalDraft>")
    lines.append("")  # trailing newline
    
    return "\n".join(lines)


def parse_fdx(fdx_content: str) -> tuple[list, dict]:
    """
    解析 FDX 文件 → (elements, meta)
    """
    from xml.etree import ElementTree as ET
    
    root = ET.fromstring(fdx_content)
    
    # 反向映射
    fdx_to_element = {v: k for k, v in ELEMENT_TO_FDX.items()}
    
    # 解析 meta
    meta = {}
    title_page = root.find("TitlePage/Content")
    if title_page is not None:
        for para in title_page.findall("Paragraph"):
            ptype = para.get("Type", "")
            text_el = para.find("Text")
            if text_el is None or not text_el.text:
                continue
            if ptype == "Title":
                meta["title"] = text_el.text
            elif ptype == "Credit":
                meta["credit"] = text_el.text
            elif ptype == "Author":
                meta["author"] = text_el.text
            elif ptype == "Source":
                meta["source"] = text_el.text
            elif ptype == "Draft Date":
                meta["draft_date"] = text_el.text
            elif ptype == "Contact":
                meta["contact"] = text_el.text
    
    # 解析 elements
    elements = []
    content = root.find("Content")
    if content is not None:
        for para in content.findall("Paragraph"):
            ptype = para.get("Type", "Action")
            etype = fdx_to_element.get(ptype, "action")
            # 拼接所有 Text 子元素
            text = "".join(t.text or "" for t in para.findall("Text"))
            if text.strip():
                elements.append({"type": etype, "text": text.strip()})
    
    return elements, meta


# ---- 使用示例 ----
if __name__ == "__main__":
    from datetime import date
    
    sample_meta = {
        "title": "Zhang and Li",
        "credit": "Written by",
        "author": "Wenden",
        "draft_date": str(date.today()),
        "contact": "wendenchina@gmail.com",
    }
    sample_elements = [
        {"type": "scene",         "text": "INT. ZHANG'S LIVING ROOM - DAY"},
        {"type": "action",        "text": "Zhang pushes the door open. Boxes everywhere. He freezes. Li is packing, doesn't look up."},
        {"type": "character",     "text": "ZHANG"},
        {"type": "dialogue",      "text": "You're leaving?"},
        {"type": "character",     "text": "LI"},
        {"type": "parenthetical", "text": "coldly"},
        {"type": "dialogue",      "text": "The contract says the 15th."},
        {"type": "character",     "text": "ZHANG"},
        {"type": "dialogue",      "text": "Can we talk?"},
        {"type": "action",        "text": "Li stops packing. Takes a deep breath."},
        {"type": "character",     "text": "LI"},
        {"type": "dialogue",      "text": "Talk about what? Three years, and when have you ever really listened?"},
        {"type": "transition",    "text": "CUT TO:"},
        {"type": "scene",         "text": "INT. CAFE - NIGHT"},
        {"type": "action",        "text": "A month later. Zhang sits alone in a corner. A cold, untouched coffee. The door opens. Li walks in."},
        {"type": "shot",          "text": "CLOSE ON: Zhang's fingers tremble slightly."},
        {"type": "character",     "text": "LI"},
        {"type": "dialogue",      "text": "You wanted to talk. I'm here."},
        {"type": "transition",    "text": "CUT TO:"},
    ]
    
    output = build_fdx(sample_elements, sample_meta)
    with open("output.fdx", "w", encoding="utf-8") as f:
        f.write(output)
    print(output)
```

## 生成的 FDX 文件示例

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no" ?>
<FinalDraft DocumentType="Script" Template="No" Version="1">
  <TitlePage>
    <Content>
      <Paragraph Type="Title"><Text>Zhang and Li</Text></Paragraph>
      <Paragraph Type="Credit"><Text>Written by</Text></Paragraph>
      <Paragraph Type="Author"><Text>Wenden</Text></Paragraph>
      <Paragraph Type="Draft Date"><Text>2026-07-09</Text></Paragraph>
      <Paragraph Type="Contact"><Text>wendenchina@gmail.com</Text></Paragraph>
    </Content>
  </TitlePage>
  <Content>
    <Paragraph Type="Scene Heading"><Text>INT. ZHANG&apos;S LIVING ROOM - DAY</Text></Paragraph>
    <Paragraph Type="Action"><Text>Zhang pushes the door open. Boxes everywhere. He freezes. Li is packing, doesn&apos;t look up.</Text></Paragraph>
    <Paragraph Type="Character"><Text>ZHANG</Text></Paragraph>
    <Paragraph Type="Dialogue"><Text>You&apos;re leaving?</Text></Paragraph>
    <Paragraph Type="Character"><Text>LI</Text></Paragraph>
    <Paragraph Type="Parenthetical"><Text>(coldly)</Text></Paragraph>
    <Paragraph Type="Dialogue"><Text>The contract says the 15th.</Text></Paragraph>
    <!-- ... 更多段落 ... -->
  </Content>
</FinalDraft>
```

## 支持的编辑器（都能直接打开这个 .fdx 文件）

| 编辑器 | 平台 | 官网 | 备注 |
|--------|------|------|------|
| **Final Draft** | macOS/Win | https://www.finaldraft.com/ | 好莱坞标准（付费，$249） |
| **Fade In** | macOS/Win/Linux | https://www.fadeinpro.com/ | 支持 FDX/Fountain 双向（$79） |
| **Movie Magic Screenwriter** | Win/Mac | https://www.entertainmentpartners.com/products/movie-magic-screenwriter/ | 制片行业标准（付费） |
| **Highland 2** | macOS | https://quoteunquoteapps.com/highland/ | 可导入 FDX（付费） |
| **KIT Scenarist** | Win/macOS/Linux | https://kitscenarist.ru/ | 支持 FDX 导入导出（免费） |
| **DubScript** | Android | Google Play | 移动端 FDX 阅读 |

## AI 应用步骤

用户说「导出成 Final Draft 文件」/「导出 FDX」时，AI 应该：

1. **准备 elements 数组** + 可选 meta 元信息
2. **调用 build_fdx()** 生成完整 FDX XML 字符串
3. **写入 `.fdx` 文件**（保存到用户工作目录，如 `~/Downloads/zhang-li.fdx`）
4. **用 `<media type="file" src="/absolute/path/output.fdx" />` 发送给用户**
5. **提醒用户**：这个文件可以直接用 Final Draft / Fade In / Movie Magic 等打开继续编辑

## 中文剧本 FDX 特殊处理

Final Draft 从 11 开始支持 Unicode（包括中文），但角色名默认按大写规则渲染。**对中文剧本**：

- 场景标题保留中文即可：`<Paragraph Type="Scene Heading"><Text>内景. 客厅 - 日</Text></Paragraph>`
- 角色名可用中文：`<Paragraph Type="Character"><Text>张三</Text></Paragraph>`（Final Draft 会显示原样，中文没大小写概念）
- 括号提示用中文全角括号也可以：`（冷冷）`
- 转场词用中文：`切至：` 也没问题

**但注意**：如果 Final Draft 使用者的界面语言是英文，Final Draft 会用英文默认样式渲染这些中文段落（字体可能是 Courier，中文显示效果不好）。**推荐**：
- 目标发行海外 → 提前用 Mode 2 术语英化再导 FDX
- 目标发行中国大陆 → 用 Mode 1 输出 Word（.docx）更合适，中文排版更好

## 常见问题

### Q1: 生成的 FDX 在 Final Draft 里打开报"文件损坏"

**A**：多半是 XML 语法错误。检查：
1. XML 声明必须是 `<?xml version="1.0" encoding="UTF-8" standalone="no" ?>`（`standalone="no"` 不能少）
2. 特殊字符必须转义（`&` `<` `>` `"` `'`）
3. 顶层结构必须完整：`<FinalDraft>` → `<TitlePage>` + `<Content>` → 关闭标签
4. 每个 `<Paragraph>` 必须有 `Type` 属性和 `<Text>` 子元素

### Q2: 场景编号自动生成

**A**：Final Draft 会在打开时自动重排场景编号。也可以显式指定：`<Paragraph Type="Scene Heading" Number="1">`。

### Q3: 双对话怎么导出

**A**：用嵌套 `<DualDialogue>`，参考"FDX 完整结构规范"里的例子。目前基础版本不含此功能，如需要可扩展 `build_fdx()`。

### Q4: 保留修订标记

**A**：需要在文档顶部加 `<Revisions>` 段定义颜色，然后在 `<Text RevisionID="N">` 上引用。属于高级功能，基础版本未含。

### Q5: Fountain 直接用 Highland 转 FDX 更好还是本 Skill 直接生成 FDX 更好

**A**：**都行**：
- 用 Highland 转：Highland 处理边缘情况（双对话、修订、字幕）比脚本完整
- 用本 Skill 直接生成：一步到位，不需要额外软件

## 参考

- Final Draft FDX 格式官方文档：https://helpcenter.finaldraft.com/hc/en-us/articles/360045290093
- FDX XML DTD 示例：https://github.com/pmarcelo/pyfountain
- Final Draft 双对话 XML 结构：https://kb.finaldraft.com/
