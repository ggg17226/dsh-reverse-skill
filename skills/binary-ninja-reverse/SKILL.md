---
name: binary-ninja-reverse
description: 用于在 Binary Ninja 中进行授权二进制分析，包括 HLIL/MLIL/LLIL 检查、字符串/导入/导出、交叉引用、类型、补丁审查、Python API 自动化，以及可选的 Binary Ninja MCP 或 localhost HTTP 集成。
---

# Binary Ninja 逆向工程

当用户明确选择 Binary Ninja、其各类 IL 对数据流分析确有实质帮助，或需要独立交叉验证 IDA/Ghidra/radare2 的结论时，使用 Binary Ninja。

## 安全起步

1. 对目标动手之前，先确认仓库的 case scope 已就绪。
2. 在 `skills/tool-index.md` 中检查 `binaryninja`；Binary Ninja 是商业软件，必须在持有有效 Vector 35 许可的情况下手动安装。
3. 打补丁或保存数据库改动时，在副本上操作。
4. 在把发现提升为结论之前，先记录导入/导出、入口点、架构与文件哈希。

## 选择集成方式

- **GUI 或 Python API**：当 Binary Ninja 已经打开，或用户希望直接交互式分析时优先使用。
- **社区 MCP 桥**：仅在被明确要求、且已审查过第三方插件边界之后使用。让 Binary Ninja 的 HTTP 监听保持在 `127.0.0.1:9009`；默认不要开启网络暴露。
- **回退方案**：当 Binary Ninja 不可用，或许可证/API 无法打开目标时，改用 `ghidra-reverse`、`ida-reverse` 或 `radare2`。

经审查的社区集成是 [`fosdickio/binary_ninja_mcp`](https://github.com/fosdickio/binary_ninja_mcp)，GPL-3.0，插件元数据版本 `1.1.0`，最低 Binary Ninja 构建号 `4000`。该仓库不是 Vector 35 的官方组件。本技能已于 2026-09-03 针对 commit `8c5134ee46e2bf44f9a4d846bd971c3e39b3e306` 核对。

Binary Ninja 侧可通过其 Plugin Manager 或从已审查的源码安装。对于 MCP stdio 桥，请固定已发布的桥版本，而不要使用无版本约束的包：

```text
npx -y binary-ninja-mcp@1.0.0 --host 127.0.0.1 --port 9009
```

只在用户选定的 MCP 客户端中注册该命令。只有当 Binary Ninja 正在运行、已打开某个二进制、且 localhost 插件端点有响应时，该桥才算就绪。

## 分析工作流

1. 枚举已打开的二进制并选择目标视图。
2. 捕获二进制状态、入口点、段、导入、导出与有代表性的字符串。
3. 在孤立解释某个函数之前，先追踪调用点与交叉引用。
4. 用 HLIL 看可读逻辑，用 MLIL SSA 看数据流，当 lift 丢失指令级行为时使用 LLIL 或反汇编。
5. 增量施加名称、注释与类型；在证据中保留原始地址。
6. 把字节补丁、原型修改与保存文件写入视为改动。仅在被要求时执行，并保留原始产物。
7. 对高影响结论，用第二个证据来源或另一款反汇编器交叉验证。

有用的 MCP 能力族包括二进制与视图选择、`list_imports`、`list_exports`、`list_strings`、`decompile_function`、`get_il`、调用者/被调用者、交叉引用、类型、注释、重命名与字节打补丁。应实时发现可用工具列表，而不要假定上游每个函数都存在。

## 输出

报告具体地址、函数名、IL 层级、支撑用的字符串/导入、置信度与复现步骤。保持仓库其余部分沿用的 Evidence → Finding → Path 链条。
