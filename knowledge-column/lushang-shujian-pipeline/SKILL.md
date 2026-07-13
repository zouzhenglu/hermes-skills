---
name: lushang-shujian-pipeline
description: "路上书简知识栏目全自动内容流水线：拆书→脚本→音频→图文→知识卡片。"
version: 1.4.0
triggers:
  - 路上书简
  - 知识栏目
  - 通勤读书
  - 拆书生成
  - 断舍离
  - 经验归档
  - 记录错误
  - wiki
platforms: [linux]
---

# 路上书简 — 内容流水线

## 概述

「路上书简」是一个通勤读书知识栏目。每周一本书，每天一期 8-12 分钟音频。
周一引读 → 周二到周六核心内容 → 周日总结。
配合公众号长文和每日知识卡片。

## 项目结构

```
~/knowledge-column/
├── config/column.yaml               # 全局配置
├── book-pool/pool.yaml              # 书籍池（待读队列）
├── readers/zouzhenglu/style.md      # 读书人风格指南
├── scripts/
│   ├── pipeline.py             # 流水线主控
│   ├── book_utils.py           # 书籍池管理 + 知识图谱
│   └── tts_utils.py            # TTS 音频生成
└── output/
    ├── audio/{book_id}/        # 音频文件 + 脚本
    ├── articles/{book_id}/     # 公众号文章
    └── cards/{book_id}/        # 知识卡片
```

## 完整流程

### Phase 1: 获取下一本书

1. 运行 `python3 ~/knowledge-column/scripts/book_utils.py next` 获取下一本待处理的书
2. 运行 `python3 ~/knowledge-column/scripts/book_utils.py concept <tag>` 查询相关知识图谱引用
3. 运行 `python3 ~/knowledge-column/scripts/book_utils.py author <name>` 查作者背景
4. 更新状态为 processing

### Phase 2: 提取书籍文本

按优先级链获取内容（详见 `references/content-fallback.md`）：

1. **用户提供电子书文件** → pymupdf 提取 PDF/EPUB/TXT
2. **Web 搜索** → 搜索「书名 核心观点」「书名 目录」
3. **Agent 内置知识** → 知名畅销书可直接凭记忆产出（告知用户）
4. **标记 needs_content** → 以上都不够时，请用户补充

**电子书文件命名规范：** 使用中文书名，不用罗马音或英文音译。
- ✅ `断舍离.epub` `刻意练习.pdf` `atomic-habits.epub`（原文书名）
- ❌ `danshari.epub` `keyilianxi.pdf`（罗马音/拼音，无法一眼辨识）

如果文件是 PDF/EPUB：
```
pip3 install --break-system-packages pymupdf
python3 -c "import pymupdf; doc=pymupdf.open('文件路径'); [print(p.get_text()) for p in doc]"
```

### Phase 3: 生成 7 期内容计划

根据每周排期（config/column.yaml 中的 weekly_schedule），结合读者风格指南（readers/zouzhenglu/style.md），生成：

1. **内容拆解**：将书的主题拆成 5 个核心模块（对应周二到周六）
2. **引读脚本**（周一）：这本书讲什么、为什么值得读、路线图
3. **总结脚本**（周日）：回顾 + 作者故事 + 同类书推荐 + 下周预告

### Phase 4: 逐期生成音频

对每期（1-7）：

1. **写脚本**：
   - 加载 readers/zouzhenglu/style.md 中的人设和风格规范
   - 严格遵守 episode_template 的结构（opening/body/closing）
   - 使用场景钩子开头，不用「大家好欢迎收听」
   - 用「走正路」的第一人称视角，沉稳、咀嚼过再讲。使用短句，句间留白。
   - 每个观点配一个生活化故事
   - 结尾：一句话带走 + 思考题 + 预告
   - 长度控制：引读 8 分钟、核心 10 分钟、总结 15 分钟

2. **检查风格**：
   - 用 humanizer 技能检查：去掉 AI 腔、讲师腔、书面语
   - 确认没有 forbidden 列表中的词
   - 确认开头是场景钩子不是「大家好」

3. **生成音频**：

   **⚠️ 语音选择法则**（详见 `references/tts-voices.md`）：
   - **默认推荐 `zh-CN-YunxiNeural`（男声，讲故事风格）**，0% 语速。温暖自然，自带叙述节奏，适合中文讲书。不要用 Yunyang（太像新闻播音）。
   - 不同读书人可用不同声音区分角色。女声 `zh-CN-XiaoxiaoNeural` 适合第二个读书人。
   - **语速用 0%（默认），不要加速。** +10% 会导致「开头太赶」，失去自然停顿感。

   **时长控制**（基于 0% 语速，250 汉字/分钟）：
   - 引读 8 分钟 ≈ 2000 汉字
   - 核心期 10 分钟 ≈ 2500 汉字
   - 总结期 15 分钟 ≈ 3750 汉字

   **脚本写作要点**（适配 TTS 朗读）：
   - 用短句。句号多，逗号少。每句话说完给声音留一口气。
   - 段落之间空一行，让 TTS 自然停顿。
   - 避免长定语、多层嵌套的复句——人听着累，TTS 读着也生硬。
   - 写完自己默读一遍，喘不过气的地方就断句。

   ```bash
   # 先估算时长
   python3 ~/knowledge-column/scripts/tts_utils.py estimate <脚本文件>

   # 确认达标后生成
   python3 ~/knowledge-column/scripts/tts_utils.py speak \
     <脚本文件> \
     ~/knowledge-column/output/audio/{book_id}/{book_id}_ep{NN}_{label}.mp3 \
     zh-CN-YunxiNeural \
     "+0%"
   ```

   Edge TTS 安装（Debian/Ubuntu 需要 `--break-system-packages`）：
   ```bash
   pip3 install --break-system-packages edge-tts
   ```

4. **发送音频给用户审听**：
   用户无法直接访问服务器文件时，用 Hermes gateway 发送：
   ```bash
   # 先查可用平台
   hermes send --list

   # 发送音频（Telegram 最可靠）
   hermes send -t telegram "说明文字\n\nMEDIA:/path/to/audio.mp3"

   # 发送文章
   hermes send -t telegram -f /path/to/article.md -s "标题"

4. **保存脚本**：
      ```bash
      python3 ~/knowledge-column/scripts/pipeline.py new-book {book_id}
      ```
      将脚本保存到 output/audio/{book_id}/scripts/ep{NN}_{label}.txt

      **音频文件命名规范**：`{book_id}_{书名}_ep{NN}_{标签}.mp3`
      例如 `book-001_断舍离_ep01_引读.mp3`。包含书名方便本地查找，上传微信素材库后也能看到。

### Phase 5: 生成公众号文章

#### 两种发布模式

| 模式 | 产出 | 适用场景 |
|------|------|---------|
| **综合文章**（默认） | 1 篇长文，整合全书精华 | 需要一篇「代表作」覆盖全书 |
| **分集文章**（`--episodes`） | 7 篇独立文章，每篇 1 张题图 | 每天一篇跟音频同步推送，读者按天追读 |

分集文章每篇遵循 **一图一文策略**（见 `references/episode-image-strategy.md`）：

⚠️ **分集文章必须是可读图文，不是播客 show notes。** 详见 `references/episode-content-rules.md`。禁止只放时间轴和金句——用户不看音频也要能获得完整价值。每篇文章必须使用 `references/article-layout.md` 中的模板组件（章节标题 H()、金句卡片 Q()、暖色提示框 X()、故事块 S()、表格 T()、正文段落 P()），禁止纯段落堆砌。

**分集文章 = 音频脚本全文 + 模板排版。** 用户明确要求「接受重复」——文章和音频讲同一内容，但文章是图文版。不要把脚本内容裁掉一半。脚本多少字，文章就保留多少字，只加排版组件。

**音频嵌入**：微信 API 不支持自动嵌入音频（mpvoice 被拦截，audio 被过滤）。文章顶部放提示，用户后台编辑器手动插入。详见 `references/episode-content-rules.md`。

**发布排期**：
- 分集文章 07:00 发 → 音频 07:30 推（文章先出，用户知道今天聊什么再听）
- 综合文章周日 20:00 发（全周完结后，不剧透）
- **1 张题图**（Hero SVG→PNG）作为文章封面和顶部视觉锚点
- **正文全用 HTML/CSS** 排版：卡片/表格/渐变色块/左边框/进度条，不依赖额外图片
- 结构框架和金句用 CSS 实现，不用图片——更灵活、更稳定、维护成本更低

所有图用 SVG 设计 → cairosvg 转 PNG（WeChat 不支持 SVG）→ 上传到微信 CDN。SVG 字体必须使用 Noto Sans CJK SC（SIL OFL），详见下方字体版权说明。

**字体版权（致命）**：Linux 服务器渲染 SVG 时，`font-family` 不能写 `'PingFang SC'` 或 `'Microsoft YaHei'`——这些字体不存在会导致中文变豆腐块（□），且为专有字体不可嵌入分发。必须使用 `'Noto Sans CJK SC'`（SIL Open Font License）。安装：`sudo apt-get install fonts-noto-cjk`。

**核心原则：文章做音频做不到的事。** 音频擅长讲故事和情绪，文章擅长表格、清单、框架、对比。文章不是音频的文字版，是音频的「使用说明书」。

⚠️ **生成文章前必须先读取** `references/article-layout.md`，严格按照其中的组件模板、配色变量和禁止项执行。配色从 `config/themes.yaml` 读取（当前主题：`forest` 墨绿森林）。

#### 5a. 先生成 Markdown 结构稿

保存到 `output/articles/{book_id}/article.md`。必须包含以下模块（顺序可调）：

1. **全局视角（全书骨架速览）**：树形结构或表格，让读者 10 秒看懂全书框架
2. **核心框架表**：一张表讲清所有关键概念（音频讲不了表）
3. **自我诊断工具**：层级/评分/清单，让读者对号入座
4. **行动指南**：可截图执行的步骤清单，每步有时间预估
5. **横向对比**：作者 vs 同类书作者（如山下英子 vs 近藤麻理惠）
6. **延伸推荐**：3 本，每本 100 字。跟书池分离，标注「后续可安排专周解读」
7. **结尾互动**：评论区问题 + 下周预告

**硬性规则：**
- 音频讲过的故事一笔带过，不重复
- 每篇文章至少一张表格（概念对比/步骤清单/层级诊断）
- 行动指南部分必须可截图执行
- 延伸推荐跟书池分离——推荐的书不等于下一本要拆的书

#### 5b. 再生成 WeChat HTML 版

基于 Markdown 结构稿，生成公众号可直接粘贴的富文本 HTML。使用内联样式，采用卡片式分层设计。关键设计要求：

- **配色**：墨绿/青灰色系（低饱和度，长时间阅读不累），米白底色，深灰正文
- **顶部**：渐变背景 + 大标题 + 期号/读书人署名，营造「杂志感」
- **正文区**：每个模块独立卡片（白色圆角 + 细阴影），模块间留白充足
- **表格**：表头深色背景 + 白色文字，数据行交替浅色
- **金句/强调**：深色背景卡片 + 白色文字作为视觉停顿
- **行动清单**：每步左边框着色 + 时间标签 + 层级缩进
- **字体层级**：标题 18px 粗体 → 副标题 → 正文 15px → 注释 12px
- **宽度**：max-width 640px，移动端友好

HTML 模板参考：`templates/article_wechat.html`

保存到 `output/articles/{book_id}/article_wechat.html`。

#### 5d. 分集文章生成流程（`--episodes` 模式）

完整步骤：

```bash
# 1. 创建目录
mkdir -p ~/knowledge-column/output/articles/{book_id}/episodes/{ep01..ep07}/images

# 2. 安装中文字体（SVG 渲染必需，SIL OFL 许可）
sudo apt-get install fonts-noto-cjk fonts-noto-cjk-extra

# 3. 生成每期的题图 SVG（使用 Noto Sans CJK SC 字体）并转 PNG
#    每篇只生成 1 张 hero 图。正文排版（框架/金句/行动清单）全部用 HTML/CSS。
#    详见 references/episode-image-strategy.md

# 4. 写每期的 WeChat HTML，题图用占位字符串 PLACEHOLDER_HERO
#    正文用已有的卡片+表格+渐变色块风格，不依赖图片

# 5. 一键发布（自动上传题图 + 替换占位 → 创建草稿）
python3 scripts/wechat_publish.py {book_id} --episodes         # 全部 7 期
python3 scripts/wechat_publish.py {book_id} --episodes --ep ep01  # 只发一期
```

**⚠️ 字体版权**：SVG 图片嵌入的字体必须是开源许可。使用 `Noto Sans CJK SC`（SIL Open Font License），**禁止**在 SVG 中引用 `PingFang SC` 或 `Microsoft YaHei`（专有字体，不可嵌入分发，且 Linux 服务器上不存在会导致中文变豆腐块）。

标题格式：`断舍离{N}：{短标题}`，N 用圈号数字 ①②③④⑤⑥⑦ 以控制在 8 个中文字符限制内。用户可在后台手动加长到 64 字。

#### 5c. 发送给用户审核

```bash
# 先发 Markdown 版（快速阅读）
hermes send -t telegram -f output/articles/{book_id}/article.md -s "📄 文章审核"

# 发 HTML 渲染截图（看视觉效果）
# 用 browser_navigate 打开 file:// 路径，再 browser_vision 截图
hermes send -t telegram "MEDIA:/screenshot/path.png"
```

### Phase 6: 发布到微信公众号

#### 6a. 用户配置

用户需要在 `~/.hermes/.env` 中设置（从 https://mp.weixin.qq.com → 设置与开发 → 基本配置 获取）：

```
WECHAT_APP_ID=wx***
WECHAT_APP_SECRET=***
```

注意：`WEIXIN_*` 是 Hermes 消息平台的凭据（聊天用），**不是**公众号 API 凭据。两个独立。

#### 6b. 生成封面图

微信草稿需要封面图（thumb_media_id）。用 Pillow 生成简单的矩形封面（墨绿色渐变 + 白色书名 + 读书人署名）：

```bash
pip3 install --break-system-packages Pillow
python3 -c "
from PIL import Image, ImageDraw, ImageFont
# 900x500, gradient background, title text, save
"
```

#### 6c. 创建草稿（⚠️ 关键：JSON 编码）

**个人订阅号（未认证）的 API 限制**：

- ❌ 不能设置 `author` 字段（会报 45110 `author size out of limit`）
- ❌ 不能设置 `digest` 字段（有时也限制）
- ❌ 标题硬限制约 **8 个中文字符**（24 字节），超过报 45003 `title size out of limit`
- ❌ 不能通过 API 自动发布，只能推送草稿 → 用户手动点「群发」
- ✅ 认证服务号无上述限制，可全自动

**⚠️ JSON 编码致命陷阱**：**永远不要用 `requests.post(json=payload)`**。Python `json.dumps` 默认 `ensure_ascii=True`，把所有中文转成 `\uXXXX` 转义符。微信 API 原样存储，草稿箱显示乱码。

**正确做法**：
```python
body = json.dumps(payload, ensure_ascii=False).encode('utf-8')
requests.post(url, data=body, headers={'Content-Type': 'application/json; charset=utf-8'})
```

详见 `references/wechat-api-notes.md`。

**已验证可用的最小 payload**：

```python
payload = {
    'articles': [{
        'title': '断舍离',            # ≤8 个中文字
        'content': html_content,
        'thumb_media_id': media_id,   # 先上传封面图获取
        'need_open_comment': 1,
    }]
}
```

#### 6d. 发布脚本

已提供 `scripts/wechat_publish.py`，支持两种模式：

```bash
source ~/.hermes/.env && export WECHAT_APP_ID WECHAT_APP_SECRET

# 综合文章模式（1 篇长文）
python3 scripts/wechat_publish.py {book_id}

# 分集文章模式（7 篇独立文章，自动上传正文插图）
python3 scripts/wechat_publish.py {book_id} --episodes
python3 scripts/wechat_publish.py {book_id} --episodes --ep ep01  # 只发一期
```

脚本自动完成：获取 token → 上传正文插图（`/cgi-bin/media/uploadimg`）→ 替换占位链接 → 上传封面 → 创建草稿。

#### 6e. 发布后

用户去 https://mp.weixin.qq.com → 草稿箱 → 编辑标题/作者/摘要 → 预览 → 群发。

详见 `references/wechat-publish.md`。

### Phase 7: 生成知识卡片（可选，需 FAL_KEY）

对每期内容，使用 baoyu-infographic 技能生成知识卡片。
风格: morandi-journal，布局: bento-grid，竖版。

**前提条件**：用户需去 https://fal.ai 免费注册，设 `FAL_KEY=xxx` 到 `~/.hermes/.env`。

### Phase 8: 收尾

1. 更新书籍状态为 completed
2. 更新知识图谱:
   ```bash
   python3 -c "
   from scripts.book_utils import add_book_to_graph
   add_book_to_graph({'id': '{book_id}', 'title': '{title}', 'author': '{author}', 'tags': {tags}})
   "
   ```
3. 汇报产出清单
4. **👉 执行 Phase 9 经验归档 + Phase 10 Git 备份**

### Phase 9: 经验归档到 Wiki ⭐

> **这是知识积累的核心环节。每次发布完成后必须执行。**
> 目标：错误记录 → 经验沉淀 → 永不重复犯错。

**Wiki 位置:** `~/wiki/`
**Wiki 索引:** `~/wiki/index.md`

#### 何时触发
每次完成 Phase 6（公众号发布）或 Phase 8（收尾）后，回顾本次流程：
- 遇到了什么新错误？
- 学到了什么新经验？
- 有什么流程可以优化？

#### 操作步骤
1. **检查是否有新错误：** 发布过程中如果遇到任何问题（哪怕立即修复了），记录到 wiki。
2. **如果已有相关页面，追加：** 如 `~/wiki/concepts/错误记录-微信公众号发布.md`
3. **如果是全新问题，新建页面：** 按 `SCHEMA.md` 的「错误记录」模板
4. **更新日志：** 在 `~/wiki/log.md` 追加 `## [YYYY-MM-DD] update | 错误记录: <简述>`
5. **关联自检清单：** 如果错误可以预防，同步更新 `~/wiki/concepts/发布流程.md` 的自检清单

#### 错误记录模板
```markdown
## 错误: <简述>
**日期:** YYYY-MM-DD
**场景:** 在做什么时发生
**现象:** 具体表现
**原因:** 根因
**修复:** 怎么解决的
**预防:** 以后怎么避免
```

#### 已有经验页面（写入前先读）
- `~/wiki/concepts/错误记录-微信公众号发布.md` — 已记录 4 个错误
- `~/wiki/concepts/内容准确性核查清单.md` — 内容归因避坑
- `~/wiki/concepts/发布流程.md` — 含自检清单
- `~/wiki/SCHEMA.md` — 标签体系和页面规则

#### 不要做的
- 不要建重复页面（先读 index.md 确认是否已有）
- 不要写流水账（只记录有复用价值的经验）
- 不要跳过（这是「以后永不重复犯错」的基础）

### Phase 10: Git 备份与版本发布 🔄

> **每次发布完成后自动执行。Git 是防止灾难性数据丢失的最后一道防线。**

涉及 3 个 Git 仓库：

| 仓库 | 本地路径 | GitHub |
|------|---------|--------|
| knowledge-column | `~/knowledge-column/` | `zouzhenglu/hermesAliRepo` |
| media-wiki | `~/wiki/` | `zouzhenglu/media-wiki` |
| hermes-skills | `~/.hermes/skills/` | `zouzhenglu/hermes-skills` |

#### 操作步骤

```bash
# 1. knowledge-column — 提交产出文件变更
cd ~/knowledge-column
git add -A
git commit -m "{book_id}-{书名}: {描述}"   # 例: "book-001-断舍离: 7期音频+文章完成"
git tag "{book_id}-{书名}-v{version}"      # 例: "book-001-断舍离-v1"
git push && git push --tags

# 2. wiki — 提交经验记录变更
cd ~/wiki
git add -A
if git diff --cached --quiet; then
  echo "wiki: 无变更"
else
  git commit -m "{book_id}: {经验摘要}"
  git push
fi

# 3. skills — 提交 skill 变更（如有）
cd ~/.hermes/skills
git add -A
if git diff --cached --quiet; then
  echo "skills: 无变更"
else
  git commit -m "skills: {变更描述}"
  git push
fi
```

#### Tag 命名规范
- 格式: `{book_id}-{书名}-v{version}` — 例: `book-001-断舍离-v1`, `book-002-专注-v1`
- 修复性变更用 patch: `book-001-断舍离-v1.1`
- 纯英文书名用英文: `book-003-atomic-habits-v1`
- 永远不删除旧 tag，保留所有发布记录

#### 原则
- **自动但不盲目** — commit 之前 `git diff --stat` 确认变更范围
- **变更内容写入 commit message** — 不要写 "update" 或 "fix"
- **tag 标记里程碑** — 每次完整发布至少打一个 tag
- **push 后验证** — `git ls-remote --tags origin` 确认 tag 已推送

## 读者风格速查

当前默认读书人：**走正路**（`readers/zouzhenglu/style.md`）。

**必须遵守：**
- 开头用场景钩子，不用「大家好欢迎收听」「今天我们来讲」
- 语气沉稳但不沉重。可以停顿，可以留白。
- 用短句。句号多，逗号少。每句话给声音留一口气。
- 用自己经历和故事讲道理，不干讲概念。
- 可以引经据典，但要马上接人话解释。
- 有原则但不教化。讲「应该」时一定配「为什么」。
- 不用「专家指出」「研究表明」等虚指。
- 不在结尾说「希望对你有所帮助」「让我们一起」。
- 不用感叹号（除非特别需要，一期不超过 1 个）。
- 完整人设见 `readers/zouzhenglu/style.md`。

**走正路** 是唯一的读书人人设（通透叙事型）。详见 `readers/zouzhenglu/style.md`。

## 每周排期

| 日期 | 类型 | 时长 | 内容 |
|------|------|------|------|
| 周一 | 引读+核心洞察 | 10min | 前半段：这本书在讲什么/为什么值得读/路线图。后半段：直接抛出全书最颠覆认知的一个观点——让周一就有独立价值，而不只是预告 |
| 周二 | 深读·一 | 10min | 核心内容第 1 部分 |
| 周三 | 深读·二 | 10min | 核心内容第 2 部分 |
| 周四 | 深读·三 | 10min | 核心内容第 3 部分 |
| 周五 | 深读·四 | 10min | 核心内容第 4 部分 |
| 周六 | 深读·五 | 10min | 核心内容第 5 部分 |
| 周日 | 总结 | 15min | 回顾 + 作者故事 + 同类推荐 + 下周预告 |

周日同步发布公众号文章。

## 书池 vs 延伸推荐（重要区分）

| | 书池 (Book Pool) | 延伸推荐 |
|---|---|---|
| 目的 | 深度解读 | 顺带推荐 |
| 投入 | 7 期音频 + 1 篇长文 + 卡片 | 音频里 2 分钟 + 文章里 100 字 |
| 来源 | 用户来选、用户给电子书 | Agent 推荐，从内置知识库里挑 |
| 触发升级 | — | 听众反响好 → 用户决定加进池子 |

**Agent 可以主动推荐延伸书单**（基于内置知识），不需要用户预先放入书池。
推荐时标注「可在后续安排专周解读」，用户觉得需要再排入池子。

## 用户如何提供电子书

用户可以通过已配置的 gateway 平台（如 Telegram）直接发送文件给 Hermes bot。
收到后 Agent 检查 `~/knowledge-column/book-pool/` 目录下的新文件。

如果用户无法通过 gateway 发文件，也可以：
- SCP/SFTP 到 `~/knowledge-column/book-pool/`
- 提供可下载的 URL

## 内容来源的诚实分级

Agent 产出内容时，必须明确告知用户内容来源：
- ✅ 「有电子书文件，基于原文拆解」——最佳
- ⚠️ 「无电子书，基于公开书评/拆书稿/训练数据」——告知用户，建议补文件
- ❌ 永远不要假装读过一本你没有原文的书。畅销书可以凭训练数据产出七八成准确度，但细节可能缺失或偏差。

## 参考文档

- `references/wechat-api-notes.md` — JSON 编码致命陷阱（ensure_ascii）、标题 8 字限制、个人号 API 权限矩阵、草稿分页删除
- `references/article-format-rules.md` — 文章里不要放的东西（阅读指南文字、API 做不到的标注）、音频命名规范
- `references/tts-voices.md` — Yunxi vs Yunyang 对比、语速选择、时长估算
- `references/pitfalls.md` — 语音选型、人设迭代、周一引读空洞等已知坑
- `references/episode-image-strategy.md` — 分集文章三层图文策略：题图/结构图/金句卡设计规范
- `templates/article_wechat.html` — WeChat 兼容 HTML 模板

## 关联资源（外部）

- **Wiki:** `~/wiki/` (GitHub: `zouzhenglu/media-wiki`) — 自媒体经验知识库，含错误记录和自检清单
- **Git 仓库:** `zouzhenglu/hermesAliRepo` ← 栏目代码+产出；`zouzhenglu/hermes-skills` ← Skills 备份
- **认证:** `~/.git-credentials` 存 GitHub token，无需每次手动输入

### 语音选择常见错误
- ❌ YunyangNeural +10% 语速 → 像新闻播报，开头太赶，听众体验差
- ❌ 长句连排不加标点 → TTS 会一口气读完，没有自然停顿
- ✅ YunxiNeural +0% 语速 → 讲故事风格，自带节奏，适合中文讲书

### TTS 失败的降级方案
如果 edge-tts 不可用，使用 Hermes 内置的 text_to_speech 工具。

### 图片生成需要 FAL_KEY
baoyu-infographic 和 image_generate 需要 FAL.ai API Key。
用户去 https://fal.ai 免费注册，设 `FAL_KEY=xxx` 到 `~/.hermes/.env` 即可。

### 用户无法访问服务器文件
用 `hermes send` 通过已配置的 gateway 平台（Telegram/微信等）发送：
```bash
hermes send --list                    # 查看可用平台
hermes send -t telegram "MEDIA:/path/to/file.mp3"
```

### 知识图谱跨书引用
每期脚本生成前，先查询知识图谱中是否有相关概念。
如果某概念在之前的书里也出现过，脚本中自动加入：「这个概念之前在讲《XXX》时也聊过……」

### ⚠️ 文章内容禁止项

以下内容**不要**出现在最终推送的文章里：

- 「阅读指南」「这不是音频的文字版」等自我说明文字 —— 用户已明确要求删除
- 文字标注的「原创声明」「AI 生成」「合集标签」—— API 做不到的功能，不要在文章里用文字假装。用户自己在后台设
- 任何解释文章定位的 meta 文字 —— 文章是成品，不是说明书

文章底部只需：栏目署名 + 干净的结尾。多余的标注一律不要。

### 草稿箱 API 检索 Bug

微信 `draft/batchget` + `draft/get` 取回的标题永远是乱码（哪怕 push 时编码正确）。**不要依赖 API 检索来验证或清理草稿。** 只能：

1. 推送新草稿 — 编码正确，后台正常显示
2. 用户手动在后台清理旧草稿
3. 不要尝试用 API 程序化清理（会因为标题乱码匹配不上而误删正常草稿）
