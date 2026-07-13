# 微信公众号发布踩坑记录

## 凭据区分

| 环境变量前缀 | 用途 | 获取地址 |
|-------------|------|---------|
| `WEIXIN_*` | Hermes 消息平台（聊天 bot） | Hermes gateway setup |
| `WECHAT_APP_ID` / `WECHAT_APP_SECRET` | 公众号 API（草稿/发布） | mp.weixin.qq.com → 设置与开发 → 基本配置 |

两者独立。`WEIXIN_*` 配了不等于公众号 API 通了。

## API 流程

```
1. GET /cgi-bin/token?grant_type=client_credential&appid=APPID&secret=SECRET
   → access_token (有效期 7200 秒)

2a. POST /cgi-bin/material/add_material?access_token=TOKEN&type=image
   → multipart/form-data, field: media
   → 返回 media_id (封面图，thumb_media_id)

2b. POST /cgi-bin/media/uploadimg?access_token=TOKEN
   → multipart/form-data, field: media
   → 返回 {"url": "http://mmbiz.qpic.cn/..."}  (正文插图 CDN URL)

3. POST /cgi-bin/draft/add?access_token=TOKEN
   → JSON body: {"articles": [{"title":"...", "content":"...", "thumb_media_id":"..."}]}
   → 返回 media_id (草稿 ID)

4. 手动发布: 用户在 mp.weixin.qq.com → 草稿箱 → 群发
```

## 正文插图工作流

文章 HTML 中的图片先用占位链接，发布时自动上传替换：

```html
<!-- 写文章时用占位符 -->
<img src="https://mmbiz.qpic.cn/sz_mmbiz_png/placeholder_hero_ep01">

<!-- 发布脚本自动：上传 hero.png → 获得 CDN URL → 替换占位符 -->
```

占位格式：`placeholder_{type}_{ep_id}`，如 `placeholder_hero_ep01`、`placeholder_structure_ep03`、`placeholder_quote_ep07`。

脚本匹配这些占位符 → 找到对应的 PNG 文件 → 调用 `uploadimg` → 替换 HTML 中的 URL → 创建草稿。

## 个人订阅号 API 限制（已验证）

| 字段 | 限制 | 错误码 |
|------|------|--------|
| title | ≤ ~8 个中文字（约 24 字节） | 45003 `title size out of limit` |
| author | 不能设置（个人号不支持） | 45110 `author size out of limit` |
| digest | 建议不设（有时也报错） | - |
| thumb_media_id | **必填**（需先上传封面图再引用） | 40007 `invalid media_id` |
| 自动发布 API | 不支持（认证服务号才可以） | - |
| content | 无明确字节限制，建议控制在 50KB 内 | - |

## 封面图要求

- 尺寸: 900×500 px 或 900×383 px（推荐 2.35:1）
- 格式: PNG 或 JPEG
- 大小: < 2MB
- 生成方式: Pillow 库直接合成（无需外部图片资源）

```bash
pip3 install --break-system-packages Pillow
python3 -c "
from PIL import Image, ImageDraw, ImageFont
img = Image.new('RGB', (900, 500), '#5b7a6f')
# draw title text, save
img.save('cover.png')
"
```

- 1×1 像素的 PNG 会报 53402 `封面裁剪失败`——必须用正经尺寸。

## 已踩坑汇总

1. `WEIXIN_*` 和 `WECHAT_APP_*` 是两个完全独立的系统，不能混用
2. 个人订阅号 title 硬限制约 8 个中文字（如「断舍离」可以，「《断舍离》：进则出，出则进」不行）
3. 草稿必须先上传封面图拿到 media_id，否则报 `invalid media_id`
4. 不要设 author 字段（个人号），不要用 1×1 像素图（封面裁剪失败）
5. 发布脚本需要先 `export $(grep WECHAT_APP ~/.hermes/.env | xargs)` 加载凭据
