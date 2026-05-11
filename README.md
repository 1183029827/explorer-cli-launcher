<h1 align="center">explorer-cli-launcher</h1>
<p align="center">
  <strong>Agent 技能：在资源管理器地址栏和右键菜单中启动 CLI 应用</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/平台-Windows-blue" alt="Windows">
  <img src="https://img.shields.io/badge/兼容-OpenCode%20%7C%20Claude%20Code-blue" alt="Compatible">
  <a href="#english">English ↓</a>
</p>

---

## 做什么的

让 Agent 帮你配置 CLI 应用，做到：

- **地址栏直接输入** `opencode` → 回车 → 在 PowerShell 中启动
- **右键文件夹** → 以该文件夹为工作目录启动 CLI

不用再先开 PowerShell、再敲命令了。

## 安装

### 方式一：Skills CLI（推荐）

```bash
npx skills add 1183029827/explorer-cli-launcher -g -y
```

### 方式二：手动克隆

```bash
git clone https://github.com/1183029827/explorer-cli-launcher.git ~/.agents/skills/explorer-cli-launcher
```

### 方式三：下载 .skill 包

从 [Releases](https://github.com/1183029827/explorer-cli-launcher/releases) 下载 `.skill` 文件，导入你的 agent。

无需依赖，无需配置，即装即用。

## 触发方式

对 agent 说这些话时，技能会自动加载：

> "帮我把 opencode 加到资源管理器地址栏启动"
> "我想在地址栏输入 claude 就能打开它"
> "给 npm 加个右键菜单，让我在任意文件夹里打开"
> "能不能右键文件夹就启动 CLI，不用我先开终端"

## Agent 会创建什么

Agent 会为每个 CLI 应用生成以下内容：

| 输出物 | 位置 | 效果 |
|--------|------|------|
| `.bat` 启动脚本 | `%USERPROFILE%\.local\bin\` | 地址栏输入即启动 |
| 注册表项 | `HKCU\Software\Classes\Directory\shell\` | 右键文件夹菜单 |
| 注册表项 | `HKCU\Software\Classes\Directory\Background\shell\` | 右键空白处菜单 |

全部写入用户空间，无需管理员权限，无 UAC 弹窗。

## 对话示例

**你说：** "把 opencode 加到地址栏和右键菜单吧"

**Agent 会：**
1. 找到 PATH 中的 `opencode.cmd`
2. 创建 `%USERPROFILE%\.local\bin\opencode.bat`
3. 注册 `HKCU\...\Directory\shell\opencode` → `"Open with OpenCode"`
4. 注册 `HKCU\...\Directory\Background\shell\opencode` → `"Open with OpenCode"`

做完后你就可以在地址栏输入 `opencode`，或者右键文件夹启动它。

## 环境要求

- Windows 10 或 11
- PowerShell 5.1+
- 目标 CLI 已安装且在 `PATH` 中
- 支持 skill 的 Agent（OpenCode、Claude Code 等）

## 卸载

对 agent 说：*"帮我把 XXX 的地址栏和右键菜单去掉"*

或手动执行：

```powershell
Remove-Item "$env:USERPROFILE\.local\bin\myapp.bat" -Force
Remove-Item "HKCU:\Software\Classes\Directory\shell\myapp" -Recurse -Force
Remove-Item "HKCU:\Software\Classes\Directory\Background\shell\myapp" -Recurse -Force
```

---

<h2 id="english">English</h2>

### What it does

An agent skill that configures CLI apps so you can:

- **Type the app name** in Explorer's address bar → launches in PowerShell
- **Right-click a folder** → launches CLI with that folder as workspace

### Install

```bash
# Recommended: via Skills CLI
npx skills add 1183029827/explorer-cli-launcher -g -y

# Or clone manually
git clone https://github.com/1183029827/explorer-cli-launcher.git ~/.agents/skills/explorer-cli-launcher

# Or download .skill from Releases
```

### Trigger phrases

> "Make opencode launch from the Explorer address bar"
> "I want to type claude in Explorer and have it work"
> "Add right-click menu for npm"

### Requirements

Windows 10/11, PowerShell 5.1+, target CLI in PATH.

## License

MIT
