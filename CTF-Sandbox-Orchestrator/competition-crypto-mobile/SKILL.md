---
name: competition-crypto-mobile
description: ctf-sandbox-orchestrator 的内部下游技能。面向密码学、编码、隐写、APK、IPA 与移动端信任边界类题目的 CTF 沙箱工作流。当用户要求解码某个数据块、还原变换链或密钥、检查隐藏的媒体载荷、Hook APK 或 IPA 的签名器、检查应用存储，或重放移动端请求签名逻辑时使用。仅在 `$ctf-sandbox-orchestrator` 已建立沙箱假设并路由到此处之后使用。
---

# 竞赛密码学与移动端

仅当 `$ctf-sandbox-orchestrator` 已激活、并已确立沙箱假设、节点归属与证据优先级之后，才把本技能作为下游专项使用。若这些前提尚未成立，先回到 `$ctf-sandbox-orchestrator`。

当题目依赖还原一条变换链、隐藏的媒体载荷、移动端签名路径或本地信任边界时，使用本技能。

除非用户明确要求英文，否则用简体中文回复。

## 快速开始

1. 先判断主导路径是密码学、隐写还是移动端。
2. 按顺序逐步还原变换，不要一上来就挑最复杂的算法。
3. 记录会影响结果的确切参数与边界。
4. Hook 能证明该行为的最窄移动端边界。
5. 复现出明文、载荷、被签名的请求或被接受的分支。

## 工作流

### 1. 密码学与编码

- 逐步重建整条链：容器格式、压缩、编码、xor 或替换、加密、完整性校验、最终解析。
- 保留确切的密钥、IV、nonce、salt、tag、偏移与字节序。

### 2. 隐写

- 检查元数据、chunk 布局、调色板、alpha 平面、LSB、缩略图、尾部追加数据与转码痕迹。
- 按证据给解码尝试排序，而不是凭蛮力好奇心乱试。

### 3. 移动端

- 从 manifest 或 plist、导出组件、deeplink、native 库、SharedPreferences、本地数据库与配置入手。
- 追踪签名器逻辑、token 存储、SSL Pinning、protobuf 或 RPC 边界，以及 native 桥调用。

## 阅读本参考

- 加载 `references/crypto-mobile.md`，获取变换检查清单、Hook 目标与证据打包方式。
- 若任务具体涉及 Android 动态跟踪、签名器 Hook、JNI 边界或 Pinning 检测，优先用 `$competition-android-hooking`。
- 若任务具体涉及 iOS 运行时跟踪、Keychain 访问、Objective-C 或 Swift Hook，或 IPA 内的 Pinning 检测，优先用 `$competition-ios-runtime`。
- 若任务具体涉及媒体载体、隐藏信道、缩略图或尾部追加数据，优先用 `$competition-stego-media`。

## 需要保留的内容

- 能证明每一级解码的关键字节
- Hook 点、被签名的字符串、请求头与本地存储路径
- 组件名、protobuf 字段、信道特有输出或尾部追加偏移
