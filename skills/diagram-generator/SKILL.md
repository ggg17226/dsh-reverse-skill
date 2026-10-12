---
name: diagram-generator
description: 从自然语言、笔记、代码片段、schema、表格或既有图表源码生成、优化、校验并渲染图表。适用于流程图、泳道图、时序图、状态图、ER 图、类图、架构/C4 风格图、依赖图、甘特图、思维导图、用户旅程图、桑基风格流量图、组织架构图、网络图等可视化模型。默认支持 Mermaid，复杂图布局用 Graphviz DOT，UML 密集的工程图用 PlantUML，直接写标记更可靠时输出 SVG。
---

# 图表生成器

## ACTION REQUIRED（读完后立刻执行）

1. `NOW`：确认当前任务是否命中本 skill 的适用范围
2. `NOW`：读取 `../tool-index.md`，校验工具可用性和实际路径
3. `NEXT`：缺工具时调用 bootstrap，不要猜路径
4. `ACT`：进入“工作流”第一步并执行，不要停在确认状态

## 目的

把杂乱或结构化的输入变成清晰、可编辑的图表。优先产出基于文本的图表源码，以便结果可被审查、纳入版本管理并继续打磨。只有当用户要求图片/PDF，或可下载的产物确有实质帮助时，才渲染成文件。

## 默认工作流

1. 明确用户意图、受众与素材来源。
2. 用下面的决策表选择图表族与绘图语言。
3. 在编写图表代码之前，先归一化实体、关系、标签、状态、分支以及时间/顺序信息。
4. 生成简洁、可读的图表源码。
5. 先在心里校验语法；需要创建文件时运行 `scripts/render_diagram.py`。
6. 返回图表源码，并附一段说明所做假设的简短备注。生成了文件时，附上输出文件链接。

不要过度追问澄清。若请求信息不足，做出合理假设并简要标注。

## 图表语言决策表

除非其他语言明显更合适，否则使用 Mermaid。

| 用户想要 | 优先使用 | 原因 |
|---|---|---|
| 流程、决策树、简单泳道 | Mermaid flowchart | 可读性好，便于粘贴进 Markdown |
| 系统与用户的交互序列 | Mermaid sequenceDiagram 或 PlantUML sequence | 文档用 Mermaid；需要 UML 形式化时用 PlantUML |
| 生命周期、状态机、状态转移 | Mermaid stateDiagram-v2 或 PlantUML state | 状态转移语法紧凑 |
| 数据库 schema、实体、关系 | Mermaid erDiagram | ER 记法可移植 |
| 类/接口/对象模型 | Mermaid classDiagram 或 PlantUML class | 文档用 Mermaid；详细 UML 用 PlantUML |
| 项目排期 | Mermaid gantt | 时间线语法简洁 |
| 层级、想法、笔记 | Mermaid mindmap | 想法梳理的合适默认选择 |
| 客户/产品旅程 | Mermaid journey | 内置 journey 记法 |
| git 历史 | Mermaid gitGraph | 内置 git 记法 |
| 依赖图、包图、大规模网络 | Graphviz DOT | 对稠密图有更好的布局引擎 |
| 带分层、集群、边界的架构图 | Mermaid flowchart 配 subgraph、Graphviz cluster 或 PlantUML C4 风格 | 按所需的保真度选择 |
| 带权重的流量/类桑基关系 | 支持时用 Mermaid sankey-beta，否则用 SVG 或 Graphviz | Mermaid 的支持程度因渲染器而异 |
| 源码语言难以表达的自定义视觉 | SVG | 可精确控制布局与样式 |

## 输出策略

- 除非用户明确只要图片，否则始终提供可编辑的源码。
- 默认只给一张最合适的图。只有在确实有用时才提供备选方案。
- 优先使用稳定、简单的语法，而不是在旧版 Mermaid/PlantUML 中可能无法渲染的花哨特性。
- 标签要短。必要时把长文本拆成图表外的注释。
- 避免含糊的节点 ID。使用 ASCII ID 加人类可读标签。
- 保留用户的术语，但在同一张图内统一大小写。
- 技术类图表中，隐含存在时要把客户端、服务、数据库、队列、外部 API、运维/用户等边界画出来。
- 业务流程图中，存在时要把正常路径、决策点、失败、重试与人工步骤区分开。
- 由不确定文本生成的图，在代码后附一节 `Assumptions`。

## Mermaid 生成规则

紧凑模板见 `references/diagram-patterns.md`。

Mermaid 通用规则：
- 以正确的图表指令开头，例如 `flowchart TD`、`sequenceDiagram`、`erDiagram`、`gantt`、`mindmap` 或 `journey`。
- 流程图除非用户要求从左到右，否则用 `flowchart TD`；架构图与流水线用 `flowchart LR`。
- 泳道或架构分层用 subgraph。给 subgraph 起可读的名字。
- 节点 ID 保持稳定且仅用 ASCII，例如 `ingest_service[Ingest Service]`。
- 含可能干扰解析器的标点时，给标签加引号。
- 分支用决策菱形：`decision{Condition?}`。
- 边标签保持一致：`-- yes -->`、`-- no -->`、`-. async .->`，或确有含义时用 `== critical ==>`。
- 时序图中先声明 participant 再写消息。人用 `actor`，系统用 `participant`。
- 条件、可选、循环与并行流程分别用 `alt/else/end`、`opt/end`、`loop/end` 与 `par/and/end` 块。

## Graphviz DOT 生成规则

大规模、稠密或对布局敏感的关系图使用 Graphviz。

- 有向关系优先用 `digraph G`，无向网络用 `graph G`。
- 在顶部设置便于布局的图属性：`rankdir=LR`、`nodesep`、`ranksep`，有帮助时加 `splines=true`。
- 用 `subgraph cluster_name` 表示边界与子系统。
- 使用朴素标签与克制的样式。
- 只有当边标签能增加含义时才使用。
- 节点很多时，用 cluster 按领域分组，避免交叉密集的全连接边。

## PlantUML 生成规则

当用户要求 UML 或需要正式 UML 记法时使用 PlantUML。

- 图表用 `@startuml` 与 `@enduml` 包裹。
- 需要时使用 `actor`、`participant`、`database`、`queue`、`collections` 或 `component` 构造型。
- 架构边界用 `package`、`rectangle` 或 `node`。
- 类图除非用户要求穷尽细节，否则只列重要字段/方法。
- 活动图使用清晰的开始/结束标记与明确的分支标签。

## SVG 生成规则

只有当文本图表语言无法可靠表达所需视觉时，才使用 SVG。

- 保持 SVG 简单、可访问、可编辑。
- 包含 `<title>` 与有意义的文本标签。
- 优先使用矩形、线条、箭头与分组，而不是复杂路径。
- 不要内嵌外部字体或远程图片。

## 渲染文件

当用户要求 PNG/SVG/PDF 时，创建源文件并运行：

```bash
python "<SKILL_ROOT>/diagram-generator/scripts/render_diagram.py" input.mmd --format svg --out output.svg
python "<SKILL_ROOT>/diagram-generator/scripts/render_diagram.py" input.dot --format png --out output.png
python "<SKILL_ROOT>/diagram-generator/scripts/render_diagram.py" input.puml --format svg --out output.svg
```

> `<SKILL_ROOT>` 是本包 `skills/` 目录的实际路径，AI 应自动检测。

该渲染器刻意对依赖保持宽容。它会尝试常见的本地工具，若渲染器不可用则给出可执行的安装提示。除非脚本成功执行且输出文件确实存在，否则不要声称已渲染出图片。

## 校验清单

定稿之前：

- 图表类型与用户任务匹配。
- 源码对所选用语言而言语法合理。
- 标签足够短，能放得下。
- 边与消息顺序准确反映输入。
- 输入不完整时已标注所做假设。
- 生成文件时，输出确实存在且能打开或大小非零。

## 常用回复模板

多数图表回答都使用这个结构：

```markdown
下面是可编辑的 [language] 版本：

```[language]
[source]
```

Assumptions:
- [only if needed]

Rendered file: [link] [only if generated]
```

英文提问用英文回答。中文提问用中文回答，除非用户另有要求。

---

## 按需自举（On-Demand Bootstrap）

### 自动化能力边界

| 工具 | 可自动安装 | 安装方式 | 说明 |
|------|-----------|---------|------|
| Mermaid CLI (mmdc) | ✓ | npm install -g @mermaid-js/mermaid-cli | 渲染 Mermaid 为 PNG/SVG |
| Graphviz (dot) | ✗ | 手动安装 | https://graphviz.org/download/ |
| PlantUML | ✗ | 需要 Java + plantuml.jar | https://plantuml.com/download |
| Python (render script) | ✓ | 已在 bootstrap 中 | `scripts/render_diagram.py` 依赖 |

### 说明

本 skill 主要输出文本格式的图表源码（Mermaid/DOT/PlantUML），不一定需要本地渲染工具。只有当用户明确要求生成 PNG/SVG/PDF 文件时才需要对应的渲染器。

如果渲染器不可用，`scripts/render_diagram.py` 会输出安装提示而不是报错。

---

## 路由上下文

**上游入口**: `skills/SKILL.md`（总控）、`routing.md`
**触发条件**: 用户说“画图”、“流程图”、“架构图”、“攻击路径图”、“时序图”、“Mermaid”、“Graphviz”、“PlantUML”
**下游出口**:
- 生成的图表可嵌入 `docs-generator/` 的报告中
- 攻击路径图可配合 `pentest-tools/` 的渗透报告

**同级关联模块**: `docs-generator/`（报告中嵌入图表）


## 任务完成自检（声称完成前 MUST 通过）

- [ ] 我是否执行了工作流中的每一步（而不是只阅读）？
- [ ] 我是否基于 `tool-index` 使用了真实工具路径？
- [ ] 我是否产出了可复现证据（命令/脚本/截图/报告）？
- [ ] 我是否完成并回写了 RULES 要求的 Checklist 项？
