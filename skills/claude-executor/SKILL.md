---
name: Claude 执行器
description: >
  调用本机 Claude Code CLI（终端版）在指定项目中执行代码任务，并实时监视会话日志。
  当用户说「启动 Claude」「帮我用 claude」「调 claude 干活」「打开 claude 终端」「让 claude 做」等涉及调用 Claude Code 的请求时触发。
  也适用于用户需要在本地项目中执行复杂代码生成、重构、调试等需要多轮交互的场景。
  注意：这是一个终端启动器 + 监视器，不是直接替代 Claude Code 本身。
---

# Claude 执行器

## 技能概述

本 Skill 的核心能力是在 Cherry Studio 中**启动一个 Claude Code 交互式终端窗口**，并**实时监视其会话日志**，在关键事件（报错、完成、提问）时以**中文翻译后通过微信推送**给用户。

**本质定位：** 启动器 + 观察员（副驾），不是替代 Claude Code 本身。Claude Code 的交互在独立终端窗口中进行，本 Skill 在 Cherry Studio 侧提供实时辅助。

---

## 工作流程

### 步骤 1：确认工作目录（安全锁）

**不管用户是否已经说了目录，这一步必须执行。**

```
Agent："🔔 在哪个目录下执行？"

用户回复目录路径，例如：E:\TI\project\new\car.102
```

**校验逻辑：**
1. 将 Windows 路径转为 Git Bash 路径（`E:\...` → `/e/...`）
2. 执行 `ls "/e/..."` 检查目录是否存在
3. 不存在则提示："❌ 目录不存在，请重新输入"
4. 存在则继续

### 步骤 2：检查是否已有活跃监视器

```
检查是否有 Monitor 正在运行（通过会话上下文判断）。

如果有 → 询问用户：
  "当前已有一个监视器在运行（上次启动的 Claude Code），是否覆盖？"
  是 → 先 stop 旧的 Monitor，再继续
  否 → 中止流程
```

### 步骤 3：启动 Claude Code 终端窗口

**备份旧日志：**
```bash
# 若 .claude-session.log 存在，重命名为 .claude-session.log.bak
mv "/e/路径/.claude-session.log" "/e/路径/.claude-session.log.bak" 2>/dev/null; true
```

**启动新 PowerShell 窗口：**
```powershell
Start-Process powershell -WindowStyle Normal @'
  cd "E:\路径"
  claude 2>&1 | Tee-Object -FilePath "E:\path\.claude-session.log" -Encoding utf8
'@
```

通过 Bash 执行上述 PowerShell 命令。

**等待确认：**
- 等待 2 秒
- 执行 `ls "/e/路径/.claude-session.log"` 确认日志文件已创建
- 确认失败则提示用户手动检查

**通知用户：**
```
✅ Claude Code 已启动，终端窗口已打开
🟢 监视器即将启动，有情况微信告诉你
```

### 步骤 4：启动 Monitor（日志监视器）

**Monitor 命令：**
```bash
tail -f "目标目录/.claude-session.log" | grep --line-buffered -E "(?i)(error|fail|undefined|cannot|does not|denied|✗|❌|(write|写入|修改|创建|删除|add|create|delete|modify)|\?\s*$|complete|done|finish|success|✅|✔)"
```

**参数：**
- 超时：30 分钟（可配置）
- persistent: false（超时后自动结束）

### 步骤 5：事件处理（Monitor 每输出一行执行一次）

当 Monitor 输出一行日志时，按以下规则处理：

**① 匹配事件类型**

逐行判断日志内容属于哪一类：

| 级别 | 匹配条件 | 处理方式 |
|:---|:---|:---|
| 🚨 报错 | 包含 error/fail/undefined/cannot/does not/denied/❌✗（忽略大小写） | 翻译后微信推送 |
| 💬 提问 | 行尾为问号（`\?\s*$`） | 微信推送 |
| ✅ 完成 | 包含 complete/done/finish/success/✅/✔（忽略大小写） | 微信推送 |
| 📝 文件变动 | 包含 write/写入/修改/创建/删除/add/create/delete/modify | 存入上下文，用户问时回答 |
| 🔄 其他 | 上述都不匹配 | 忽略，不推送不存储 |

**② 报错中文翻译**

抓取原始报错行，翻译为中文：

| 常见原文模式 | 中文翻译 |
|:---|:---|
| `'X' undeclared` | 编译错误：`X` 未声明 |
| `expected 'X' before 'Y'` | 语法错误：在 `Y` 前缺少 `X` |
| `undefined reference to 'X'` | 链接错误：找不到 `X` 的实现 |
| `unused variable 'X'` | 警告：变量 `X` 未使用 |
| `implicit declaration of function 'X'` | 警告：函数 `X` 隐式声明（缺头文件） |
| `X: No such file or directory` | 找不到文件或目录 `X` |
| `permission denied` | 权限不足 |
| `command not found` | 命令未找到 |
| `segmentation fault` | 程序崩溃：内存访问越界 |
| 其他 error/fail | 翻译核心语义，附原始行 |

**简化原则：** 速度比精度重要。抓到大类关键词就翻译大类，不追求逐字翻译。

**③ 微信推送格式**

```
🚨 [Claude 执行器] 编译错误：变量 `TIM_Init` 未声明
    └─ 原始：error: 'TIM_Init' undeclared (first use in this function)
```

```
💬 [Claude 执行器] Claude 在问你："是否要创建这个文件？"
```

```
✅ [Claude 执行器] 任务完成：编译通过
```

```
📝 [Claude 执行器] 已修改文件：main.c（第 42-58 行）
```

### 步骤 6：用户查询处理

当用户在 Cherry Studio 中问"怎么样了""他在干什么""进度如何"等：

1. 读取日志文件最后 30 行：`tail -30 "路径/.claude-session.log"`
2. 用中文总结 Claude Code 当前在做什么
3. 如果有最近的报错，重点指出

### 步骤 7：结束流程

当用户说"搞定了""干完了""结束""停"等：

```
1. 停止 Monitor（通过 TaskStop 工具）
2. 读取完整会话日志
3. 输出改动总结：
   ├─ 修改了哪些文件
   ├─ 主要改动内容
   ├─ 编译是否通过
   └─ 本次会话耗时
4. 可选：询问用户是否保留日志文件
```

---

## 模式说明

### 默认模式：优化模式
- 用户输入需求后，Agent 帮忙整理成清晰、结构化的 prompt
- 补充上下文、明确任务目标，让 Claude Code 拿到 prompt 就能直接干
- 优化后的 prompt 显示给用户确认后传入

### 透传模式
- 当用户说"透传""原话传""直接传""不要优化"时
- 用户的原始输入原样传给 Claude Code 作为开场 prompt
- 适用于用户已经写好精确 prompt 的场景

---

## 安全与边界

1. **目录安全锁：** 每次调用**必须**先问用户目录，确认后才执行
2. **目录校验：** 启动前检查目录是否存在，不存在则拒绝
3. **进程隔离：** Claude Code 在新窗口中独立运行，不干扰 Cherry Studio
4. **防重复：** 启动前检查是否已有活跃监视器
5. **日志安全：** 日志只写入项目目录下的 `.claude-session.log`
6. **一次一个会话：** 不支持同时监视多个 Claude Code 实例
7. **超时保护：** Monitor 最多持续 30 分钟

## 注意事项

- Shell 环境是 **Git Bash**（路径格式：`/e/xxx` 而非 `E:\xxx`）
- Claude Code 路径：`C:\Users\FLYpig\.local\bin\claude.exe`（已在 PATH 中）
- PowerShell 命令通过 Bash 的 `powershell.exe -Command` 执行
- 不要同时启动多个 Monitor，会话上下文中记录 Monitor 状态
