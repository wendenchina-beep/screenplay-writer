# 共享词典 · 场景术语（内外景 + 时间）

3 个 Mode 都会查这个词典。场景标题（Scene Heading）的核心结构：

```
<内外景> <地点> - <时间>
```

例：
- 中文：`内景. 客厅 - 日`
- 英文：`INT. LIVING ROOM - DAY`
- 繁中：`內景. 客廳 - 日`

## 内外景术语表

| 语义 | 简体中文 | English | 繁體中文 | 别名（识别时兼容） |
|------|---------|---------|---------|-----------|
| 室内 | `内景` | `INT.` | `內景` | 内景 / 内 / 室内 / Inside / Interior / Indoor |
| 室外 | `外景` | `EXT.` | `外景` | 外景 / 外 / 室外 / Outside / Exterior / Outdoor |
| 室内到室外 | `内/外景` | `INT./EXT.` | `內/外景` | 内外景 / 内→外 / Interior/Exterior |
| 室外到室内 | `外/内景` | `EXT./INT.` | `外/內景` | 外内景 / 外→内 / Exterior/Interior |

## 时间术语表

| 语义 | 简体中文 | English | 繁體中文 | 别名（识别时兼容） |
|------|---------|---------|---------|-----------|
| 白天 | `日` | `DAY` | `日` | 白天 / 白日 / 日间 / 中午 |
| 夜晚 | `夜` | `NIGHT` | `夜` | 夜里 / 夜晚 / 晚上 / 深夜 |
| 早晨 | `早晨` | `MORNING` | `早晨` | 早上 / 上午 / Morning |
| 黎明 | `黎明` | `DAWN` | `黎明` | 清晨 / 破晓 / Dawn |
| 黄昏 | `黄昏` | `DUSK` | `黃昏` | 傍晚 / 日落 / Dusk |
| 连续 | `连续` | `CONTINUOUS` | `連續` | 承接 / 紧接 / Continuous |
| 稍后 | `稍后` | `LATER` | `稍後` | 之后 / 少顷 / Later |

## 场景标题识别规则

Skill 扫描一行文本时，按以下顺序判断是否为场景标题：

1. **首字符**：以 `内景/外景/INT./EXT./內景/外景/INT/EXT` 开头（大小写不敏感）
2. **含分隔**：中间用空格 / `-` / `.` / `：`
3. **末尾时间**：以时间术语（日/夜/DAY/NIGHT 等）结尾

匹配到就归类为 `scene` 元素。

## 常见规范化前后对照

| 用户原文 | 规范后（中文） | 规范后（英文） |
|---------|-------------|--------------|
| `1. 张家客厅，白天` | `内景. 张家客厅 - 日` | `INT. ZHANG'S LIVING ROOM - DAY` |
| `室内 - 咖啡馆 - 傍晚` | `内景. 咖啡馆 - 黄昏` | `INT. CAFE - DUSK` |
| `街道，深夜` | `外景. 街道 - 夜` | `EXT. STREET - NIGHT` |
| `INT./EXT. Car - continuous` | `内/外景. 车内 - 连续` | `INT./EXT. CAR - CONTINUOUS` |

## Skill 内部处理伪代码

```python
def normalize_scene_heading(raw_text: str, target_lang: str = "zh-CN"):
    # 1. 检测内外景
    place = detect_place(raw_text)          # int / ext / int-ext / ext-int
    # 2. 剥离内外景，取剩下的
    remaining = strip_place(raw_text)
    # 3. 检测时间
    time = detect_time(remaining)           # day / night / morning / dawn / ...
    # 4. 剥离时间，剩下的就是地点
    location = strip_time(remaining).strip()
    # 5. 按目标语言拼接
    return f"{PLACE_MAP[place][target_lang]} {location} - {TIME_MAP[time][target_lang]}"
```
