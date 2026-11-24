# skills/：能力市场的技能

这里是乾元官方能力市场的**技能**条目（ADR 0024）。每个子目录是一个条目：目录名即条目名（安装 ID），
内容与 Core 的技能目录格式完全相同，安装后原样放在 `<用户根>\skills\<名>\`，Core 在新会话中加载。

当前上架 Word 文档（`docx`）、PDF 文档（`pdf`）、PowerPoint 演示文稿（`pptx`）、技能创建器
（`skill-creator`）和 Excel 电子表格（`xlsx`）。源码来自
[Anthropic Skills `e4bb5d9`](https://github.com/anthropics/skills/tree/e4bb5d960d87126ff3b3a8af1a104b7db67560ba/skills)，
每个安装包都包含各自的 Apache-2.0 `LICENSE.txt`。市场只分发源码及随附文件，不安装技能运行时可能需要的第三方工具。
五个市场图标均使用 Material Icon Theme 5.40.0 的现成图形（`word`、`pdf`、`powerpoint`、`skill`、`table`），
原样转成 PNG；[MIT 许可证](https://github.com/skaduv/primus/blob/da4584b7bb18d6b160bf1dc1b62a63355dfc2eba/apps/desktop/src/components/icons/LICENSE-material-icon-theme)也随图标元数据保留。

```
skills/
└─ <名>/
   ├─ SKILL.md        必需：技能本身（frontmatter 的 name 与目录名相同、description 必填）
   ├─ market.yaml     必需：市场元数据（见下），不进安装包
   ├─ README.md       可选：详情页显示的说明；没有时用 SKILL.md 去掉 frontmatter 后的正文。不进安装包
   ├─ icon.svg        可选：图标（market.yaml 的 icon 指向它），不进安装包
   └─ …               SKILL.md 引用的其他文件（脚本、模板、参考资料），原样打进安装包
```

## 上架、更新与下架

- **上架**：新建 `skills/<名>/`，写好 `SKILL.md` 与 `market.yaml`，提交到 `dev`，经 PR 合入本仓库 `main` 后由 Market 工作流发布。
- **更新**：改了任何会进安装包的文件，都要在 `market.yaml` 里**升版本号**。已发布的版本号不能复用：同名同版本的
  内容变了，构建失败（已安装用户的 SHA-256 会对不上）；版本号也不能比已发布的低。旧版本的安装包保留在 Release 上。
- **下架**：删掉目录。下次发布时该条目从索引中消失，它的附件被删除；已安装的用户不受影响。
- 本地检查：在构建工具仓库 `skaduv/primus` 运行 `pnpm build:market --root <本仓库路径>`（输出到 `dist/market/`，不发布）。

## `market.yaml` 规范（技能）

已知键严格校验，**出现未知键即构建失败**（防止拼写错误被悄悄忽略）。

| 键 | 必填 | 说明 |
|---|---|---|
| `title` | 是 | 显示名。字符串（视为中文），或 `{ zh: …, en: … }` |
| `description` | 否 | 一句话简介，写法同 `title` |
| `version` | 是 | SemVer 版本号，如 `1.0.0`；预发布版本（`1.1.0-beta.1`）不会被推荐安装 |
| `category` | 否 | 分类键，只能用构建工具 `scripts/build-market.mjs` 中 `MARKET_META.categories` 已声明的：`office` 办公、`writing` 写作、`dev` 开发、`data` 数据、`learning` 学习、`life` 生活 |
| `labels` | 否 | 标签键数组，只能用 `MARKET_META.labels` 已声明的标签，如 `featured`（精选） |
| `engines` | 否 | `{ primus: "<兼容范围>" }`，如 `">=0.1.6"`、`">=1.0.0 <2.0.0"`；支持 `>=` `>` `<=` `<` `=`、空格为“且”、`||` 为“或”；缺省为兼容所有版本 |
| `changelog` | 否 | 这个版本的更新说明，写法同 `title` |
| `icon` | 否 | 条目目录下的 `.svg` 或 `.png` 文件名，≤ 256 KiB；SVG 不能含脚本或事件属性 |

示例：

```yaml
title: 示例技能
description: 说明技能擅长处理什么任务。
category: office
labels: [featured]
version: 1.0.0
engines:
  primus: ">=0.1.6"
changelog: 首次上架。
```

## 生成器的检查

- 条目名只能是字母、数字、`-`、`_`（最长 64），与 `mcp/`、`plugins/` 下的条目不区分大小写不重名；
- `SKILL.md` 按 Core 的规则解析（与能力中心 `skill-frontmatter.ts` 相同）：Core 不加载的（YAML 无效、缺 description）
  与 Core 只告警的（名称不是小写字母 / 数字 / 连字符、description 超过 1024 字）都拒绝，`name` 必须与目录名相同；
- 密钥扫描：仓库里不得出现真实的 Key / Token / 私钥；
- 不能包含符号链接；`.` 开头的文件与目录不打包；
- 最多 5000 个文件、单个文件 32 MiB、合计 64 MiB（与薄壳解压上限一致）。
