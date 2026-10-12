---
name: competition-zip-archive
description: ctf-sandbox-orchestrator 的内部下游技能。面向 ZIP 与 PKZIP 压缩包题目、传统 ZipCrypto 识别、使用 bkcrack 做已知明文恢复、基于密钥的解密与可复现解压的 CTF 沙箱工作流。当用户要求解出加密 ZIP 题目、检查 ZipCrypto 元数据、从已知文件前缀恢复密钥，或在不以口令暴力破解为起点的情况下解锁压缩包时使用。仅在 `$ctf-sandbox-orchestrator` 已建立沙箱假设并路由到此处之后使用。
---

# 竞赛 ZIP 压缩包

仅当 `$ctf-sandbox-orchestrator` 已激活、并已确立沙箱假设、证据优先级与分析项目根目录之后，才把本技能作为下游专项使用。若这些前提尚未成立，先回到 `$ctf-sandbox-orchestrator`。

当决定性路径是加密的 ZIP/PKZIP 压缩包，而不是上传解析器或通用加密数据块时，使用本技能。当题目给出可预测的文件、格式头、模板或其他可恢复明文时，优先走传统 ZipCrypto 的已知明文路径。不要以盲目口令暴力破解开始。

除非用户明确要求英文，否则用简体中文回复。命令与工具输出保持原样。

## 快速开始

1. 留存原始压缩包、计算哈希，并在分析项目的 `work/<case>/` 目录下操作副本。
2. 在尝试口令攻击之前，先确认实际的压缩包格式并列出条目。
3. 判断该条目是否使用传统 ZipCrypto。`bkcrack` 无法恢复 WinZip AES 或其他现代加密。
4. 构造精确的已知明文候选。该攻击至少需要 12 个已知明文字节，其中至少 8 个字节连续。
5. 用 `bkcrack` 恢复内部密钥，然后生成未加密副本并解压。
6. 把命令、条目名、已知明文来源、恢复出的密钥、输出哈希与最终 flag 作为证据保留。

## 工具准备

在猜测可执行文件位置之前，先走常规 tool index 与 bootstrap 路径：

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File skills/scripts/refresh-tool-index.ps1
powershell -NoProfile -ExecutionPolicy Bypass -File skills/scripts/bootstrap-reverse.ps1 -Capability bkcrack
```

在 Kali 上，使用等价的能力命令：

```bash
bash kali/scripts/bootstrap-reverse.sh bkcrack
```

Windows 能力固定在 `v1.8.1` 发行版，并会校验 GitHub asset 摘要。若手动安装该工具，之后需要刷新 `skills/tool-index.md`。

## 工作流

### 1. 确定压缩包真相

保持原始文件不可变，并记录：

```bash
sha256sum challenge.zip
file challenge.zip
bkcrack -L challenge.zip
```

在 Windows 上，用 `Get-FileHash` 代替 `sha256sum`。`bkcrack -L` 会显示条目名、压缩方法与加密状态。不要仅凭 `.zip` 扩展名推断加密类型。

检查将用于提供已知明文的那个条目。有用的候选是 stored（未压缩）或其他可预测的文件，例如 PNG、PDF、文本模板，或题目生成的配置。本地 ZIP 头（`PK\x03\x04`）是该压缩包条目的元数据，不是加密成员内部的明文，因此它本身并不是有用的已知明文样本。

### 2. 用已知明文恢复密钥

当密文条目是 `flag.txt`、且 `known.zip` 中存在匹配的明文条目时：

```bash
bkcrack -C challenge.zip -c flag.txt -P known.zip -p flag.txt
```

对于原始密文与明文文件：

```bash
bkcrack -c cipherfile -p plainfile
```

明文必须与加密条目中所表示的字节一致。若该条目经过 deflate 压缩，未压缩的副本并不会自动是正确的输入；应使用匹配的 ZIP 夹具或精确的压缩字节。

若已知字节从某个偏移开始，添加 `-o <offset>`。若只有 8-11 个字节连续，可用稀疏提示把它们与其他已知字节组合：

```bash
bkcrack -c cipherfile -p plainfile -x 25 4b4f -x 30 21
```

成功运行会得到三个内部 ZipCrypto 密钥。按打印结果原样记录；它们不是原始口令。

### 3. 解锁并验证

用恢复出的密钥生成新的压缩包，保持源文件不被改动：

```bash
bkcrack -C challenge.zip -k K0 K1 K2 -D unlocked.zip
7z t unlocked.zip
7z x unlocked.zip -ounpacked
```

把 `K0 K1 K2` 替换为 `bkcrack` 打印出的十六进制值。用压缩包测试命令以及哈希或精确 flag 比对来验证输出。若只需要某一个原始成员，可用 `-d` 写出其解密后的字节；deflate 压缩的原始数据可能需要 bkcrack 附带的 `inflate.py` 辅助脚本。

### 4. 前置条件不满足时改道

- WinZip AES 或其他现代加密模式：停止本路径，识别该题特有的原语。
- 没有可靠的已知明文：先检查文件名、元数据、压缩方式选择、题目源码与其他条目，再考虑口令恢复。
- 已知明文短于要求：寻找更多连续字节，或使用有证据支撑的稀疏偏移。
- 问题属于应用上传/解析链：交给 `$competition-file-parser-chain`。
- 解压之后问题属于通用密文或自定义密码：交给 `$competition-crypto-mobile`。

## 需要保留的证据

- 原始与工作副本路径，以及 SHA-256 哈希
- `bkcrack -L` 输出与被选中的加密成员
- 精确的明文夹具、压缩方法、偏移与稀疏字节提示
- 密钥恢复命令与恢复出的三个内部密钥
- `unlocked.zip` 的验证输出、解压路径与最终产物哈希

决策表与证据检查清单见 `references/zip-archive.md`。
