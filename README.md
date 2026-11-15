<p align="center">
  <img src="assets/logo.png" alt="乾元 Primus" width="144" />
</p>

<h1 align="center">乾元 Primus</h1>

<p align="center">面向所有人的桌面 Agent 应用</p>

<p align="center">
  <a href="https://github.com/skaduv/primus-releases/releases/latest">
    <img src="https://img.shields.io/github/v/release/skaduv/primus-releases?style=flat-square&label=latest" alt="最新版本" />
  </a>
  <img src="https://img.shields.io/badge/Windows-11%20%7C%20x64%20%2F%20ARM64-0078D4?style=flat-square&logo=windows11&logoColor=white" alt="Windows 11 x64 / ARM64" />
  <img src="https://img.shields.io/badge/release-official-181717?style=flat-square&logo=github&logoColor=white" alt="官方发布" />
</p>

<p align="center">
  选择一个项目目录，用自然语言让 Agent 读代码、改代码、运行命令、排查问题。
</p>

<p align="center">
  <a href="https://github.com/skaduv/primus-releases/releases/latest"><strong>下载最新版本 →</strong></a>
</p>

> [!NOTE]
> 上方下载入口始终指向当前标记为 **Latest** 的正式版本。发布新版本后会自动更新，无需修改本 README。

## 关于 Primus

乾元（Primus）是一款面向开发与项目工作的 Windows 桌面 Agent 应用。它把自然语言对话、项目文件和命令行工作流放在同一个桌面工作区里，让 Agent 理解项目上下文，并通过工具完成实际的代码与排查任务。

- **项目级协作**：直接选择项目目录，让 Agent 在真实项目中读取文件、分析代码、应用修改并运行命令。
- **桌面原生体验**：基于 Tauri 构建，提供会话、文件与变更等桌面工作流。
- **持续更新**：安装后可随应用退出自动检查并升级到新版本。

## 下载与安装

前往 [**GitHub Releases**](https://github.com/skaduv/primus-releases/releases/latest)，根据电脑架构下载安装包。大多数电脑使用 x64；骁龙等 Windows on ARM 设备使用 ARM64。

| 设备 | 推荐安装包 |
| --- | --- |
| Intel / AMD 64 位电脑 | `PrimusSetup_<版本>_x64_zh-CN.exe` |
| Windows on ARM 设备 | `PrimusSetup_<版本>_arm64_zh-CN.exe` |
| 需要 MSI 安装时 | `Primus_<版本>_<架构>_zh-CN.msi` |

1. 打开[最新版本下载页](https://github.com/skaduv/primus-releases/releases/latest)。
2. 下载与电脑架构匹配的 `.exe` 安装程序。
3. 双击运行安装程序，按提示完成安装。

如果双击 `.msi` 提示无法识别文件类型，请改用对应的 `.exe` 安装程序；安装完成后的应用与直接安装 MSI 相同。

## 自动更新

使用当前版本的安装程序或 MSI 安装后，Primus 会在应用退出时自动安装已发布的新版本。

如果电脑上安装过较早的 NSIS 版本，只需运行一次当前版本的安装程序或 MSI，升级时会自动卸载旧版并保留原有数据。

## 系统要求

- Windows 11
- x64 或 ARM64
- 可访问互联网，用于 Agent 服务和版本更新

## 安全提示

> [!WARNING]
> 当前安装包尚未进行代码签名。首次运行时 Windows SmartScreen 可能会拦截，请确认文件来自本仓库后，点击「更多信息」→「仍要运行」。

请只从本仓库的 Releases 页面下载安装包。每个版本的发布说明都会提供安装包文件的 SHA-256，可用于校验下载是否完整：

```powershell
Get-FileHash .\<下载的文件名> -Algorithm SHA256
```

## 仓库说明

本仓库是乾元 Primus 的**官方发布仓库**，用于分发 Windows 安装包、发布说明与应用自动更新清单。

- [下载最新版本](https://github.com/skaduv/primus-releases/releases/latest)
- [浏览全部版本](https://github.com/skaduv/primus-releases/releases)

软件版权由作者保留。
