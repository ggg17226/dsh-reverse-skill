---
name: competition-websocket-runtime
description: ctf-sandbox-orchestrator 的内部下游技能。面向 WebSocket 与 SSE 握手、认证材料、订阅状态、实时消息 schema、重连行为与帧驱动运行时效果的 CTF 沙箱工作流。当用户要求检查 WebSocket 或 SSE 握手、解码帧、追踪订阅、跟踪重连逻辑、检查实时连接建立阶段发送的认证材料，或解释实时帧如何改变渲染或持久化状态时使用。仅在 `$ctf-sandbox-orchestrator` 已建立沙箱假设并路由到此处之后使用。
---

# 竞赛 WebSocket 运行时

仅当 `$ctf-sandbox-orchestrator` 已激活、并已确立沙箱假设、节点归属与证据优先级之后，才把本技能作为下游专项使用。若这些前提尚未成立，先回到 `$ctf-sandbox-orchestrator`。

当决定性行为由实时握手与帧流承载，而不是单次 HTTP 请求时，使用本技能。

除非用户明确要求英文，否则用简体中文回复。

## 快速开始

1. 先梳理握手：origin、路径、头部、cookie、query、认证令牌与升级响应。
2. 把连接建立、订阅消息、保活、服务端推送与重连逻辑分开。
3. 用同一条链记录消息 schema、topic 或 channel 身份与状态副作用。
4. 把帧与渲染、存储或后端可见的效果对应起来。
5. 复现能到达决定性状态变化的最小「握手加帧」序列。

## 工作流

### 1. 梳理实时握手

- 记录初始 HTTP 或 SSE 请求、升级头部、cookie、令牌、query 参数、origin 校验与协商出的协议。
- 注明认证材料是通过头部、cookie、query string 还是初始应用帧携带的。
- 让路由、订阅端点与会话身份保持一一对应。

### 2. 解码消息流

- 把 subscribe、unsubscribe、ack、heartbeat、服务端推送、重连与终止帧分开。
- 还原对行为有意义的消息类型、channel ID、schema 字段与序列关系。
- 把传输层保活与应用层业务消息区分开。

### 3. 收敛到决定性实时路径

- 把结论压缩为最小序列：握手 → 认证或订阅帧 → 被推送或被接受的帧 → 产生的状态变化。
- 把规范帧顺序与任何重放用的最小顺序并排保留。
- 若难点是没有运行时 UI 或应用状态关联的通用协议重组，切回更贴合的协议技能。

## 阅读本参考

- 加载 `references/websocket-runtime.md`，获取握手检查清单、帧检查清单与证据打包方式。

## 需要保留的内容

- 握手头部、cookie、query 参数、认证材料、协商的子协议与 channel ID
- 帧 schema、订阅消息、服务端推送、重连流程与产生的状态变化
- 能证明决定性分支的最小可重放实时序列
