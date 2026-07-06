# 用 Pi Agent 搭建 AI 视频工作流：从安装到实战

## 一、Pi Agent 是什么？

在聊实操之前，先说说 Pi Agent 到底是什么、它和市面上其他 AI 编程 Agent 有什么不同。

目前 AI Agent 领域的主流工具——如 Cloud Code、Codex、Open Code——从名字里的"Code"就能看出来，核心定位是帮你写代码。它们预装了代码索引、测试运行、Git 操作、编码规范等一整套围绕代码交付设计的工具，开箱即用。

但问题是，不是每个人每天都在写代码。更多人日常需要搜资料、读 PDF、整理表格、做汇报、做视频。

Pi Agent 走了一条截然不同的路。**Pi 是一个极简的终端 AI coding agent harness**。它的核心哲学是：**Adapt pi to your workflows, not the other way around**——让 Agent 适应你的工作流，而不是让你去适应 Agent 的默认形态。

翻译成人话就是：Pi 只给你最基础的积木——默认只提供 `read`、`write`、`edit`、`bash` 四个工具——其他的能力，你需要什么就自己往上加什么。通过 **Extensions**（TypeScript 扩展）、**Skills**（可复用功能包）、**Prompt Templates**（提示模板）和 **Themes**（主题），你可以把 Pi 改造成任何你想要的样子。

这就是 Pi 和 Codex、Open Code 最大的区别：**Codex 是开箱即用的完整产品，Pi 是你可以自己搭积木的 Agent 底座**。每个人根据自己的需求装上不同的 Skill 和 Extension，最后用到的 Pi 都不一样。

> 有一个数据很能说明问题：在 OpenRouter 的排行榜上，Pi 每天的 token 消耗量排在第五名，紧跟 Cloud Code 后面。OpenAI Codex 的负责人甚至公开表示，他们大约有 5% 的生产流量已经跑在了 Pi Agent 上面。同样规模的任务，Pi 的 token 消耗大概只有 Cloud Code 的三分之一甚至更少。

## 二、实操案例：用 Pi Agent 搭建 AI 视频工作流

接下来，我用一个真实的案例，带你走一遍从零开始搭建 Pi Agent 视频工作流的全过程。

> **案例目标**：用 Pi Agent 自动完成一个 AI 视频的制作——搜索素材、生成图片、合成配音、渲染成片。

用到的主要工具包括：
- **Tavily**：AI 优化的搜索引擎，用于搜集素材和资料
- **PDF 工具**：读取和处理文档资料
- **Edge TTS**：微软 Edge 的神经网络 TTS，免费高质量配音
- **GPT Image-2**：OpenAI 的图像生成模型
- **HyperFrames**：让 AI Agent 用 HTML 写视频、渲染成 MP4 的框架

### 第一步：安装 WSL（Windows 用户必看）

Pi 对 Windows 的原生适配存在一些问题，但在 WSL 中运行非常丝滑。所以第一步是装好 WSL。

**1. 检查 CPU 虚拟化是否开启**

打开任务管理器 → 性能选项卡 → 查看"虚拟化"是否已启用。如果未启用，需要进 BIOS 开启。

**2. 启用 Windows 功能**

在任务栏搜索"功能"，打开"启动或关闭 Windows 功能"，勾选：
- **适用于 Linux 的 Windows 子系统**
- **虚拟机平台**

然后重启电脑。

**3. 安装 WSL Ubuntu**

以管理员身份打开 cmd 或 PowerShell，运行：
`wsl --install --web-download`
这会自动下载并安装 WSL 和 Ubuntu 系统。

安装完成后，在 PowerShell 中输入 `wsl` 就能启动默认的 WSL 系统。

**4. WSL 与 Windows 文件互访**

- 在 WSL 中查看 Windows 文件：`explorer.exe .`
- 在 Windows 中查看 WSL 文件：打开"我的电脑" → 点开右下角的 Linux

**5. 配置 WSL 网络（解决 VPN 和代理问题）**

WSL 默认是 NAT 网络模式，无法直接使用本机的 VPN 和代理。我们需要改成**镜像模式**。

在 `C:\Users\你的用户名\` 目录下新建一个 `.wslconfig` 文件，内容如下：
[wsl2]
networkingMode=mirrored

text
然后在 PowerShell 中执行 `wsl --shutdown` 重启 WSL，等 8 秒再重新打开。

### 第二步：安装 Pi Agent

进入 WSL 环境，执行安装命令：
`curl -fsSL https://pi.dev/install.sh | sh`
或者用 npm：
`npm install -g @earendil-works/pi-coding-agent`

配置 API Key（以 Anthropic 为例）：
可以用 `/login` 命令交互式登录，选择你喜欢的模型提供商。

### 第三步：安装各种工具（Extensions & Skills）

现在开始往 Pi 上"搭积木"。**最省事的方式是直接用自然语言指挥 Pi 一键完成**——你只需对 Pi 说：

> "请帮我安装所有视频工作流所需的扩展和技能，包括 Tavily 搜索、PDF 处理、Edge TTS 配音、GPT Image-2 生图和 HyperFrames 渲染，并配置好对应的 API 密钥（Tavily 和 OpenAI）。"

Pi 会根据你的指令自动执行安装。当然，如果你想手动控制，也可以分别运行下面的命令（仅供参考）：

**1. Tavily 搜索工具**

Tavily 是一个专为 AI Agent 优化的搜索引擎。先注册获取 API Key（有免费额度），然后安装扩展：
`pi install npm:@jmcombs/pi-tavily-search`
配置 API Key：
`export TAVILY_API_KEY="tvly-your-key-here"`

**2. PDF 工具**

安装 PDF 处理 Skill，让 Pi 能读取和解析 PDF 文档（具体包名请参考官方仓库，或直接让 Pi 自行查找安装）。

**3. Edge TTS 配音工具**

Edge TTS 通过 `node-edge-tts` 库调用微软 Edge 的在线神经网络 TTS 服务，完全免费、无需 API Key。安装对应的 Skill 即可使用。

**4. GPT Image-2 生图工具**

安装 GPT Image-2 扩展，Pi 就能调用 OpenAI 的 `gpt-image-2` 模型生成图片：
`pi install npm:@kuzat/pi-extension-gpt-image-2`

**5. HyperFrames 视频渲染**

HyperFrames 是一个开源框架，让 AI Agent 用 HTML、CSS 和 JavaScript 来"写"视频，然后渲染成 MP4。安装后，Pi 就能自动生成视频脚本、动画效果和合成代码。

### 第四步：串联成完整的视频工作流

所有工具装好之后，你只需要给 Pi 一条指令，比如：

> "帮我做一个 30 秒的产品介绍视频。用 Tavily 搜索产品资料，读取 PDF 文档提取关键信息，用 GPT Image-2 生成配图，用 Edge TTS 生成配音，最后用 HyperFrames 渲染成 MP4。"

Pi 会自动拆解任务、依次调用各个工具、交付完整的视频成品。

这就是 Pi Agent 的真正威力——**它不是让你多了一个"写代码的工具"，而是让你拥有一个可以自由组装的全能工作流引擎**。

## 三、写在最后

Pi Agent 的价值不在于它"有什么"，而在于它"没有什么"。它没有内置 MCP、没有子智能体、没有 plan mode、没有权限弹窗——因为它把选择权交给了你。

> 官网首页写着一句话："There are many agent harnesses, but this one is yours."

在这个"大而全"成为主流的时代，Pi 选择做减法。它只提供一个极简的底座，剩下的——用什么模型、装什么工具、搭什么工作流——全由你来决定。

而当你把 Pi 和 Tavily、Edge TTS、GPT Image-2、HyperFrames 这些工具组合在一起时，它就不再只是一个"编程助手"了——它变成了一个能帮你做视频、做 PPT、做调研、做任何事情的**全能数字员工**。

这才是 2026 年真正会用 AI 的人在做的事：**把 AI 当成工作流引擎，而不是问答机器**。