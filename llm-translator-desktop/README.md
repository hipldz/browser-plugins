# Windows 桌面 LLM 翻译 · LLM Translator

[返回项目总览](../README.md)

Windows 10 / 11 常驻系统托盘的文本处理工具。通过 `Ctrl+Shift+X` 读取当前应用选区，发送到配置的 Chat Completions API，按当前 Prompt 处理后原位替换或悬浮显示。也可在应用内手动输入文本。

项目使用 Python 和 PyQt5，可直接运行源码，也可在 Windows 上打包为单文件 EXE。

## 功能

- **全局选区处理**：默认快捷键 `Ctrl+Shift+X`，支持修改；读取选区前等待组合键松开，复制失败时进行一次内部重试。
- **两种输出方式**：替换原文，或显示可拖动、复制、关闭的置顶悬浮卡片。
- **独立翻译页面**：输入、执行和查看结果均在应用内完成，不读取或修改外部应用选区。
- **Prompt 管理**：内置「中英互译」「学术润色」，支持新增、复制、编辑、删除和切换模式。
- **概览与历史**：查看本次运行中最近 60 条记录，历史详情与翻译页输入相互独立；退出后清空。
- **系统托盘**：查看状态、切换 Prompt、输出方式和主题，暂停或恢复快捷键监听，接收完成通知。
- **浅色与深色主题**：从窗口右上角、设置页或托盘面板切换，并保存选择。
- **凭据存储**：API Key 保存在 Windows 凭据管理器，普通设置单独保存为 JSON。

## 安装与启动

### 环境要求

- Windows 10 / 11 交互式桌面环境。
- 从源码运行需要 Python 3.10 及以上；运行打包后的 EXE 无需安装 Python。
- 可访问的 Chat Completions API，以及对应的 API Key 和模型名称。

### 从源码运行

下载或克隆仓库，在仓库根目录打开 PowerShell 后执行：

```powershell
cd llm-translator-desktop
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe app.py
```

如果终端已经位于 `llm-translator-desktop/`，无需再次执行 `cd`。依赖包括 PyQt5、UI Automation、keyring 和 Windows 支持库。

首次启动后：

1. 打开「设置」，填写 API Key、Base URL 和 Model，点击「保存设置」。
2. 在「概览」选择 Prompt 和输出方式。
3. 在 Edge、Firefox、Word 等应用中选中文字，按 `Ctrl+Shift+X`。

隐藏到托盘启动：

```powershell
.\.venv\Scripts\pythonw.exe app.py --minimized
```

关闭主窗口默认隐藏到托盘；右键托盘图标选择「退出」才会结束程序。也可在设置中开启启动时隐藏到托盘。需要开机启动时，可将指向 `pythonw.exe`、参数为完整 `app.py` 路径及 `--minimized` 的快捷方式放入 Windows 启动文件夹（`Win+R` → `shell:startup`）。

## API 配置

| 配置 | 说明 |
| --- | --- |
| Base URL | API 根地址，例如 `https://your-api.example/v1`；自动附加 `/chat/completions`，已包含完整路径时不重复追加 |
| Model | 服务商支持的 Chat Completions 模型 ID，必填 |
| API Key | 在设置中填写并保存到 Windows 凭据管理器；留空表示保持已保存密钥 |
| 超时 | 默认 30 秒，允许 1–120 秒 |
| 原文长度上限 | 默认 10,000 字符，允许 1–100,000 字符 |
| 写回实现 | `paste` 通过 Ctrl+V 写回，`type` 通过 Unicode 输入事件写回 |

远程接口要求 HTTPS；`localhost`、`127.0.0.1`、`::1` 允许 HTTP。Base URL 不接受用户名、密码、查询参数或 fragment。请求使用 Bearer 鉴权、非流式 Chat Completions JSON，接口需返回 `choices[0].message.content`。

### Prompt 模式

在「Prompt」页编辑并保存模式，使用「设为当前模式」，或从概览和托盘面板切换。

| 内置模式 | 用途 |
| --- | --- |
| 中英互译 | 将中文或英文翻译为另一种语言，保留含义、语气和风格 |
| 学术润色 | 按原文语言润色为专业、严谨、流畅的正式文档风格 |

Prompt 原样作为 `system` 消息，原文单独作为 `user` 消息。无需写 `{{text}}` 或 `{text}`，也不会替换 `{target_language}`；处理风格和语言方向均由 Prompt 决定。

示例：

```text
请将原文润色为自然、严谨的学术英语。
保留实验细节、物种名称、单位、数值、引文和事实，不添加新结论。
仅输出修改后的文本。
```

任务开始时会固定本次 Prompt 和输出方式，请求过程中切换模式只影响下一次任务。正在处理时再次触发快捷键会跳过新请求，不会将旧选区排队。

## 使用

### 选区输出

| 模式 | 行为 | 适用场景 |
| --- | --- | --- |
| 替换原文（默认） | 检查目标窗口、焦点和可获取的选区状态，再尝试写回结果 | 输入框、Word 等可编辑控件 |
| 悬浮显示 | 保留原文，弹出置顶结果卡片，可拖动标题栏、复制或关闭 | 阅读网页、查词、查看润色候选 |

可在概览、设置或托盘面板中切换。悬浮卡片记住本次运行中拖动后的位置；悬浮模式允许在处理期间切换窗口，替换模式则应保持原窗口、焦点和选区不变。

### 应用内翻译

进入「翻译」页，输入文本并点击「执行」。结果只显示在应用内，并加入概览历史；即使当前输出方式为替换原文，也不会向外部应用发送粘贴指令。查看历史不会覆盖翻译页尚未处理的输入。

### 托盘状态

| 颜色 | 状态 |
| --- | --- |
| 蓝色 | 就绪，监听已开启 |
| 橙色 | 正在读取选区、调用 API 或写回 |
| 绿色 | 最近一次处理成功 |
| 红色 | 最近一次处理失败 |
| 灰色 | 已暂停快捷键监听 |

右键托盘图标可切换 Prompt、输出和主题，暂停或恢复监听、打开窗口、退出程序。完成或失败通知可在设置中关闭，其显示也受 Windows 通知设置和勿扰模式影响。

## 配置与数据存储

| 数据 | 位置或行为 |
| --- | --- |
| API Key | 当前 Windows 用户的凭据管理器，服务名 `SelTranslate.ChatCompletions` |
| 普通配置 | `%LOCALAPPDATA%\SelTranslate\settings.json`，包含 API 地址、模型、Prompt、快捷键、输出、主题等 |
| 临时密钥覆盖 | 环境变量 `SELECT_TRANSLATOR_API_KEY`，请求时优先于已保存密钥 |
| 会话历史 | 仅保存在内存中，退出后清空；`app.py` 不主动将会话文本写盘 |
| 配置示例 | [settings.example.json](settings.example.json)，不含密钥，实际配置由界面生成 |

内部配置目录和凭据服务沿用 `SelTranslate`，更新程序不会因显示名称变化而丢失已有设置或密钥。`settings.json` 不允许包含明文 `api_key`。

当前配置版本为 `2`。读取旧版本配置时，会将 Prompt 列表重置为两项预设，不迁移旧版四个内置模式或旧自定义模式；其他普通设置仍可沿用。升级前请备份需要保留的自定义 Prompt。旧 `config.json` 不再自动迁移。

## 常见问题与限制

- **提示快捷键被占用**：退出仍在运行的旧进程，或在设置中修改快捷键。
- **无法读取网页选区**：程序优先使用 UI Automation，失败后内部模拟 Ctrl+C；可根据界面提示区分复制未更新、剪贴板不可读或焦点改变。
- **无法替换或替换被中止**：请求期间保持窗口、焦点和选区不变；部分自定义编辑器不兼容，普通权限程序也可能无法向管理员权限窗口写回。可切换悬浮输出。
- **连续处理同一输入框**：每次等待任务结束后，再次执行 `Ctrl+A` → `Ctrl+Shift+X`，两次成功处理之间无需额外等待。
- **剪贴板内容变化**：内部复制和粘贴会暂用剪贴板，并尽力恢复原纯文本，无法保证恢复图片、HTML 等格式。
- **文本隐私**：执行任务会发送文本至配置的 API 服务。旧命令行入口 `main.py` 仍可能写本地调试日志；日常使用应从 `app.py` 启动。

## 打包为单文件 EXE

在 Windows 上进入本项目目录，使用同一虚拟环境执行：

```powershell
.\.venv\Scripts\python.exe -m pip install pyinstaller pillow
.\.venv\Scripts\python.exe build_exe.py
```

构建结果为 `dist\LLM Translator.exe`，目标电脑无需安装 Python。脚本会打包 `assets/` 资源并从品牌 SVG 生成 ICO，个人密钥和设置不打入 EXE。

PyInstaller 无法在 Linux 上交叉构建该 Windows EXE。单文件程序首次启动需要解包资源，可能比直接运行源码稍慢。

## 源码结构

| 文件 | 职责 |
| --- | --- |
| `app.py` | PyQt5 主界面、托盘、主题和会话历史 |
| `desktop.py` | Windows 全局热键线程与后台任务 |
| `main.py` | UI Automation、剪贴板与原位写回底层逻辑，兼有旧命令行入口 |
| `translator.py` | API 参数校验和 Chat Completions 请求 |
| `modes.py` / `hotkeys.py` | Prompt 模式与快捷键解析校验 |
| `settings_store.py` / `app_paths.py` | 普通配置读写、版本处理与用户目录 |
| `secret_store.py` | Windows 凭据管理器访问 |
| `floating_result.py` | 悬浮结果卡片 |
| `vector_icons.py` / `assets/` | 矢量图标与托盘状态图标 |
| `windows_titlebar.py` | 原生标题栏主题适配 |
| `build_exe.py` | Windows 单文件 EXE 打包 |
| `tests/test_core.py` | 配置、Prompt、请求构造等检查 |

真实选区读取、剪贴板和写回行为需要在 Windows 交互式桌面中验证。

## 许可证

[MIT License](../LICENSE)
