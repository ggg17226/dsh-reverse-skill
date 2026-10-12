---
name: competition-kernel-container-escape
description: ctf-sandbox-orchestrator 的内部下游技能。面向内核攻击面、namespace 与 cgroup 边界、容器隔离假设、syscall 路径与逃逸原语验证的 CTF 沙箱工作流。当用户要求分析容器到宿主机的逃逸路径、内核漏洞利用前置条件、namespace 穿越、capability 滥用，或证明某个利用原语是否越过沙箱边界时使用。仅在 `$ctf-sandbox-orchestrator` 已建立沙箱假设并路由到此处之后使用。
---

# 竞赛内核容器逃逸

仅当 `$ctf-sandbox-orchestrator` 已激活、并已确立沙箱假设、节点归属与证据优先级之后，才把本技能作为下游专项使用。若这些前提尚未成立，先回到 `$ctf-sandbox-orchestrator`。

当决定性的步骤是证明「容器化上下文」与「宿主机或更高权限内核上下文」之间发生了边界穿越时，使用本技能。

除非用户明确要求英文，否则用简体中文回复。

## 快速开始

1. 先梳理运行时隔离：namespace、cgroup、seccomp、capability、LSM 与挂载边界。
2. 把「利用前置条件」「利用原语」与「边界穿越证明」三者分开。
3. 记录内核版本、配置线索、运行时选项与可达的 syscall 面。
4. 把带插桩的观察结果与未改动的原始题目路径分开。
5. 复现一条最小的「原语到边界穿越」链。

## 工作流

### 1. 梳理隔离与内核攻击面

- 记录 namespace 映射、cgroup 模式、capability、seccomp profile、AppArmor 或 SELinux 状态、已挂载文件系统与运行时 socket。
- 注明内核版本、发行版构建线索、模块暴露情况与容器运行时行为。
- 让宿主机与容器的观察结果都对应到确切的节点与上下文。

### 2. 证明利用原语与穿越

- 说明可控输入、触发条件、受影响对象，以及可观察到的内核或运行时状态变化。
- 捕获穿越前后的身份、namespace、挂载或进程可见性，以证明边界穿越。
- 把「仅导致崩溃」的行为与「稳定获得能力」区分开。

### 3. 收敛到决定性逃逸链

- 压缩为：前置状态 → 原语触发 → 边界穿越证据 → 得到的宿主机级能力。
- 说明根因是内核漏洞、运行时错误配置、capability 超授，还是 namespace 泄漏。
- 若路径主要依赖初始立足点之后的凭据重放，交给 Linux credential pivot 技能。

## 阅读本参考

- 加载 `references/kernel-container-escape.md`，获取隔离检查清单、原语检查清单与对照原则。

## 需要保留的内容

- 内核与运行时上下文、capability 集合、seccomp 或 LSM 状态，以及 namespace 映射
- 原语触发数据、边界穿越证据与得到的能力
- 一条从容器上下文到宿主机相关效果的最小可复现链
