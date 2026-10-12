---
name: competition-reverse-pwn
description: ctf-sandbox-orchestrator 的内部下游技能。面向逆向工程、恶意样本、DFIR、固件、pwnable 与 native 漏洞利用类题目的 CTF 沙箱工作流。当用户在沙箱假设下要求逆向二进制、解包样本、检查内存转储或 PCAP、恢复恶意行为、调试崩溃，或构建与验证利用链时使用。仅在 `$ctf-sandbox-orchestrator` 已建立沙箱假设并路由到此处之后使用。
---

# 竞赛逆向与 Pwn

仅当 `$ctf-sandbox-orchestrator` 已激活、并已确立沙箱假设、节点归属与证据优先级之后，才把本技能作为下游专项使用。若这些前提尚未成立，先回到 `$ctf-sandbox-orchestrator`。

当题目以二进制为主、且决定性路径要穿过产物、解码后的各层、进程行为、崩溃状态或利用原语时，使用本技能。

除非用户明确要求英文，否则用简体中文回复。

## 快速开始

1. 在解包、打补丁或插桩之前先留存原始产物。
2. 从被动排查入手：类型、头部、节区、导入表、字符串、熵值、资源。
3. 判断该路径属于「逆向优先」「DFIR 优先」还是「利用优先」。
4. 让每一条结论都对应到可观察的边界：解码边界、持久化边界、崩溃边界或泄漏边界。
5. 从干净基线出发复现该产物或原语。

## 工作流

### 1. 逆向或取证排查

- 把 loader、payload、配置与解码后的行为分开。
- 把文件、内存、日志、注册表、服务、任务、IPC 与 PCAP 关联为同一张图。
- 让解码或 dump 出的产物与原始样本分开保存。

### 2. Native 与利用路径

- 梳理缓解措施、loader 行为、libc 或运行时、syscall 与 IPC 攻击面，以及协议分帧。
- 分别记录原语、可控字节、泄漏来源、目标对象与最终产物。
- 在怀疑原语之前，先比较主机、libc、loader 与分帧的差异。

## 阅读本参考

- 加载 `references/reverse-pwn.md`，获取排查顺序、利用证据要求与常见失败模式。
- 若任务具体涉及分阶段载荷边界、配置 blob、beacon 参数或解码出的 IOC 字段，优先用 `$competition-malware-config`。
- 若任务具体涉及固件分区、启动链、解出的文件系统或升级包信任边界，优先用 `$competition-firmware-layout`。
- 若任务具体涉及上传解析、预览、压缩包解压、转换器或反序列化链，优先用 `$competition-file-parser-chain`。
- 若任务具体涉及 source map、产出的 bundle、chunk 注册表，或从线上前端产物重建隐藏运行时结构，优先用 `$competition-bundle-sourcemap-recovery`。
- 若任务具体涉及容器到宿主机的边界穿越、内核利用前置条件、namespace 或 cgroup 穿越，或逃逸原语验证，优先用 `$competition-kernel-container-escape`。
- 若任务具体涉及从抓包中重建协议、流或传输的产物，优先用 `$competition-pcap-protocol`。
- 若任务具体涉及自定义二进制或文本协议、且真正的阻碍是重放状态、消息顺序或校验和逻辑，优先用 `$competition-custom-protocol-replay`。
- 若任务具体涉及跨 EVTX、PCAP、注册表、邮件或磁盘产物重建时间顺序，优先用 `$competition-forensic-timeline`。

## 需要保留的内容

- 偏移、哈希、节区名、导入表、配置 blob、mutex、注册表键
- 崩溃偏移、寄存器、堆或栈形态、泄漏地址与协议步骤
- 原始、解码、dump 与插桩后的产物，分别作为独立文件保存
