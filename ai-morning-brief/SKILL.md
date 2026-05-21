# AI 早报 Skill

生成面向中文读者的 AI 晨报：国际 + 中文 AI 圈双轨聚合，输出可直接发布到公众号、知乎或朋友圈。

工作流参考 [Horizon](https://github.com/Thysrael/Horizon)（AI 新闻雷达）：多源抓取、去重、AI 评分过滤、背景补充、结构化成稿。

## When to use this skill

- 每天一条命令，自动汇总 AI 领域重要动态，无需手动刷 10+ 来源
- 需要公众号 / 知乎 / Newsletter 可直接粘贴的简体中文早报
- 关注 DeepSeek、Qwen、Kimi、GLM 等中文 AI 圈，同时覆盖 Anthropic、OpenAI、Google 等国际动态
- 已安装 Horizon 时，可与 Horizon 日报合并，减少漏报

## Workflow

按顺序执行；任一步失败则跳过该步，不阻塞整体成稿。

### Step 0 — HelloGitHub 月刊（最优先）

在抓取新闻**之前**，检查 [HelloGitHub 月刊](https://hellogithub.com/periodical) 是否有新一期。

**检测：**
1. GitHub Releases API：`https://api.github.com/repos/521xueweihan/HelloGitHub/releases/latest`
2. 备用：`https://hellogithub.com/periodical/volume/{N}`

**判定为「有更新」：** `published_at` 在最近 14 天内，或发布月份为当前自然月。

**输出：** 若有更新，在标题下方、今日一句话**之前**插入：

```
> 🔔 **HelloGitHub 月刊更新**：第 {N} 期（{YYYY-MM-DD}）— [阅读月刊](https://hellogithub.com/periodical/volume/{N})
```

并列出 2～3 个与 AI / 开发工具相关的推荐项目。无新刊则跳过本块。

---

### Step 1 — 多源抓取（24 小时内）

并行抓取，只保留过去 **24 小时** 的内容（与 Horizon `time_window_hours: 24` 一致）。

#### 国际 — Lab / 聚合（RSS 或 API）

| 来源 | URL | 类型 |
|------|-----|------|
| Anthropic News | `https://raw.githubusercontent.com/Olshansk/rss-feeds/main/feeds/feed_anthropic_news.xml` | RSS |
| OpenAI News | `https://openai.com/news/`（RSS 失败则抓首页） | RSS / 页面 |
| Google Developers Blog | `https://developers.googleblog.com/feeds/posts/default` | RSS |
| HuggingFace Blog | `https://huggingface.co/blog/feed.xml` | RSS |
| Meta AI Blog | `https://ai.meta.com/blog/` | 页面 / RSS 备用 |
| Claude Code Changelog | `https://code.claude.com/docs/en/changelog/rss.xml` | RSS |
| The Decoder | `https://the-decoder.com/feed/` | RSS |
| TLDR AI | `https://tldr.tech/api/latest/ai` | API |
| Hacker News | `https://hacker-news.firebaseio.com/v0/topstories.json`（取 Top 30，标题含 AI/LLM/agent/model 等关键词） | API |

#### 中文 — 媒体（首页/RSS）

| 来源 | URL |
|------|-----|
| 机器之心 | `https://www.jiqizhixin.com/` |
| 量子位 | `https://www.qbitai.com/` |
| 极客公园 | `https://www.geekpark.net/` |
| 36氪 | `https://www.36kr.com/` |
| 新智元 | `https://www.ainewera.com/` |
| 雷锋网 AI | `https://www.leiphone.com/category/ai` |
| 虎嗅 | `https://www.huxiu.com/` |

#### 模型厂商 / GitHub

| 来源 | URL |
|------|-----|
| DeepSeek | `https://github.com/deepseek-ai`（releases / 近期动态） |
| Qwen | `https://github.com/QwenLM` |
| GLM/智谱 | `https://github.com/THUDM` |
| GitHub 趋势（AI） | OSS Insight API：`https://api.ossinsight.io/q/trends/repos/?period=past_24_hours&language=Python`（可再加 TypeScript / Rust / Jupyter Notebook）；或 `https://github.com/trending` 作备用 |

抓取失败则跳过，不中断流程；不要为了补齐栏目虚构标题、数字或链接。

---

### Step 2 — 去重（Horizon: Deduplicate）

合并指向**同一事件**的条目：
- 相同或规范化后的 URL（忽略 `utm_*` 等参数）
- 标题高度相似（同一产品发布、同一论文）
- 跨平台重复（HN + RSS + 中文媒体同一新闻）

保留信息最全的一条（优先含链接、评论数、官方来源），其余丢弃。

---

### Step 3 — AI 评分与过滤（Horizon: Score & Filter）

对每条候选按 **0–10** 打分（可参考 Horizon [scoring.md](https://github.com/Thysrael/Horizon/blob/main/docs/scoring.md)）：

| 分数 | 档位 | 映射到早报 |
|------|------|----------|
| 9–10 | 突破性 | 🔴 重大 |
| 7–8 | 高价值 | 🟡 重要 |
| 5–6 | 值得关注 | ⚪ 备选（仅在当日新闻稀少、且确有信息量时收录 1–2 条） |
| ≤4 | 噪声 | 丢弃 |

**评分维度：** 技术深度与新颖性、行业影响、表述质量、社区讨论质量（HN/Reddit 评论）、互动信号（点赞/评论需配合实质内容）。

**过滤阈值：**
- 正文条目：**≥ 7.0** 才进入「国际动态 / 中文圈」
- **≥ 9.0** 额外进入「🔥 今日精选」
- 已进入「🔥 今日精选」的事件，后续「国际动态 / 中文圈」不再重复出现
- 全局控制在：国际 3–6 条、中文 3–5 条、精选 1–3 条

---

### Step 4 — 背景补充（Horizon: Enrich，轻量版）

对 **🔴 今日精选** 中涉及陌生概念、公司、论文、项目的条目，各补 **1 句** 背景说明（不展开成长文）：
- 这是什么 / 为何重要
- 读者无领域背景也能读懂标题

若条目来自 Horizon，优先复用其 `detailed_summary` / 背景字段；其余条目在 1–2 句摘要中自带必要上下文即可，不必逐条 enrich。

---

### Step 5 — 可选：合并 Horizon 日报

仅在用户明确要求「合并 Horizon」、或本轮上下文已提供 Horizon 摘要时执行。若本机已 clone [Horizon](https://github.com/Thysrael/Horizon)（默认 `~/Horizon` 或用户指定路径）：

1. 确认已配置 `data/config.json` 且含 AI 相关源（RSS、HN、GitHub 等，见 [config.example.json](https://github.com/Thysrael/Horizon/blob/main/data/config.example.json)）
2. 执行：`uv run horizon --hours 24`（需在 Horizon 目录、已 `uv sync`）
3. 读取最新 `data/summaries/` 下中文或英文 Markdown
4. 将 Horizon 中 **AI 相关且未与 Step 1–3 重复** 的高分条目并入候选池，重新按 Step 3–4 筛选后写入早报

未安装 Horizon、未配置 `.env` / `data/config.json`、或运行失败：跳过该步，仅用 Step 1 手动抓取结果；若用户明确要求合并 Horizon，需要在最终回复里说明跳过原因。

Horizon 能力参考：飞书/邮件/Webhook/MCP 发布见 [Horizon README](https://github.com/Thysrael/Horizon/blob/main/README_zh.md)。

---

### Step 6 — 生成早报（固定结构）

```
# 🤖 AI 早报 | {TODAY_DATE}（{WEEKDAY}）

{Step 0 的 HelloGitHub 提醒块（若有）}

> 📌 **今日一句话**：{用一句话概括今日最重要的一条 AI 进展}

---

## 🔥 今日精选

{仅 score≥9 / 🔴 条目，1–3 条；格式：🔴 **[来源]** [标题] — [1–2 句，含背景补充]（附链接更佳）}

---

## 🌍 国际动态

{3–6 条，排除今日精选已出现事件；格式：🔴/🟡/⚪ **[来源]** [事件] — [1–2 句说明]}

---

## 🇨🇳 中文圈

{3–5 条，排除今日精选已出现事件；无重大新闻时写：今日暂无中文圈重大动态}

---

## ⭐ GitHub 今日趋势（AI）

{Top 3，格式：[`owner/repo`](https://github.com/owner/repo) — [一句话] ⭐{stars 或 stars_gained}；必须给出 GitHub 仓库链接}

---

## 💡 今日洞察

{2–3 句：综合今日新闻的含义；具体、可判断，避免空泛套话}

---

**朋友圈版 ↓**（可直接复制）

🤖 AI早报 {SHORT_DATE}

{3–4 条，每条 ≤25 字，口语化}

👉 完整早报见评论区
```

---

### Step 7 — 质量检查

- HelloGitHub 提醒（若有）必须在标题后、今日一句话前
- 日期均为近 24 小时；过期丢弃
- 已去重，无同一事件重复出现
- 不虚构来源、链接、star 数、百分比或发布日期；不确定则降级为「未见可靠来源」
- GitHub 今日趋势必须使用 Markdown 链接：[`owner/repo`](https://github.com/owner/repo)；拿不到仓库 URL 的条目不收录
- 中文圈至少 1 条，否则注明「今日暂无中文圈重大动态」
- 朋友圈版总字数 ≤ 120 字
- 专有名词保留英文（Claude、GPT、Gemini 等），正文简体中文

---

### Step 8 — 保存与发布到 AI-News-weekly

质量检查通过后，必须把最终 Markdown 正文保存到：

`/home/lebhoryi/Project/AI-News-weekly/2026/AI简报{YYYYMMDD}.md`

规则：
- `{YYYYMMDD}` 使用当天日期，例如 `AI简报20260521.md`
- 若当天文件已存在，覆盖当天文件；不要改动其它历史简报
- 保存后进入 `/home/lebhoryi/Project/AI-News-weekly/2026`
- 只 `git add` 本次生成的 Markdown 文件，不要 `git add .`
- 若 `git status --short` 显示没有变化，则说明内容未变，跳过 commit 和 push
- 若有变化，按仓库既有风格提交并推送：

```bash
cd /home/lebhoryi/Project/AI-News-weekly/2026
git add "AI简报{YYYYMMDD}.md"
git commit -m "add {MMDD}"
git push
```

其中 `{MMDD}` 例如 `0521`。如果 `git push` 因认证或网络失败，保留本地 commit，并在最终回复中说明失败原因。

## Output format

Markdown，可直接粘贴到微信公众号编辑器、知乎或 Notion；同时保存到 AI-News-weekly 仓库并执行 add / commit / push。

## Notes

- 写作语言：简体中文；公司名、模型名保留原文
- 用具体事实，避免「某厂有新动态」式空话
- 中文实验室重大发布（DeepSeek、Qwen 等）优先进入「今日精选」或中文圈靠前位置
- 可与 `china-hot-mcp` 联动，叠加微博/知乎 AI 话题热度
- 进阶：用 [Horizon Wizard](https://github.com/Thysrael/Horizon)（`uv run horizon-wizard`）生成个性化 `config.json`，再定时跑 `uv run horizon` 作第二数据源

## References

- [Horizon](https://github.com/Thysrael/Horizon) — AI 新闻雷达（中英双语日报、评分、去重、enrich、MCP）
- [Horizon 配置说明](https://github.com/Thysrael/Horizon/blob/main/docs/configuration.md)
- [Horizon 评分机制](https://github.com/Thysrael/Horizon/blob/main/docs/scoring.md)
- [HelloGitHub 月刊](https://hellogithub.com/periodical)
