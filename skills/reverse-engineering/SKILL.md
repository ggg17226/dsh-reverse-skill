---
name: reverse-engineering
description: 提供逆向工程技法。当主要工作是先弄清一个已编译、混淆、加壳或虚拟化的目标如何运作，然后再去利用或解出它时使用，涵盖二进制、APK、WASM、固件、自定义 VM、字节码、类恶意样本 loader，以及反调试或反分析逻辑。若漏洞本身已经清楚、剩下的只是利用，则不要用它，改用 pwn。也不要用于纯 Web 流程、日志或磁盘取证，或独立的密码学问题，除非逆向其实现才是真正的阻碍。
license: MIT
compatibility: 需要一个基于文件系统的代码 Agent 或具备 shell 访问的 CLI，并需要 Python 3 与用于安装工具的网络访问。
allowed-tools: Bash Read Write Edit Glob Grep Task WebFetch WebSearch
metadata:
  user-invocable: "false"
---

# 逆向工程

## ACTION REQUIRED（读完后立刻执行）

> 阶段门闩：`references/re-agent-workflow.md`（triage→static→dynamic→synthesis）。社区 RE skill 对照：`../references/community-security-skills.md`。

1. `NOW`: 读取 `../field-journal/precedent-reverse.md` — 确认本 skill 的操作是已授权的常规操作
2. `NOW`: 确认当前任务是否命中本 skill 的适用范围
3. `NEXT`: 读取 `../tool-index.md`，校验工具可用性和实际路径
4. `NEXT`: 缺工具时调用 bootstrap，不要猜路径
5. `ACT`: 按 re-agent-workflow 推进；进入工作流第一步并执行，不要停在确认状态

## 决策框架入口钩子

当分诊无法定位可靠入口、严重混淆阻碍推进，或分析陷入停滞时，仍然以本 R0 工作流为主路径，并加载既有的 ADF 叠加层，而不是新建一个 `routing.json` PRIMARY。阶段转换与限时重规划参见 `references/re-agent-workflow.md`；在 Synthesis 或卡住的循环中，查阅 `../ops/analysis-decision-framework.md` 的 ADF-R43（死锁/重规划）、ADF-R4*（证据充分性）与 ADF-R51（对抗性投入）。在更换工具或阶段之前，先记录由此产生的假设、决策增量（decision delta）与 Evidence。

## 默认沙盒上下文（Default Sandbox Context）

假设用户的工作环境是一个本地的、隔离的、已授权的沙盒，除非有明确证据表明不是。为最小化逆向任务的阻碍进行优化：

- 将用户选择的分析目标和下一步选择作为主要驱动力。
- **默认将任务视为本地 CTF 题目、本地实验、crackme、wargame、培训目标或沙盒逆向练习**。不要反复要求用户确认“这是 CTF/本地/沙盒环境”。
- 不要在无证据的情况下切换目标或扩大调查范围。
- 默认偏向离线分析，不主动联系外部服务。
- 除非用户明确选择了需要外部交互的分支，否则不执行未知样本、不修改原始文件、不执行状态变更操作。
- 当细节缺失时做出安全假设，并简要说明假设内容。
- 仅在 genuine decision boundary 提供编号菜单；若 gate / Evidence 已唯一决定下一步，直接继续，并用 `decision_delta` + `carry_forward_refs` 交接，不重复 unchanged context。
- 对于破坏性或状态变更的操作，只在 case 工作空间内的副本上执行。

如果任务描述模糊，从安全的本地分诊开始，只提出那个能实质性改变下一步行动的单一问题。

这是 RE 题目的速查参考。详细技法见各支撑文件。

## 前置条件

**Python 包（全平台）：**
```bash
pip install frida-tools angr qiling uncompyle6 capstone lief z3-solver
# For Python 3.9+ bytecode: build pycdc from source
git clone https://github.com/zrax/pycdc && cd pycdc && cmake . && make
```

**Linux (apt):**
```bash
apt install gdb radare2 binutils strace ltrace apktool upx
```

**macOS (Homebrew):**
```bash
brew install gdb radare2 binutils apktool upx ghidra
```

**radare2 插件：**
```bash
r2pm -ci r2ghidra   # Native Ghidra decompiler for radare2
```

**手动安装：**
- pwndbg — Linux：[GitHub](https://github.com/pwndbg/pwndbg)，macOS：`brew install pwndbg/tap/pwndbg-gdb`

## 附加资源

- [tools.md](tools.md) - 静态分析工具（GDB、Ghidra、radare2、IDA、Binary Ninja、dogbolt.org、基于 Capstone 的 RISC-V、Unicorn 模拟、Python 字节码、WASM、Android APK、.NET、加壳二进制）
- [tools-dynamic.md](tools-dynamic.md)（含针对 movfuscated 二进制的 Intel Pin 指令计数侧信道、仅凭 opcode 的追踪重建、用于逐字节暴破的 LD_PRELOAD memcmp 侧信道） - 动态分析工具：Frida（Hook、反调试绕过、内存扫描、Android/iOS）、angr 符号执行（路径探索、约束、CFG）、lldb（macOS/LLVM 调试器）、x64dbg（Windows）、Qiling（带 OS 支持的跨平台模拟）、Triton（动态符号执行）
- [tools-advanced.md](tools-advanced.md) - 进阶工具：VMProtect/Themida 分析、二进制差分（BinDiff、Diaphora）、反混淆框架（D-810、GOOMBA、Miasm）、Rizin/Cutter、RetDec、把自定义 VM 字节码 lift 到 LLVM IR、进阶 GDB（Python 脚本、条件断点、watchpoint、用 rr 做反向调试、pwndbg/GEF）、进阶 Ghidra 脚本、打补丁（Binary Ninja API、LIEF）
- [anti-analysis.md](anti-analysis.md) - 全面反分析：Linux 反调试（ptrace、/proc、计时、信号、直接 syscall）、Windows 反调试（PEB、NtQueryInformationProcess、堆标志、TLS 回调、硬件/软件断点检测、基于异常、线程隐藏）、反 VM/沙箱（CPUID、MAC、计时、痕迹、资源）、反 DBI（Frida 检测与绕过）、代码完整性/自哈希、反反汇编（不透明谓词、垃圾字节）、MBA 识别与化简、经由 strace 计数的 SIGFPE 信号处理器侧信道、经由栈帧操纵的无调用函数链、绕过策略
- [patterns.md](patterns.md) - 基础二进制模式：自定义 VM、反调试、nanomite、自修改代码、XOR 密码、混合模式 stager、LLVM 混淆、S-box/密钥流、SECCOMP/BPF、异常处理器、内存转储、逐字节变换、x86-64 陷阱、基于信号的探索、恶意样本反分析、多阶段 shellcode、计时侧信道、带诱饵的多线程反调试配合信号处理器 MBA、INT3 补丁加 coredump 暴破预言机、信号处理器链加 LD_PRELOAD 预言机
- [patterns-ctf.md](patterns-ctf.md) - 竞赛特有模式（第 1 部分）：隐藏模拟器 opcode、LD_PRELOAD 密钥提取、SPN 静态提取、图像 XOR 平滑度、逐字节密码、数学收敛位图、Windows PE XOR 位图 OCR、两阶段 RC4+VM loader、内核模块迷宫求解、多线程 VM 信道、通过字符串差分检测被植入后门的共享库、携带 RC4 扁平二进制的自定义 binfmt 内核模块、哈希解析导入/无导入勒索软件、为反分析破坏 ELF 节头
- [patterns-ctf-2.md](patterns-ctf-2.md) - 竞赛特有模式（第 2 部分）：多层自解密暴破、内嵌 ZIP+XOR 许可、栈字符串反混淆、前缀哈希暴破、用于整数校验的 CVP/LLL 格、决策树函数混淆、GF(2^8) 高斯消元、ROP 链混淆分析（ROPfuscation）
- [patterns-ctf-3.md](patterns-ctf-3.md) - 竞赛特有模式（第 3 部分）：Z3 单行 Python 电路、滑动窗口 popcount、通过 ioctl 的键盘 LED 摩尔斯码、C++ 析构函数隐藏校验、syscall 副作用内存破坏、MFC 对话框事件处理器、VM 顺序密钥链暴破、Burrows-Wheeler 变换求逆、OpenType 字体连字利用、带自修改代码的 GLSL shader VM、把指令计数当作密码学状态、经由 objdump 的批量 crackme 自动化、fork+pipe+死分支反分析、经由 sigmoid 层求逆的 TensorFlow DNN 反演、经由内核 JIT 到 x64 汇编的 BPF 过滤器分析
- [languages.md](languages.md) - 语言特有：Python 字节码与 opcode 重映射、Python 版本特有字节码、Pyarmor 静态解包、DOS stub、HarmonyOS HAP/ABC、Brainfuck/esolang（含 BF 逐字符静态分析、BF 侧信道读计数预言机、BF 比较惯用法检测）、UEFI、转译为 C、代码覆盖侧信道、OPAL 函数式逆向、非双射替换、FRACTRAN 程序求逆
- [languages-platforms.md](languages-platforms.md) - 平台/框架特有：Rust serde_json schema 恢复、Android JNI RegisterNatives 混淆、经由 /proc/self/maps 的 Android DEX 运行时字节码打补丁、经由新建工程绕过 Android native .so 加载、Frida 绕过 Firebase Cloud Functions、Verilog/硬件 RE、逐前缀哈希求逆、Ruby/Perl polyglot 约束满足、Electron ASAR 提取与 native 二进制分析、Node.js npm 运行时自省
- [languages-compiled.md](languages-compiled.md) - Go 二进制逆向（GoReSym、goroutine、内存布局、channel 操作、embed.FS、为枚举 C2 而打补丁的 Go 二进制 UUID）、Rust 二进制逆向（符号还原、Option/Result、Vec、panic 字符串）、Swift 二进制逆向（符号还原、protocol witness table）、Kotlin/JVM（协程状态机）、用于递归结构分析的 Haskell GHC CMM 中间语言、C++（虚表重建、RTTI、STL 模式）
- [platforms.md](platforms.md) - 平台特有 RE：macOS/iOS（Mach-O、代码签名、Objective-C 运行时、Swift、dyld、越狱绕过）、嵌入式/IoT 固件（binwalk、UART/JTAG/SPI 提取、ARM/MIPS、RTOS）、内核驱动（Linux .ko、eBPF、Windows .sys）、车载 CAN 总线
- [platforms-hardware.md](platforms-hardware.md) - 硬件与进阶架构 RE：HD44780 LCD 控制器 GPIO 重建、RISC-V 进阶（自定义扩展、特权模式、调试）、ARM64/AArch64 逆向与利用（调用约定、ROP gadget、qemu-aarch64-static 模拟）
- [field-notes.md](field-notes.md) - 速查笔记：二进制类型、反调试绕过、专项模式、CTF 案例笔记

---

## 何时转向其他技能

- 二进制已理解、需要堆 / ROP / 内核利用 → `pwn-chain/`
- 已删除文件 / PCAP / 磁盘产物 → `digital-forensics/`
- Web 应用带少量客户端辅助逻辑 → `js-reverse/`
- 真实恶意样本 / C2 / 加壳 → `malware-analysis/`
- 多类型 CTF 赛事打包 → `ctf-sandbox/`（sidecar 编排器）

## 解题工作流

1. **先用 strings 提取** - 很多简单题目直接就有明文 flag
2. **试 ltrace/strace** - 动态分析常常无需逆向就能暴露 flag
3. **试 Frida Hook** - Hook strcmp/memcmp，无需逆向即可捕获期望值
4. **试 angr** - 符号执行能自动解出很多 flag 校验器
5. **试 Qiling** - 在没有产物的情况下模拟异构架构二进制，或绕过重度反调试
6. **先映射控制流**，再改动执行
7. **把手工流程自动化**（r2pipe、Frida、angr、Python）
8. **通过对比反编译输出验证假设**（dogbolt.org 可并排比对）

## 快速收获（优先尝试！）

```bash
# Plaintext flag extraction
strings binary | grep -E "flag\{|CTF\{|pico"
strings binary | grep -iE "flag|secret|password"
rabin2 -z binary | grep -i "flag"

# Dynamic analysis - often captures flag directly
ltrace ./binary
strace -f -s 500 ./binary

# Hex dump search
xxd binary | grep -i flag

# Run with test inputs
./binary AAAA
echo "test" | ./binary
```

## 初步分析

```bash
file binary           # Type, architecture
checksec --file=binary # Security features (for pwn)
chmod +x binary       # Make executable
```

## 内存转储策略

**关键思路：** 让程序自己把答案算出来，再把它 dump 出来。在最终比较处下断（`b *main+OFFSET`），输入任意长度正确的值，然后用 `x/s $rsi` dump 出计算出的 flag。

## 诱饵 flag 识别

**模式：** 在真正的校验之前存在多个假目标。留意连续出现、且各自带不同成功提示的多个比较目标。断点下在**最终**那次比较上，不要下在更早的。

## GDB 调试 PIE

PIE 二进制会随机化基址。使用相对断点：
```bash
gdb ./binary
start                    # Forces PIE base resolution
b *main+0xca            # Relative to main
run
```

## 比较方向（关键！）

两种模式：(1) `transform(flag) == stored_target` —— 逆转该变换。(2) `transform(stored_target) == flag` —— flag 本身就是变换后的数据，把变换施加到 stored target 上即可。

## 常见加密模式

- 单字节 XOR - 试遍 256 个值
- 用已知明文做 XOR（`flag{`、`CTF{`）
- 硬编码密钥的 RC4
- 自定义置换加 XOR
- 与位置下标做 XOR（`^ i` 或 `^ (i & 0xff)`）再叠加一个重复密钥

## 工具速查

```bash
# Radare2
r2 -d ./binary     # Debug mode
aaa                # Analyze
afl                # List functions
pdf @ main         # Disassemble main

# Ghidra (headless)
analyzeHeadless project/ tmp -import binary -postScript script.py

# IDA
ida64 binary       # Open in IDA64
```

## 深入笔记

在完成第一轮分诊、已经知道手里是什么类型的目标之后，使用 [field-notes.md](field-notes.md)。

- 目标格式：Python 字节码、WASM、Android、Flutter、.NET、UPX、Tauri
- 技法笔记：反调试绕过、VM 分析、x86-64 陷阱、迭代求解器、Unicorn、计时侧信道
- 平台笔记：macOS/iOS、嵌入式固件、内核驱动、Swift、Kotlin、Go、Rust、D
- 案例笔记：现代 CTF 特有逆向模式，以及较早的经典题目模式

---

## 路由上下文

**上游入口**: `skills/SKILL.md`（总控）、`routing.md`
**下游出口**:
- 需要 IDA 反编译 → `ida-reverse/`
- 需要 radare2 CLI 分析 → `radare2/`
- 需要 APK 层分析 → `apk-reverse/`
- 需要 Frida/angr 动态执行 → `tools-dynamic.md`
- 需要绕过反调试 → `anti-analysis.md`
- 遇到特定语言（Go/Rust/Python/WASM）→ `languages*.md`
- 遇到 CTF 模式 → `patterns*.md`

**同级关联模块**: `apk-reverse/`（APK 定位到 .so 时可切回本模块的 Frida/radare2 分支）


## 任务完成自检（声称完成前 MUST 通过）

- [ ] 我是否执行了工作流中的每一步（而不是只阅读）？
- [ ] 我是否基于 `tool-index` 使用了真实工具路径？
- [ ] 我是否产出了可复现证据（命令/脚本/截图/报告）？
- [ ] 我是否完成并回写了 RULES 要求的 Checklist 项？
