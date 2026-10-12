---
name: competition-dpapi-credential-chain
description: ctf-sandbox-orchestrator 的内部下游技能。面向 DPAPI masterkey、vault blob、浏览器凭据库、受保护密钥、域备份密钥与「密钥到被接受」重放链的 CTF 沙箱工作流。当用户要求检查 DPAPI blob 或 masterkey、恢复浏览器或 vault 凭据、追踪 DPAPI 上下文或备份密钥的使用，或解释受保护的 Windows 密钥如何转化为被接受的访问或权限时使用。仅在 `$ctf-sandbox-orchestrator` 已建立沙箱假设并路由到此处之后使用。
---

# 竞赛 DPAPI 凭据链

仅当 `$ctf-sandbox-orchestrator` 已激活、并已确立沙箱假设、节点归属与证据优先级之后，才把本技能作为下游专项使用。若这些前提尚未成立，先回到 `$ctf-sandbox-orchestrator`。

当决定性的 Windows 密钥受 DPAPI 保护、而难点在于证明「哪个上下文能解开它」以及「明文在哪里被接受」时，使用本技能。

除非用户明确要求英文，否则用简体中文回复。

## 快速开始

1. 把受保护 blob、masterkey、解密上下文与最终接受服务四者分开。
2. 在得出宽泛结论之前，先记录 SID、用户或机器上下文、masterkey 路径、vault 或浏览器存储位置，以及目标重放点。
3. 把 DPAPI 来源产物、解包步骤、明文密钥与接受边界放在同一条链上。
4. 区分本地用户 DPAPI、机器 DPAPI、域备份密钥的使用，以及应用特有的加密封装。
5. 复现能证明决定性边界的最小「DPAPI → 被接受访问」路径。

## 工作流

### 1. 梳理受保护密钥与 DPAPI 上下文

- 记录 blob 来源、masterkey 位置、SID、保护器作用域（protector scope）、用户 profile 路径、凭据库位置，以及任何应用层封装（例如浏览器加密或 vault 元数据）。
- 注明决定性的值是在 Credential Manager、Vault、浏览器 cookie、浏览器保存的密码、Wi-Fi profile、RDP 文件，还是自定义应用存储中。
- 让受保护产物、候选 masterkey 与账号或机器上下文保持一一对应。

### 2. 证明解包与接受

- 说明该密钥是如何被解密的：用户登录材料、机器上下文、域备份密钥，还是另一个已恢复的保护器。
- 记录明文类型、目标主机或服务、重放方式，以及由此得到的会话、令牌或数据访问。
- 把「blob 解密成功」与「访问实际被接受」区分开。

### 3. 收敛到决定性 DPAPI 链

- 把结论压缩为最小序列：受保护产物 → masterkey 或解包上下文 → 明文密钥 → 被接受的重放或访问 → 得到能力。
- 明确说明决定性边界位于 masterkey 恢复、DPAPI 作用域混淆、应用封装处理，还是接受该密钥的服务。
- 若任务扩展到通用 LSASS 票据材料或完整的 Windows 横向移动，交回更贴合的主机或跳板技能。

## 阅读本参考

- 加载 `references/dpapi-credential-chain.md`，获取 blob 检查清单、masterkey 检查清单与证据打包方式。

## 需要保留的内容

- Blob 路径、masterkey 路径、SID、保护器作用域、存储名称与应用封装细节
- 被恢复的明文所解锁的确切接受服务或数据集
- 一条能证明该边界的最小「受保护产物 → 被接受访问」序列
