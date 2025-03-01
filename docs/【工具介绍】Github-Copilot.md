# Codespace 是什么
Codespace 是 GitHub 提供的一种基于云的开发环境。以下是关于 Codespace 的详细介绍：

### 主要特点
- **便捷性**
    - 无需在本地安装复杂的开发工具和依赖项。开发者可以直接在浏览器中访问一个完整的开发环境，随时随地开始编码，只需要有网络连接和一个浏览器即可。
    - 例如，当你在不同的设备上工作或者在旅行中需要临时处理代码，只需登录 GitHub 并打开相应的 Codespace 就能继续工作。
- **一致性**
    - 为团队成员提供一致的开发环境。在团队开发中，不同成员可能使用不同的操作系统和本地开发配置，这可能导致一些兼容性问题。而 Codespace 可以确保每个团队成员都在相同的环境中进行开发，减少因环境差异带来的问题。
    - 比如，团队可以通过配置 `.devcontainer` 文件来定义 Codespace 的开发环境，包括使用的编程语言版本、安装的扩展等，保证每个人的环境一致。
- **可定制性**
    - 开发者可以根据自己的需求定制 Codespace 的环境。通过 `.devcontainer` 文件，可以指定所需的操作系统、编程语言、工具和扩展等。
    - 例如，如果你需要开发一个 Python 项目，可以在 `.devcontainer` 文件中指定 Python 的版本，并安装相关的 Python 包，如 `numpy`、`pandas` 等。
 
### 主要用途
- **快速上手新项目**
    - 当你开始一个新的项目时，不需要花费时间来设置本地开发环境。只需创建一个 Codespace，就可以立即开始编写代码。
    - 例如，当你克隆一个开源项目的仓库后，直接在该仓库中创建一个 Codespace，就可以在预配置好的环境中对项目进行修改和调试。
- **协作开发**
    - 方便团队成员之间的协作。团队成员可以在相同的 Codespace 环境中进行开发，更容易共享代码和解决问题。
    - 例如，团队成员可以同时在一个 Codespace 中进行代码审查、调试和合并请求等操作。
- **教育和培训**
    - 在教育领域，Codespace 可以为学生提供一致的开发环境，方便教师进行教学和指导。
    - 例如，教师可以为学生提供一个包含特定项目代码和开发环境配置的仓库，学生只需创建相应的 Codespace 就可以开始学习和实践。
 
### 使用方法
- **创建 Codespace**
    - 在 GitHub 仓库页面，点击 `Code` 按钮，然后选择 `Codespaces` 选项卡，点击 `Create codespace on main`（或其他分支）按钮即可创建一个新的 Codespace。
- **配置环境**
    - 通过在仓库根目录下创建 `.devcontainer` 文件夹，并在其中添加 `devcontainer.json` 文件来配置 Codespace 的环境。
 
总的来说，Codespace 为开发者提供了一种灵活、便捷的开发方式，尤其是在远程协作和快速开发方面具有很大的优势。

# `github-copilot-guide` 目录下各个 Markdown 文件的内容介绍

### `0-welcome.md`
此文件内容简短，仅包含一个 HTML 注释 `<!-- readme -->`，可能是作为欢迎页面的占位或起始标记。

### `1-copilot-extension.md`
该文件主要介绍在 Codespace 中启用 Copilot 的步骤，具体内容如下：
- 推荐在新的浏览器标签页中进行后续操作以便参考说明。
- 讲述在打开 Codespace 之前，创建开发容器（`.devcontainer/devcontainer.json`）并将 Copilot 添加到扩展列表的方法，包含创建文件及文件内容的配置。
- 介绍提交文件、打开 Codespace、验证 Codespace 运行、查看 Copilot 扩展等一系列操作，期间需等待一定时间。

### `2-skills-javascript.md`
此文件主要围绕在 JavaScript 文件中查看 AI 代码建议展开：
- 先肯定用户创建安装了 Copilot 的 Codespace 的工作，介绍 Copilot 支持的语言。
- 详细说明添加 JavaScript 文件并开始编写代码的活动步骤，如创建 `skills.js` 文件、输入函数头，Copilot 会自动给出函数体建议，按 `Tab` 键可接受建议。
- 还包括将代码推送到仓库的操作步骤，如 `git add`、`git commit`、`git push` 等，最后需等待并刷新仓库页面。

### `3-copilot-hub.md`
文件主要介绍查看 GitHub Copilot 选项卡中的多个建议：
- 提醒用户拉取最新代码（`git pull`）。
- 介绍添加另一个 JavaScript 方法并查看所有建议的活动步骤，包括创建 `member.js` 文件、输入函数头、查看 Copilot 建议、打开完成面板选择解决方案等。
- 同样包含将代码推送到仓库的操作，最后需等待并刷新仓库页面。

### `4-copilot-comment.md`
该文件主要介绍使用注释让 Copilot 生成代码：
- 说明创建新文件 `comments.js`。
- 讲述在文件中输入注释（如 `// Create web server`），Copilot 会给出代码块建议，可通过多种方式选择并接受解决方案。

### `X-finish.md`
此文件为课程完成页面，内容包括：
- 祝贺用户完成课程，并展示庆祝图片。
- 回顾用户完成的所有任务，如在 Codespace 中设置 Copilot、使用 Copilot 接受建议代码等。
- 提供后续学习和资源链接，如 GitHub Copilot 的相关文档、企业账户信息、入门指南、配置设置等。
- 给出下一步建议，如分享课程反馈、学习其他 GitHub 技能、阅读 GitHub 入门文档、探索项目等。

# 4-copilot-comment.md的翻译
<!--
  <<< 作者备注：步骤 4 >>>
  开始此步骤前，先对前一个步骤予以确认。
  定义相关术语并链接到 docs.github.com。
-->

## 步骤 4：使用注释让 Copilot 生成代码

_很不错，你已经会使用 Copilot 选项卡啦！_ 🥳

你现在已经利用了 Copilot 快速选项卡的自动建议功能以及 Copilot 中心来接受人工智能生成的建议了。

现在，让我们来看看如何利用注释来生成 Copilot 建议！

### :keyboard: 活动：将最新代码拉取到 Codespace 仓库。

> **注意**
> 在下一个活动之前，必须先进行拉取操作。

1. 使用 VS Code 终端拉取最新代码：

   ```
   git pull
   ```

### :keyboard: 活动：从注释生成 Copilot 建议代码。

1. 在 Codespace 中的 VS Code 资源管理器窗口内创建一个新文件。（如果你之前关闭了 Codespace，请重新打开或创建一个新的 Codespace。）
2. 将文件命名为 `comments.js`。
3. 在文件中输入以下注释：
   ```
   // 创建 Web 服务器
   ```
4. 按 `enter` 键换行。
5. Copilot 将会给出一个代码块建议。
6. 将鼠标悬停在红色波浪线上，然后选择 `...`。

   > **注意**
   > 如果你没有看到 Copilot 代码块建议或者红色波浪线以及三个点 `...`，可以按 `control + enter` 组合键调出 GitHub Copilot 完成面板。

7. 点击 `打开完成面板`。Copilot 将会合成大约 10 个不同的代码建议。你应该会看到类似如下的内容：
   ![2023 年 4 月 25 日下午 3:59:42 的屏幕截图](https://user-images.githubusercontent.com/26442605/234425509-74ea96e0-bbd6-417b-84c5-73546ac7b2cd.png)
8. 找到你喜欢的解决方案，然后点击 `接受解决方案`。
9. 你的 `comments.js` 文件将会用你选择的解决方案进行更新。

### :keyboard: 活动：从 Codespace 将代码推送到你的仓库。

1. 使用 VS Code 终端将 `comments.js` 文件添加到仓库中：

   ```
   git add comments.js
   ```

2. 接着在 VS Code 终端中暂存并将更改提交到仓库：

   ```
   git commit -m "Copilot 第三次提交"
   ```

3. 最后从 VS Code 终端将代码推送到仓库：

   ```
   git push
   ```

**等待约 60 秒，然后刷新你的仓库主页以进行下一步操作。**
