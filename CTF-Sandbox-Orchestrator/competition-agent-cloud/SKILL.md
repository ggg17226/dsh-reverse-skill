---
name: competition-agent-cloud
description: ctf-sandbox-orchestrator 的内部下游技能。面向 AI Agent、提示注入（prompt injection）、MCP 或工具链、云、容器、CI/CD 与供应链类题目的 CTF 沙箱工作流。当用户要求在沙箱假设下分析「提示到工具」的执行流、检索投毒、被挂载的密钥、部署漂移、运行时与清单不一致，或镜像仓库来源与 CI 产物时使用。仅在 `$ctf-sandbox-orchestrator` 已建立沙箱假设并路由到此处之后使用。
---

# 竞赛 Agent 与云环境

仅当 `$ctf-sandbox-orchestrator` 已激活、并已确立沙箱假设、节点归属与证据优先级之后，才把本技能作为下游专项使用。若这些前提尚未成立，先回到 `$ctf-sandbox-orchestrator`。

当题目的解题路径由「提示到工具」的执行、检索与记忆边界、部署漂移，或构建与发布来源驱动时，使用本技能。

除非用户明确要求英文，否则用简体中文回复。

## 快速开始

1. 先判断主导路径是 Agent 侧还是基础设施侧。
2. 梳理一条最小控制链：不可信输入 → 模型可见上下文 → 工具或部署副作用。
3. 把「签入（checked-in）的意图」与「运行时的真实状态」区分开。
4. 用紧凑的证据区块保留提示、工具参数、清单、挂载与来源追踪步骤。
5. 用最小上下文与最少插桩复现该利用或错误配置。

## 工作流

### 1. Agent 与提示注入

- 把提示、工具 schema、检索到的分块、规划器笔记、记忆文件与交接信息都视为题目取证对象。
- 证明一条最小链条：不可信内容 → 模型可见指令 → 工具副作用。
- 把「声称具备的能力」与「运行时实际暴露的能力」区分开。

### 2. 云、容器与 CI/CD

- 把构建期、部署期与运行期拆开分析。
- 将 compose 或 kube 清单与实时的挂载、环境变量、日志与流量相互对照。
- 追踪来源链：源码 → 依赖解析 → 构建 → 发布 → 运行时消费方。

## 阅读本参考

- 加载 `references/agent-cloud.md`，获取控制栈检查清单、部署真相检查清单与证据打包方式。
- 若任务具体涉及提示边界滥用或「检索内容到工具」的漂移，优先用 `$competition-prompt-injection`。
- 若任务具体涉及 CI、依赖来源、镜像仓库漂移或交付产物，优先用 `$competition-supply-chain`。
- 若任务具体涉及队列消息体、异步 worker 漂移、重试或仅存在于 worker 的运行时状态，优先用 `$competition-queue-worker-drift`。
- 若任务具体涉及 SSRF 打向内部控制面、元数据端点或由元数据派生的令牌跳板，优先用 `$competition-ssrf-metadata-pivot`。
- 若任务具体涉及代理与上游的解析差异、歧义头部、路径归一化漂移或请求走私行为，优先用 `$competition-request-normalization-smuggling`。
- 若任务具体涉及元数据服务访问、实例或工作负载身份、链路本地令牌路径或元数据派生权限，优先用 `$competition-cloud-metadata-path`。
- 若任务具体涉及 kube API 权限、服务账号信任、准入控制行为、控制器漂移或集群密钥暴露，优先用 `$competition-k8s-control-plane`。
- 若任务具体涉及实时挂载、sidecar、init 容器或仅运行时可见的密钥暴露，优先用 `$competition-container-runtime`。
- 若任务具体涉及容器到宿主机的边界穿越、内核面前置条件或逃逸原语验证，优先用 `$competition-kernel-container-escape`。

## 需要保留的内容

- 提示片段、检索到的分块、规划器状态转移与最终工具参数
- 与实时挂载或路由相对应的 compose 或 Kubernetes 片段
- 产物哈希、依赖漂移、CI 步骤与最终运行时消费方
