---
name: competition-linux-credential-pivot
description: ctf-sandbox-orchestrator 的内部下游技能。面向 Linux 凭据产物、服务令牌、SSH 材料、云与容器密钥、socket 级信任与主机到主机跳板链的 CTF 沙箱工作流。当用户要求追踪 Linux 认证产物、被接受的令牌或密钥重放、socket 或服务账号信任边界、sudo 或 capability 滥用，或解释跨 Linux 题目节点的横向移动时使用。仅在 `$ctf-sandbox-orchestrator` 已建立沙箱假设并路由到此处之后使用。
---

# 竞赛 Linux 凭据跳板

仅当 `$ctf-sandbox-orchestrator` 已激活、并已确立沙箱假设、节点归属与证据优先级之后，才把本技能作为下游专项使用。若这些前提尚未成立，先回到 `$ctf-sandbox-orchestrator`。

当决定性边界是 Linux 凭据材料、以及该材料在哪里被接受时，使用本技能。

除非用户明确要求英文，否则用简体中文回复。

## 快速开始

1. 把「凭据存储位置」与「权限被接受」区分开。
2. 在下结论之前，先记录用户、进程、namespace、socket、密钥文件与服务信任边界。
3. 把产物恢复、重放路径与由此得到的能力放在同一条链上。
4. 区分本地提权与横向主机跳板。
5. 复现一条最小的「产物到被接受访问」路径。

## 工作流

### 1. 梳理凭据与信任产物

- 记录 SSH 密钥、agent socket、kubeconfig、云令牌、服务账号密钥、环境变量、配置文件与进程内存线索。
- 注明 sudoers 规则、capability、setuid 二进制、systemd unit 上下文与 namespace 边界。
- 让每份产物都对应到其所有者、作用域与预期的接受服务。

### 2. 证明重放与跳板

- 说明密钥、令牌、socket 或密钥材料在哪里被接受：SSH、API、Unix socket、容器运行时或控制面端点。
- 记录目标主机、协议、主体，以及由此得到的会话或权限。
- 把「认证成功」与「获得有用能力」区分开。

### 3. 收敛到决定性 Linux 跳板链

- 压缩为：恢复出的产物 → 被接受的重放路径 → 跳板主机或权限变化 → 得到的能力。
- 说明根因是密钥处理薄弱、令牌泄漏、socket 信任、sudo 或 capability 滥用，还是 namespace 穿越。
- 若链条转向内核漏洞利用边界，交给 kernel container escape 技能。

## 阅读本参考

- 加载 `references/linux-credential-pivot.md`，获取产物检查清单、重放矩阵与证据打包方式。

## 需要保留的内容

- 产物路径、所有者、作用域、接受服务与由此得到的主体
- 确切的跳板顺序，以及协议与目标主机或 namespace
- 一条能证明能力获得的最小可重放链
