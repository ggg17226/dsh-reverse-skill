---
name: competition-k8s-control-plane
description: ctf-sandbox-orchestrator 的内部下游技能。面向 Kubernetes API 分析、服务账号信任、RBAC 边界、准入与控制器行为、集群密钥、工作负载变更与命名空间范围内的漂移的 CTF 沙箱工作流。当用户要求检查 kube API 权限、服务账号令牌、RoleBinding 或 ClusterRoleBinding 边界、准入 webhook、控制器创建的 pod、密钥暴露，或解释实时工作负载为何与清单不一致时使用。仅在 `$ctf-sandbox-orchestrator` 已建立沙箱假设并路由到此处之后使用。
---

# 竞赛 K8s 控制面

仅当 `$ctf-sandbox-orchestrator` 已激活、并已确立沙箱假设、节点归属与证据优先级之后，才把本技能作为下游专项使用。若这些前提尚未成立，先回到 `$ctf-sandbox-orchestrator`。

当决定性路径要穿过 Kubernetes 控制面状态、API 权限或控制器行为，而不只是单个容器的运行时，使用本技能。

除非用户明确要求英文，否则用简体中文回复。

## 快速开始

1. 把清单意图与实时集群状态分开：API 对象、变更、控制器、密钥与最终工作负载。
2. 先确定当前生效的主体：服务账号、kubeconfig 身份、节点凭据、webhook 还是控制器。
3. 梳理从最小控制面边界到其工作负载效果的关系。
4. 用紧凑的证据区块保留 RBAC、服务账号、owner reference、命名空间边界与密钥消费方。
5. 复现能产生决定性工作负载或密钥效果的最小集群操作。

## 工作流

### 1. 梳理 API 信任路径

- 记录 namespace、服务账号、Role、ClusterRole、绑定、准入钩子、控制器，以及它们能够改动的资源。
- 区分读取权限、创建权限、patch 权限、exec 权限与密钥访问权限。
- 让主体、verb、资源、namespace 与最终对象保持一一对应。

### 2. 追踪从变更到工作负载状态

- 说明一个 API 动作如何变成 pod、卷挂载、密钥暴露、环境变量注入、job 运行或控制器创建的产物。
- 把签入的 YAML 与经过默认值填充、准入变更或控制器协调之后的实时对象对照。
- 把「pod 运行时行为」与「集群级变更逻辑」区分开。

### 3. 收敛到决定性集群路径

- 把结论压缩为最小链条：主体 → API 权限 → 被改动的对象 → 由此产生的工作负载、密钥或路由效果。
- 让 kube 对象、实时 describe 输出与被消费的密钥或配置路径都对应到同一个 namespace 与控制器。
- 若问题收窄为某一个容器的挂载或运行时偏差，切回更贴合的 container-runtime 技能。

## 阅读本参考

- 加载 `references/k8s-control-plane.md`，获取 RBAC 检查清单、控制器检查清单与证据打包方式。
- 若难点在于元数据服务可达性、工作负载身份、实例凭据或元数据派生权限，优先用 `$competition-cloud-metadata-path`。

## 需要保留的内容

- Namespace、服务账号、verb、资源类型、RoleBinding 或 ClusterRoleBinding，以及 owner reference 链
- 准入变更、生成的工作负载、被挂载的密钥与控制器造成的漂移
- 能产生决定性效果的确切 API 动作或对象差异
