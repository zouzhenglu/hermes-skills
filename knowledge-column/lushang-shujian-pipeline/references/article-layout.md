# 路上书简 · 公众号文章排版规范

## 核心原则

- 每篇 WeChat 安全 HTML：`<section>` + 内联样式，无外部 CSS
- 所有颜色从 `config/themes.yaml` 读取，通过 `c = themes['themes'][theme_name]['colors']` 加载
- 字体仅用开源栈：`'Noto Sans CJK SC', 'WenQuanYi Zen Hei', 'PingFang SC', sans-serif`
- 禁止 `-apple-system`、`Microsoft YaHei`（方正专有字体，有嵌入授权争议）
- 禁止 `box-shadow`、`linear-gradient`、`border-radius` > 4px

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

### Q() 金句卡片
```html
<section style="margin:14px 0;padding:0 16px;">
<p style="background:{{primary}};color:#fff;padding:14px 18px;border-radius:4px;text-align:center;font-size:14px;line-height:2.2;">金句</p>
</section>
```

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

## 严格禁止

- ❌ `<div>` 替代 `<section>`
- ❌ `box-shadow`、`linear-gradient`、`border-radius` > 4px
- ❌ `-apple-system`、`Microsoft YaHei`
- ❌ 文字标注「原创声明」「AI生成」「合集」「阅读指南」
- ❌ 标题含「30 秒」「X 步」等装饰文字（会被微信进度条提取到底部）
- ❌ 分集文章只放时间轴 — 必须是脚本全文图文版
- ❌ 分集文章裁切脚本 — 脚本多少字就保留多少字
