---
name: competition-oauth-oidc-chain
description: ctf-sandbox-orchestrator 的内部下游技能。面向 OAuth、OIDC、重定向流程、state 或 nonce 处理、PKCE、令牌交换、刷新逻辑、声明映射与被接受的登录路径的 CTF 沙箱工作流。当用户要求追踪重定向、回调参数、scope、state、nonce、PKCE、刷新令牌、授权同意，或解释 OAuth 或 OIDC 链如何变成被接受的身份或权限时使用。仅在 `$ctf-sandbox-orchestrator` 已建立沙箱假设并路由到此处之后使用。
---

# 竞赛 OAuth 与 OIDC 链

仅当 `$ctf-sandbox-orchestrator` 已激活、并已确立沙箱假设、节点归属与证据优先级之后，才把本技能作为下游专项使用。若这些前提尚未成立，先回到 `$ctf-sandbox-orchestrator`。

当难点在于证明 OAuth 或 OIDC 流程如何被构造、交换并最终被接受时，使用本技能。

除非用户明确要求英文，否则用简体中文回复。

## 快速开始

1. 按顺序梳理认证链：入口路由、重定向、authorize 请求、回调、令牌交换、刷新与最终接受服务。
2. 在改动任何内容之前，先记录 scope、state、nonce、PKCE 材料、redirect URI 与承载声明的令牌。
3. 把「持有令牌」与「身份实际被接受」区分开。
4. 把浏览器可见的重定向与服务端可见的令牌交换放在同一条紧凑链上。
5. 复现能证明决定性身份边界的最小「重定向到接受」流程。

## 工作流

### 1. 梳理重定向与令牌路径

- 记录 issuer、client ID、redirect URI、authorize 参数、回调参数、令牌端点与刷新路径。
- 注明哪些值由用户控制、推导而来、被缓存或被校验：`state`、`nonce`、PKCE verifier、audience、scope 或 prompt。
- 让浏览器重定向、服务端交换与由此得到的会话状态保持一一对应。

### 2. 证明「令牌到身份」的接受

- 说明 code、ID token、access token 或 refresh token 如何变成应用会话、声明映射、tenant 选择或被接受的权限。
- 记录令牌声明、过期时间、audience、subject、scope，以及确切接受它的应用或后端边界。
- 把「UI 登录成功」与「后端授权成功」区分开。

### 3. 收敛到决定性 OAuth 链

- 把结论压缩为最小序列：入口请求 → 重定向 → 回调 → 令牌或声明被接受 → 得到能力。
- 若某个参数改动是关键，则保留一条规范正常流程与一条最小改动流程。
- 若任务扩展到认证链之外的通用 Web 路由或存储行为，切回更通用的 web-runtime 技能。

## 阅读本参考

- 加载 `references/oauth-oidc-chain.md`，获取重定向检查清单、令牌检查清单与证据打包方式。
- 若难点在于签发之后的 JWT 头部解析、声明归一化、密钥查找或令牌校验混淆，优先用 `$competition-jwt-claim-confusion`。

## 需要保留的内容

- Redirect URI、参数、code、令牌声明、scope，以及接受它的服务或回调
- 声明或令牌变成被接受的应用身份的确切位置
- 一条最小的可重放「重定向到接受」序列
