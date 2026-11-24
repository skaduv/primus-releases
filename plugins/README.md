# plugins/：能力市场的插件

这里将放乾元官方能力市场的**插件**条目（ADR 0024 第 4.1、7.3 节；原“连接器”并入插件）。**M3 之前不接受条目**：
构建工具仓库 `skaduv/primus` 的 `scripts/build-market.mjs` 发现这里有子目录时直接失败，因为插件包加入 Core `packages`、OAuth / API Key 连接与加载检查还没有实现。

计划中的结构（M3 落地时以本文件更新后的规范为准）：

```
plugins/
└─ <名>/
   ├─ market.yaml                       元数据；可声明 `mcp`（托管 MCP + 认证方式，即原“连接器”）
   ├─ extensions/ skills/ prompts/ themes/   Core 包的约定目录（都可选）
   └─ mcp/                              可选：插件自带的 MCP 服务器，结构同 mcp/
```

- 插件扩展与 Core 同权限运行：只上架经过 PR 审查的插件。
- 安装后内容在 `<用户根>\plugins\<名>\`，以本地路径加入 Core `settings.json` 的 `packages`，对所有工作区生效。
- 条目名与 `skills/`、`mcp/` 下的条目不区分大小写不重名。
