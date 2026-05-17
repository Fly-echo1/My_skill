# 嵌入式开发模式

> 嵌入式项目全流程开发模式：需求 → 设计 → 计划 → 子 Agent 驱动实现 + 微信实时汇报。

## ⚠️ 版权声明

本 Skill 是一个**工作流编排器**，它描述如何调用外部 Skill 完成嵌入式开发全流程。

- 本文件夹内的 `SKILL.md` 和 `README.md` 为原创作品
- `embedded-dev` **不包含** Superpowers 系列 Skill 的任何代码或内容
- 运行时需要依赖以下**外部 Skill**（版权归原作者所有，需从市场自行安装）：
  - `brainstorming`
  - `writing-plans`
  - `subagent-driven-development`
  - `verification-before-completion`
  - `dispatching-parallel-agents`

## 设计背景

从零开始一个嵌入式项目涉及多个阶段：需求分析、方案设计、代码实现、测试验证。
本 Skill 串联 Superpowers 工作流（brainstorming → writing-plans → subagent-driven-development），
加上微信实时推送，形成一条完整的嵌入式开发流水线。

## 核心功能

| 功能 | 说明 |
|:---|:---|
| 🧠 **需求分析** | 引导用户描述项目上下文（MCU、语言、约束） |
| 📐 **方案设计** | 调用 brainstorming 产出设计文档 |
| 📋 **实施计划** | 调用 writing-plans 分解为可执行任务 |
| 🤖 **子 Agent 开发** | 派发独立子 Agent 按任务实施，审查后再继续 |
| 📱 **微信通知** | 子 Agent 提问、审查不通过、任务完成时推送到手机 |

## 工作流程

```
用户输入"开始嵌入式开发"
  → ① 询问是否需要微信推送
  → ② 确认项目上下文（MCU、平台、语言、约束）
  → ③ 启动 brainstorming → 产出设计文档
  → ④ 启动 writing-plans → 产出实施计划
  → ⑤ 启动 subagent-driven-development → 逐任务实施
       ├─ 派发实施子 Agent
       ├─ 规范审查 → 通过/打回
       ├─ 质量审查 → 通过/打回
       └─ 进入下一个任务
  → ⑥ 全部完成 → 最终代码审查 + 收尾
```

## 关键技术细节

### 微信推送规则

| 事件 | 推送内容 |
|:---|:---|
| 子 Agent 提问 | 💬 [UART驱动] 波特率可配置还是固定？ |
| 规范审查不通过 | ❌ [UART驱动] 缺少错误处理 |
| 质量审查不通过 | ⚠ [环形缓冲区] 魔法数字问题 |
| 任务完成 | ✅ [UART驱动] 完成，进入下一项 |
| 全部完成 | 🎉 全部 X 个任务完成！ |

### 汇报格式

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 [任务 1/5] UART 驱动
   ├─ ▶ 派发实施子 Agent...
   ├─ ✅ 实施完成
   ├─ ▶ 派发规范审查子 Agent...
   ├─ ✅ 规范审查通过
   └─ 🔄 进入下一个任务...
━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 上下文确认模板

启动时会向用户确认：
- **项目/任务描述**：做什么
- **目标 MCU/平台**：STM32、ESP32、Linux 等
- **开发语言**：C、C++、Rust
- **约束条件**：功耗、实时性、内存限制

## 依赖 Skill（需提前安装）

| Skill | 用途 |
|:---|:---|
| `brainstorming` | 需求分析与设计 |
| `writing-plans` | 制定实施计划 |
| `subagent-driven-development` | 逐任务实施 |
| `verification-before-completion` | 完成前验证 |
| `dispatching-parallel-agents` | 并行任务调度 |

以上 Skill 可在 Cherry Studio 市场中搜索安装。

## 使用示例

```
你：开始嵌入式开发

我：📱 需要微信推送进度到手机吗？
你：需要

（扫码绑定微信）

我：确认项目上下文：
    - 项目：STM32F407 平衡车
    - MCU：STM32F407VGT6
    - 语言：C
    - 约束：实时性要求高，RAM 192KB
    对吗？
你：对

我：🚀 启动 brainstorming...
    📐 设计文档完成
    📋 实施计划完成
    🤖 开始子 Agent 开发...
    📱 微信：🔄 [任务1/3] 电机驱动
    ...
```

## 注意事项

- 子 Agent **串行执行**（防止写冲突）
- 子 Agent 提问时必须等待用户回复
- 所有代码改动前确认不在 main/master 分支
