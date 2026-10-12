---
name: competition-identity-windows
description: ctf-sandbox-orchestrator 的内部下游技能。面向 Active Directory、Kerberos、LDAP、OAuth、企业消息系统、Windows 主机取证、凭据材料与横向移动类题目的 CTF 沙箱工作流。当用户要求追踪票据或令牌、检查邮箱规则、分析 Windows 主机证据、理解 AD 信任路径，或解释跨沙箱关联节点的横向移动链时使用。仅在 `$ctf-sandbox-orchestrator` 已建立沙箱假设并路由到此处之后使用。
---

# 竞赛 Windows 身份

仅当 `$ctf-sandbox-orchestrator` 已激活、并已确立沙箱假设、节点归属与证据优先级之后，才把本技能作为下游专项使用。若这些前提尚未成立，先回到 `$ctf-sandbox-orchestrator`。

当题目围绕身份流转、可重放的凭据、Windows 主机产物、企业邮件或横向移动展开时，使用本技能。

除非用户明确要求英文，否则用简体中文回复。

## 快速开始

1. 先梳理身份或跳板链，再深入每一份主机产物。
2. 把「持有凭据」与「权限被接受」区分开。
3. 在同一时间线上关联身份证据、主机证据与邮件证据。
4. 用紧凑的证据区块保留票据、SID、事件 ID、邮箱规则与跳板主机。
5. 用最小可行链条复现出权限边界或邮件效果。

## 工作流

### 1. 身份与 AD

- 追踪主体来源、同步路径、令牌或票据签发、声明变换（claims transformation）、组解析与接受服务。
- 当 Kerberos 是重点时，记录票据类型、SPN、委派模式、PAC 或组数据、加密类型与缓存位置。

### 2. Windows 主机与跳板

- 把 SAM、SECURITY、SYSTEM、NTDS、DPAPI、LSA secrets、ETW、Sysmon、PowerShell、服务、计划任务、WMI、WinRM、SMB 与 RDP 关联为同一张跳板图。
- 把移动过程表述为具体链条：立足点 → 恢复出的产物 → 重放路径 → 跳板主机 → 得到能力。

### 3. 企业消息系统

- 把钓鱼诱饵、授权同意日志、邮箱规则与身份提供方事件关联在一起，使邮件路径与权限路径保持连通。

## 阅读本参考

- 加载 `references/identity-windows.md`，获取票据、主机与企业消息系统的检查清单。
- 若任务主要是主机到主机的跳板、Kerberos 重放或 Windows 权限链，优先用 `$competition-windows-pivot`。
- 若任务具体涉及约束委派、非约束委派、RBCD、S4U 或票据接受证明，优先用 `$competition-kerberos-delegation`。
- 若任务具体涉及 AD CS、证书模板、EKU、注册权限、PKINIT 或基于证书的权限，优先用 `$competition-ad-certificate-abuse`。
- 若任务具体涉及 OAuth 或 OIDC 声明、回调流程、scope、授权同意或被接受的登录身份，优先用 `$competition-oauth-oidc-chain`。
- 若任务具体涉及 DPAPI masterkey、vault blob、浏览器或 vault 密钥、备份密钥使用，或「受保护密钥到访问」链，优先用 `$competition-dpapi-credential-chain`。
- 若任务具体涉及 LSASS 内存、票据缓存、与 LUID 关联的材料、DPAPI 上下文或可重放的主机凭据产物，优先用 `$competition-lsass-ticket-material`。
- 若任务具体涉及邮箱规则、转发、OAuth 授权同意、代理访问或传输层邮件滥用，优先用 `$competition-mailbox-abuse`。
- 若任务具体涉及强制认证、中继目标，或证明哪个服务接受被中继的认证，优先用 `$competition-relay-coercion-chain`。

## 需要保留的内容

- SID、SPN、票据字段、事件 ID、邮箱规则与重放点
- 确切的主机到主机跳板顺序，以及接受凭据或票据的服务
- 原始产物、解析摘要与推导出的时间线，分别作为独立输出
