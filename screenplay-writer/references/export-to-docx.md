# 导出到 Word 文档（.docx）

**用途**：把 Skill 处理好的剧本按 4 种格式规范生成 Word 文档，保留字体、字号、边距、缩进、大小写、加粗、斜体全部样式。

**技术栈**：Python + `python-docx` 库（或 AI 用 docx skill 也行）

## 单位换算

python-docx 的原生单位是 EMU，但常用封装：

```python
from docx.shared import Pt, Inches, RGBColor, Emu

# 从 CSS px (@96 DPI) 换算到 python-docx 单位
def px_to_pt(px: float) -> float:
    return px * 72 / 96      # 12 px = 9 pt

def px_to_inches(px: float) -> float:
    return px / 96           # 96 px = 1 inch
```

规范里所有 `px` 值都能通过上面两个函数换到 python-docx 需要的单位。

## 通用生成模板

```python
from docx import Document
from docx.shared import Pt, Inches, RGBColor, Cm
from docx.enum.text import WD_ALIGN_PARAGRAPH, WD_LINE_SPACING
from docx.oxml.ns import qn
from docx.oxml import OxmlElement

# 5 种格式的完整规范（对齐 vocab-format-specs.md）
FORMATS = {
    "hollywood": {
        "page": {"kind": "letter", "width_in": 8.5, "height_in": 11,
                 "top_in": 1.0, "right_in": 1.0, "bottom_in": 1.0, "left_in": 1.5},
        "font": {"name": "Courier New", "size_pt": 12},
        "elements": {
            # 数值直接对应剧本行业公开标准的英寸值（× 96 = px）
            "scene":         {"left_px": 0,   "align": "left",   "before_px": 32, "after_px": 0, "upper": True,  "bold": True},
            "action":        {"left_px": 0,   "align": "left",   "before_px": 16, "after_px": 0},
            "character":     {"left_px": 192, "align": "center", "before_px": 16, "after_px": 0, "upper": True},   # 2.0"
            "parenthetical": {"left_px": 144, "align": "left",   "before_px": 0,  "after_px": 0},                  # 1.5"
            "dialogue":      {"left_px": 96,  "align": "left",   "before_px": 0,  "after_px": 0},                  # 1.0"
            "transition":    {"left_px": 384, "align": "right",  "before_px": 16, "after_px": 0, "upper": True},   # 4.0"
            "shot":          {"left_px": 0,   "align": "left",   "before_px": 16, "after_px": 0, "upper": True,  "bold": True},
            "section":       {"left_px": 0,   "align": "left",   "before_px": 32, "after_px": 0, "bold": True},
            "note":          {"left_px": 0,   "align": "left",   "before_px": 16, "after_px": 0, "italic": True},
        },
    },
    "eastAsia": {
        "page": {"kind": "a4", "width_in": 8.27, "height_in": 11.69,
                 "top_in": 0.79, "right_in": 0.75, "bottom_in": 0.79, "left_in": 0.75},
        "font": {"name": "Microsoft YaHei", "size_pt": 12},
        "elements": {
            "scene":         {"left_px": 0,   "align": "left",  "before_px": 16, "after_px": 10, "bold": True},
            "action":        {"left_px": 0,   "align": "left",  "before_px": 0,  "after_px": 10},
            "character":     {"left_px": 0,   "align": "left",  "before_px": 10, "after_px": 0,  "bold": True},
            "parenthetical": {"left_px": 125, "align": "left",  "before_px": 0,  "after_px": 0},
            "dialogue":      {"left_px": 125, "align": "left",  "before_px": 0,  "after_px": 8},
            "transition":    {"left_px": 470, "align": "right", "before_px": 14, "after_px": 8},
            "shot":          {"left_px": 0,   "align": "left",  "before_px": 12, "after_px": 8, "bold": True},
            "section":       {"left_px": 0,   "align": "left",  "before_px": 20, "after_px": 8, "bold": True},
            "note":          {"left_px": 0,   "align": "left",  "before_px": 8,  "after_px": 8, "italic": True},
        },
    },
    # stage 和 audio 同样按 vocab-format-specs.md 表格补全
    "animation": {
        "page": {"kind": "letter", "width_in": 8.5, "height_in": 11,
                 "top_in": 1.0, "right_in": 1.0, "bottom_in": 1.0, "left_in": 1.5},
        "font": {"name": "Courier New", "size_pt": 12},
        "elements": {
            "scene":         {"left_px": 0,   "align": "left",   "before_px": 20, "after_px": 12, "upper": True, "bold": True},
            "action":        {"left_px": 0,   "align": "left",   "before_px": 0,  "after_px": 10},
            "character":     {"left_px": 192, "align": "left",   "before_px": 12, "after_px": 0,  "upper": True, "bold": True},
            "parenthetical": {"left_px": 144, "align": "left",   "before_px": 0,  "after_px": 0,  "italic": True},
            "dialogue":      {"left_px": 96,  "align": "left",   "before_px": 0,  "after_px": 10},
            "transition":    {"left_px": 384, "align": "right",  "before_px": 14, "after_px": 8,  "upper": True},
            "shot":          {"left_px": 0,   "align": "left",   "before_px": 12, "after_px": 8,  "upper": True, "bold": True},
            "section":       {"left_px": 0,   "align": "left",   "before_px": 24, "after_px": 10, "bold": True},
            "note":          {"left_px": 0,   "align": "left",   "before_px": 8,  "after_px": 8,  "italic": True},
            "visual":        {"left_px": 0,   "align": "left",   "before_px": 12, "after_px": 8,  "upper": True, "bold": True},
            "sfx":           {"left_px": 0,   "align": "left",   "before_px": 8,  "after_px": 8,  "upper": True, "bold": True},
        },
        "font_eastAsia": "Songti SC",  # 中文降级字体（宋体，对齐迪士尼/Pixar 动画剧本传统）
    },
}

ALIGN_MAP = {
    "left": WD_ALIGN_PARAGRAPH.LEFT,
    "right": WD_ALIGN_PARAGRAPH.RIGHT,
    "center": WD_ALIGN_PARAGRAPH.CENTER,
}


def px_to_pt(px):
    return Pt(px * 72 / 96)


def build_docx(elements, format_id, output_path):
    """
    elements: [{"type": "scene", "text": "..."}, ...]
    format_id: "hollywood" / "eastAsia" / "stage" / "audio" / "animation"
    output_path: str, 输出 .docx 文件路径
    """
    fmt = FORMATS[format_id]
    doc = Document()
    
    # 1. 设置页面
    section = doc.sections[0]
    section.page_width  = Inches(fmt["page"]["width_in"])
    section.page_height = Inches(fmt["page"]["height_in"])
    section.top_margin    = Inches(fmt["page"]["top_in"])
    section.right_margin  = Inches(fmt["page"]["right_in"])
    section.bottom_margin = Inches(fmt["page"]["bottom_in"])
    section.left_margin   = Inches(fmt["page"]["left_in"])
    
    # 2. 设置默认字体
    style = doc.styles["Normal"]
    style.font.name = fmt["font"]["name"]
    style.font.size = Pt(fmt["font"]["size_pt"])
    # 中文字体也要设置（不然 Word 会用系统默认）
    rpr = style.element.get_or_add_rPr()
    rfonts = rpr.find(qn("w:rFonts"))
    if rfonts is None:
        rfonts = OxmlElement("w:rFonts")
        rpr.append(rfonts)
    rfonts.set(qn("w:eastAsia"), fmt["font"]["name"])
    
    # 3. 逐个元素写入
    for element in elements:
        layout = fmt["elements"][element["type"]]
        para = doc.add_paragraph()
        
        # 对齐
        para.paragraph_format.alignment = ALIGN_MAP[layout["align"]]
        # 左缩进
        para.paragraph_format.left_indent = px_to_pt(layout["left_px"])
        # 段前/段后间距
        para.paragraph_format.space_before = px_to_pt(layout["before_px"])
        para.paragraph_format.space_after  = px_to_pt(layout["after_px"])
        # 行距（1.0 单倍行距，Courier 12pt 时为 16px）
        para.paragraph_format.line_spacing = 1.0
        
        # 文本内容（应用大小写）
        text = element["text"]
        if layout.get("upper"):
            text = text.upper()
        run = para.add_run(text)
        run.font.name = fmt["font"]["name"]
        run.font.size = Pt(fmt["font"]["size_pt"])
        if layout.get("bold"):
            run.font.bold = True
        if layout.get("italic"):
            run.font.italic = True
    
    # 4. 页码（右上角）
    add_page_number_header(doc)
    
    doc.save(output_path)


def add_page_number_header(doc):
    """在页眉右上角加页码，字号 9pt，颜色灰色"""
    section = doc.sections[0]
    header = section.header
    para = header.paragraphs[0] if header.paragraphs else header.add_paragraph()
    para.alignment = WD_ALIGN_PARAGRAPH.RIGHT
    run = para.add_run()
    run.font.size = Pt(9)
    run.font.color.rgb = RGBColor(0x6b, 0x72, 0x80)
    
    # 插入 PAGE 域
    fld_char1 = OxmlElement("w:fldChar")
    fld_char1.set(qn("w:fldCharType"), "begin")
    instr = OxmlElement("w:instrText")
    instr.text = "PAGE"
    fld_char2 = OxmlElement("w:fldChar")
    fld_char2.set(qn("w:fldCharType"), "end")
    run._r.append(fld_char1)
    run._r.append(instr)
    run._r.append(fld_char2)


# ---- 使用示例 ----
if __name__ == "__main__":
    sample = [
        {"type": "scene",         "text": "内景. 张家客厅 - 日"},
        {"type": "action",        "text": "张三推门进来，看着满地的行李箱，愣住了。李四正在收拾东西，头也没抬。"},
        {"type": "character",     "text": "张三"},
        {"type": "dialogue",      "text": "你要走？"},
        {"type": "character",     "text": "李四"},
        {"type": "parenthetical", "text": "（冷冷）"},
        {"type": "dialogue",      "text": "合同上写着 15 号搬走。"},
        {"type": "transition",    "text": "切至："},
    ]
    build_docx(sample, "eastAsia", "output.docx")
```

## AI 应用步骤

用户说「导出成 Word 文档」时，AI 应该：

1. **确认格式**：默认 `eastAsia`（中文剧本）或 `hollywood`（英文剧本），也可以问用户
2. **准备 elements 数组**：把 Skill 处理后的剧本按类型分好
3. **调用 build_docx()** 或**用 docx skill** 生成 `.docx` 文件
4. **保存到用户工作目录**（默认 `~/Downloads/` 或 `.` 当前目录）
5. **用 `<media type="file" src="..." />` 发送给用户**

## 用 docx skill 的替代方案

如果本 Skill 部署环境里已经有 `docx` skill（Anthropic 的 docx skill 支持模板填充），可以：

1. 让 Skill 输出一个**中间 JSON**（`{elements: [...], format: "hollywood"}`）
2. 调用 `docx` skill 时传这个 JSON + 一份对应格式的 .docx 模板
3. `docx` skill 会按模板 style 填充

**中间 JSON 结构**：

```json
{
  "meta": {
    "title": "示例剧本",
    "author": "张三",
    "format": "hollywood",
    "language": "zh-CN"
  },
  "elements": [
    {"type": "scene", "text": "内景. 张家客厅 - 日"},
    {"type": "action", "text": "..."},
    {"type": "character", "text": "张三"},
    {"type": "dialogue", "text": "你要走？"}
  ]
}
```

## 常见问题

### Q1: 中文字体在 Word 里显示乱码 / 变成宋体

**A**：python-docx 默认不设置 `eastAsia` 字体属性，Word 会 fallback 到系统默认。必须显式设置：

```python
rfonts = rpr.find(qn("w:rFonts"))
if rfonts is None:
    rfonts = OxmlElement("w:rFonts")
    rpr.append(rfonts)
rfonts.set(qn("w:eastAsia"), "Microsoft YaHei")
```

上面模板代码已包含。

### Q2: 好莱坞格式的左缩进（1.5"）Word 里显示错乱

**A**：确保 `section.left_margin = Inches(1.5)`。python-docx 默认 1"，必须显式改。

### Q3: transition 靠右显示不对齐

**A**：`WD_ALIGN_PARAGRAPH.RIGHT` 是段落右对齐，不是设置文字位置。如果对齐还不对，检查是否设了 `left_indent`（transition 不应该有 left_indent，因为它是靠右）。

### Q4: 页码没显示

**A**：Word 页码是通过 `PAGE` 域实现的，模板代码里的 `fldChar` + `instrText` 就是干这个的。如果没显示，检查是否放在 `section.header` 而不是 body。

## 输出文件

生成完后的 `.docx` 文件：
- Word / WPS Office / Google Docs 都能打开
- 保留所有排版（字体、字号、缩进、大小写、加粗、斜体、行距、页面边距）
- 可以直接打印或提交项目
