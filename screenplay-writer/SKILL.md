---
name: screenplay-writer
description: 剧本格式化与结构化辅助 Skill —— 给 AI 用的工业级剧本排版规范手册。AI 读完可以直接产出符合行业标准（好莱坞 / 东亚 / 舞台剧 / 广播剧 / 动画剧本）的 Word 文档（.docx）或 PDF 文件，字体、字号、边距、缩进、大小写、加粗、斜体、分页全部精确对齐 Final Draft / 剧本行业标准。支持 3 种使用模式：纯格式转换、局部术语优化、故事结构重塑。当用户说"帮我把剧本改成标准格式导出 Word""生成剧本 PDF""规范剧本术语""按三幕式重组我的故事""INT./EXT. 场景标题""FDX 格式""转场词写法""救猫咪 15 拍""起承转合"等时使用。
---

# Screenplay Writer

## Core Rule

**这是一份给 AI 用的工业级剧本规范手册。**

Skill 的最终产出是**用户可以直接用的 Word 文档（.docx）或 PDF 文件**，不是给人读的 Markdown。

**核心工作方式**：AI 读完本 Skill 里的规范 → 按规范识别、处理用户剧本 → **直接调用 docx / pdf 生成能力**输出符合行业标准的剧本文件。

**中间不需要用户看 Markdown**。用户最终拿到的是 `.docx` 或 `.pdf`，跟 Final Draft / 剪映剧本模块导出的效果同级。

## 3 Modes 路由

首次触发时先判断用户属于哪种：

| 模式 | 用户诉求（关键词） | Skill 提供 | 触发时读 |
|------|------------------|----------|---------|
| **1️⃣ Mode 1: 格式转换** | "改成标准格式" / "导出 Word" / "生成剧本 PDF" / "好莱坞格式" | 5 种剧本格式的完整排版规范 + Word/PDF 生成模板 | `references/mode-1-format-conversion.md` + `references/vocab-format-specs.md` + `references/export-to-docx.md` 或 `references/export-to-pdf.md` |
| **2️⃣ Mode 2: 术语优化** | "规范术语" / "统一场景标题" / "把 CUT TO 改成中文" | 场景术语词典（内外景/时间）+ 转场词典（切至/叠化/淡入等）+ 多语言映射 | `references/mode-2-term-refinement.md` |
| **3️⃣ Mode 3: 结构重塑** | "按三幕式重组" / "救猫咪节拍" / "英雄旅程" / "起承转合" / "我这剧本结构乱" | 5 种经典故事结构模板 + 节拍分析 + 场次重排建议 | `references/mode-3-structure-reshape.md` |

**模式可叠加**：用户可以说"先按 Mode 3 重组，然后 Mode 1 转成好莱坞 Word"——AI 按顺序跑，最终交付一份 Word 文件。

**模式判定示例**：
- 用户："这是我剧本，转成好莱坞标准 PDF" → **Mode 1**（输出 .pdf）
- 用户："帮我做成 Word 版本的剧本" → **Mode 1**（输出 .docx，默认东亚格式如果是中文）
- 用户："帮我把所有'切换到'统一改成'切至'" → **Mode 2**（术语处理完后询问是否要导出成 Word/PDF）
- 用户："我这个故事感觉节奏散，能不能按救猫咪 15 拍重组一下" → **Mode 3**
- 用户："先按三幕式改，再导出成好莱坞 Word" → **Mode 3 → Mode 1**（顺序执行，最终 .docx）

## Interaction Rule

- **用户不明确要哪种模式** → 一句话反问：「你是要 (1) 换格式导出 Word/PDF，还是 (2) 规范术语，还是 (3) 重组结构？」
- **用户没说要 Word 还是 PDF** → **默认输出 Word**（.docx 可打开可微调，用户满意后自己再导 PDF）；中文剧本默认 `eastAsia` 格式，英文剧本默认 `hollywood`
- **拿不准的字段（默认语言、默认格式）** → 用中文/东亚标准作为默认，跑完让用户 review
- **模式跑完不要给用户看 Markdown**，直接给最终 `.docx` / `.pdf` 文件；用户要看中间态时再给
- **AI 交付文件时必须用** `<media type="file" src="/absolute/path" caption="..." />` 送给用户，不能只打印路径

## Shared Vocabulary（3 Mode 都会用到的共享词典）

以下词典 3 个 Mode 都可能查：

- `references/vocab-scene-terms.md` — 场景内外景 + 时间术语（多语言）
- `references/vocab-transitions.md` — 转场词典（9 种转场 + 5 种语言）
- `references/vocab-format-specs.md` — **5 种剧本格式的工业级精确排版参数**（页面/字体/字号/行距/元素缩进/对齐/大小写/加粗/斜体）

## Export Pipeline（关键）

不管是 Mode 1 / 2 / 3，最终交付都走这条流水线：

```
用户剧本 → Skill 识别元素类型（scene/action/character/dialogue/...）
       → 用户选目标产出形态
       → 按对应规范生成文件
       → <media /> 发送给用户
```

### 4 种可交付产出

| 文件类型 | 生成规范 | 谁能打开 | 可编辑性 |
|---------|--------|--------|--------|
| **`.docx` Word** | `references/export-to-docx.md` + `vocab-format-specs.md` | Word / WPS / Google Docs | ⭐⭐⭐⭐⭐ 通用编辑 |
| **`.pdf` PDF** | `references/export-to-pdf.md` + `vocab-format-specs.md` | PDF 阅读器 | ⭐ 只读为主 |
| **`.fountain` Fountain** | `references/export-to-fountain.md` | **Highland / Slugline / Fade In / Beat / KIT Scenarist** | ⭐⭐⭐⭐⭐ 剧本编辑器原生 |
| **`.fdx` Final Draft XML** | `references/export-to-fdx.md` | **Final Draft / Fade In / Movie Magic / Highland / KIT Scenarist** | ⭐⭐⭐⭐⭐ 好莱坞行业标准 |

**默认选择规则**：
- 用户没明说 → **默认输出 Word**（.docx）
- 用户说「发行 / 好莱坞 / 制片公司」→ 输出 **FDX** + 附一份 **PDF**
- 用户说「继续在编辑器改」→ 询问用哪个编辑器：
  - Highland / Slugline / Fade In / Beat → 输出 **Fountain**
  - Final Draft / Movie Magic → 输出 **FDX**
- 用户说「打印 / 存档」→ 输出 **PDF**

### 中英剧本的默认差异

- **中文剧本** → 默认 `.docx`（东亚格式，中文排版最好）；用户明说要发行海外时再术语英化 + 输出 FDX
- **英文剧本** → 默认 `.docx`（好莱坞格式）；说"给制作人 / 发行"时输出 FDX

**AI 生成 .docx 时必须做到**：
- 页面尺寸 / 边距按 vocab-format-specs 中"页面参数"表格
- 默认字体 / 字号按"字体"表格
- 每个段落按"元素排版参数"表格设置：左缩进（px→pt 换算）、对齐、段前段后间距、大写、加粗、斜体
- 中文字体必须显式设置 `w:eastAsia` 属性（见 export-to-docx.md Q1）
- 页码放右上角，9pt 灰色

**AI 生成 .pdf 时必须做到**：
- CSS `@page size` 用规范里的纸张（Letter / A4）
- CSS `@page margin` 用规范里的边距（好莱坞 `1in 1in 1in 1.5in`）
- 每个元素用对应 class（`.scene` `.character` `.dialogue` 等）
- CSS 里精确设置 `margin-left` / `width` / `text-align` / `text-transform` / `font-weight` / `font-style` / `line-height`
- 通过 `@top-right { content: counter(page); }` 加页码

**AI 生成 .fountain 时必须做到**：
- 标题页用 `Key: Value` 格式
- 中文场景标题必须加 `.` 前缀强制识别
- 中文角色名必须加 `@` 前缀强制识别
- 中文转场必须加 `>` 前缀强制识别
- 相邻元素之间的空行规则（character → dialogue 之间不空行）
- 见 `references/export-to-fountain.md` 完整代码

**AI 生成 .fdx 时必须做到**：
- XML 声明必须是 `<?xml version="1.0" encoding="UTF-8" standalone="no" ?>`
- 顶层 `<FinalDraft DocumentType="Script" Template="No" Version="1">`
- 标题页 `<TitlePage><Content><Paragraph Type="Title|Credit|Author">…`
- 正文 `<Content><Paragraph Type="Scene Heading|Action|Character|Parenthetical|Dialogue|Transition|Shot|General">`
- 特殊字符 XML 转义（`&` `<` `>` `"` `'`）
- 见 `references/export-to-fdx.md` 完整代码

## Mode 1: 格式转换（输出 Word / PDF）

**输入**：用户的剧本文本（可以是 TXT / Markdown / Fountain / 从 DOCX 粘贴 / FDX XML 片段）

**Skill 做什么**：
1. **识别现有格式**（好莱坞 / 东亚 / 舞台剧 / 广播剧 / 动画剧本 / 无格式散文）
2. **识别每行元素类型**（scene / action / character / parenthetical / dialogue / transition / shot / section / note）
3. **按目标格式的完整规范** —— 见 `vocab-format-specs.md` —— **生成 Word 或 PDF 文件**
4. **交付给用户**：`<media type="file" src="/absolute/path/screenplay.docx" caption="标准好莱坞格式剧本" />`

**支持 5 种目标格式**：
- **好莱坞标准**（Hollywood）— Letter 纸、左 1.5"、Courier 12pt、~54 行/页
- **东亚影视**（East Asia）— A4、Microsoft YaHei、场景加粗
- **舞台剧**（Stage）— 场景居中大写、动作缩进斜体
- **广播剧/播客**（Audio）— 场景大写、角色 + 括号内动作提示
- **动画剧本**（Animation）— 沿用好莱坞页面 + 新增 `VISUAL:` / `SFX:` 元素，供分镜师和音效师定位

**输出文件格式**：
- **`.docx`**（默认，Word/WPS/Google Docs 打开）
- **`.pdf`**（打印/存档/发行）
- **`.fountain`**（可导入 Highland / Slugline / Fade In / Beat / KIT Scenarist 继续编辑）
- **`.fdx`**（可导入 Final Draft / Movie Magic 好莱坞行业标准继续编辑）

详见：
- `references/mode-1-format-conversion.md`
- `references/export-to-docx.md`
- `references/export-to-pdf.md`
- `references/export-to-fountain.md`
- `references/export-to-fdx.md`

## Mode 2: 术语优化

**输入**：用户已有剧本

**Skill 做什么**：
1. 扫描 → 规范化术语（场景标题、转场词、时间术语、括号提示）
2. **询问用户是否要导出 Word/PDF**——如果要就走 Mode 1 的流水线；否则输出规范化后的文本

详见 `references/mode-2-term-refinement.md`。

## Mode 3: 结构重塑

**输入**：用户的故事梗概或已有剧本

**Skill 做什么**：
1. 分析现有故事的场次和节拍点
2. 对齐目标结构模板（5 选 1）
3. 输出结构诊断报告 + 场次重排建议
4. **询问用户是否要导出成 Word 版本的场次大纲**——如果要就用 Mode 1 的流水线导出

详见 `references/mode-3-structure-reshape.md`。

## Skill 边界

**做**：
- ✅ 生成符合行业标准的 Word / PDF 剧本文件（Mode 1 主线）
- ✅ 术语规范（词典替换）
- ✅ 结构分析和重组建议
- ✅ FDX / Fountain 导入导出

**不做**：
- ❌ **不写具体剧情内容**（本 Skill 不代替编剧构思）
- ❌ **不给你的剧本打分/评优**
- ❌ **不做人物弧线设计 / 对话生成**

如果用户想要"帮我写一段吵架戏" → 明确告诉他这不在本 Skill 范围内。

## Final Check

Mode 1 输出 Word/PDF 后必查：
- [ ] 页面尺寸和边距完全按规范（好莱坞 Letter + 左 1.5"，东亚 A4 + 边距 72-76px）
- [ ] 字体和字号按规范（好莱坞 Courier New 12pt，东亚 Microsoft YaHei 12pt）
- [ ] 每个元素的左缩进、对齐、大小写、加粗、斜体符合表格
- [ ] 页码在右上角
- [ ] 分页规则合理（不出现孤行寡行）
- [ ] 用 `<media type="file" src="..." caption="..." />` 发送给用户

Mode 2 输出后必查：
- [ ] 场景标题格式统一（如所有场景都是 `内景. 客厅 - 日` 结构）
- [ ] 转场词全部映射到规范词典
- [ ] 目标语言一致
- [ ] 询问了用户是否需要导出 Word/PDF

Mode 3 输出后必查：
- [ ] 节拍点数量 = 目标模板节拍数
- [ ] 每个节拍点有对应的场次建议
- [ ] 场次总数合理
- [ ] 询问了用户是否需要导出成 Word 大纲
