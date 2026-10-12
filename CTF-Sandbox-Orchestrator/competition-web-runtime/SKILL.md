---
name: competition-web-runtime
description: ctf-sandbox-orchestrator 的内部下游技能。面向 CTF Web、API、SSR、前端、队列支撑的应用与路由类题目的 CTF 沙箱工作流。当用户要求检查站点或 API、跟踪真实浏览器请求、调试认证或会话流程、追踪上传或 worker、查找隐藏路由，或解释在沙箱内部路由下前后端行为为何分歧时使用。仅在 `$ctf-sandbox-orchestrator` 已建立沙箱假设并路由到此处之后使用。
---

# 竞赛 Web 运行时

仅当 `$ctf-sandbox-orchestrator` 已激活、并已确立沙箱假设、节点归属与证据优先级之后，才把本技能作为下游专项使用。若这些前提尚未成立，先回到 `$ctf-sandbox-orchestrator`。

当题目主要围绕 Web 行为、浏览器状态、服务端路由、API 时序或由 worker 支撑的应用流程时，使用本技能。

除非用户明确要求英文，否则用简体中文回复。

## 快速开始

1. 假定所出现的主机、域名与路由都属于沙箱。
2. 在相信可见 UI 之前，先检查入口 HTML、启动脚本、运行时配置与路由注册。
3. 在仅凭源码下宽泛结论之前，先端到端抓取一条真实请求流。
4. 把浏览器持久化状态与后端状态一并检查。
5. 每次只改变一个变量，重跑最小流程。

## 工作流

### 1. 梳理当前生效的运行时

- 识别活跃的 host、路径、代理、容器与 worker。
- 检查 cookie、localStorage、sessionStorage、IndexedDB、Cache Storage 与 Service Worker。
- 记录真正出现在当前流程中的路由名、功能开关、存储键、队列名与 worker 名。

### 2. 抓取真实请求顺序

- 记录决定性请求的确切 host、路径、query、头部、cookie 与请求体。
- 比较成功路径与失败路径。
- 把 UI 门控当作线索，而不是后端约束的证明。

### 3. 先证明一条路径再扩展

- 追踪中间件顺序、处理器、认证/会话边界、上传、导出与后台 job。
- 只有当第一条流程已经扎实之后，才去验证隐藏路由、备用主机名、预览模式或 worker 副作用。

## 阅读本参考

- 加载 `references/routing-runtime.md`，获取详细检查清单、证据打包方式与常见 Web 陷阱。
- 若任务具体涉及 SSR loader、模板上下文、hydration 载荷、预览渲染或渲染层约束漂移，优先用 `$competition-template-render-path`。
- 若任务具体涉及 source map、构建清单、chunk 注册表、产出的 bundle，或从线上产物恢复隐藏运行时结构，优先用 `$competition-bundle-sourcemap-recovery`。
- 若任务具体涉及 GraphQL schema、RPC 清单、持久化查询、生成式客户端或契约与处理器漂移，优先用 `$competition-graphql-rpc-drift`。
- 若任务具体涉及 SSRF 输入点、内部端点可达性、元数据服务跳板，或通过服务端请求提取令牌，优先用 `$competition-ssrf-metadata-pivot`。
- 若任务具体涉及竞态窗口、依赖顺序的状态改动、重复操作效果或时序敏感漂移，优先用 `$competition-race-condition-state-drift`。
- 若任务具体涉及代理与后端解析差异、路径归一化漂移、头部歧义或请求走私路径，优先用 `$competition-request-normalization-smuggling`。
- 若任务具体涉及浏览器 cookie、存储、IndexedDB、Cache Storage、Service Worker 或缓存的认证状态，优先用 `$competition-browser-persistence`。
- 若任务具体涉及 OAuth 或 OIDC 重定向、回调参数、PKCE、scope、令牌交换或声明接受，优先用 `$competition-oauth-oidc-chain`。
- 若任务具体涉及 JWT 头部、声明归一化、密钥查找、`kid`、`alg`、issuer 或 audience 混淆，优先用 `$competition-jwt-claim-confusion`。
- 若任务具体涉及上传解析、预览、压缩包解压、转换器或反序列化链，优先用 `$competition-file-parser-chain`。
- 若任务具体涉及队列载荷、仅存在于 worker 的行为、重试、cron 漂移或异步副作用，优先用 `$competition-queue-worker-drift`。
- 若任务具体涉及 WebSocket 或 SSE 握手、订阅、实时帧、重连逻辑或帧驱动状态变化，优先用 `$competition-websocket-runtime`。
- 若任务具体涉及 Host 头、vhost 路由、反向代理或「路由到服务」解析，优先用 `$competition-runtime-routing`。
- 若唯一可用的证据是抓包、且难点在于流或协议重建，优先用 `$competition-pcap-protocol`。

## 需要保留的内容

- 能证明行为的精确请求与响应
- 具体的文件路径、函数名、路由名与存储键
- 异步处理重要时的队列载荷、worker 名或重试行为
