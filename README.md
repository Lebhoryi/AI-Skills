# AI-Skills

个人 Agent Skills 集合。这里收纳了一组面向 Claude Code、Cursor、OpenClaw 等 Agent 环境的技能：从 AI 早报、内容制图、概念学习，到代码诊断、TDD、PRD/Issue 拆解和技能开发。

## 快速使用

按你的 Agent 客户端约定，把需要的 skill 目录复制到对应的 skills 目录即可。

```bash
# Claude Code 示例
mkdir -p ~/.claude/skills
cp -R ~/AI-Skills/<skill-name> ~/.claude/skills/

# Cursor 示例
mkdir -p ~/.cursor/skills
cp -R ~/AI-Skills/<skill-name> ~/.cursor/skills/
```

如果某个 skill 自带依赖或资源，请优先阅读该目录下的 `SKILL.md` 或 `README.md`。

## 重点 Skill

### `ai-morning-brief`

生成面向中文读者的 AI 晨报，覆盖国际 AI 动态和中文 AI 圈，可直接输出公众号、知乎、Newsletter 或朋友圈版本。

注：`ai-morning-brief` 已兼容 [Horizon](https://github.com/Thysrael/Horizon)。如果本机已安装并配置 Horizon，可在生成早报时合并 Horizon 过去 24 小时的高分条目，复用其多源抓取、去重、AI 评分和背景补充能力；未安装 Horizon 时仍可独立生成早报。

### `ljg-card`

内容制图 skill，把文本、URL 或文件内容铸造成 PNG 视觉资产。支持长图、信息图、多卡、小红书大字卡、视觉笔记、漫画和白板等模具，默认输出到 `~/Downloads/`。

### `ljg-plain` / `ljg-learn` / `ljg-think`

一组认知写作 skill：

- `ljg-plain`：把复杂内容改写成聪明的 12 岁孩子也能懂的白话。
- `ljg-learn`：从八个维度解剖概念，并生成 org-mode 笔记。
- `ljg-think`：沿着一个问题一路纵向追问，钻到不可再分的本质。

## Skill 索引

### 内容与知识

| Skill | 用途 |
| --- | --- |
| `ai-morning-brief` | 生成中文 AI 早报，兼容 Horizon 数据源合并 |
| `ljg-card` | 把内容生成 PNG 卡片、信息图、漫画、白板等视觉资产 |
| `ljg-plain` | 白话解释或改写复杂内容 |
| `ljg-learn` | 八维概念解剖，输出 org-mode 笔记 |
| `ljg-think` | 对观点或问题做纵向深挖 |
| `docx` | 创建、读取、编辑和分析 `.docx` Word 文档 |
| `mem-search` | 搜索跨会话记忆，找回过去解决过的问题 |

### 工程与代码

| Skill | 用途 |
| --- | --- |
| `diagnose` | 按“复现 -> 最小化 -> 假设 -> 插桩 -> 修复 -> 回归测试”诊断 bug |
| `tdd` | 用红绿重构循环做测试驱动开发 |
| `requesting-code-review` | 完成重要任务后发起代码审查 |
| `improve-codebase-architecture` | 找代码架构中的深模块机会和可测试性改进点 |
| `zoom-out` | 要求 Agent 升高抽象层，解释代码区域的模块地图和调用关系 |
| `brainstorming` | 在写代码前澄清意图、需求和设计方案 |

### 产品、Issue 与计划

| Skill | 用途 |
| --- | --- |
| `to-prd` | 把当前上下文整理成 PRD 并发布到项目 issue tracker |
| `to-issues` | 把计划或 PRD 拆成可独立领取的垂直切片 issue |
| `setup-matt-pocock-skills` | 为仓库初始化 issue tracker、triage labels 和 domain docs 配置 |
| `grill-me` | 对计划或设计进行连续追问，逼近共同理解 |
| `grill-with-docs` | 基于项目领域文档和 ADR 压测方案，并在决策形成时更新文档 |

### Agent 能力与工作流

| Skill | 用途 |
| --- | --- |
| `skill-creator` | 创建、改进、评测和打包 Agent Skill |
| `find-skills` | 从开放技能生态中搜索并安装合适的 skill |
| `caveman` | 超压缩沟通模式，减少废话和 token 消耗 |
| `pua` | 高压执行/闭环风格 skill，用于反复失败、质量不足或需要强力推进的场景 |

## 目录约定

每个顶层目录通常代表一个独立 skill：

```text
skill-name/
  SKILL.md      # 必需：skill 元信息与执行说明
  README.md     # 可选：面向人的介绍文档
  assets/       # 可选：模板、图片、脚本依赖等资源
  references/   # 可选：按需加载的参考资料
  scripts/      # 可选：可复用脚本
```

`ljg-card/node_modules/` 中的 Playwright skill 属于依赖包内部内容，不作为本仓库的主要 skill 维护入口。

## 维护建议

- 新增 skill 时，至少提供 `SKILL.md`，并在本 README 的索引中补一行。
- 如果 skill 有独立安装步骤、外部依赖或输出路径，放到该 skill 自己的 `README.md`。
- 面向 Agent 的触发条件写在 `SKILL.md` frontmatter 的 `description` 中；面向人的介绍写在 README 中。
