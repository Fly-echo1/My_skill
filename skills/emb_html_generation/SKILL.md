---
name: emb_html_generation
description: 嵌入式项目文档生成器。自动扫描嵌入式项目源码，分析MCU型号、驱动模块、外设配置，一键生成简洁有技术感的HTML文档。依赖全局安装的 mkdocs 和 mkdocs-material。
trigger: 用户要求为嵌入式项目生成文档、创建技术文档、制作HTML报告、给项目写文档、建立项目文档站
args:
  project-dir: 项目目录路径，默认当前目录或上次使用的目录
  output-dir: 输出目录，默认 site/
---

# 嵌入式项目文档生成器

## 工作流程

### 1. 确定项目路径

- 如果用户指定了 `project-dir` 参数，使用该路径
- 否则从当前对话上下文中推断项目路径
- 最后确认用户确认路径

### 2. 扫描项目结构

```bash
# 探索项目结构
find <project-dir> -maxdepth 3 -type f \( -name "*.c" -o -name "*.h" -o -name "*.syscfg" -o -name "*.ioc" -o -name "*.cfg" \) 2>/dev/null | sort
```

分析以下信息：
- **MCU 型号**：从 `.syscfg`、`_hal.c`、`main.c` 头文件或启动文件中提取
- **驱动模块**：列出 Drivers/ 下的子目录
- **传感器/外设**：从驱动目录名或源码包含的头文件推断
- **IDE/工具链**：检查 `.cproject`、`.project`、`.uvprojx`、`.ioc` 等
- **SDK 版本**：从 SDK 路径或版本头文件推断

### 3. 分析核心源码

读取关键文件：
- `main.c` / `main.h` — 主逻辑和功能模块
- 各驱动模块的头文件 — 了解 API 和功能
- 项目配置文件 — 了解硬件配置

### 4. 生成 Markdown 文档

在 `<project-dir>/docs/` 下生成以下文件：

#### `docs/index.md` — 项目概览
- 项目名称和简介
- MCU 型号、IDE、SDK 信息
- 硬件架构图（ASCII）
- 核心功能列表（表格）
- 硬件组件清单（表格）
- 快速开始指南

#### `docs/architecture.md` — 软件架构
- 目录结构树
- 主循环流程图（ASCII）
- 中断系统表格
- 模块依赖关系

#### `docs/drivers.md` — 驱动模块详解
- 每个驱动模块的功能描述
- 关键 API 说明
- 硬件接口（I2C/SPI/UART/GPIO）

#### `docs/control.md` — 控制算法
- PID 参数表格
- 控制流程图（如果项目使用了 PID 控制）

#### `docs/pinout.md` — 引脚映射
- 引脚分配表格（如果有 `.syscfg` 或 `.ioc` 文件）
- 外设功能说明

#### `docs/debug.md` — 调试与使用
- UART 输出格式
- OLED 显示布局
- 调试技巧

### 5. 创建 MkDocs 配置

在项目目录下创建 `mkdocs.yml`：

```yaml
site_name: <项目名>
site_description: <描述>

use_directory_urls: false

theme:
  name: material
  palette:
    - scheme: slate
      primary: indigo
      accent: cyan
      toggle:
        icon: material/weather-sunny
        name: Light mode
    - scheme: default
      primary: indigo
      accent: cyan
      toggle:
        icon: material/weather-night
        name: Dark mode
  features:
    - navigation.tracking
    - navigation.indexes
    - navigation.top
    - content.code.copy
    - search.highlight
    - search.share

markdown_extensions:
  - pymdownx.highlight:
      anchor_linenums: true
      pygments_lang_class: true
  - pymdownx.inlinehilite
  - pymdownx.snippets
  - pymdownx.superfences
  - pymdownx.tabbed:
      alternate_style: true
  - admonition
  - pymdownx.details
  - tables
  - toc:
      permalink: true

nav:
  - 项目概览: index.md
  - 软件架构: architecture.md
  - 驱动模块: drivers.md
  - 控制算法: control.md
  - 引脚映射: pinout.md
  - 调试与使用: debug.md
```

### 6. 构建 HTML

```bash
cd <project-dir>
mkdocs build --clean
```

### 7. 打开结果

```bash
start "" "<project-dir>/site/index.html"
```

### 8. 汇报总结

告诉用户：
- 生成了多少页面
- 输出目录在哪
- 已自动打开浏览器

## 风格指南

- 表格使用 `|` 管道符格式
- ASCII 流程图风格：圆角框用 `┌───┐`，箭头用 `─►`
- 代码块标注语言类型
- 所有文档用中文撰写
- 色调偏暗色技术风，表格整齐
- 标题用 `#` 层级，三级标题封顶

## 依赖检查

如果 mkdocs 或 mkdocs-material 未安装，自动安装：

```bash
pip install mkdocs-material
```
