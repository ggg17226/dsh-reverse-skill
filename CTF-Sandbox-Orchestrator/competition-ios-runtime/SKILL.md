---
name: competition-ios-runtime
description: ctf-sandbox-orchestrator 的内部下游技能。面向 IPA 运行时分析、Frida Hook、Objective-C 或 Swift 方法跟踪、Keychain 检查、SSL Pinning 绕过、URL Scheme 处理与 iOS 请求签名还原的 CTF 沙箱工作流。当用户要求 Hook 某个 IPA、跟踪 Objective-C 或 Swift 运行时行为、检查 Keychain 或 plist 状态、绕过 Pinning、分析 deeplink 或 universal link，或重放被接受的 iOS 请求时使用。仅在 `$ctf-sandbox-orchestrator` 已建立沙箱假设并路由到此处之后使用。
---

# 竞赛 iOS 运行时

仅当 `$ctf-sandbox-orchestrator` 已激活、并已确立沙箱假设、节点归属与证据优先级之后，才把本技能作为下游专项使用。若这些前提尚未成立，先回到 `$ctf-sandbox-orchestrator`。

当决定性路径需要穿过实时的 iOS 信任边界、而仅靠静态字符串或 plist 值无法解决时，使用本技能。

除非用户明确要求英文，否则用简体中文回复。

## 快速开始

1. 把原始 IPA、解出的 bundle，以及任何解密或重签名后的副本分别留存为独立产物。
2. 在开展大范围运行时 Hook 之前，先从 `Info.plist`、entitlements、URL scheme、framework、Keychain 使用情况与本地应用存储入手。
3. 选择能证明该行为的最窄运行时边界：签名器、信任评估器、Keychain 访问处、Objective-C 或 Swift 方法，或网络请求构造器。
4. 在声称已理解该信任路径之前，先让静态 bundle 证据与实时 Hook 输出相互印证。
5. 用最小的 Hook 集合复现出被接受的请求、令牌或被门控的分支。

## 工作流

### 1. iOS 静态排查

- 梳理 bundle 结构、`Info.plist`、entitlements、URL scheme、universal link、内嵌 framework 与 app group 路径。
- 记录可能的信任边界：请求签名器、设备绑定、证书校验、越狱检测、Keychain 访问或本地缓存加载。
- 注明敏感逻辑位于 Objective-C、Swift、内嵌 framework，还是随包的 Web 页面中。

### 2. Hook 运行时边界

- 优先 Hook 请求构造器、加密辅助函数、信任评估器、Keychain 读取或 Objective-C selector，而不是大范围的 UI 处理器。
- 在会改变服务端接受结果的那个边界上，记录明文输入、请求头、nonce、签名字符串与输出。
- 只做刚好足以暴露真实请求路径的 Pinning 或环境检测补丁与绕过。

### 3. 重放被接受的路径

- 重建最小的有状态序列：本地令牌、设备标识、请求体、签名、请求头与信任校验。
- 让 Hook 日志、bundle 路径、plist 键与本地存储产物都对应到同一个会话或账号状态。
- 若任务主要转向变换算法还原而非 iOS 运行时，切回更通用的密码学或移动端技能。

## 阅读本参考

- 加载 `references/ios-runtime.md`，获取 Hook 目标、存储检查清单与证据打包方式。

## 需要保留的内容

- Bundle 路径、entitlements、plist 键、selector、类名、Hook 点与请求头名称
- Keychain 条目、本地数据库或 plist 路径、URL scheme 与 app group 存储位置
- 能证明该 iOS 信任边界的最小可重放请求或分支
