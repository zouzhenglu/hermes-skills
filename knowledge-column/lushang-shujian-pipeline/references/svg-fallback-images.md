# SVG 降级图片生成方案

当 `image_generate` 不可用时（FAL_KEY 未配置 / 额度不足），用纯 SVG → cairosvg → PNG 生成视觉资产。

## 适用场景

- 公众号头像
- 文章封面图 / 题图
- 知识卡片（简单排版）
- 任何可以通过几何图形 + 文字表达的静态图

## 前置条件

```bash
# 字体（SIL OFL，必须安装）
sudo apt-get install fonts-noto-cjk fonts-noto-cjk-extra

# SVG 转 PNG
pip3 install --break-system-packages cairosvg
```

## 工作流

1. **手写 SVG** — 在 `/home/admin/knowledge-column/output/images/` 下创建 `.svg` 文件
   - 尺寸通常 800x800（头像）或 900x500（封面）
   - 字体：`font-family="'Noto Sans CJK SC', sans-serif"`
   - 配色：从 `config/themes.yaml` 取，当前主题 forest（墨绿 #3d5a4f）

2. **转 PNG**
   ```bash
   cairosvg input.svg -o output.png -W 800 -H 800
   ```

3. **验证** — 用 `vision_analyze` 检查效果（居中、对比度、小尺寸可读性）

## 头像设计要点

微信头像以圆形裁剪显示，设计时注意：

- 核心视觉元素必须居中，不能依赖四角
- 用大面积色块 + 简单图形，不要细密纹理（缩小后糊掉）
- 前景与背景对比度要足够（深底浅图 或 浅底深图）
- 800x800 是安全尺寸，微信会压缩但不会严重失真

## 与 image_generate 的取舍

| | SVG 手写 | image_generate |
|---|---|---|
| 可控性 | 100% 精确 | 描述驱动，需多轮调整 |
| 速度 | 秒级 | 秒级（API） |
| 复杂场景 | 不适合照片级/有机纹理 | 适合 |
| 依赖 | 无外部 API | 需要 FAL_KEY |
| 文字渲染 | 精确 | 不可控（可能乱码） |

**结论**：封面图、头像、卡片等几何/文字为主的设计优先用 SVG；照片级/复杂艺术效果用 image_generate。
