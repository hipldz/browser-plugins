# Browser Plugins 与 LLM 翻译工具

个人浏览器扩展与文本处理工具合集，包含媒体播放监控、浏览器代理，以及浏览器、Windows 桌面和 VS Code 三种使用场景下的 LLM 翻译工具。

各子项目独立使用。浏览器扩展可直接加载源码；桌面工具和 VS Code 扩展的运行方式见各自 README。

## 项目导航

| 项目 | 适用环境 | 主要功能 | 文档 |
| --- | --- | --- | --- |
| `bilibili-monitor` | Chrome / Edge、Firefox | 手动开启标签页监控，尝试恢复暂停的音视频，可配置遮罩选择器 | [媒体播放监控](bilibili-monitor/README.md) |
| `http-proxy` | Chrome / Edge、Firefox | 配置 HTTP 代理、切换代理开关、设置绕过地址 | [浏览器代理](http-proxy/README.md) |
| `llm-translation` | Chrome / Edge、Firefox | 右键或快捷键处理网页选区，支持悬浮结果、原文替换与手动输入 | [浏览器 LLM 翻译](llm-translation/README.md) |
| `llm-translator-desktop` | Windows 10 / 11 | 全局快捷键处理应用选区，支持托盘、悬浮结果和原位写回 | [Windows 桌面翻译](llm-translator-desktop/README.md) |
| `vsce-llm-translation` | VS Code 1.85.0 及以上 | 编辑器选区翻译、Hover 结果、原文替换与侧边栏输入 | [VS Code LLM 翻译](vsce-llm-translation/README.md) |

## 浏览器扩展快速安装

下载或克隆仓库后，根据浏览器和项目选择目录：

| 项目 | Chrome / Edge 加载目录 | Firefox 临时加载的文件 |
| --- | --- | --- |
| 媒体播放监控 | `bilibili-monitor/chrome/` | `bilibili-monitor/firefox/manifest.json` |
| 浏览器代理 | `http-proxy/chrome/` | `http-proxy/firefox/manifest.json` |
| LLM 翻译 | `llm-translation/chrome/` | `llm-translation/firefox/manifest.json` |

### Chrome / Edge

1. 打开 `chrome://extensions/` 或 `edge://extensions/`。
2. 开启「开发者模式」，点击「加载已解压的扩展程序」。
3. 选择表格中对应的 `chrome/` 目录，该目录下应直接包含 `manifest.json`。
4. 从工具栏打开扩展，按子项目文档完成设置。

### Firefox

1. 打开 `about:debugging#/runtime/this-firefox`。
2. 点击「临时载入附加组件」，选择表格中对应的 `manifest.json`。
3. 临时加载的扩展会在 Firefox 重启后移除；此方式适合源码调试。

浏览器内部页面、扩展商店等受限页面无法注入脚本，媒体监控和网页选区翻译应在普通网页中使用。

## 选择翻译工具

三种翻译工具均需要自行配置 LLM API。它们各自保存设置和 Prompt，不会自动同步。

| 配置 | 浏览器版 | Windows 桌面版 | VS Code 版 |
| --- | --- | --- | --- |
| API 地址 | 完整 Chat Completions 请求地址 | Base URL，自动补全 `/chat/completions`，也接受完整路径 | 完整 Chat Completions 请求地址 |
| Prompt 原文插入方式 | `{{text}}`；省略时追加原文 | Prompt 作为 `system` 消息，原文单独作为 `user` 消息 | `{{text}}`；省略时追加原文 |
| API Key 存储 | 扩展本地存储 | Windows 凭据管理器 | VS Code SecretStorage |
| 默认选区快捷键 | `Alt+T` | `Ctrl+Shift+X` | `Alt+T` |

执行翻译会将文本发送至配置的 API 服务。具体接口格式、输出方式和使用限制见各子项目文档。

## 仓库结构

```text
browser-plugins/
├── bilibili-monitor/        # 媒体播放监控扩展
├── http-proxy/              # 浏览器代理扩展
├── llm-translation/         # 浏览器 LLM 翻译扩展
├── llm-translator-desktop/  # Windows Python 桌面工具
├── vsce-llm-translation/    # TypeScript VS Code 扩展
├── LICENSE
└── README.md
```

## 许可证

仓库采用 [MIT License](LICENSE)。VS Code 子项目也附有独立的 [MIT 许可证文件](vsce-llm-translation/LICENSE)。
