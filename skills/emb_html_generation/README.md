# 嵌入式项目文档生成器

> 自动扫描嵌入式项目源码，一键生成简洁有技术感的 HTML 文档站。

## 设计背景

嵌入式项目通常缺乏良好的文档。手动写文档费时费力，且容易过时。
本 Skill 通过分析项目源码结构，自动提取 MCU 型号、驱动模块、外设配置等信息，
生成一套可直接部署的 MkDocs HTML 文档站，暗色技术风格。

## 核心功能

| 功能 | 说明 |
|:---|:---|
| 🔍 **自动扫描** | 分析 `.c`、`.h`、`.syscfg`、`.ioc` 等文件 |
| 📦 **模块提取** | 识别 MCU 型号、驱动模块、传感器、外设 |
| 📄 **多页文档** | 生成概览、架构、驱动、引脚、调试等页面 |
| 🎨 **暗色主题** | MkDocs Material 主题，暗色技术风 |
| 🌐 **一键部署** | 自动 `mkdocs build` 并打开浏览器 |
| 📊 **表格输出** | 引脚映射、API 说明、功能清单全是整齐表格 |

## 工作流程

```
用户输入项目路径
  → ① 扫描项目结构（find *.c *.h *.syscfg ...）
  → ② 分析 MCU 型号、驱动模块、外设
  → ③ 读取 main.c / 驱动头文件核心内容
  → ④ 生成 Markdown 文档（index/architecture/drivers/pinout/debug）
  → ⑤ 创建 mkdocs.yml 配置文件
  → ⑥ mkdocs build → 打开浏览器预览
```

## 关键技术细节

### 项目扫描

```bash
find <项目路径> -maxdepth 3 -type f \( -name "*.c" -o -name "*.h" \
  -o -name "*.syscfg" -o -name "*.ioc" -o -name "*.cfg" \) | sort
```

从扫描结果中提取：
- **MCU 型号**：`.syscfg` 配置、启动文件、`main.c` 头文件
- **驱动模块**：`Drivers/` 目录结构
- **外设列表**：驱动目录名或源码中 `#include` 的头文件
- **工具链**：`.cproject`、`.uvprojx`、`.ioc` 等

### 文档结构

| 文件 | 内容 |
|:---|:---|
| `docs/index.md` | 项目概览、硬件架构 ASCII 图、功能表格、快速开始 |
| `docs/architecture.md` | 目录结构、主循环流程图、中断系统、模块依赖 |
| `docs/drivers.md` | 各驱动模块功能、关键 API、硬件接口 |
| `docs/control.md` | PID 参数表格、控制流程图（如果有 PID） |
| `docs/pinout.md` | 引脚分配表（如果有 `.syscfg`/`.ioc`） |
| `docs/debug.md` | UART 输出格式、OLED 布局、调试技巧 |

### 主题配置

MkDocs Material 暗色主题，支持明暗切换，深色靛蓝主色调。

### 依赖

- Python
- `mkdocs` + `mkdocs-material`（未安装时自动 pip install）

## 使用示例

```
你：帮 E:\TI\project\new\car.102 生成文档

我：扫描项目中... 找到 28 个源文件
    识别到 MCU：MSPM0G3507
    识别到驱动：GPIO、UART、I2C、TIM、ADC、WWDG
    正在生成文档...
    ✅ 文档已生成！
    📄 6 个页面已创建
    📂 输出目录：E:\TI\project\new\car.102\site\
    🌐 已自动打开浏览器
```

## 注意事项

- 项目需要网络安装 mkdocs-material（如未安装）
- 扫描深度为 3 层目录，更深层的文件需手动指定
- 文档语言为中文
