---
name: competition-browser-persistence
description: ctf-sandbox-orchestrator 的内部下游技能。面向浏览器 cookie、localStorage、sessionStorage、IndexedDB、Cache Storage、Service Worker、离线缓存与客户端会话持久化的 CTF 沙箱工作流。当用户要求检查浏览器状态、重放被缓存的认证或会话行为、解释页面加载后行为为何不同，或追踪存储的客户端状态如何改变请求、渲染或访问权限时使用。仅在 `$ctf-sandbox-orchestrator` 已建立沙箱假设并路由到此处之后使用。
---

# 竞赛浏览器持久化

仅当 `$ctf-sandbox-orchestrator` 已激活、并已确立沙箱假设、节点归属与证据优先级之后，才把本技能作为下游专项使用。若这些前提尚未成立，先回到 `$ctf-sandbox-orchestrator`。

当决定性分支位于浏览器持有的状态中、而不仅仅存在于可见 HTML 或后端源码里时，使用本技能。

除非用户明确要求英文，否则用简体中文回复。

## 快速开始

1. 先确定当前生效的持久化面：cookie jar、localStorage、sessionStorage、IndexedDB、Cache Storage 还是 Service Worker。
2. 在改动状态之前，先记录 origin、scope、domain、path、过期时间与键名。
3. 把存储的状态绑定到一个具体效果：请求头、渲染分支、被缓存的响应、离线行为或隐藏路由的访问。
4. 把启动时注入的状态与运行时被修改的状态分开。
5. 复现能到达决定性分支的最小有状态序列。

## 工作流

### 1. 梳理浏览器状态面

- 检查 cookie、存储桶、Service Worker 注册、缓存条目，以及启动过程中暴露的临时全局变量。
- 记录每一项状态实际作用于哪个 origin、host、路由或功能开关。
- 把认证 token、刷新凭据、CSRF 状态、缓存响应与功能开关分别放入独立的证据区块。

### 2. 把状态与运行时行为对应起来

- 说明存储的状态如何变成请求头、角色推导、路由可见性、被缓存的 API 数据或离线兜底行为。
- 对比干净状态与被改动状态的运行结果，每次只改变一个变量。
- 把「仅影响 UI 的状态」与「被后端接受的状态」区分开。

### 3. 收敛到决定性持久化链

- 把结论压缩为最小链条：初次访问页面或登录 → 状态被持久化 → 后续请求或渲染分支 → 得到能力。
- 让提取出的存储内容、Service Worker 脚本与重放步骤都对应到同一 origin 与路由。
- 若问题扩展到浏览器持久化之外的通用 Web 路由或 worker 行为，切回更通用的 web-runtime 技能。

## 阅读本参考

- 加载 `references/browser-persistence.md`，获取浏览器状态检查清单、Service Worker 检查清单与证据打包方式。

## 需要保留的内容

- Cookie 属性、存储键、数据库名、缓存键、Service Worker scope 与 origin 边界
- 每个决定性状态项所导致的确切请求或渲染效果
- 最小可用路径的「干净状态 vs 被改动状态」复现步骤
