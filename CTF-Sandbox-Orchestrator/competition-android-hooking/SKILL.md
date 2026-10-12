---
name: competition-android-hooking
description: ctf-sandbox-orchestrator 的内部下游技能。面向 Android APK Hook、Frida 跟踪、请求签名还原、SSL Pinning 绕过、JNI 边界检查与应用信任边界分析的 CTF 沙箱工作流。当用户要求 Hook 某个 APK、分析签名逻辑、跟踪 Java 或 native 边界、绕过 Pinning 或 Root 检测、检查 SharedPreferences 或应用数据库，或重放被服务端接受的移动端请求时使用。仅在 `$ctf-sandbox-orchestrator` 已建立沙箱假设并路由到此处之后使用。
---

# 竞赛 Android Hook

仅当 `$ctf-sandbox-orchestrator` 已激活、并已确立沙箱假设、节点归属与证据优先级之后，才把本技能作为下游专项使用。若这些前提尚未成立，先回到 `$ctf-sandbox-orchestrator`。

当决定性路径需要穿过 Android 应用的实时信任边界、而仅靠静态字符串无法解决时，使用本技能。

除非用户明确要求英文，否则用简体中文回复。

## 快速开始

1. 在打补丁或重签名之前，先留存原始 APK、解包资源与反编译产物。
2. 从 manifest、导出组件、deeplink、native 库、prefs、本地数据库与随包配置入手。
3. 决定要 Hook 的最窄运行时边界：签名器、加密辅助函数、JNI 桥、WebView 桥或请求构造器。
4. 在声称已理解某个信任边界之前，先让静态证据与动态跟踪相互印证。
5. 用最小的 Hook 集合复现出被签名的请求、被接受的 token 或被门控的分支。

## 工作流

### 1. Hook 之前先做静态排查

- 梳理包结构、导出的 activity、service、receiver、provider 与 deeplink 处理器。
- 记录 SSL Pinning 逻辑、Root 检测、功能开关、token 存储位置、SharedPreferences、SQLite 表，以及 protobuf 或 RPC 边界。
- 判断敏感逻辑位于 Java、Kotlin、JNI 还是随包的 WebView 中。

### 2. Hook 最窄的边界

- 优先 Hook 请求签名器、加密辅助函数、keystore 访问、protobuf 编解码或 JNI 封送（marshaling），而不是大范围的 UI Hook。
- 在真正改变信任结果的那个边界上记录明文输入、签名字符串、请求头、nonce 与输出。
- 若 Pinning 或环境检测阻碍推进，只做刚好足以暴露真实请求路径的补丁或 Hook。

### 3. 重放被接受的路径

- 重建能到达服务端被接受分支的最小序列：本地状态、nonce、请求体、签名与请求头。
- 让 Hook 日志、抓到的请求形态与本地存储路径都对应到同一个账号或会话状态。
- 若题目的重心转向变换算法还原而非 Android 运行时，切回更通用的密码学或移动端技能。

## 阅读本参考

- 加载 `references/android-hooking.md`，获取 Hook 目标、存储检查清单与证据打包方式。

## 需要保留的内容

- Hook 点、类名、JNI 符号、签名器输入输出与请求头名称
- SharedPreferences、本地数据库记录、deeplink、导出组件与 token 存储路径
- 能证明该信任边界的最小可重放请求或分支
