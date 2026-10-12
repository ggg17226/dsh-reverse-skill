---
name: competition-bundle-sourcemap-recovery
description: ctf-sandbox-orchestrator 的内部下游技能。面向 source map、构建清单、chunk 注册表、产物 bundle、混淆 loader 流程与前端运行时还原的 CTF 沙箱工作流。当用户要求重建线上 JavaScript 结构、检查 source map 或 chunk map、追踪 bundle 加载过程、从产物中还原隐藏路由或 API，或基于构建后的前端产物解释运行时行为时使用。仅在 `$ctf-sandbox-orchestrator` 已建立沙箱假设并路由到此处之后使用。
---

# 竞赛 Bundle 与 Sourcemap 还原

仅当 `$ctf-sandbox-orchestrator` 已激活、并已确立沙箱假设、节点归属与证据优先级之后，才把本技能作为下游专项使用。若这些前提尚未成立，先回到 `$ctf-sandbox-orchestrator`。

当运行时的真相存在于构建产物、source map、chunk 表或混淆 loader 流程中，而不仅存在于签入的源码里时，使用本技能。

除非用户明确要求英文，否则用简体中文回复。

## 快速开始

1. 从线上提供的产物集合入手：入口 HTML、构建清单、bootstrap bundle、chunk map 与 source map。
2. 在开始大范围人工反混淆之前，先记录 chunk id、路由 chunk、loader 函数、端点字符串与配置键。
3. 重建能解释「当前到底哪个产物在运行」的最小运行时图。
4. 除非已证明两边一致，否则把线上产物的真相与仓库源码分开看待。
5. 复现能证明决定性行为的最小「产物 → 运行时」边界。

## 工作流

### 1. 梳理线上产物集合

- 记录入口 HTML、script 标签、preload 提示、manifest 文件、asset map、chunk 注册表与 source map URL。
- 记录框架特有的产物，例如路由 manifest、client reference manifest 或懒加载表（若存在）。
- 让产出的文件名、hash 后缀与路由归属保持一一对应。

### 2. 重建运行时结构

- 顺着 bootstrap 代码、chunk loader、模块注册表、字符串解码器与懒加载边界追踪。
- 利用 source map、manifest 文件与稳定的符号簇，还原路由名、API 调用、功能开关与隐藏面板。
- 把「构建时的意图」与「当前实际被提供的 bundle」区分开。

### 3. 收敛到决定性 bundle 路径

- 把结论压缩为最小序列：线上产物 → loader 路径 → 模块或符号 → 运行时效果。
- 明确说明决定性弱点位于 manifest 漂移、chunk 加载、隐藏路由代码、字符串解码，还是过时的源码假设。
- 若任务从构建产物转向 SSR 或模板渲染约束，交回更贴合的 template-render 技能。

## 阅读本参考

- 加载 `references/bundle-sourcemap-recovery.md`，获取产物检查清单、反混淆检查清单与证据打包方式。

## 需要保留的内容

- 线上文件名、chunk id、manifest 条目、source map 路径、还原出的符号与端点字符串
- 能证明该运行时分支的确切执行 bundle 或模块
- 一条能到达决定性效果的最小「产物 → 运行时」序列
