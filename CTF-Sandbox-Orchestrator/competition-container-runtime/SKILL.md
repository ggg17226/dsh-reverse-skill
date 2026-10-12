---
name: competition-container-runtime
description: ctf-sandbox-orchestrator 的内部下游技能。面向容器实时运行时分析、被挂载的密钥、sidecar、namespace、init 容器、entrypoint 漂移与「路由到容器」解析的 CTF 沙箱工作流。当用户询问为什么实时容器与清单不一致、被挂载的密钥在哪里被消费、sidecar 或 init 容器如何改变运行时状态，或哪条路由解析到哪个实时容器时使用。仅在 `$ctf-sandbox-orchestrator` 已建立沙箱假设并路由到此处之后使用。
---

# 竞赛容器运行时

仅当 `$ctf-sandbox-orchestrator` 已激活、并已确立沙箱假设、节点归属与证据优先级之后，才把本技能作为下游专项使用。若这些前提尚未成立，先回到 `$ctf-sandbox-orchestrator`。

当题目的实质是「实时容器或 pod 现在到底在做什么」，而不是「签入的清单声称它该做什么」时，使用本技能。

除非用户明确要求英文，否则用简体中文回复。

## 快速开始

1. 把意图与现实拆开：manifest、镜像、启动过程、实时挂载、实时路由、实时进程。
2. 梳理链路：宿主机 → 代理 → 容器或 pod → 挂载卷 → 消费进程。
3. 把密钥、渲染后的配置、init 输出与 sidecar 输出同静态清单分开保存。
4. 证明一条从「被挂载或注入的状态」到「可达行为」的最小实时路径。
5. 用最小的运行时特定链条复现该效果。

## 工作流

### 1. 梳理实时运行时

- 将 compose 或 kube 清单与正在运行的容器、pod、挂载卷、环境变量、sidecar、init 容器及 entrypoint 对照比较。
- 确定究竟是哪个进程在消费被挂载的密钥、渲染后的配置或共享卷的输出。

### 2. 追踪路由与挂载边界

- 把虚拟主机、反向代理、service、容器端口、文件系统挂载与运行时生成的文件路径一并对应起来。
- 记录决定性状态是「打进镜像的」「由环境变量注入的」「后来才挂载的」，还是「由 init 或 sidecar 进程写入的」。

### 3. 报告运行时偏差

- 指出实时运行时与签入意图发生分歧的最早时间点。
- 保留一条从 manifest 或 compose 意图到实时消费方行为的紧凑证据链。

## 阅读本参考

- 加载 `references/container-runtime.md`，获取运行时检查清单、挂载链检查清单与常见的实时/静态对照陷阱。
- 若难点在于 kube API 权限、服务账号信任、RBAC 边界、准入变更或控制器创建的工作负载漂移，优先用 `$competition-k8s-control-plane`。
- 若难点在于 Host 头路由、路径前缀重写或跨节点的「路由到服务」映射，优先用 `$competition-runtime-routing`。
- 若难点在于证明容器到宿主机的越界、内核攻击面前置条件或稳定的逃逸原语，优先用 `$competition-kernel-container-escape`。
- 若难点在于拿到容器立足点后重放 Linux 密钥、socket 信任边界或宿主机间跳板，优先用 `$competition-linux-credential-pivot`。

## 需要保留的内容

- 与实时挂载或路由相对应的 compose/Kubernetes 片段
- 容器 ID、pod 名、挂载路径、sidecar 输出、渲染后的配置路径与消费进程
- 仅在运行时才变得可达的确切路由或文件路径
