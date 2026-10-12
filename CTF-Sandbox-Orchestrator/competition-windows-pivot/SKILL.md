---
name: competition-windows-pivot
description: ctf-sandbox-orchestrator 的内部下游技能。面向 Kerberos、WinRM、SMB、RDP、Windows 凭据材料、可重放票据、委派边界与主机到主机跳板链的 CTF 沙箱工作流。当用户要求重放 Kerberos 材料、追踪 WinRM、SMB 或 RDP 跳板、理解主机到主机的权限移动，或证明哪个 Windows 服务接受了凭据或票据时使用。仅在 `$ctf-sandbox-orchestrator` 已建立沙箱假设并路由到此处之后使用。
---

# 竞赛 Windows 跳板

仅当 `$ctf-sandbox-orchestrator` 已激活、并已确立沙箱假设、节点归属与证据优先级之后，才把本技能作为下游专项使用。若这些前提尚未成立，先回到 `$ctf-sandbox-orchestrator`。

当题目路径以主机到主机移动、可重放票据材料或 Windows 权限边界为主时，使用本技能。

除非用户明确要求英文，否则用简体中文回复。

## 快速开始

1. 把跳板压缩为具体链条：立足点 → 恢复出的产物 → 重放路径 → 跳板主机 → 得到的能力。
2. 把「存储的凭据材料」与「可用的权限」区分开。
3. 让主机证据、票据证据与权限效果处于同一条时间线上。
4. 为每一份被重放的产物记录确切的接受服务或主机。
5. 复现仍能证明权限边界的最小跳板。

## 工作流

### 1. 恢复重放材料

- 在当前路径中检查 SAM、SECURITY、SYSTEM、NTDS、DPAPI、LSA secrets、浏览器存储、PowerShell 历史、ETW、Sysmon 与事件日志。
- 按「实际可在哪里使用」区分口令、哈希、票据、cookie、vault blob 或 gMSA 材料。

### 2. 追踪跳板链

- 梳理实际使用的协议：WinRM、SMB、RDP、WMI、管理共享、远程注册表或服务控制。
- 当 Kerberos 是重点时，记录 SPN、委派、PAC 或组数据、加密类型与接受服务。
- 当 AD 边界是重点时，检查 ACL、GPO 链接、SIDHistory、委派、证书模板与复制权限。

### 3. 报告该边界

- 让跳板路径保持具体且可重放。
- 说明哪份产物越过了哪条边界，以及目标主机上出现了什么能力。

## 阅读本参考

- 加载 `references/windows-pivot.md`，获取跳板检查清单、Kerberos 证据区块与常见重放误区。
- 若任务具体涉及 DPAPI masterkey、浏览器或 vault 存储、受保护 blob，或证明恢复出的 DPAPI 密钥在哪里被接受，优先用 `$competition-dpapi-credential-chain`。
- 若任务具体涉及 LSASS 内存、票据缓存、可重放的会话材料或主机常驻凭据提取，优先用 `$competition-lsass-ticket-material`。
- 若任务具体涉及委派边界、SPN 信任、S4U 流程，或哪个服务接受被委派的票据，优先用 `$competition-kerberos-delegation`。
- 若难点在于强制认证、强制原语、中继目标，或接受中继认证的服务，优先用 `$competition-relay-coercion-chain`。

## 需要保留的内容

- 主机名、登录 ID、SID、SPN、票据字段、服务名与事件 ID
- 确切的重放点，以及由此产生的登录会话、令牌或组成员变化
- 原始主机产物与推导出的时间线分别保存
