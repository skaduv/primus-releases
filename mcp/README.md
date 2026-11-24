# mcp/：能力市场的 MCP 服务器

这里是乾元官方能力市场的 **MCP 服务器**条目（ADR 0024 第 4.1、6、7 节）。每个子目录是一个条目：目录名即条目名
（安装 ID，也是写进 `mcp.json` 的服务器名），安装后运行文件在 `<用户根>\mcp\<名>\`，服务器条目写进用户级 `mcp.json`，
`command` 为安装目录里内置运行时（`node.exe` / `python.exe`）的绝对路径。

```
mcp/
└─ <名>/
   ├─ market.yaml     必需：市场元数据 + 服务器模板（见下），不进安装包
   ├─ README.md       可选：详情页显示的说明，不进安装包
   ├─ icon.svg        可选：图标（market.yaml 的 icon 指向它），不进安装包
   └─ server/         必需：stdio 服务器的源码，内容即安装包的根目录（entry 相对它）
      ├─ package.json + package-lock.json    Node 有依赖时（dependencies 或 optionalDependencies 非空必须有锁文件）
      └─ requirements.txt                    Python 有依赖时（每行 包名==版本）
```

当前没有上架 MCP。市场（M2）支持带内置运行时的**本地 stdio** 服务器；远程（http）服务器随插件在 M3 加入。

## 依赖与按架构出包

- 内置 `node.exe` 没有 npm、`npx`，也没有 `uvx`：依赖由 CI 在对应架构的 Windows 运行器（x64：`windows-2025`，
  ARM64：`windows-11-arm`）上预装，随安装包分发，用户机器上不联网装任何东西。
  - Node：`npm ci --omit=dev --ignore-scripts`（**不运行安装脚本**；原生模块必须以预编译的可选依赖提供，
    不能靠 node-gyp 现场编译）。依赖装在包内 `node_modules/`。
  - Python：内置 `python.exe -m pip install --target site-packages --no-compile --only-binary=:all: --no-deps -r requirements.txt`
    （只装轮子，**不解析依赖**：`requirements.txt` 必须逐行列出并锁定全部直接与传递依赖，如只写 `requests==2.32.3`
    而漏掉 `urllib3`、`idna`、`certifi`、`charset-normalizer` 时服务器导入失败，冒烟不通过、不发布）。安装时 `mcp.json` 条目会带 `PRIMUS_PYTHON_PACKAGES=<安装目录>\site-packages`，内置 Python 的
    `sitecustomize.py` 据此把随包依赖加入 `sys.path`（ADR 0010），字节码缓存在安装目录的 `pycache\`。
- 包根的 `install.json`（不区分大小写）为乾元安装记录保留，不能作为文件或目录随包发布；子目录里的同名文件不受影响。
- 不要提交 `node_modules/`、`site-packages/`、`__pycache__/`；`.` 开头的文件不打包。
- 每个声明的架构出一个包：`mcp-<名>-<版本>-<x64|arm64>.zip`（确定性 zip）。没有依赖的条目两个包内容相同。
- 冒烟：CI 用与用户机器相同的方式合成 `mcp.json` 条目（声明的环境变量填占位值），先直接按 MCP stdio 协议
  `initialize` → `tools/list`，再用 Core 命令行 `mcp list --json` 实测，**至少列出一个工具**，否则不发布。
  所以需要 Key 的服务器在没有真实 Key 时也要能启动并列出工具（调用工具时再提示缺 Key）。源码没变、发布时沿用的
  已发布安装包同样下载、核对 SHA-256 后重新冒烟（依赖或内置运行时变了时旧包也必须还能启动）。`arch` 不含当前架构的
  条目在该架构上不预装、不冒烟。
- 本地检查：从构建工具仓库 `skaduv/primus` 运行 `pnpm build:market --root <本仓库路径>`（无依赖的条目直接打包）；有依赖的条目需要它声明的每个架构的预装副本：分别在
  x64 与 ARM64 的 Windows 上先执行 `node scripts/build-core.mjs --platform win-<x64|arm64>` 生成同架构的内置运行时
  （`dist/runtime`，全新检出时没有），再执行 `node scripts/market-mcp.mjs prepare --arch <x64|arm64> --runtime dist/runtime
  --root <本仓库路径> --out market-prebuilt`（原生的可选依赖按运行器架构安装，不能在 x64 上代装 ARM64），把两份 `market-prebuilt/<arch>/`
  放到同一目录后再 `node scripts/build-market.mjs --root <本仓库路径> --prebuilt market-prebuilt`。

## 上架、更新与下架

与技能相同（见 [`skills/README.md`](../skills/README.md)）：提交到 `dev`、经 PR 合入本仓库 `main`；改了 `server/` 里任何文件都要升版本号。
索引记下源码的 SHA-256（`sourceSha256`）：同名同版本源码没变时沿用已发布的安装包（依赖重装的字节未必相同），
源码变了而版本号没升则构建失败。

## `market.yaml` 规范（MCP）

已知键严格校验，**出现未知键即构建失败**。与技能共有的键（`title`、`description`、`version`、`category`、`labels`、
`engines`、`changelog`、`icon`）写法与 `skills/README.md` 相同，另有：

| 键 | 必填 | 说明 |
|---|---|---|
| `runtime` | 是 | `node` 或 `python`：用安装目录里的内置运行时启动 |
| `entry` | 是 | 入口文件，`server/` 内的相对路径（`/` 分隔），如 `index.mjs`、`src/server.py` |
| `args` | 否 | 入口文件之后的固定参数（字符串数组，每项 ≤ 1024 字符） |
| `arch` | 否 | 要出包的架构：`[x64, arm64]`（缺省为两者） |
| `exposure` | 否 | 工具暴露方式：`codemode`（缺省）、`deferred`、`direct`、`hidden` |
| `env` | 否 | 需要用户填写的环境变量：`{ 变量名: 声明 }`，见下 |

环境变量声明（只声明，**不带值**；仓库里不得出现真实 Key）：

| 键 | 必填 | 说明 |
|---|---|---|
| `label` | 是 | 表单上显示的名字，写法同 `title` |
| `description` | 否 | 说明（在哪里申请、格式），写法同 `title` |
| `required` | 否 | `true` 时不填不能安装（缺省 `false`） |
| `secret` | 否 | `true` 时表单用密码输入框（缺省 `false`）。值仍**明文**保存在本机（ADR 0024 第 7.2 节） |
| `helpUrl` | 否 | 获取指引的 https 地址 |

变量名只能是字母或 `_` 开头的字母、数字、`_`、`-`；不能用保留名：`PRIMUS_` 开头的、`PATH`、`PATHEXT`、`NODE_OPTIONS`、
`NODE_PATH`、`PYTHONPATH`、`PYTHONHOME`、`SYSTEMROOT`、`COMSPEC`（不区分大小写）。

示例：

```yaml
title: 高德天气
description: 用高德开放平台查询国内城市的实况天气与未来几天的预报。
category: life
version: 1.0.0
engines:
  primus: ">=0.1.7"
runtime: node
entry: index.mjs
env:
  AMAP_KEY:
    label: 高德 Web 服务 Key
    description: 在高德开放平台控制台创建应用并添加“Web 服务”类型的 Key。
    required: true
    secret: true
    helpUrl: https://console.amap.com/dev/key/app
```

## 生成器的检查

- 条目名规则、与 `skills/`、`plugins/` 不区分大小写不重名，与技能相同；
- 条目目录下只能有 `market.yaml`、`README.md`、图标与 `server/`；入口文件必须在 `server/` 里；
- 依赖清单：Node 有 `dependencies` 或 `optionalDependencies` 时必须有 `package-lock.json`，Python 的 `requirements.txt` 每行 `包名==版本`（预装用 `--no-deps`，传递依赖也必须逐个列出并锁定）；
  运行时与清单不匹配（Node 带 `requirements.txt` 等）拒绝；
- 预装副本（`--prebuilt <目录>/<arch>/<名>/`）里的源码必须与仓库逐字节一致，多出的文件只能在 `node_modules/`
  （Node）或 `site-packages/`（Python）下；
- 密钥扫描覆盖源码与预装的依赖；不能包含符号链接；
- 最多 20,000 个文件、单个文件 128 MiB、单个安装包 256 MiB（与薄壳 `market/install.rs` 的解压上限一致）。
