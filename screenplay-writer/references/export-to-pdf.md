# 导出到 PDF

**用途**：把 Skill 处理好的剧本按 4 种格式规范生成 PDF 文件，保留字体、字号、边距、缩进、大小写、加粗、斜体、分页等全部专业排版。

**推荐技术栈**：**HTML + CSS → PDF**（用 Playwright / Chromium / WeasyPrint）

**为什么用 HTML+CSS**：
- CSS 的 `@page` 语法能精确控制页面尺寸、边距、分页
- 中英日韩混排渲染保真
- 字体、字号、缩进都能一比一对应到 CSS 属性
- 分页规则跟浏览器排版一致，跟 Word 差异小

## 4 种格式的 CSS 模板

### 好莱坞标准（Hollywood）

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Screenplay</title>
<style>
  @page {
    size: 8.5in 11in;                     /* Letter */
    margin: 1in 1in 1in 1.5in;            /* top right bottom LEFT (左 1.5") */
    @top-right {
      content: counter(page);
      font-size: 9pt;
      color: #6b7280;
      font-family: "Courier New", monospace;
    }
  }
  body {
    font-family: "Courier New", monospace;
    font-size: 12pt;
    line-height: 16px;                    /* = round(12 × 96/72) */
    color: #111827;
    margin: 0;
  }
  .scene       { font-weight: 700; text-transform: uppercase; margin-top: 32px; margin-bottom: 0; }
  .action      { margin-top: 16px; margin-bottom: 0; }
  .character   { margin-left: 192px; width: 384px; text-align: center; text-transform: uppercase; margin-top: 16px; }  /* 2.0" left, 4.0" width */
  .parenthetical { margin-left: 144px; width: 240px; }                                                                  /* 1.5" left, 2.5" width */
  .dialogue    { margin-left: 96px;  width: 336px; }                                                                    /* 1.0" left, 3.5" width */
  .transition  { margin-left: 384px; width: 192px; text-align: right; text-transform: uppercase; margin-top: 16px; }    /* 4.0" left, 2.0" width */
  .shot        { font-weight: 700; text-transform: uppercase; margin-top: 16px; }
  .section     { font-weight: 700; margin-top: 32px; }
  .note        { font-style: italic; margin-top: 16px; color: #4b5563; }
  .scene, .action, .shot, .section, .note { width: 576px; }
</style>
</head>
<body>
<p class="scene">INT. LIVING ROOM - DAY</p>
<p class="action">Zhang pushes the door open. Boxes everywhere. He freezes.</p>
<p class="character">ZHANG</p>
<p class="dialogue">You're leaving?</p>
<p class="character">LI</p>
<p class="parenthetical">(coldly)</p>
<p class="dialogue">The contract says the 15th.</p>
<p class="transition">CUT TO:</p>
</body>
</html>
```

### 东亚影视（East Asia）

```html
<style>
  @page {
    size: A4;
    margin: 76px 72px 76px 72px;
    @top-right { content: counter(page); font-size: 9pt; color: #6b7280; }
  }
  body {
    font-family: "Microsoft YaHei", "PingFang SC", "思源黑体", sans-serif;
    font-size: 12pt;
    line-height: 16px;
    color: #111827;
    margin: 0;
  }
  .scene       { font-weight: 700; width: 650px; margin-top: 16px; margin-bottom: 10px; }
  .action      { width: 650px; margin-bottom: 10px; }
  .character   { font-weight: 700; width: 120px; margin-top: 10px; }
  .parenthetical { margin-left: 125px; width: 410px; }
  .dialogue    { margin-left: 125px; width: 470px; margin-bottom: 8px; }
  .transition  { margin-left: 470px; width: 180px; text-align: right; margin-top: 14px; margin-bottom: 8px; }
  .shot        { font-weight: 700; width: 650px; margin-top: 12px; margin-bottom: 8px; }
  .section     { font-weight: 700; width: 650px; margin-top: 20px; margin-bottom: 8px; }
  .note        { font-style: italic; width: 650px; margin-top: 8px; margin-bottom: 8px; color: #4b5563; }
</style>
```

### 舞台剧（Stage）

```html
<style>
  @page {
    size: 8.5in 11in;
    margin: 88px 86px 88px 92px;
    @top-right { content: counter(page); font-size: 9pt; color: #6b7280; }
  }
  body {
    font-family: "Times New Roman", "Source Han Serif", serif;
    font-size: 12pt;
    line-height: 16px;
    color: #111827;
    margin: 0;
  }
  .scene       { font-weight: 700; text-transform: uppercase; text-align: center; width: 610px; margin-top: 20px; margin-bottom: 12px; }
  .action      { font-style: italic; margin-left: 40px; width: 530px; margin-bottom: 10px; }
  .character   { text-transform: uppercase; text-align: center; margin-left: 210px; width: 190px; margin-top: 14px; }
  .parenthetical { text-align: center; margin-left: 130px; width: 350px; }
  .dialogue    { margin-left: 85px; width: 440px; margin-bottom: 10px; }
  .transition  { text-transform: uppercase; text-align: right; margin-left: 380px; width: 230px; margin-top: 14px; margin-bottom: 8px; }
  .shot        { text-transform: uppercase; width: 610px; margin-top: 12px; margin-bottom: 8px; }
  .section     { font-weight: 700; width: 610px; margin-top: 20px; margin-bottom: 8px; }
  .note        { font-style: italic; margin-left: 40px; width: 530px; margin-top: 8px; margin-bottom: 8px; color: #4b5563; }
</style>
```

### 广播剧（Audio）

```html
<style>
  @page {
    size: A4;
    margin: 78px 78px 78px 78px;
    @top-right { content: counter(page); font-size: 9pt; color: #6b7280; }
  }
  body {
    font-family: "Arial", "PingFang SC", sans-serif;
    font-size: 12pt;
    line-height: 16px;
    color: #111827;
    margin: 0;
  }
  .scene       { font-weight: 700; text-transform: uppercase; width: 620px; margin-top: 16px; margin-bottom: 10px; }
  .action      { width: 620px; margin-bottom: 10px; }
  .character   { font-weight: 700; text-transform: uppercase; width: 180px; margin-top: 10px; }
  .parenthetical { font-style: italic; margin-left: 185px; width: 360px; }
  .dialogue    { margin-left: 185px; width: 435px; margin-bottom: 8px; }
  .transition  { text-transform: uppercase; text-align: right; margin-left: 420px; width: 200px; margin-top: 14px; margin-bottom: 8px; }
  .shot        { font-weight: 700; width: 620px; margin-top: 10px; margin-bottom: 8px; }
  .section     { font-weight: 700; width: 620px; margin-top: 18px; margin-bottom: 8px; }
  .note        { font-style: italic; width: 620px; margin-top: 8px; margin-bottom: 8px; color: #4b5563; }
</style>
```

### 动画剧本（Animation）

```html
<style>
  @page {
    size: Letter;
    margin: 96px 96px 96px 144px;
    @top-right { content: counter(page); font-size: 9pt; color: #6b7280; }
  }
  body {
    font-family: "Courier New", "Courier", "Songti SC", "SimSun", monospace;
    font-size: 12pt;
    line-height: 16px;
    color: #111827;
    margin: 0;
  }
  .scene       { font-weight: 700; text-transform: uppercase; width: 576px; margin-top: 20px; margin-bottom: 12px; }
  .action      { width: 576px; margin-bottom: 10px; }
  .character   { font-weight: 700; text-transform: uppercase; margin-left: 192px; width: 384px; margin-top: 12px; }
  .parenthetical { font-style: italic; margin-left: 144px; width: 240px; }
  .dialogue    { margin-left: 96px; width: 336px; margin-bottom: 10px; }
  .transition  { text-transform: uppercase; text-align: right; margin-left: 384px; width: 192px; margin-top: 14px; margin-bottom: 8px; }
  .shot        { font-weight: 700; text-transform: uppercase; width: 576px; margin-top: 12px; margin-bottom: 8px; }
  .section     { font-weight: 700; width: 576px; margin-top: 24px; margin-bottom: 10px; }
  .note        { font-style: italic; width: 576px; margin-top: 8px; margin-bottom: 8px; color: #4b5563; }
  .visual      { font-weight: 700; text-transform: uppercase; width: 576px; margin-top: 12px; margin-bottom: 8px; }
  .sfx         { font-weight: 700; text-transform: uppercase; width: 576px; margin-top: 8px; margin-bottom: 8px; }
</style>
```

**动画特有元素**：
- `.visual` — `VISUAL:` / `画面：` 分镜画面提示，供分镜师定位（大写加粗）
- `.sfx` — `SFX:` / `音效：` 音效指令，供音效师定位（大写加粗）

## 用 Playwright 生成 PDF

```python
from playwright.sync_api import sync_playwright
from pathlib import Path

def html_to_pdf(html_str: str, output_path: str, page_size: str = "Letter"):
    """
    html_str: 完整 HTML 字符串
    output_path: str, PDF 输出路径
    page_size: "Letter" / "A4"
    """
    # 先把 HTML 写到临时文件（Playwright 需要 file:// URL）
    tmp_html = Path(output_path).with_suffix(".html")
    tmp_html.write_text(html_str, encoding="utf-8")
    
    with sync_playwright() as p:
        browser = p.chromium.launch()
        page = browser.new_page()
        page.goto(f"file://{tmp_html.absolute()}")
        page.pdf(
            path=output_path,
            format=page_size,        # "Letter" 或 "A4"
            print_background=True,
            margin={"top": "0", "right": "0", "bottom": "0", "left": "0"},  # 边距已在 CSS @page 里
            prefer_css_page_size=True,
        )
        browser.close()
    
    tmp_html.unlink()  # 清理临时 HTML
```

## 用 WeasyPrint 生成 PDF（推荐，无需浏览器）

```python
from weasyprint import HTML

def html_to_pdf_weasy(html_str: str, output_path: str):
    HTML(string=html_str).write_pdf(output_path)
```

WeasyPrint 支持 CSS Paged Media，`@page` 语法完美支持，中文字体也没问题。

## AI 应用步骤

用户说「导出成 PDF」时，AI 应该：

1. **确认格式**：默认 `hollywood`（英文）/ `eastAsia`（中文）
2. **生成完整 HTML 字符串**：
   - 头部按格式 ID 选对应 CSS
   - body 里每个元素用对应的 class 名（`.scene` / `.action` / `.character` / …）
   - **必要时对文本做 HTML 转义**（`&` → `&amp;` 等）
3. **调用 pdf skill 或 Playwright/WeasyPrint**
4. **保存到用户工作目录**
5. **用 `<media type="file" src="..." />` 发送给用户**

## 分页控制

CSS Paged Media 的分页规则：

- **强制换页**：`page-break-before: always;` 加到某元素前
- **避免拆分**：`page-break-inside: avoid;` 加到对话组（`.character + .dialogue`）
- **孤行寡行**：`orphans: 2; widows: 2;` 避免段落末尾 1 行留在页首/页末

**推荐加到通用 CSS**：

```css
.character { page-break-after: avoid; }         /* 角色名后不换页 */
.parenthetical { page-break-before: avoid; page-break-after: avoid; }
.scene { page-break-before: auto; page-break-after: avoid; }  /* 场景标题后跟内容不换页 */
```

## 常见问题

### Q1: 中文字体不显示 / 变成方框

**A**：CSS 里必须列出中文字体：`font-family: "Microsoft YaHei", "PingFang SC", "思源黑体", sans-serif;`。系统里没有对应字体时会 fallback，Chromium 通常有 Noto Sans CJK。

### Q2: 生成的 PDF 边距对不上

**A**：Playwright 的 `page.pdf(margin={...})` 会**覆盖** CSS `@page margin`。要用 CSS 里的边距就传 `margin={"top": "0", ...}` + `prefer_css_page_size=True`。

### Q3: 分页规则跟 Word 差异大

**A**：CSS Paged Media 跟 Word 分页规则不完全一致（浏览器按 CSS 布局，Word 按自己引擎）。要求跟 Word 一模一样时，走 export-to-docx.md 直接生成 Word。

### Q4: PDF 打开后能不能选中文字复制

**A**：可以。HTML+CSS→PDF 生成的是**矢量 PDF**（文字层保留），不是图片 PDF。

## 输出文件

生成完后的 `.pdf` 文件：
- 任何 PDF 阅读器都能打开
- 保留所有排版（跟 CSS 定义完全一致）
- 文字可选中、可复制、可搜索
- 可以直接打印或提交项目
