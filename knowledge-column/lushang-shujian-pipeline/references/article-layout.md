# 路上书简 · 公众号文章排版规范

## 核心原则

- 每篇 WeChat 安全 HTML：内联样式，无外部 CSS，无 `<link>` 或 `@import`
- 所有颜色从 `config/themes.yaml` 读取，通过 `c = themes['themes'][theme_name]['colors']` 加载
- **正文字体栈**（系统字体引用不涉及嵌入版权）：
  `-apple-system, BlinkMacSystemFont, 'PingFang SC', 'Noto Sans CJK SC', 'Microsoft YaHei', sans-serif`
- **SVG 图片字体**（嵌入图片必须开源）：
  `'Noto Sans CJK SC', 'WenQuanYi Zen Hei', sans-serif`
- `<div>` 和 `<section>` 都可以用，WeChat 不区别对待
- `box-shadow`、`linear-gradient` 微信都支持，但用之前确认色差够大（两色太近不如纯色）

## 分集 vs 综合文章

| | 分集（7篇） | 综合（1篇） |
|---|---|---|
| 发布 | 每天 07:00 | 周日 20:00 |
| 内容 | 音频脚本全文 + 排版组件 | 表格/清单/框架/对比 |
| 规则 | 脚本多少字文章就保留多少字，不裁切 | 做音频做不到的事 |

## Python 组件函数

```python
with open('config/themes.yaml') as f:
    c = yaml.safe_load(f)['themes']['forest']['colors']

def H(text):  # 章节标题 — 左边框+主色
def P(text):  # 正文段落
def Q(text):  # 金句卡片 — 深色背景
def S(text):  # 故事块 — 暖色背景+强调色左边框
def X(text):  # 练习/思考卡 — 暖色居中
def T(rows):  # 表格 — rows 二维列表
```

## 组件 HTML 模板

### H() 章节标题
```html
<section style="margin:22px 0 0;padding:0 16px;">
<p style="font-size:18px;font-weight:bold;color:{{primary_dark}};border-left:3px solid {{primary}};padding-left:8px;margin:0 0 10px;">标题</p>
</section>
```

### P() 正文
```html
<section style="margin:0;padding:0 16px;">
<p style="font-size:14px;color:{{text}};line-height:2;margin:0 0 10px;">正文</p>
</section>
```

### Q() 金句卡片（纯色背景，干净利落）
```html
<section style="margin:14px 0;padding:0 16px;">
<p style="background:{{primary}};color:#fff;padding:14px 18px;border-radius:4px;text-align:center;font-size:14px;line-height:2.2;">金句</p>
</section>
```

> 如需渐变：`background:linear-gradient(135deg,{{primary_dark}},{{primary}})` — 注意两色跨度要够大才看得出。

### S() 故事块
```html
<section style="margin:14px 0;padding:0 16px;">
<p style="background:{{bg_warm}};padding:14px 16px;border-radius:4px;font-size:14px;color:{{text}};line-height:2;border-left:3px solid {{accent}};">故事</p>
</section>
```

### X() 练习卡
```html
<section style="margin:16px 0;padding:0 16px;">
<p style="background:{{bg_warm}};padding:12px 16px;border-radius:4px;text-align:center;font-size:14px;color:{{text_warm}};line-height:2;">练习</p>
</section>
```

### T() 表格
```html
<section style="margin:14px 0;padding:0 16px;">
<table style="width:100%;border-collapse:collapse;font-size:13px;">
<tr><td style="padding:5px 6px;color:{{text_light}};border-bottom:1px solid {{border_light}};">列</td></tr>
</table>
</section>
```

## 标题区

```html
<section style="margin:0;padding:18px 16px 14px;background:{{bg}};">
<p style="text-align:center;color:{{primary}};font-size:22px;font-weight:bold;">标题</p>
<p style="text-align:center;color:{{text_muted}};font-size:13px;">副标题</p>
<p style="text-align:center;color:{{text_muted}};font-size:11px;letter-spacing:2px;">路 上 书 简 · 第 N 期</p>
<hr style="border:0;border-top:1px solid {{border}};margin:14px 0 10px;">
</section>
```

## 文章底部

```html
<section style="margin:24px 0 30px;padding:0 16px;">
<p style="text-align:center;font-size:11px;color:{{text_muted}};margin:0;">路上书简 · 通勤路上的读书伙伴</p>
</section>
```

## 严格禁止

### CSS 禁止（微信真的会过滤）
- ❌ 外部样式表（`<link>`、`@import`）
- ❌ `position: fixed/sticky`
- ❌ `@keyframes` 动画

### 内容禁止
- ❌ 文字标注「原创声明」「AI生成」「合集」「阅读指南」
- ❌ 标题含「30 秒」「X 步」等装饰文字（会被微信进度条提取到底部）
- ❌ 分集文章只放时间轴 — 必须是脚本全文图文版
- ❌ 分集文章裁切脚本 — 脚本多少字就保留多少字
- ❌ 文章底部放元信息说明（文章是成品，不是说明书）

### 字体禁止（仅 SVG 图片，正文不受限）
- ❌ SVG 图片中引用 `'PingFang SC'`（专有字体不可嵌入 + Linux 无此字体→豆腐块）
- ❌ SVG 图片中引用 `'Microsoft YaHei'`（同上）
- ✅ SVG 图片必须用 `'Noto Sans CJK SC'`（SIL Open Font License）

## 经验教训

### ⚠️ 不要全局机械替换 CSS 值
- 渐变两色跨度不够（如 `#5b7a6f→#6b8e80` 色差极小）时，纯色比渐变更好
- border-radius 不要全局改：大卡片可 8px，小元素保持 4px
- 改字号必须同步调 line-height，否则行距变挤
- 排版优化要逐元素判断，不能正则一把梭
