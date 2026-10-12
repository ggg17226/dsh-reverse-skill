---
name: case-review
description: 在报告交接之前，审查 reverse-skill 的案件包是否具备 scope 就绪性、Evidence 到 Finding 到 Path 的可追溯性、work item 覆盖度、时间线引用，以及可选的产物哈希完整性。
---

# Evidence 图审查

当逆向工程、取证、CTF 或授权安全案件需要可辩护的交接时，使用本技能。它审计既有的 `work/<case>/` 案件包，不改变案件，也不接触目标。

## 范围

本技能覆盖：

- scope 元数据与目标活动就绪性
- Evidence 记录结构与可复现性字段
- work item 与时间线条目对 Evidence 的引用
- 报告 Markdown 中结构化的 Finding 与 Path
- 对案件内产物的可选 SHA-256 校验
- 用于报告交接的 Markdown 或 JSON 审查结果

它 MUST NOT 进行侦察、漏洞利用、动态插桩或改动目标。那些动作属于被路由到的分析技能，并且需要案件 scope 门。

## ACTION REQUIRED（读完后立刻执行）

1. `NOW`：读取 `../field-journal/precedent-reverse.md`，确认本次是对既有授权案件包的审查。
2. `NOW`：确认案件路径，并选择只读审查模式。
3. `NEXT`：读取 `../tool-index.md`；本技能只使用 Python 3 标准库，不需要 bootstrap。
4. `NEXT`：运行 `python3 scripts/review_case.py <case-root> --format markdown`。
5. `ACT`：解决每一个错误，然后重新运行审查，之后才能声称交接完成。

## 工具依赖

| 工具 | 是否必需 | 用途 | 可否自动自举 |
|------|----------|---------|---------------|
| Python 3.9+ | 是 | 运行只读的案件审查脚本 | 否，使用平台自带的 Python |

不需要网络访问，也不需要第三方包。

## 工作流

### 阶段 1：受理（Intake）

对既有案件目录运行审查：

```bash
python3 skills/case-review/scripts/review_case.py work/<case> --format markdown
```

确认 `scope.md`、`timeline.md`、`workitems.md` 与 `evidence/` 均存在。非严格审查只报告 scope 警告，而严格审查会把警告视为交接阻塞项。

## 建议下一步（选一个编号）

1. 修复 scope.md 中的授权、范围或 network_profile 字段
2. 继续检查 Evidence 记录的可复现命令和来源
3. 导出当前 review 结果并附到阶段性报告
4. 换 JSON 输出接入 CI 或其他审查工具
5. 暂停，先确认审查范围

### 阶段 2：可追溯性（Traceability）

审查以下检查项：

- 引用了不存在的 Evidence ID
- Finding 缺少 `evidence_ids`
- Path 使用了不允许的 `path_type`，或缺少 Evidence 引用
- work item 与时间线条目指向未知的 Evidence
- 未被任何记录引用的 Evidence
- 已标记为 Validated 但置信度很低的 Finding

离线观察只有在 `notes` 字段明确记录了该离线限制时，才可使用 `repro_command: n/a`。

当其他工具需要稳定字段时使用 JSON：

```bash
python3 skills/case-review/scripts/review_case.py work/<case> --format json
```

## 建议下一步（选一个编号）

1. 补写缺失的 Evidence，并保留原始命令
2. 将候选 Finding 绑定到 Evidence 后重新审查
3. 为调用链或攻击链补充 P-id 和 Path 步骤
4. 生成 Markdown handoff summary
5. 换回 PRIMARY skill 继续分析

### 阶段 3：固定性校验（Fixity verification）

当某条 Evidence 记录同时包含 `content_hash` 与 `artifact_path` 时，校验案件内的产物：

```bash
python3 skills/case-review/scripts/review_case.py work/<case> --verify-hashes --strict
```

脚本接受 `sha256:<64 位十六进制字符>`，并检查该产物仍位于案件根目录之内。哈希不匹配属于硬失败。

PowerShell 的 Evidence 辅助脚本可以在追加记录的同时记录哈希：

```powershell
powershell -File skills/scripts/append-evidence.ps1 -CaseRoot work\<case> -Id E-001 -Title "Sample hash" -ReproCommand "sha256sum evidence/sample.bin" -ArtifactPath "evidence\sample.bin"
```

## 建议下一步（选一个编号）

1. 修复 hash mismatch 或替换已污染的工作副本
2. 为未固定的原始文件补充 SHA-256 和 artifact_path
3. 继续进入报告生成阶段
4. 导出 JSON 结果供 CI 保存
5. 暂停并请求人工复核

### 阶段 4：交接（Handoff）

在最终报告或向专项交接之前使用严格模式：

```bash
python3 skills/case-review/scripts/review_case.py work/<case> --strict --format markdown > work/<case>/report/case-review.md
```

除非显式使用 shell 重定向保存输出，否则该命令对案件而言是只读的。本审查不构成法律意见，也不替代组织自身的证据处理流程。

## 建议下一步（选一个编号）

1. 将通过的 review 结果交给 `docs-generator/` 生成正式报告
2. 回到 PRIMARY skill 补齐新的分析证据
3. 归档 Markdown 和 JSON review 结果
4. 暂停并请求人工复核

## 语言行为约定

- 内部推理、工具选择与阶段控制：英文。
- 用户可见消息、章节标签、报告与下一步菜单：中文，除非用户要求其他语言。
- 默认双语标签为中文在前、英文在后，用 `/` 分隔。

## 自举边界

本技能没有第三方依赖。若 Python 3 不可用，唯一允许的补救动作是在当前平台已登记 Python 能力时走仓库的 bootstrap 路径。若未登记此类能力，就停止并报告缺失的运行时。不要猜可执行文件路径、不要下载包，也不要从本技能内部执行手动安装。

## 路由上下文

**上游入口**：任何已经产出案件包的逆向、取证、CTF 或授权安全技能。

**下游出口**：`docs-generator/` 用于生成正式报告；当证据图不完整时回到原来的 PRIMARY skill。

**关联模块**：`ops/evidence-finding-path.md`、`ops/timeline-workitem.md`、`digital-forensics/`、`reverse-engineering/` 与 `docs-generator/`。

## 参考资料

- [NIST SP 800-86: Guide to Integrating Forensic Techniques into Incident Response](https://csrc.nist.gov/pubs/sp/800/86/final)
- [SWGDE Best Practices for Computer Forensic Acquisitions](https://www.swgde.org/documents/published-complete-listing/17-f-002-2-1/)
- [SWGDE Best Practices for Archiving Digital and Multimedia Evidence](https://www.swgde.org/documents/published-complete-listing/19-f-003-best-practices-for-archiving-digital-and-multimedia-evidence/)

## 任务完成自检

- [ ] 我是否审查了 scope.md、timeline.md、workitems.md 和 evidence/？
- [ ] 所有 Finding 是否引用了现存 Evidence？
- [ ] 所有 Path 是否包含合法 path_type 和 Evidence 引用？
- [ ] 是否执行了 hash verification，或记录了未执行原因？
- [ ] 是否以 strict 模式重新运行并保存了 review 结果？
