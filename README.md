# CloudSentinel GitOps 配置仓库种子

此目录是一套可复制为独立私有仓库 `cloudsentinel-gitops` 的完整种子。正式启用后，源码仓库继续保留模板和发布脚本，Argo CD 只读取独立配置仓库；不要让 Argo CD 直接跟踪应用源码仓库。

## 当前河源 Compact Lab 入口

Argo CD ApplicationSet 已指向 `compact-lab-staging` 与
`compact-lab-production`。Compact Overlay 复用原七节点 Lab 资源，但把镜像拉取
地址切换为北京 ACR 公网 Endpoint，并按组合式 Workload Label 调度到河源的
`worker-01`、`worker-02`。数据层入口是
`platform/cloudsentinel-data/overlays/lab-compact`，监控入口是
`platform/cloudsentinel-monitoring/overlays/lab-compact`。

河源公网边缘入口固定在 `worker-02`：Staging Web 使用 HTTPS NodePort
`30443`，Production Web 使用独立证书源 Secret
`cloudsentinel-production-web-public-tls` 和 HTTPS NodePort `30444`，Grafana
使用 `30300`。Production 继续保持人工 Sync，`externalTrafficPolicy: Local`
确保其 NodePort 只在存在本地 Web Endpoint 的边缘节点接收流量。

原 `lab-*`、数据 `overlays/lab` 和监控 `overlays/lab` 继续保存北京七节点历史
基线，不应改写成河源拓扑。Compact 数据层仍是单副本 MySQL/Redis、静态 Retain
Local PV、NetworkPolicy 和逻辑备份 CronJob，不是高可用生产方案。

## 初始化前必须替换

- 企业 RDS/Tair Overlay 中的 `REPLACE_*` 与域名可以保持休眠，但任何被 Argo CD 引用的 `lab-*`、`bootstrap` 和数据目录不得含占位符；ACR Registry/Namespace 已固定为北京个人版实例和 `cloudsentinel0306`；
- 三个镜像的真实 SHA-256 digest；
- Staging/Production 的外部密钥对象路径；
- Argo CD `repoURL` 与 CODEOWNERS 团队。

`validate-gitops` 对当前激活路径拒绝占位符，并渲染两个 Compact 应用 Overlay、
Secret Overlay、Compact 数据层与监控、Secret Store 和 Argo CD Bootstrap。
`platform/cloudsentinel-secrets` 负责先物化 Secret，使数据初始化和 PreSync
Migration 开始前凭证已存在。

北京七节点 Overlay 继续使用 ACR VPC Endpoint。河源 Compact Overlay 使用
`crpi-1s64ln3ptbvgkqof.cn-beijing.personal.cr.aliyuncs.com` 公网 Endpoint，源
`.dockerconfigjson` 必须包含同一个公网认证服务器。ACR 个人版和跨地域公网拉取
无生产 SLA并可能限流，只适用于当前学习集群。

## 分支保护

GitHub Free 私有仓库无法强制分支保护或 CODEOWNERS 审批，因此当前基线要求操作者自律使用 PR、检查 `validate-gitops`，并禁止把 GitHub App 配置为绕过流程。CODEOWNERS 只用于责任标记，不等价于强制审批。Production 晋级工作流只复制已经在 Staging 使用的 digest；合并后仍必须由操作者在 Argo CD 手工 Sync。若升级 GitHub Pro，应立即为 `main` 启用分支保护、必需状态检查、对话解决和线性历史。

数据层是单点实验配置。不得删除 `cloudsentinel-data` Namespace、数据 Application、PVC 或 PV；这些操作即使保留底层目录，也会破坏声明式绑定和恢复流程。正式生产必须切回托管数据服务或经验证的高可用平台。
