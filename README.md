# My Skill

> 个人为 **Cherry Studio** 创作的 Agent Skill 合集。

---

## ⚠️ 版权声明

本仓库仅包含**我个人原创创作**的 Agent Skill。

- 每个 Skill 文件夹内的 `SKILL.md` 和 `README.md` 均为我的原创作品
- 本仓库**不包含** Cherry Studio 的任何预装系统提示词、内置 Agent 配置
- 本仓库**不包含**来自市场 / 第三方作者的 Skill 代码或内容

如果您在自己的项目中使用或参考了本仓库的内容，请保留原始出处。

---

## Skill 列表

| Skill | 说明 | 状态 |
|:---|:---|---:|
| [Claude 执行器](./skills/claude-executor/) | 启动终端 Claude Code + 实时日志监视 + 微信推送 | ✅ |
| [嵌入式文档生成器](./skills/emb_html_generation/) | 自动扫描嵌入式项目源码生成 HTML 文档 | ✅ |
| [嵌入式开发模式](./skills/embedded-dev/) | 嵌入式项目全流程编排（Superpowers 工作流 + 微信汇报） | ✅ |

## 使用方式

1. 将 Skill 文件夹放入 Cherry Studio 的 Skills 目录
2. 在 Cherry Studio 中启用该 Skill
3. 根据各 Skill 的 README 说明进行调用

## 外部依赖声明

以下 Skill 在运行时需要调用其他作者的作品，特此声明：

| Skill | 外部依赖 | 版权归属 |
|:---|:---|:---|
| `emb_html_generation` | MkDocs Material 主题（`mkdocs-material`） | MIT License，[squidfunk](https://github.com/squidfunk) |
| `embedded-dev` | Superpowers 系列 Skill（brainstorming、writing-plans、subagent-driven-development 等） | 原作者所有，需从市场自行安装 |

本仓库不提供上述依赖的任何代码或主题文件，仅描述如何调用它们。

## License

MIT © Fly-echo1
