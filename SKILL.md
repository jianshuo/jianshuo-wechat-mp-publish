---
name: jianshuo-wechat-mp-publish
description: Use when the user wants to write or publish a 微信公众号 (WeChat Official Account) article — they share rough thoughts, a draft, or notes and ask for help polishing, generating a cover image (题图) and explanation illustration (解释图), or preparing the article for upload to mp.weixin.qq.com. Triggers include "写一篇微信文章", "公众号", "润色", "题图", "发公众号", "/jianshuo-wechat-mp-publish".
---

# jianshuo-wechat-mp-publish

帮助用户写微信公众号文章。**轻润色，不重写。** 自动生成题图和解释图，输出可直接粘贴到公众号后台的内容包。

## Core Principle

**保留作者的语气和节奏。** 用户的思路和表达方式是文章的灵魂。你只做四件事：

1. 修明显错字和重复字
2. 调整段落（微信读者习惯短段落，每段 1–3 句）
3. 抚平特别拗口的句子（保守，能不动就不动）
4. 准备配套素材（题图、标题候选、摘要）

**不要做的事：**
- 不要改变作者的用词偏好
- 不要加 AI 味儿的连接词（"首先"、"其次"、"综上所述"、"总而言之"、"值得注意的是"）
- 不要把口语改成书面语
- 不要加 emoji（除非原文有）
- 不要重新组织段落顺序
- 不要"提升"作者的表达——他写他的，你只是清洁工

## When This Skill Fires

- 用户提供一段思路、草稿、或语音转写文字
- 用户说"帮我写一篇公众号"、"润色一下"、"准备发布"
- 用户在公众号写作工作目录下工作（默认 `~/wechat-publish/` 或 `~/code/wechat-publish/`，可由用户配置）

## Workflow

### Step 0: 接收输入

用户会以以下形式给你内容：
- 完整草稿（最常见）
- 几段散乱的思路 / bullet points
- 一段长文字，没有分段
- 语音转写（可能有错字、重复）

如果输入太散，**问一个问题**："这是想写一篇文章，还是几个独立想法？" —— 但只问这一次。

### Step 1: 轻润色

打开一个 markdown 文件，把用户的内容粘进去。然后**只**做下面这些：

- 修错字（"的得地"乱用、同音字错字、重复字"我我"）
- 段落切分：每 1–3 句一段。微信里长段落很难读
- 拗口的地方做最小改动。如果改动后语气变了，宁可不改
- 标点统一：中文用全角逗号句号，英文/数字之间空格
- 保留原本的开头和结尾——这是作者的标志性特征

**改动的尺度参考：** 如果你改的字数超过原文的 5%，你改太多了。退回去。

### Step 2: 标题候选

给用户 **3 个标题候选**：

- A) 直白型：直接说文章讲什么
- B) 故事型：从一个场景或冲突切入
- C) 用户原文里的一句话：从草稿里摘最有味道的一句

不要做：标题党、夸张、"震惊"、"必看"。

### Step 3: 摘要 (50–80 字)

公众号摘要是发到朋友圈/对话框时的预览。要点：

- 不是文章第一段的复制
- 一句话说清楚读者会获得什么
- 用作者的语气，不是营销腔

### Step 4: 配图（每篇两张）

每篇文章配 **两张图**:

- **题图 cover.png** — 进入文章前的封面,**严格 2.35:1**(900×383, 即 900÷383=2.349),进 WeChat 编辑器封面字段。强字体、强构图、文字主导
- **解释图 illustration.png** — 正文里的配图,**比例由内容决定**(模型自选),帮读者一眼看懂文章核心结构。扁平卡通,有标签和流程

**先问用户题图怎么处理**:

> 题图想怎么处理？
> A) 我有图片，我提供路径
> B) 用文字封面（标题 + 简洁背景，HTML/CSS 渲染，免费、即时）
> C) **AI 生成（GPT Image，按词义出概念图，需 OPENAI_API_KEY，每张约 $0.05–0.20）**
> D) 跳过，待会儿手动处理

**如果选 A**：把用户提供的图片复制到文章目录，重命名为 `cover.png`/`cover.jpg`。

**如果选 B**：

```bash
~/.claude/skills/jianshuo-wechat-mp-publish/render-cover.sh "标题文字" "副标题或日期" /path/to/output/cover.png
```

输出 900×383 PNG，2.35:1 微信主封面比例。

**如果选 C**：

```bash
~/.claude/skills/jianshuo-wechat-mp-publish/gen-cover-ai.sh <article-folder> ["目标字词"]
```

- 不传第二个参数时，从 `meta.json` 取 `title` 当目标字词
- 内部调用 `gpt-image-2-skill`（自动选 provider：Codex `~/.codex/auth.json` 或 `OPENAI_API_KEY`）
- 默认尺寸 `1536x1024`（最接近 2.35:1 的 landscape），自动 sips 居中裁到 900×383
- 原图保存为 `cover-raw.png`，裁剪后是 `cover.png`
- `cover-prompt.md` 作为 `--instructions`（设计哲学），短生成指令作为 `--prompt`——这样 gpt-5.4 能消化长 prompt 后再调 image_generation 工具
- 可调环境变量：`WECHAT_PUBLISH_IMAGE_SIZE`（默认 `1536x1024`）、`WECHAT_PUBLISH_IMAGE_QUALITY`（默认 `high`）

**前置依赖**：必须装好 `gpt-image-2-skill`：

```bash
git clone https://github.com/Wangnov/gpt-image-2-skill /tmp/g
cp -r /tmp/g/skills/gpt-image-2-skill ~/.claude/skills/
```

并且至少有以下一种鉴权：
- **推荐**：Codex `~/.codex/auth.json`（ChatGPT Plus 计划即可，**不需要 OpenAI 组织验证**，gpt-image-2 的中文字渲染明显比 gpt-image-1 准确）
- 或 `OPENAI_API_KEY`（需要在 platform 验证组织才能用 gpt-image-2）

**鉴权预检**（在跑 `gen-cover-ai.sh` 之前必做，避免烧 API 又拿不到结果）：

```bash
[ -f ~/.codex/auth.json ] && echo "✓ Codex auth (recommended, free on ChatGPT Plus)" \
  || [ -n "$OPENAI_API_KEY" ] && echo "✓ OPENAI_API_KEY (needs org verification for gpt-image-2)" \
  || echo "✗ NO AUTH — set up one of: ~/.codex/auth.json (recommended) OR export OPENAI_API_KEY=..."
```

如果两个都没有，**不要跑** `gen-cover-ai.sh`（它会失败但消息可能模糊）。直接告诉用户：
- 推荐路径：跑 `codex login` 走 ChatGPT Plus 配额（免费、不用组织验证）
- 备选路径：去 platform.openai.com 拿 API key 并完成组织验证（gpt-image-2 必需）
- 临时路径：选 B 文字封面（`render-cover.sh`），无需任何 API

**目标字词**的选择：文章标题往往是长短语（如「AI 能力的三个简单层次」），但 prompt 模板对单字 / 两字词更友好。可以建议用户挑核心概念字词：

> 目标字词用什么？默认是文章标题。建议挑一个核心概念字词（1–4 字），比如「AI 能力的三个简单层次」可以用「三层」或「层次」。

**然后生成解释图**(无需问用户,自动跑):

```bash
~/.claude/skills/jianshuo-wechat-mp-publish/gen-illustration.sh <article-folder>
```

- 读 `article.md` 全文,作为 instructions 传给 gpt-image-2
- 模型理解文章核心结构后,生成扁平卡通解释图
- **不裁剪**,模型自选画幅(双行对照通常出 3:2,流程类用横长条,层级深度用竖版)
- 输出 `illustration.png`,直接用作正文配图

如果用户对某张图不满意,直接重跑对应脚本——每次结果不同。

### Step 5: 输出文件包

在用户的工作目录下（默认 `~/wechat-publish/articles/`）创建文件夹：

```
articles/2026-05-09-{slug}/
├── article.md           # 润色后的 markdown 源文件
├── article.html         # 转成 HTML，直接粘贴用
├── cover.png            # 题图 900×383 (2.35:1 严格)
├── illustration.png     # 解释图（任意比例，模型自选）
├── meta.json            # { title, summary, author, date, slug }
└── original.md          # 用户原始输入，备份
```

`{slug}` 从标题生成：拼音首字母 + 关键词，限制 30 字符以内。例如"我的第一台 Mac" → `my-first-mac`。

**article.html 转换规则：**
- 用 `pandoc` 或简单的 markdown 解析（不需要复杂样式，公众号编辑器会重新排版）
- 保留段落分隔（`<p>`）
- 保留加粗（`<strong>`）和列表
- 不要内联 CSS——公众号会清掉

```bash
pandoc article.md -f markdown -t html -o article.html
# 如果没有 pandoc:
# 用 Python 的 markdown 包 / Node 的 marked / 或手写最简实现
```

### Step 6: 发布（Tier 1 自动化）

文章包准备好后，**告诉用户运行**：

```bash
~/.claude/skills/jianshuo-wechat-mp-publish/publish.sh <workspace>/articles/YYYY-MM-DD-{slug}
```

或如果用户已经 `cd` 进文章目录：

```bash
~/.claude/skills/jianshuo-wechat-mp-publish/publish.sh
```

`publish.sh` 自动做的事：

1. 打开浏览器到 https://mp.weixin.qq.com/
2. 在 Finder 中显示 cover.png（拖进编辑器封面区）
3. 把 article.html 在浏览器另开一个标签页（备用，rich-text 复制源）
4. 把正文 HTML 以 **rich text 格式**放到剪贴板（Cmd+V 直接出排版，不是源码）
5. 终端显示交互菜单，按 1/2/3/4 在剪贴板里切换：
   - `1` 标题
   - `2` 作者
   - `3` 摘要
   - `4` 正文 HTML（重新放回剪贴板）
   - `q` 退出

**典型用户流程（用了 publish.sh 之后约 2 分钟）：**
1. 终端跑 publish.sh
2. 切到浏览器 → 扫码登录公众号
3. 点"新的创作 → 图文消息"
4. 编辑器：正文区 Cmd+V（默认剪贴板就是 rich-text 正文）
5. 切回终端按 `1` → 切到浏览器 → 标题字段 Cmd+V
6. 终端按 `2` → 浏览器作者字段 Cmd+V
7. 终端按 `3` → 浏览器摘要字段 Cmd+V
8. Finder 里把 cover.png 拖到封面区
9. 手机预览，发布

**如果剪贴板正文粘出来排版乱了**：浏览器切到第二个 article.html 标签页 → Cmd+A → Cmd+C → 粘进编辑器（这是 rich-text 复制最可靠的来源）。

输出给用户的最后一段话，固定格式：

```
准备好了。文章在 articles/2026-05-09-{slug}/

发布：
  ~/.claude/skills/jianshuo-wechat-mp-publish/publish.sh articles/2026-05-09-{slug}

它会打开浏览器、显示题图、把正文推到剪贴板。在终端按数字键切换标题/作者/摘要。

article.md 是源文件，下次改用这个。
```

## File Layout (skill 自身)

```
~/.claude/skills/jianshuo-wechat-mp-publish/
├── SKILL.md                       # 本文件
├── cover-template.html            # 文字题图模板（HTML+CSS）
├── render-cover.sh                # 渲染文字题图（免费、即时）
├── cover-prompt.md                # AI 题图 prompt 模板（[目标字词] 占位符）
├── gen-cover-ai.sh                # 题图: 2.35:1 强约束, 自动裁到 900×383
├── illustration-prompt.md         # AI 解释图 prompt 模板（[文章内容] 占位符）
├── gen-illustration.sh            # 解释图: 比例自适应, 不裁剪
└── publish.sh                     # Tier 1 发布助手（开浏览器 + 剪贴板）
```

依赖的外部 skill：
- `gpt-image-2-skill`（github.com/Wangnov/gpt-image-2-skill）—— gen-cover-ai.sh 走这里调 gpt-image-2

## Polish Heuristics (具体到字)

错字模式 → 改：
- "的得地" 误用：根据语法判断
- 重复字："我我"、"是是"、"了了" → 删一个
- 同音字：考虑上下文（"在"vs"再"，"做"vs"作"）

段落切分时机：
- 一句话讲完一个意思，下一句换主语 → 分段
- 出现"但是"、"不过"、"所以"、"后来"在句首 → 考虑分段
- 一段超过 80 字 → 找最近的句号分

不要分段：
- 排比句、列举（保持节奏）
- 对话（按对话格式）

## Anti-Patterns (绝对不做)

| 不要 | 原因 |
|------|------|
| 把"我觉得"改成"笔者认为" | 改变了作者身份 |
| 加"小标题"打断行文 | 微信读者不需要导航 |
| 把口语句尾"吧/呢/啊"删掉 | 删掉就不是这个人写的了 |
| 在结尾加"欢迎关注"、"点赞在看" | 作者会自己决定要不要 |
| 把"今天"改成具体日期 | 作者用"今天"是有意为之 |
| 自己加举例 / 引用 / 数据 | 这是写作，不是补全 |
| 改动后没给 diff，直接全文输出 | 用户看不见你改了什么 |

## Showing the Diff

每次润色完，**先告诉用户你改了什么**，再问要不要继续：

```
我改了 7 处：
1. L3: "我我觉得" → "我觉得"（重复字）
2. L8: 长段落 (120字) 拆成两段
3. L15: "通过…的方式" → "用…"（口语化保留）
...

要看完整结果吗？
```

如果改动 ≤ 3 处，可以直接给完整结果，不用列 diff。

## Running the Skill (实操步骤)

1. 确认工作目录（默认 `~/wechat-publish/`，可由用户配置）
2. 接收用户输入（粘贴或文件）
3. 写 `original.md`（用户原始输入）
4. 写 `article.md`（润色版）→ 列 diff 给用户
5. 用 AskUserQuestion 问标题候选
6. 用 AskUserQuestion 问题图选择
7. 渲染题图（如选 B）
8. 生成 `article.html`、`meta.json`
9. 输出发布指引

## Done When

- [ ] `articles/YYYY-MM-DD-{slug}/` 文件夹存在
- [ ] 包含 article.md、article.html、cover.png、meta.json、original.md
- [ ] meta.json 字段齐全
- [ ] 用户拿到了发布指引
- [ ] 用户没说"再改改"
