# [Fatebook](https://fatebook.io)

[English](./README.md) | [中文](./README.zh-CN.md)

制作和追踪预测的最快方式。

此仓库包含 [fatebook.io](https://fatebook.io)、[Fatebook for Slack](https://fatebook.io/for-slack) 和 [Fatebook for Chrome](https://fatebook.io/extension)。

你可以在 [Issues](https://github.com/Sage-Future/fatebook/issues) 中报告 bug，或在我们的 [Discord](https://discord.gg/mt9YVB8VDE) 中提出功能请求。

## 技术栈

- Prisma
- Next.js
- Vercel
- Slack API
- TRPC
- Tailwind

## 用户制作的集成

使用 [Fatebook API](https://fatebook.io/api-setup)，Fatebook 用户已创建以下集成：

- [一个 Obsidian 插件，可在编辑器中直接创建和预览 Fatebook 问题](https://github.com/GarretteBaker/obsidian-fatebook)
- [一个 iOS 快捷指令，用于创建 Fatebook 问题](https://www.icloud.com/shortcuts/25903acfcd3d4fd5bed31c2f50322928)
- [一个 Alfred 工作流，用于创建 Fatebook 问题](https://github.com/Calebp98/alfred-fatebook-workflow)
- [一个 Emacs 插件，用于创建 Fatebook 问题](https://github.com/sonofhypnos/fatebook.el)

在 [Discord](https://discord.gg/mt9YVB8VDE)分享你自己的 API 创作！

## 贡献指南

如果你对贡献感兴趣，[告诉我们](https://github.com/Sage-Future/fatebook/issues)，我们可以帮助你快速上手——感谢你的贡献！

以下说明假设你使用 macOS 和 Homebrew。

### 克隆仓库

```shell
git clone https://github.com/Sage-Future/fatebook.git
cd fatebook
cp .env.example .env
```

### 为本地主机创建 TLS 证书

```shell
brew install mkcert
mkcert -install
mkcert localhost
```

### 设置数据库

```shell
brew install postgresql@15
# 或者从 https://www.postgresql.org/download 下载并安装
# 如果你这样做，请按照其中关于启动数据库的说明操作

# 按照安装程序提供的说明将 postgresql@15 添加到你的 PATH
# 例如，在某些系统上它会告诉你这样做：
echo 'export PATH="/opt/homebrew/opt/postgresql@15/bin:$PATH"' >> ~/.zshrc
. ~/.zshrc

# 如果你使用 Bash 而不是 Z shell，请将上述命令中的 `~/.zshrc` 替换为 `~/.bash_profile`

# 启动 postgres
LC_ALL="C" /opt/homebrew/opt/postgresql@15/bin/postgres -D /opt/homebrew/var/postgresql@15
```

打开一个新的终端窗口并运行以下命令：

```shell
# 创建 `fatebook` 角色
createuser fatebook

# 创建 `fatebook_development` 数据库，由 `fatebook` 用户拥有
createdb fatebook_development --owner fatebook
```

1. 运行 `npm install` 安装依赖。
2. 运行 `npm run vercel-build` 设置 Fatebook 数据库模式。

### 启动服务器

1. 运行 `npm run dev`。
2. 访问 https://localhost:3000。你应该能够登录并创建预测。

### 运行测试

使用 `npm run test` 运行 Jest 测试。

你还可以通过运行 `npm run test -- --coverage` 生成覆盖率报告，报告将打印到控制台并保存到 `coverage/` 目录。查看它的最佳方式是在 `coverage/lcov-report/index.html` 中的 HTML 格式（运行 `open coverage/lcov-report/index.html`）。

### 在 VSCode 调试器中运行

有一个用于 VSCode/Cursor 的启动配置叫做 "Next.js: debug fatebook full stack"，可以在调试器中运行 Web 应用（前端和后端，断点可以在客户端或 SSR 期间工作）。

### 开发 Fatebook 浏览器扩展

Fatebook 有 Chrome 和 Firefox [扩展](https://fatebook.io/extension)。源代码在 [/chrome-extension](https://github.com/Sage-Future/fatebook/tree/main/chrome-extension) 中

测试 Chrome 扩展的更改：

1. 按照上述一般 Fatebook 开发设置步骤操作
2. 在 `after.js` 中，确保 `extensionInfo.isDev = true`
3. 运行 `npm run dev`，并检查你的开发服务器是否在 https://localhost:3000 运行
4. 转到 `chrome://extensions`
5. 如果你已安装生产版 Fatebook 扩展，请将其禁用
6. 选择 "Load unpacked"，然后选择 chrome-extension 目录
7. 在 `chrome://extensions/shortcuts` 中，将 "Make a new prediction" 的快捷方式设置为 Cmd-Shift-F（如果尚未设置）
8. 提交之前，确保 `after.js` 中的 `extensionInfo.isDev = false`

如果你更改了 `chrome-extension` 中文件的内容，要在你的浏览器中看到更改，你需要重新加载扩展：

1. 转到 `chrome://extensions`
2. 点击 Fatebook for Chrome 卸载版本旁边的刷新按钮
3. 在要测试扩展的任何标签页中刷新

对 Fatebook 本身的更改（例如对 `/pages/embed/q/[id].tsx` 的更改）将自动反映，无需重新加载扩展。

<details>
  <summary>发布 Chrome 扩展更新</summary>

贡献者——维护者将测试并部署你的更改。

维护者说明：

1. 在 Chrome 和 Firefox 中本地测试更改
2. 在 `after.js` 中设置 `extensionInfo.isDev = false`
3. 部署对 fatebook.io 的更改，并使用生产环境测试你的本地卸载扩展
4. 在 `manifest.json` 中增加 `version` 字段
5. 运行 `npm run zip`
6. 将 Chrome 扩展上传到 Web Store，将 Firefox 版本上传到 Mozilla 附加组件。通常审批需要 <24 小时

</details>

### 附录

如果你想对 Slack 集成进行更改或测试，你需要设置一个 Slack 应用。这比较复杂——[联系我们](https://github.com/Sage-Future/fatebook/issues)，我们会帮助你完成设置。

<details>
  <summary>可选附加步骤：或者设置你自己的 Google OAuth 凭据</summary>

你的 `.env.example` 已预填充了共享的 Fatebook 开发人员 Google OAuth 凭据。如果你想要创建自己的凭据（例如要更改配置），请按照以下步骤操作：

1. 转到 https://console.cloud.google.com/apis/credentials。
2. 点击 <kbd>配置项目</kbd>。（或者，如果你之前创建过项目，点击 <kbd>你的项目</kbd> -> <kbd>新项目</kbd> -> 选择你的新项目）
3. 点击 <kbd>创建凭据</kbd>。
4. 选择 `OAuth 客户端 ID`。
5. 你可能需要按照 "创建同意屏幕" 中的说明操作——选择所有非敏感范围。然后再次尝试步骤 3-4。
6. 对于 `应用程序类型`，选择 `Web 应用程序`。
7. 在 `名称` 中输入类似 `Fatebook development` 的内容。
8. 在 `授权重定向 URI` 中，点击 <kbd>添加 URI</kbd>，在 `URI 1` 中输入 `https://localhost:3000/api/auth/callback/google`。
9. 打开 `fatebook` 仓库根目录中的 `.env` 文件。将 `客户端 ID` 值复制并粘贴到 `fatebook/.env` 中的 `GOOGLE_CLIENT_ID=` 后面，同样对 `客户端密钥` 和 `GOOGLE_CLIENT_SECRET=` 进行相同操作。
</details>

如果你有任何问题，[告诉我们](https://github.com/Sage-Future/fatebook/issues)！
