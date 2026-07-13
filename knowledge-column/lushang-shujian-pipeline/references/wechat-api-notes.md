# 微信公众号 API 踩坑笔记（个人订阅号）

## JSON 编码：血的教训

**永远不要用 `requests.post(json=payload)`。** Python 的 `json.dumps` 默认 `ensure_ascii=True`，会把所有中文变成 `\uXXXX` 转义符。微信 API 收到后原样存储，草稿箱显示乱码。

**正确做法：**
```python
body = json.dumps(payload, ensure_ascii=False).encode('utf-8')
requests.post(url, data=body, headers={'Content-Type': 'application/json; charset=utf-8'})
```

## 个人订阅号 API 能力矩阵

| 功能 | API 字段 | 个人订阅号 | 认证服务号 |
|------|---------|-----------|-----------|
| 推送草稿 | `draft/add` | ✅ | ✅ |
| 上传封面 | `material/add_material?type=image` | ✅ | ✅ |
| 开启评论 | `need_open_comment: 1` | ✅ | ✅ |
| 仅粉丝评论 | `only_fans_can_comment` | ✅ | ✅ |
| 设置作者 | `author: "xxx"` | ❌ 报 45110 | ✅ |
| 设置摘要 | `digest: "xxx"` | ❌ 不稳定 | ✅ |
| 原创声明 | `claim_original: 1` | ❌ 后台手动 | ✅ |
| AI 标注 | `ai_content_type: 1` | ❌ 后台手动 | ✅ |
| 合集归类 | 无 API | ❌ 后台手动 | ❌ 也需手动 |
| 赞赏 | 无 API | ❌ 后台手动 | ❌ |
| 自动群发 | `freepublish/submit` | ❌ | ✅ |

## 标题限制

个人订阅号草稿标题硬限制约 **8 个中文字符**（24 字节 UTF-8），超长报 `45003 title size out of limit`。

- ❌ `「断舍离」：进则出，出则进，然后，再出。`（14 字，报错）
- ❌ `断舍离：进则出，出则进`（10 字，报错）
- ✅ `断舍离：进则出`（7 字，通过）
- ✅ `断舍离`（3 字，通过）

发布后在后台可手动加长到 64 字。

## 封面图规格

- 推荐 900×500 px（2.35:1 或 16:9）
- 不能用 1×1 的极小图，会报 `53402 封面裁剪失败`
- 用 Pillow 生成简单封面即可：深色渐变 + 白色书名 + 署名

## 语音素材

微信素材库支持上传 MP3 作为语音素材：
```
/cgi-bin/material/add_material?type=voice
```
但仅限嵌入公众号文章播放，不能用于外部播客平台（无公网 URL）。

## 文章底部规范

**禁止在文章里用文字标注 API 做不到的功能。** 文章是成品，不是说明书。

❌ 不要在文章底部加：
- 「原创声明：本文由路上书简原创」—— 用户自己在后台开
- 「AI 辅助生成标注」—— 用户在后台开
- 「合集：路上书简」—— 用户在后台选
- 「阅读指南」「这不是音频的文字版」等自我说明
- 任何解释文章定位的 meta 文字

✅ 底部只需：
- 栏目署名（「路上书简 · 通勤路上的读书伙伴」）
- 播客收听入口（小宇宙 / Apple Podcasts / 喜马拉雅）
- 点赞/在看/分享引导（可选）
- 下周预告（总结期）

## 发布流程

1. API 推送草稿到后台
2. 用户去 mp.weixin.qq.com → 草稿箱
3. 改标题（可加长到 64 字）
4. 补作者名
5. 选合集
6. 开启原创/AI标注/赞赏（如有权限）
7. 预览 → 群发
