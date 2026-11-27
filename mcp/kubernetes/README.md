通过本机 `kubectl`（及可选 Helm v3）管理 Kubernetes 集群。

- **连接**：默认读取 `~/.kube/config`；也可填写 `KUBECONFIG_PATH`，或填写 `K8S_SERVER` 与 `K8S_TOKEN`。
- **功能**：资源增删改查、Pod 日志、容器执行、扩缩容与回滚、端口转发、节点维护与 Helm 操作。
