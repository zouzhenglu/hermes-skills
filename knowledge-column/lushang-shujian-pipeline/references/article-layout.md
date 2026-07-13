# 路上书简 · 公众号文章排版规范

## 核心原则

- 每篇 WeChat 安全 HTML：内联样式，无外部 CSS，无 `<link>` 或 `@import`
- 所有颜色从 `config/themes.yaml` 读取，通过 `c = themes['themes'][theme_name]['colors']` 加载
- **正文字体栈**（系统字体按优先级回退，无嵌入版权问题）：
  `-apple-system, BlinkMacSystemFont, 'PingFang SC', 'Noto Sans CJK SC', 'Microsoft YaHei', sans-serif`
- **SVG 图片字体**（嵌入图片必须用开源字体）：
  `'Noto Sans CJK SC', 'WenQuanYi Zen Hei', sans-serif`
- **CSS 可用性**（经过实际发布验证）：`box-shadow` ✅ · `linear-gradient` ✅ · `border-radius` 任意值 ✅ · `<div>` ✅

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

def H(text):   # 章节标题 — 左边框+主色
def P(text):   # 正文段落
def Q(text):   # 金句卡片 — 渐变/纯色深色背景
def S(text):   # 故事块 — 暖色背景+强调色左边框
def X(text):   # 练习/思考卡 — 暖色居中
def T(rows):   # 表格 — rows 二维列表，自动表头深色+数据行交替
def Card(html):    # 白色卡片容器 — 微阴影+8px圆角
def Badge(text):   # 标签 pill — 圆角14px，用于日期/类型标记
def Tip(text):     # 底部注释 — 暖色背景+暖棕文字
def Bar(pct):      # 进度条 — CSS百分比+渐变填充
```

## 组件 HTML 模板

### H() 章节标题
```html
<section style="margin:22px 0 0;padding:0 16px;">
<p style="font-size:18px;font-weight:bold;color:{{primary_dark}};border-left:4px solid {{primary}};padding-left:10px;margin:0 0 10px;">标题</p>
</section>
```

### P() 正文
```html
<section style="margin:0;padding:0 16px;">
<p style="font-size:15px;color:{{text}};line-height:2;margin:0 0 12px;">正文</p>
</section>
```

### Q() 金句卡片（渐变背景）
```html
<section style="margin:16px 0;padding:0 16px;">
<p style="background:linear-gradient(135deg,{{primary}},{{primary_mid}});color:#fff;padding:16px 20px;border-radius:8px;text-align:center;font-size:15px;line-height:2.2;letter-spacing:1px;">金句</p>
</section>
```

### S() 故事块
```html
<section style="margin:16px 0;padding:0 16px;">
<p style="background:{{bg_warm}};padding:14px 16px;border-radius:8px;font-size:14px;color:{{text}};line-height:2;border-left:4px solid {{accent}};">故事</p>
</section>
```

### X() 练习卡
```html
<section style="margin:16px 0;padding:0 16px;">
<p style="background:{{bg_warm}};padding:14px 16px;border-radius:8px;text-align:center;font-size:14px;color:{{text_warm}};line-height:2;">练习</p>
</section>
```

### T() 表格（表头深色+数据行交替）
```html
<section style="margin:16px 0;padding:0 16px;">
<table style="width:100%;border-collapse:collapse;font-size:13px;border-radius:8px;overflow:hidden;">
<tr style="background:{{primary}};color:#fff;">
  <td style="padding:8px 10px;">表头1</td>
  <td style="padding:8px 10px;">表头2</td>
</tr>
<tr style="background:{{bg_card}};">
  <td style="padding:8px 10px;border-bottom:1px solid {{border_light}};">数据</td>
  <td style="padding:8px 10px;border-bottom:1px solid {{border_light}};">数据</td>
</tr>
<tr style="background:{{bg_warm}};">
  <td style="padding:8px 10px;border-bottom:1px solid {{border_light}};">数据</td>
  <td style="padding:8px 10px;border-bottom:1px solid {{border_light}};">数据</td>
</tr>
</table>
</section>
```

### Card() 白色卡片容器
```html
<section style="margin:16px 0;padding:0 16px;">
<div style="background:{{bg_card}};border-radius:8px;padding:16px;box-shadow:0 1px 6px rgba(0,0,0,0.06);">
  卡片内容（可嵌套 H/P/S/X 等组件）
</div>
</section>
```

### Badge() 标签 pill
```html
<span style="display:inline-block;background:{{primary}};color:#fff;padding:3px 12px;border-radius:14px;font-size:12px;">标签</span>
```

### Tip() 底部注释
```html
<section style="margin:12px 0;padding:0 16px;">
<p style="background:{{bg_warm}};padding:10px 14px;border-radius:8px;font-size:13px;color:{{text_warm}};line-height:1.8;">注释内容</p>
</section>
```

### Bar() 进度条
```html
<section style="margin:12px 0;padding:0 16px;">
<div style="background:{{border_light}};border-radius:10px;height:12px;overflow:hidden;">
  <div style="background:linear-gradient(90deg,{{primary}},{{primary_mid}});width:{{pct}}%;height:100%;border-radius:10px;"></div>
</div>
</section>
```

## 标题区

```html
<section style="margin:0;padding:24px 16px 16px;background:linear-gradient(180deg,{{bg}},{{bg_warm}});">
<p style="text-align:center;color:{{primary}};font-size:22px;font-weight:bold;letter-spacing:1px;">标题</p>
<p style="text-align:center;color:{{text_muted}};font-size:13px;margin:4px 0;">副标题</p>
<p style="text-align:center;color:{{text_muted}};font-size:11px;letter-spacing:2px;">路 上 书 简 · 第 N 期</p>
<hr style="border:0;border-top:1px solid {{border}};margin:14px 0 10px;">
</section>
```

## 文章底部

```html
<section style="margin:28px 0 14px;padding:0 16px;">
<hr style="border:0;border-top:1px solid {{border_light}};margin:0 0 14px;">
<p style="text-align:center;font-size:11px;color:{{text_muted}};letter-spacing:1px;">路上书简 · 通勤路上的读书伙伴</p>
</section>
```

## 字号层级（手机端）

| 元素 | 字号 | 字重 |
|------|------|------|
| 文章标题 | 22px | bold |
| 章节标题 H() | 18px | bold |
| 正文 P() | 15px | normal |
| 金句 Q() | 15px | normal |
| 表格 T() | 13px | normal |
| 注释/署名 | 11-12px | normal |
| Badge() | 12px | normal |

## 严格禁止

### CSS 禁止（微信真的会过滤）
- ❌ 外部样式表（`<link>`、`@import`）
- ❌ `position: fixed/sticky`（微信会剥离）
- ❌ `@keyframes` 动画（微信会剥离）

### 内容禁止
- ❌ 文字标注「原创声明」「AI生成」「合集」「阅读指南」
- ❌ 标题含「30 秒」「X 步」等装饰文字（会被微信进度条提取到底部）
- ❌ 分集文章只放时间轴 — 必须是脚本全文图文版
- ❌ 分集文章裁切脚本 — 脚本多少字就保留多少字
- ❌ 文章底部放元信息说明（文章是成品，不是说明书）

### 字体禁止（仅 SVG 图片，正文不受限）
- ❌ SVG 图片中引用 `'PingFang SC'`（Apple 专有，不可嵌入分发 + Linux 服务器无此字体→豆腐块）
- ❌ SVG 图片中引用 `'Microsoft YaHei'`（Microsoft 专有，不可嵌入分发 + Linux 服务器无此字体→豆腐块）
- ✅ SVG 图片必须用 `'Noto Sans CJK SC'`（SIL Open Font License，商用/嵌入全自由）
