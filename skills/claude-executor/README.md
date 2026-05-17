# Claude 执行器

> 在 Cherry Studio 中调用终端 Claude Code 并实时监视会话。

## 设计背景

Claude Code CLI 在大型项目代码生成上能力很强，但 Cherry Studio 无法直接与之交互。
本 Skill 搭建了一座桥梁：**Cherry Studio 做指挥中心，Claude Code 在终端干活，我在旁边实时观察汇报。**

## 核心功能

| 功能 | 说明 |
|:---|:---|
| 🔐 **目录安全锁** | 每次调用必须先确认工作目录，防止误操作 |
| 🪟 **启动终端** | 自动打开 PowerShell 窗口，cd 到指定目录启动 Claude Code |
| 👀 **实时监视** | Monitor 工具后台跟踪 `.claude-session.log`，grep 筛选关键事件 |
| 🌏 **中文翻译** | 英文报错实时翻译为中文（覆盖 10+ 常见错误类型） |
| 📱 **微信推送** | 报错、Claude 提问、任务完成时推送通知到手机 |
| 💬 **随时查问** | 用户问"怎么样了"，读取日志用中文总结当前进度 |
| 📋 **干完总结** | 停止 Monitor 后输出改动文件清单和耗时 |

## 工作流程

```
用户输入需求
  → ① 问目录（安全锁）
  → ② 校验目录存在 + 备份旧日志
  → ③ PowerShell 启动新窗口，运行 claude + Tee-Object 写日志
  → ④ Monitor 持久跟踪 tail -f 日志
  → ⑤ 关键事件 → 中文翻译 → 微信推送
  → ⑥ 用户说"搞定了" → 停止 Monitor → 输出总结
```

## 关键技术细节

### 终端启动

```powershell
Start-Process powershell -WindowStyle Normal @'
  cd "E:\路径"
  claude 2>&1 | Tee-Object -FilePath "E:\路径\.claude-session.log" -Encoding utf8
'@
```

- `Start-Process` 不阻塞 Skill 流程
- `Tee-Object` 同时输出到终端窗口和日志文件
- `.claude-session.log` 放在项目目录内，每次启动自动备份旧日志

### 日志监视

```bash
tail -f "路径/.claude-session.log" \
  | grep --line-buffered -E "(?i)(error|fail|undefined|...)"
```

- `--line-buffered` 确保每行日志立即输出，不攒批
- Monitor 超时 30 分钟自动结束

### 事件推送规则

| 级别 | 触发词 | 推送 |
|:---|:---|---:|
| 🚨 报错 | error/fail/undefined/cannot/denied | 📱 微信 |
| 💬 提问 | 行尾为 `?` | 📱 微信 |
| ✅ 完成 | complete/done/success | 📱 微信 |
| 📝 文件变动 | write/create/modify/delete | 🖥️ 聊天 |

### 环境依赖

- **Claude Code CLI**：已安装并在 PATH 中
- **Shell**：Git Bash（路径格式 `/e/xxx`）
- **PowerShell**：Windows 自带，用于启动新终端窗口
- **微信通道**：已配置（用于推送通知）

## 使用示例

```
你：帮我启动 Claude，在 E:\TI\project\new\car.102 干活
我：🔔 在哪个目录下执行？
你：E:\TI\project\new\car.102
我：✅ Claude Code 已启动，有情况微信告诉你
    🟢 监视器已启动

（你在终端里与 Claude Code 交互...）

📱 微信：🚨 [Claude 执行器] 编译错误：变量 `TIM_Init` 未声明

你（切回 Cherry Studio）：他现在怎么样了？
我：Claude Code 刚才编译时报错了，缺少 TIM_Init 的定义，
    他正在检查是否有包含相关头文件。

（你干完了）

你：搞定了
我：📋 本次改动总结：
    ├─ 修改：main.c（添加 TIM 初始化，第42-58行）
    ├─ 编译：⚠️ 有 1 个警告（未使用变量）
    └─ 耗时：约 15 分钟
```

## 注意事项

- 一次只能监视一个 Claude Code 会话
- 支持**透传模式**：说"透传"即可原话传给 Claude Code，不做优化
- 如果不想要监视功能，可以说"不用看了"提前结束
