# Kubernetes

通过本机 `kubectl` 与 `helm` 连接并管理 Kubernetes 集群的核心 stdio MCP 服务器，基于 [Flux159/mcp-server-kubernetes](https://github.com/Flux159/mcp-server-kubernetes) v4.2.3（MIT 许可证）整理，移除了远程 HTTP/SSE 服务、OpenTelemetry 遥测、示例文档与部署脚手架，仅保留核心 MCP 能力。

## 使用前提

1. 本机已安装 `kubectl` 并位于系统 `PATH` 中；
2. 已配置可用的 kubeconfig（默认读取系统 `KUBECONFIG` 或 `~/.kube/config`，也可在下方环境变量中指定 `KUBECONFIG_PATH`）；
3. 如需使用 Helm 相关工具，本机需安装 Helm v3 并位于 `PATH` 中。

## 核心工具

### 查询与诊断（只读）

- `kubectl_get`：查询资源列表或详情（默认对 Secret 数据进行脱敏）
- `kubectl_describe`：查看资源详细描述与事件
- `kubectl_logs`：查看 Pod、Deployment 或 Job 日志
- `kubectl_context`：查看与切换 kubeconfig 上下文（只读模式下仅允许查询）
- `kubectl_reconnect`：刷新 kubeconfig 配置状态
- `explain_resource`：查看 Kubernetes 资源字段说明
- `list_api_resources`：列出集群支持的 API 资源类型
- `ping`：检查 MCP 服务连通性

### 创建、变更与发布

- `kubectl_apply`：应用 YAML 清单或文件
- `kubectl_create`：创建资源
- `kubectl_patch`：更新资源部分字段
- `kubectl_scale`：扩缩容副本数
- `kubectl_rollout`：查看发布状态、历史、重启与回滚
- `port_forward` / `stop_port_forward`：建立或停止本地端口转发

### Helm 管理

- `install_helm_chart`：安装 Helm Chart（支持模板应用模式）
- `upgrade_helm_chart`：升级已安装的 Release
- `uninstall_helm_chart`：卸载 Release

### 运维与高危操作

- `kubectl_delete`：删除资源
- `kubectl_generic`：执行受安全参数校验保护的通用 `kubectl` 命令
- `node_management`：对节点执行 `cordon` / `drain` / `uncordon`
- `exec_in_pod`：在容器内执行命令
- `cleanup`：清理本会话跟踪的端口转发与资源

### 内置资源与诊断提示词

- 资源 URI：`k8s://default/pods`、`k8s://default/deployments`、`k8s://default/services`、`k8s://namespaces`、`k8s://nodes`
- 提示词：`k8s-diagnose`（按关键词与命名空间执行系统化的 Pod/节点排障流程）

## 可选环境变量

| 变量名 | 说明 |
|---|---|
| `KUBECONFIG_PATH` | 自定义 kubeconfig 文件路径；留空时使用系统 `KUBECONFIG` 或 `~/.kube/config` |
| `ALLOW_ONLY_READONLY_TOOLS` | 设为 `true` 时仅暴露只读查询与诊断工具 |
| `ALLOW_ONLY_NON_DESTRUCTIVE_TOOLS` | 设为 `true` 时禁用删除、卸载 Helm、节点排空、容器内执行与 `kubectl_generic` |
| `MASK_SECRETS` | 默认开启 Secret 输出脱敏；设为 `false` 关闭 |
| `SPAWN_MAX_BUFFER` | `kubectl`/`helm` 输出缓冲区上限（字节），大型集群可设为 `5242880` |
