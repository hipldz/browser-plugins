# VS Code LLM 翻译 · LLM Translator

[返回项目总览](../README.md)

基于大语言模型的 VS Code 文本处理扩展，支持中英互译、文本润色和自定义 Prompt。可在编辑器中处理选区，也可通过侧边栏手动输入。

## 功能

- **编辑器选区翻译**：右键菜单或 `Alt+T` 执行当前模式。
- **Hover 结果**：高亮选区并自动弹出仅包含处理结果的悬浮窗口，便于复制。
- **替换文本**：启用后直接替换编辑器选中内容。
- **侧边栏输入**：配置 API、管理模式、输入文本并查看结果。
- **自定义 Prompt**：内置「中英互译」「中英润色」，支持新增、编辑、删除自定义模式。
- **密钥存储**：API Key 使用 VS Code SecretStorage，其他配置保存至扩展全局状态。

## 预览

![VS Code LLM Translator 侧边栏与翻译结果](doc/preview.png)

## 安装与开发

要求 VS Code 1.85.0 及以上。源码使用 TypeScript，开发和打包需要 Node.js 与 npm。

### 从源码调试

在仓库根目录打开终端：

```bash
cd vsce-llm-translation
npm ci
npm run compile
code .
```

在打开的子项目窗口中按 `F5`，使用已有的「Run Extension」配置启动扩展开发宿主；该配置会调用默认编译监视任务。在开发宿主中点击 Activity Bar 的「LLM Translator」图标进入配置面板。

### 打包和安装 VSIX

在子项目目录中执行：

```bash
npm run publish
```

此脚本实际执行本地 `vsce package`，会先编译并在当前目录生成 `.vsix`，不会发布到 Marketplace。

在 VS Code 扩展视图的菜单中选择「从 VSIX 安装」，或执行：

```bash
code --install-extension ./llm-translator-1.0.9.vsix
```

文件名随 `package.json` 中的版本号变化，请以实际输出为准。

## API 配置

点击 Activity Bar 的「LLM Translator」图标打开侧边栏。配置修改后会在字段失去焦点或选择变化时自动保存。

| 字段 | 说明 |
| --- | --- |
| 终端节点 | 完整的 Chat Completions 请求地址，不自动补全路径；支持 `{{modelName}}` 占位符 |
| API Key | 接口密钥，使用 VS Code SecretStorage 保存 |
| 模型名称 | 按服务商要求填写；非空时加入请求体的 `model` 字段，并替换地址中的模型占位符 |
| 翻译模式 | 必选，选择内置或自定义模式 |
| 替换文本 | 控制编辑器选区输出方式，不影响侧边栏手动输入 |

地址示例：

```text
https://your-api.example/v1/chat/completions
```

模型名位于路径中的地址示例：

```text
https://your-api.example/openai/deployments/{{modelName}}/chat/completions
```

接口需接受 POST JSON，`messages` 中包含 `user` 消息及文本内容数组，并返回 `choices[0].message.content`。请求同时携带 `Authorization: Bearer <API Key>` 和 `api-key` 请求头。实际地址、模型名及所需查询参数以服务商提供的接口为准。

## 使用

### 编辑器选区

1. 选中需要处理的文本。
2. 按 `Alt+T`，或右键选择「翻译选中的文本 (LLM Translation)」。
3. 状态栏显示处理进度，完成后在选区位置自动打开 Hover 结果。
4. 如果已勾选「替换文本」，则直接将结果写入编辑器选区。

默认展示模式会在 60 秒后清除结果与高亮，切换文件也会清除。替换模式下，请在等待结果时保持编辑器选区不变。

### 侧边栏输入

在侧边栏输入框键入或粘贴文本，点击「执行」，结果显示在面板下方。此方式使用当前模式和 API 配置，不会替换编辑器选区。

### 快捷键与命令

| 默认快捷键 | 命令 | 功能 |
| --- | --- | --- |
| `Alt+T` | `llm-translator.translate` | 编辑器获得文本焦点时，翻译选中文本 |
| `Ctrl+Shift+Alt+L` | `llm-translator.openPanel` | 打开翻译侧边栏 |

可在 VS Code「键盘快捷方式」中按命令名称搜索并修改绑定，也可从命令面板执行。

## Prompt 模式

| 内置模式 | 用途 |
| --- | --- |
| 中英互译 | 自动识别中文或英文，翻译为另一种语言 |
| 中英润色 | 按原文语言润色为专业、严谨的正式文档风格 |

在「新增模式」中填写模式名称和提示词，使用 `{{text}}` 作为原文占位符。省略占位符时，程序会在提示词末尾追加原文。

```text
请将以下内容翻译为简洁、准确的中文技术文档，保留术语和数字，仅输出译文：

{{text}}
```

点击「新增」后会自动选择新模式。取消「锁定提示词」可编辑当前模式，修改自动保存。内置模式不可删除，自定义模式可通过提示词区域的删除按钮移除。

## 数据与使用范围

- 执行翻译会将选区或手动输入发送到配置的 API 服务。
- API Key 由 VS Code SecretStorage 管理，终端节点、模型、Prompt 和输出偏好保存在 `globalState` 中。
- 配置与浏览器版、Windows 桌面版各自独立，不会自动同步。
- 编辑器选区命令需要活动文本编辑器和非空选区；侧边栏可用于独立输入。

## 源码结构

| 文件 | 职责 |
| --- | --- |
| `src/extension.ts` | 命令注册、选区翻译、Hover 和原文替换 |
| `src/translatorPanel.ts` | 侧边栏 Webview 与消息处理 |
| `src/translationService.ts` | Prompt 拼接与 API 请求 |
| `src/storageService.ts` | 普通配置和密钥存储 |
| `src/defaultPrompts.ts` | 内置模式与类型定义 |
| `media/panel.js` | 侧边栏交互 |
| `package.json` | 扩展清单、快捷键与构建脚本 |

开发检查命令为 `npm run compile` 和 `npm run lint`。

## 许可证

[MIT License](LICENSE)
