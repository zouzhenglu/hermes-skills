# WeChat CSS 实测兼容表

> 基于微信公众号实际发布验证。测试环境：个人订阅号，2026-07。
> 规则：以微信客户端实际渲染为准，不以编辑器预览为准。

## 可用 ✅

| CSS 属性 | 验证状态 | 推荐值 | 备注 |
|----------|---------|--------|------|
| `color` | ✅ | `#2c2c2c` 正文 | 始终可用 |
| `background` / `background-color` | ✅ | 任意 hex | 始终可用 |
| `background: linear-gradient(...)` | ✅ | `135deg, #5b7a6f, #6b8e80` | 金句卡片、标题区已验证 |
| `box-shadow` | ✅ | `0 1px 6px rgba(0,0,0,0.06)` | 微量阴影做卡片层次，微信不过滤 |
| `border-radius` | ✅ | `8px` 卡片, `14px` pill | 任意值均可，已验证 4-14px |
| `border` / `border-left` / `border-right` | ✅ | `4px solid #5b7a6f` | 左边框标题标志 |
| `font-size` | ✅ | `15px` 正文, `18px` 标题 | 建议不低于 14px |
| `font-weight` | ✅ | `bold` / `normal` | 400/700 均可用 |
| `line-height` | ✅ | `2` (中文), `2.2` (金句) | 单位可用数字或 px |
| `text-align` | ✅ | `center` / `left` | 始终可用 |
| `padding` / `margin` | ✅ | 标准值 | 始终可用 |
| `letter-spacing` | ✅ | `1-2px` | 标题和署名已验证 |
| `<section>` | ✅ | 推荐 | 语义标签 |
| `<div>` | ✅ | 可用 | 与 section 无差异 |
| `<table>` / `<tr>` / `<td>` | ✅ | 内联样式 | 表头深色 + 数据行交替 |
| `<img src="...">` | ✅ | 微信 CDN URL | 需先 `/cgi-bin/media/uploadimg` 上传 |
| `<hr>` | ✅ | `border-top:1px solid` | 分隔线 |
| `font-family` (系统栈) | ✅ | `-apple-system, ..., sans-serif` | 引用系统字体不涉及嵌入 |

## 会被微信过滤 ❌

| CSS 属性 | 现象 | 替代方案 |
|----------|------|---------|
| `<link>` / `@import` | 完全剥离 | 必须内联 style |
| `position: fixed` / `sticky` | 剥离 | 不用 |
| `@keyframes` 动画 | 剥离 | 静态替代 |
| `mpvoice` 标签 | 被拦截 | 后台编辑器手动插入音频 |
| `<audio>` 标签 | 被过滤 | 同上 |
| `font-family: 'PingFang SC'` (SVG内) | Linux 渲染变豆腐块 □ | `'Noto Sans CJK SC'` (SIL OFL) |
| `font-family: 'Microsoft YaHei'` (SVG内) | 同上 + 版权风险 | 同上 |

## 字体版权说明

| 场景 | 可用字体 | 原因 |
|------|---------|------|
| HTML 正文 `font-family` | `-apple-system, PingFang SC, Microsoft YaHei, ...` | 引用系统已安装字体，不嵌入，无版权问题 |
| SVG 图片 `font-family` | `'Noto Sans CJK SC'` | 嵌入图片分发，必须 SIL OFL 许可 |

## 字号层级（手机端推荐）

```
22px bold   — 文章大标题
18px bold   — 章节标题 H()
15px normal — 正文 P() / 金句 Q()
13px normal — 表格 T()
12px normal — Badge() / 音频提示
11px normal — 署名 / 底部
```
