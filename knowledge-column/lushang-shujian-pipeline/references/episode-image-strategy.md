# 分集文章 — 图文策略

## 核心原则：图片做减法，排版做加法

经过实践验证的最佳方案：**每篇 1 张题图 + 正文全用 HTML/CSS 排版**。

之前的三层策略（hero + structure + quote 三张图）存在以下问题：
- 结构图和金句卡做成图片，在微信里不如 HTML/CSS 排版灵活
- SVG 转 PNG 依赖系统字体，Linux 服务器无 macOS/Windows 字体 → 中文全部变豆腐块（□）
- 维护成本高（21 张图 vs 7 张图），出错点多
- 已发布的综合文章证明：卡片+表格+渐变色块的 HTML 排版效果极好，且微信内稳定

## 当前方案：一图一文

| 元素 | 实现方式 | 理由 |
|------|---------|------|
| 题图（Hero） | 1 张 SVG→PNG，900×1200 | 文章封面 + 顶部视觉锚点，暗绿渐变 + 大字标题 |
| 框架/对比/分类 | HTML table + 色块卡片 | 微信原生渲染，不依赖图片，排版灵活 |
| 金句 | CSS gradient 卡片 | 已发布文章验证过，效果稳定 |
| 行动步骤 | 编号卡片 + 左边框色条 | 跟已发布文章同风格 |

## ⚠️ 字体版权（致命坑）

**所有 SVG 图片必须使用开源字体，不能写 macOS/Windows 系统字体名。**

SVG 中 `font-family="'PingFang SC'"` 或 `'Microsoft YaHei'` 在 Linux 服务器上渲染时字体不存在 → cairosvg 回退到无字体 → **中文全部变成豆腐块（□）**。即使部署到有这些字体的平台，也存在版权风险（PingFang SC 是 Apple 专有字体，Microsoft YaHei 是 Microsoft 专有字体，不可嵌入图片分发）。

**唯一推荐字体**：`Noto Sans CJK SC`（Google + Adobe 联合发布，**SIL Open Font License**，商用/嵌入/分发全自由）

安装：
```bash
sudo apt-get install fonts-noto-cjk fonts-noto-cjk-extra
```

SVG 中的正确写法：
```xml
font-family="'Noto Sans CJK SC','WenQuanYi Zen Hei',sans-serif"
```

备选回退字体 `WenQuanYi Zen Hei` 是 GPL-2 + 字体嵌入例外，也可合法嵌入图片，但不如 Noto 的 SIL OFL 干净（SIL OFL 是字体嵌入的黄金标准）。

## 题图设计规范

- 尺寸：900×1200（竖版，适配手机全屏）
- 配色：暗绿渐变背景（`#2c3e33` → `#5b7a6f`），米白文字（`#f5e6c8`），浅绿装饰元素（`#8bab9e`）
- 布局：期数徽章（顶部半透明 pill）→ 书名+作者 → 大字标题（44px 粗体，`letter-spacing:4`）→ 分隔线 → 副标题 → 底部金句 + 署名
- 氛围：极简、留白、圆圈线条做装饰
- 字体：`'Noto Sans CJK SC'`（SIL OFL），粗体用于标题，常规用于正文
- 标题格式：`断舍离{N}：{短标题}`，其中 N 用圈号数字 ①②③④⑤⑥⑦

生成命令：
```bash
python3 -c "
import cairosvg
cairosvg.svg2png(url='hero.svg', write_to='hero.png', output_width=900, output_height=1200)
"
```

## 正文排版（无图片，纯 HTML/CSS）

正文全部用 HTML/CSS 内联样式，参考已发布综合文章的风格：

- **卡片**：白色圆角 `border-radius:8px` + 细阴影 `box-shadow:0 1px 4px rgba(0,0,0,0.06)`
- **左边框卡片**：`border-left:4px solid #5b7a6f` 用于步骤/重点，不同颜色区分层级
- **渐变金句卡片**：`linear-gradient(135deg,#5b7a6f,#6b8e80)` 背景 + 白色文字
- **表格**：表头深色背景 `background:#5b7a6f;color:#fff` + 数据行交替 `#faf7f3` 底色
- **进度条**：CSS width 百分比 + 渐变填充（用于 7·5·1 比例等）
- **badge**：`border-radius:14px` pill 形状 + 暗绿背景，用于日期/类型标签
- **tip 条**：`background:#faf7f3` + 暖棕文字，用于底部注释和「走正路体感」

**CSS 字体栈**（正文引用系统字体，不嵌入，无版权问题）：
```css
font-family: -apple-system, BlinkMacSystemFont, 'PingFang SC', 'Noto Sans CJK SC', 'Microsoft YaHei', sans-serif;
```

## 占位符与发布

文章 HTML 中使用固定占位符 `PLACEHOLDER_HERO`（纯文本字符串），发布脚本自动替换：
```html
<img src="PLACEHOLDER_HERO" alt="...">
```

`wechat_publish.py` 的 `replace_placeholder_hero()` 函数负责：找到占位符 → 上传对应 `hero.png` 到微信 CDN（`/cgi-bin/media/uploadimg`）→ 替换为真实 URL。

## 不推荐的做法

| ❌ 不推荐 | 原因 |
|-----------|------|
| 每篇 3 张以上图 | 维护成本高，效果不如 HTML 排版，出错点多 |
| AI 生成照片 | 细节诡异，与极简气质冲突 |
| SVG 用平台字体名 | Linux 渲染变豆腐块 + 版权风险（PingFang/Microsoft YaHei 不可嵌入） |
| 结构图做成图片 | HTML table/CSS 更灵活可控，微信原生支持 |
| GIF/动图 | 微信 300 帧限制，与极简主题冲突 |
| 漫画/手绘风 | 跟「走正路」沉稳通透人设不搭 |
