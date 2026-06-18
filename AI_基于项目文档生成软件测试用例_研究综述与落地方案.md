# AI 基于项目文档生成软件测试用例：研究综述与落地方案

日期：2026-06-16  
角色视角：软件测试工程师 + AI 研究科学家

## 摘要

基于近期论文和工程实践证据，LLM 可以有效辅助从项目文档、SRS、用户故事、接口说明和代码上下文中生成测试用例，尤其适合快速形成系统测试/验收测试的初稿、发现遗漏场景、生成边界/异常/权限/状态类补充用例。但当前研究也很明确：它不适合“无监督全自动产出最终测试资产”。复杂条件、隐含业务规则、跨文档冲突、缺失上下文和测试 oracle 错误，都会导致 AI 生成无依据、冗余或错误的用例。

因此，推荐的落地路线不是“一个提示词生成全部测试用例”，而是：

> 文档证据库 + 原子需求卡片 + 测试设计技术选择 + 分步生成 + 证据校验 + 覆盖矩阵 + 多模型/多轮审查 + 人工确认。

这个方案的核心目标是：提高生成效率和覆盖面，同时把幻觉限制在可发现、可追踪、可拒绝的范围内。

## 1. 论文与研究证据

### 1.1 从需求规格生成系统测试用例是可行的，但要分步

Bhatia 等人在 *System Test Case Design from Requirements Specifications* 中使用五个真实交付过的软件工程项目 SRS，采用 prompt chaining：先让模型理解 SRS，再按 use case 逐个生成测试设计。论文报告约 87% 生成用例有效，约 15% 的有效用例是开发团队此前没有考虑到的。这说明 LLM 的价值不仅是提速，也能补漏；但仍有 13% 不适用或冗余，需要审查。  
来源：https://arxiv.org/html/2412.03693v1

### 1.2 高层测试用例更适合作为第一阶段产物

Hasan 等人在 *Automatic High-Level Test Case Generation using Large Language Models* 中强调，实践中的主要痛点往往不是写脚本，而是测试工作与业务需求对齐。高层测试用例用于说明“测试什么”和“预期行为是什么”，更适合作为详细脚本之前的桥梁。  
来源：https://arxiv.org/abs/2503.17998

Masuda 等人在 *Generating High-Level Test Cases from Requirements using LLM: An Industry Study* 中提出 GHL 方法：先根据需求文档生成适用的测试设计技术，再基于每种技术生成高层测试用例。实验表明该方式在部分数据集上有不错召回，但不同项目差异明显，也指出 RAG 对每个业务领域定制成本较高。  
来源：https://arxiv.org/html/2510.03641v1

### 1.3 复杂需求会显著降低准确性

Scania 汽车软件需求场景的研究使用 GPT-4o 从真实汽车需求生成黑盒测试，评价指标包括 coverage、completeness、correctness、precision。结果显示约 63% 的需求文档可生成可用测试套件，简单需求效果更好；复杂条件逻辑会带来不一致和错误，理想场景可大幅提速，但实际仍需要人工复核。  
来源：https://www.diva-portal.org/smash/get/diva2%3A1986692/FULLTEXT01.pdf

另一项黑盒需求测试研究也指出，ChatGPT 生成测试套件覆盖率较高，但会出现代码错误、冗余和持续误解某些指令的问题；更详细的提示可以降低失败/冗余比例，但不能替代人工监督。  
来源：https://www.diva-portal.org/smash/get/diva2%3A1917788/FULLTEXT01.pdf

### 1.4 LLM 测试生成的通用结论：有潜力，但 oracle 与覆盖仍是难点

综述论文 *Software Testing with Large Language Models* 总结，LLM 已被用于单元测试生成、test oracle 生成、系统测试输入生成、调试和修复，但挑战仍包括高覆盖率、oracle 正确性、严格评估和真实场景应用。  
来源：https://www.eecs.yorku.ca/~wangsong/papers/LLM4Test.pdf

MDPI 的综述 *A Review of Large Language Models for Automated Test Case Generation* 将方法归类为 prompt engineering、feedback-driven、fine-tuning/pre-training、hybrid approaches，并指出虽然覆盖、可用性和正确性有提升，但存在性能不稳定、编译错误等问题。  
来源：https://www.mdpi.com/2504-4990/7/3/97

### 1.5 幻觉来源：需求冲突、上下文缺失、检索失败和生成失败

*LLM Hallucinations in Practical Code Generation* 指出，实际代码生成中最常见的幻觉类型是 task requirement conflicts，并将原因归结为训练数据质量、意图理解能力、知识获取能力和仓库级上下文意识不足。论文也验证了基于仓库检索的 RAG 能作为轻量缓解方式。  
来源：https://arxiv.org/html/2409.20550v1

经典 RAG 论文 *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks* 提出用参数化记忆 + 外部非参数记忆结合，以改善知识密集任务中的事实性、可更新性和来源追踪。  
来源：https://arxiv.org/abs/2005.11401

但 RAG 本身也会幻觉。*Hallucination Mitigation for Retrieval-Augmented Large Language Models* 将 RAG 幻觉分为检索失败和生成失败：数据源低质/过期、查询歧义、检索策略不足、上下文噪声、上下文冲突等都会造成错误。  
来源：https://www.mdpi.com/2227-7390/13/5/856

### 1.6 可借鉴的幻觉抑制方法

Self-consistency 通过采样多条推理路径，再选择一致性最高的答案，提高复杂推理任务表现。测试生成中可转化为“多次独立生成 + 聚类/投票 + 差异审查”。  
来源：https://arxiv.org/abs/2203.11171

SelfCheckGPT 利用同一提示下多个样本之间的一致性检测幻觉：如果模型真正掌握事实，多次回答应较一致；如果是幻觉，回答更容易分歧。  
来源：https://arxiv.org/abs/2303.08896

Chain-of-Verification 先生成草稿，再生成验证问题并独立回答，最后修订答案。测试用例生成中可转化为“每条用例都必须回答：它来自哪条需求？预期结果是否有证据？是否与其他规则冲突？”  
来源：https://www.research-collection.ethz.ch/server/api/core/bitstreams/468e77de-b21f-4ede-b179-8a52b01a1c5a/content

## 2. 总体结论

1. AI 生成测试用例最可靠的入口是“高层测试用例/测试设计”，不是直接生成自动化脚本。
2. 单轮提示词不可控，必须采用 prompt chaining 或 agent workflow。
3. RAG 有价值，但不是万能。必须保证文档源质量、分块粒度、检索召回、冲突处理和引用追踪。
4. 所有测试用例都必须落到需求证据、代码证据或人工确认之一。没有证据的预期结果只能标为“待确认”，不能进入正式测试集。
5. 减少幻觉的关键不是让模型“更聪明”，而是让流程具备拒绝、追踪、校验和复核能力。

## 3. 可落地架构

### 3.1 输入资产

建议收集以下项目材料：

| 类型 | 示例 | 用途 |
|---|---|---|
| 需求文档 | PRD、SRS、用户故事、验收标准 | 生成功能测试与验收测试 |
| 设计文档 | 架构设计、概要设计、接口设计 | 识别模块边界、依赖和异常路径 |
| 接口文档 | OpenAPI、报文规范、字段表 | 生成接口测试、字段校验、兼容性测试 |
| 业务规则 | 权限矩阵、流程图、状态机、计算公式 | 生成决策表、状态迁移、边界测试 |
| 历史资产 | 已有测试用例、缺陷单、生产问题、变更记录 | 回归测试、风险补强、缺陷模式补漏 |
| 代码与配置 | Controller、Service、SQL、配置、定时任务 | 校验真实行为、生成单元/集成测试 |

### 3.2 中间资产

不要直接从文档到测试用例。中间必须有四类结构化资产：

1. `evidence_index.jsonl`：文档证据索引，包含来源文件、章节、段落、页码、哈希、原文片段。
2. `requirement_cards.yaml`：原子需求卡片，包含 actor、trigger、precondition、business rule、input、output、exception、source。
3. `coverage_model.xlsx`：覆盖模型，按功能、角色、状态、字段、接口、异常、权限、数据、非功能维度展开。
4. `test_cases.xlsx`：正式测试用例，必须引用 requirement id 和 evidence id。

推荐流水线：

```text
项目文档/代码/历史缺陷
  -> 文档解析与证据索引
  -> 原子需求卡片
  -> 测试设计技术选择
  -> 高层测试用例生成
  -> 覆盖矩阵审查
  -> 幻觉/冲突/冗余校验
  -> 人工确认
  -> 自动化脚本生成或测试管理系统入库
```

## 4. 生成流程设计

### 阶段 A：文档摄取与证据索引

目标：让模型只能在可追溯证据上工作。

处理动作：

1. 解析 PDF、Word、Excel、Markdown、接口规范、代码注释。
2. 按章节、表格、字段、流程步骤进行结构化切分。
3. 每个片段生成稳定 ID，例如 `DOC-PRD-001#sec3.2#p12`。
4. 保存原文、页码/行号、文件哈希、更新时间。
5. 对文档冲突或重复版本建立优先级，例如“最新签字版 > 当前代码 > 历史需求 > 会议纪要”。

关键规则：

> 后续任何测试用例的预期结果都必须引用至少一个证据 ID；否则标为 `Needs Confirmation`。

### 阶段 B：原子需求卡片生成

将长文档拆成小而明确的需求卡片：

| 字段 | 说明 |
|---|---|
| req_id | 稳定需求 ID |
| source_ids | 来源证据 ID |
| actor | 用户/系统/外部机构 |
| trigger | 触发动作 |
| precondition | 前置条件 |
| rule | 业务规则 |
| input | 输入字段/数据 |
| output | 输出/状态变化/提示 |
| exception | 异常或失败分支 |
| ambiguity | 歧义点 |
| confidence | 高/中/低 |

模型提示原则：

```text
只抽取文档中明确出现的需求。
不要补充业务规则。
如果文档缺少预期结果、状态变化或异常处理，写入 ambiguity。
每个字段必须引用 source_ids。
```

### 阶段 C：选择测试设计技术

让 AI 先决定“用什么测试方法”，再生成用例。推荐覆盖：

| 技术 | 适用对象 | 示例 |
|---|---|---|
| 等价类 | 输入字段、枚举、类型 | 有效/无效证件类型 |
| 边界值 | 金额、日期、长度、数量 | 最小金额、最大金额、超限 |
| 决策表 | 多条件业务规则 | 权限 + 状态 + 金额 |
| 状态迁移 | 订单、审批、支付、任务流 | 新建->提交->审核->完成 |
| 场景法 | 端到端流程 | 用户完整办理业务 |
| 错误推测 | 历史缺陷与高风险点 | 重复提交、网络超时 |
| 组合测试 | 多参数组合 | pairwise 覆盖 |
| 安全测试 | 登录、权限、敏感数据 | 越权、脱敏、注入 |
| 兼容/回归 | 升级、接口版本、迁移 | 新旧报文兼容 |
| 非功能 | 性能、可用性、审计 | 并发、日志、幂等 |

### 阶段 D：高层测试用例生成

先生成自然语言测试用例，不急着写脚本。

推荐字段：

| 字段 | 说明 |
|---|---|
| tc_id | 测试用例 ID |
| req_ids | 覆盖的需求 ID |
| evidence_ids | 证据 ID |
| feature | 功能点 |
| risk_level | P0/P1/P2 |
| test_type | 正向/反向/边界/权限/状态/接口/回归 |
| precondition | 前置条件 |
| test_data | 测试数据 |
| steps | 操作步骤 |
| expected_result | 预期结果 |
| design_technique | 测试设计方法 |
| automation_hint | 可自动化建议 |
| confidence | 高/中/低 |
| review_status | Confirmed/Needs Confirmation/Rejected |
| open_questions | 待确认问题 |

### 阶段 E：验证与反幻觉

每条测试用例生成后立即做校验，而不是等全部生成完。

校验清单：

| 校验项 | 规则 |
|---|---|
| 证据校验 | `expected_result` 必须能追溯到文档/代码/人工确认 |
| 需求覆盖 | 每个 P0/P1 需求至少有正向、反向、边界或异常覆盖 |
| 冲突检测 | 同一状态/字段/接口的预期结果不能互相矛盾 |
| 冗余检测 | 语义近似且覆盖相同 req 的用例合并 |
| 漏测检测 | 按角色、状态、字段、异常、权限、接口逐维扫描空洞 |
| 自一致性 | 同一需求独立生成 3 次，分歧点进入人工复核 |
| Chain-of-Verification | 对每条高风险用例生成验证问题并独立回答 |
| 执行校验 | 自动化脚本必须编译、运行、通过；接口测试必须能 mock 或真实调用 |
| 变更校验 | 文档或代码变更后，只重生成受影响 req 的用例 |

## 5. Skills 化方案

这里的 skills 不是让模型“记住一切”，而是把稳定流程固化成可重复执行的工作说明。

建议拆成五个 skill：

### 5.1 `project-doc-ingestion`

职责：解析项目文档并生成证据索引。

规则：

- 不总结为不可追踪的大段文字。
- 每条事实必须有文件、页码/章节/行号。
- 识别版本、冲突、重复文档。
- 输出 `evidence_index.jsonl` 和 `source_conflicts.md`。

### 5.2 `requirement-card-extractor`

职责：把证据转成原子需求卡片。

规则：

- 一张卡只表达一个可测试规则。
- 缺失信息写入 `ambiguity`。
- 不从常识补业务规则。
- 输出 `requirement_cards.yaml`。

### 5.3 `test-design-planner`

职责：为每张需求卡选择测试设计技术。

规则：

- 字段类需求必须考虑等价类和边界值。
- 多条件规则必须考虑决策表。
- 流程/状态类需求必须考虑状态迁移。
- 涉及登录、权限、金额、审批、外部接口时自动升为高风险。
- 输出 `coverage_model.xlsx` 或 `coverage_model.md`。

### 5.4 `testcase-generator`

职责：生成高层测试用例。

规则：

- 只生成可追溯到 req/evidence 的用例。
- 每条用例必须有预期结果。
- 无明确预期结果时标为 `Needs Confirmation`。
- 禁止把猜测写成确认事实。
- 输出 `test_cases.xlsx` 或 `test_cases.md`。

### 5.5 `hallucination-and-coverage-auditor`

职责：审查幻觉、冲突、重复和漏测。

规则：

- 没有 evidence 的用例拒绝进入正式集。
- 同一需求多次生成结果不一致时标红。
- 对 P0/P1 需求做覆盖空洞扫描。
- 输出 `validation_report.md`、`gap_report.md`、`open_questions.md`。

## 6. 推荐 Prompt/Agent 工作流

### 6.1 需求抽取 Prompt

```text
你是需求分析和测试设计专家。
任务：从给定证据片段中抽取原子需求卡片。

约束：
1. 只能使用输入证据，不允许补充外部常识。
2. 每条需求必须引用 source_id。
3. 如果预期结果、异常处理、状态变化缺失，写入 ambiguity。
4. 输出 YAML，字段固定为 req_id/source_ids/actor/trigger/precondition/rule/input/output/exception/ambiguity/confidence。
```

### 6.2 测试设计 Prompt

```text
你是资深测试架构师。
任务：为每条 requirement card 选择测试设计技术并说明覆盖理由。

必须考虑：
- 正向路径
- 反向路径
- 边界值
- 权限与角色
- 状态迁移
- 异常与超时
- 数据一致性
- 审计日志
- 回归影响

不要生成测试用例，只输出覆盖模型。
```

### 6.3 测试用例生成 Prompt

```text
你是高级测试工程师。
任务：基于 requirement cards 和 coverage model 生成高层测试用例。

硬性规则：
1. 每条测试用例必须引用 req_ids 和 evidence_ids。
2. expected_result 必须来自 evidence，不能猜测。
3. 找不到证据时 review_status=Needs Confirmation。
4. 不要为了数量制造重复用例。
5. 输出表格字段：tc_id, req_ids, evidence_ids, feature, risk_level, test_type, precondition, test_data, steps, expected_result, design_technique, automation_hint, confidence, review_status, open_questions。
```

### 6.4 反幻觉审查 Prompt

```text
你是测试用例审查员，不负责生成新用例。
任务：检查输入测试用例是否有幻觉、冲突、重复、漏测。

逐条判断：
1. expected_result 是否有 evidence 支撑？
2. steps 是否引入了文档未定义的页面、按钮、接口、字段？
3. test_data 是否违反字段约束？
4. 是否和其他用例/需求冲突？
5. 是否覆盖了对应测试设计技术？

输出：
- accepted_cases
- rejected_cases
- needs_confirmation
- duplicate_groups
- coverage_gaps
- questions_for_business
```

## 7. 幻觉控制策略

### 7.1 证据优先

模型回答前先检索证据，生成后反向验证证据。每条测试用例必须能回答：

- 这条用例覆盖哪个需求？
- 需求来自哪个文档、哪一页、哪一段？
- 预期结果是否在证据中出现？
- 如果没有出现，是不是已经标记为待确认？

### 7.2 分层输出

将输出分成三类：

| 类型 | 进入正式测试集 | 说明 |
|---|---|---|
| Confirmed | 可以 | 文档/代码/人工确认有证据 |
| Needs Confirmation | 不直接进入 | 可能合理，但缺少证据 |
| Rejected | 不进入 | 与证据冲突、重复或明显错误 |

### 7.3 不把 AI 生成内容重新当事实源

生成的用例、总结和解释不能直接进入 RAG 主证据库。可以进入 `generated_candidates/`，但必须经人工确认或可执行验证后，才升级为正式知识。

### 7.4 多样性生成 + 一致性过滤

对 P0/P1 需求使用多次独立生成：

```text
same requirement card
  -> generation A
  -> generation B
  -> generation C
  -> semantic clustering
  -> common cases accepted for review
  -> disagreements become open questions
```

### 7.5 Chain-of-Verification

对每条高风险用例生成验证问题：

| 验证问题 | 示例 |
|---|---|
| 证据问题 | 哪条需求说明了该预期结果？ |
| 角色问题 | 该用户是否有权限执行该动作？ |
| 状态问题 | 当前状态是否允许该迁移？ |
| 数据问题 | 测试数据是否满足字段约束？ |
| 冲突问题 | 是否与其他文档规则矛盾？ |

只有全部通过，才进入 `Confirmed`。

## 8. 评估指标

建议用以下指标判断方案是否有效：

| 指标 | 定义 | 目标建议 |
|---|---|---|
| Validity | 人工确认有效用例比例 | PoC 目标 >= 80% |
| Traceability | 有 req/evidence 引用的用例比例 | 100% |
| Unsupported Expected Result | 预期结果无证据比例 | 正式集 0% |
| Requirement Coverage | 需求被至少一条用例覆盖比例 | P0/P1 100% |
| Technique Coverage | 测试设计技术覆盖完整度 | 高风险模块 >= 90% |
| Redundancy Rate | 重复/等价用例比例 | <= 10% |
| Gap Discovery | AI 发现人工遗漏有效用例数 | 记录并复盘 |
| Review Cost | 人工审查耗时 | 比纯人工下降 |
| Automation Pass Rate | 生成脚本编译/运行通过比例 | 按项目逐步提高 |
| Defect Detection | 回归/探索中发现缺陷数 | 与历史基线对比 |

不要只用“生成了多少条用例”作为成功标准。数量越大，幻觉和冗余通常也越多。

## 9. 两周 PoC 实施计划

### 第 1-2 天：选择试点模块

选择一个边界清楚、文档较完整、风险较高但规模可控的模块，例如登录、权限、支付、审批、报文接口、数据迁移。

产出：

- `source_inventory.md`
- `evidence_index.jsonl`
- `source_conflicts.md`

### 第 3-4 天：抽取需求卡片

产出：

- `requirement_cards.yaml`
- `ambiguity_list.md`

人工快速审查需求卡片，先修正源头，不急着生成测试。

### 第 5-6 天：生成覆盖模型

产出：

- `coverage_model.md`
- `risk_matrix.md`

重点确认 P0/P1、状态、权限、异常、边界、接口兼容性。

### 第 7-8 天：生成高层测试用例

产出：

- `test_cases_draft.md`
- `open_questions.md`

只生成高层用例，不直接写自动化脚本。

### 第 9-10 天：审查与反幻觉

产出：

- `validation_report.md`
- `gap_report.md`
- `rejected_cases.md`

人工确认后形成第一版正式测试集。

### 第 11-14 天：自动化和度量

对 Confirmed 且稳定的用例生成接口测试、单元测试或 E2E 脚本。

产出：

- `test_cases_v1.xlsx`
- `traceability_matrix.xlsx`
- `automation_candidates.md`
- `poc_metrics.md`

## 10. 推荐落地目录

```text
ai-testcase-workbench/
  sources/
    docs/
    api/
    code_refs/
    defects/
  parsed/
    evidence_index.jsonl
    source_conflicts.md
  requirements/
    requirement_cards.yaml
    ambiguity_list.md
  design/
    coverage_model.md
    risk_matrix.md
  generated/
    test_cases_draft.md
    generated_candidates/
  validation/
    validation_report.md
    gap_report.md
    rejected_cases.md
    open_questions.md
  approved/
    test_cases_v1.xlsx
    traceability_matrix.xlsx
  automation/
    api/
    unit/
    e2e/
```

## 11. 人机分工

| 角色 | AI 负责 | 人负责 |
|---|---|---|
| 需求分析 | 抽取、归类、发现歧义 | 确认业务含义和优先级 |
| 测试设计 | 推荐技术、生成覆盖矩阵 | 判断风险和测试策略 |
| 用例生成 | 生成初稿、补充边界/异常 | 确认预期结果 |
| 反幻觉 | 证据校验、冲突检测、重复检测 | 处理业务冲突和未定义规则 |
| 自动化 | 生成脚本初稿、修复编译错误 | 维护框架和执行环境 |
| 度量 | 统计覆盖、缺口、有效率 | 决定发布准入标准 |

## 12. 最终建议

建议把 AI 测试用例生成定位为“测试设计增强系统”，而不是“自动测试工程师替代品”。第一阶段只做高层测试用例和覆盖补漏；第二阶段再生成可执行脚本；第三阶段通过历史缺陷、生产问题、代码覆盖、变更影响分析持续增强。

最小可行版本必须具备三条底线：

1. 每条测试用例有来源证据。
2. 每条预期结果可验证。
3. 每个 AI 输出都有审查状态。

如果这三条做不到，AI 生成越多，测试资产债务越重；如果做到，AI 会成为很好的测试“放大器”：提速、补漏、统一格式，并让需求歧义更早暴露。



---

# 第二部分：优化版完整落地实施方案（基于首次 PoC 复盘）

> 本部分新增于 2026-06-18。第一部分（1–12 节）是方法论与计划；本部分基于首次 PoC——深圳前海联合交易中心《现货竞买和竞卖交易业务细则》（QME）——的实际执行复盘，对方法论给出可执行的优化，并整理成端到端的完整落地实施方案。第一部分的五条结论与底线（高层用例优先、prompt chaining、证据优先、分层输出、人工确认）全部保留；本部分只做四件事：**做实（可执行）、做诚实（指标）、可治理（产物）、可对标（金标准）**。

## 13. 首次 PoC 复盘与优化总览

首次 PoC 按第 4 节 A–E 阶段完整跑通：产出 47 条证据、41 张需求卡、66 条高层测试点，可追溯性 100%、需求覆盖 100%、反幻觉纪律到位（显式拒绝 5 类无依据写法）。这验证了"方法论可行"这一核心命题。

但把产物放到真实交付标准下、并与既有人工用例库（CTCS 竞价模块 1,363 条 / 全库 17,666 条）对照后，暴露出 7 个必须修正的差距：

| 编号 | 差距 | 现象 | 对应优化（本部分） |
|---|---|---|---|
| G1 | 用例不可执行 | 预期多为复述规则，缺步骤/数据/可判定 oracle | §14 两层用例模型 + 系统输入注入 |
| G2 | 指标误导 | 突出"100% 覆盖"，而 Validity 仍"待人工评审" | §17 分层指标体系（诚实口径） |
| G3 | 系统测试面覆盖低 | 66 规则点 vs 同业务 1,363 条真实用例 | §14 可执行化 + §16 对标基准 |
| G4 | 32% Needs Confirmation | NC 实为开放问题，却计入"已生成用例" | §15 输入闸门 + NC 生命周期 |
| G5 | 产物 / ID 混乱 | QME、QH 两套 ID 并存，4+ 份重叠 Excel、残留锁文件 | §18 产物治理与版本规范 |
| G6 | 无金标准对标 | 手握 1,363 条人工用例却未做基准 | §16 对标既有用例库 |
| G7 | 单输入封顶质量 | 仅 1 份 PDF，外部管理办法/公告未接入 | §15 输入充分性闸门 |

优化主线一句话：**把"对单一文档的规则抽取"升级为"对系统的、可执行、可对标、可治理的测试设计"**，并让指标只讲能被验证的真话。

## 14. 两层用例模型与系统输入注入（解决 G1 / G3）

第 4 节阶段 D 的"高层测试用例"在实践中会退化成"换句话说的规则"。根因是客观的：业务规则文档本身不含系统页面、字段、接口，因此只能产出规则级测试点。解决办法不是让模型更聪明，而是**把用例显式分成两层，并在两层之间插入"系统输入注入"步骤**。

| 层 | 名称 | 来源 | 稳定性 | 是否可执行 | 典型字段 |
|---|---|---|---|---|---|
| L1 | 规则测试点 | 需求卡 + 证据 | 高（随规则变） | 否 | req_ids、evidence_ids、rule_point、oracle_intent |
| L2 | 可执行用例 | L1 + 系统输入资产 | 中（随系统变） | 是 | steps、test_data、expected_result(oracle)、system_refs |

L1 即当前 PoC 的产物，保留其可追溯价值；L2 由 L1 注入"系统输入资产"实例化得到。

**系统输入资产（必须由项目方提供，是质量天花板的钥匙）：**

| 资产 | 内容 | 解锁的用例能力 |
|---|---|---|
| 字段清单 | 场次创建/公告/报价表单的字段、类型、约束、枚举 | 等价类/边界值的具体取值与断言 |
| 公告模板 | 必填项、价格/数量/时间要素 | 字段完整性、模板条件必填 |
| 页面与 API 地图 | 入口路径、按钮、接口、状态码 | 具体 steps 与接口断言 |
| 配置与规则参数 | 担保/费率/成交原则/降幅/时间间隔的取值来源 | 数值型 oracle、计算校验 |

**注入前后对照（同一条加价竞买规则，TC-QME-012）：**

L1 规则测试点（注入前，即当前 PoC 产物）：

```text
预期：加价竞买应以初始价为起点，由低至高按最小加价幅度整倍数加价回应。（复述规则，不可判定）
```

L2 可执行用例（注入"报价表单字段：初始价 / 最小加价幅度 / 最大加价幅度"后）：

```text
前置：单价加价竞买场次已开始，初始价=100，最小加价幅度=5，最大加价幅度=20
步骤：进入"报价报量"页面 -> 提交报价 -> 提交数量
测试数据：报价=105（一倍步长，正向）；报价=103（非整倍数，反向）；报价=121（超最大幅度，边界）
预期(oracle)：105 接受；103 拒绝并提示"须为最小加价幅度的整数倍"；121 拒绝（>初始价+最大幅度）
```

L2 的粒度即对齐 CTCS 真实用例（"报价值 ≤ 初始价＋最大价格变动幅度，增量为最小变动幅度的整数倍"）。

**用例 schema 扩展（在第 4 节阶段 D 字段基础上新增）：** `layer`(L1/L2)、`system_refs`(字段/页面/接口 ID)、`oracle_type`(枚举/边界/状态/计算/权限)、`executability`(可执行/需输入/不可执行)。

## 15. 输入充分性闸门与 Needs-Confirmation 生命周期（解决 G4 / G7）

PoC 有 32% 用例停在 Needs Confirmation，且只有一份 PDF，导致担保金额、费率、结算交收等只能"引用而不能测"。修正：在生成前加一道**输入充分性闸门**，在生成后给 NC 一个**受管理的生命周期**。

**输入充分性闸门（阶段 B 之后、C 之前）：** 对每张需求卡判定证据充分度并据此路由——

| 充分度 | 判据 | 路由 |
|---|---|---|
| 完整 | 规则、输入、输出/oracle 均有证据 | 直接生成 L2 可执行用例 |
| 部分 | 规则清楚但缺系统字段/取值 | 生成 L1 并挂"待注入"，补输入后实例化为 L2 |
| 外部依赖 | 明确 defer 到外部办法/公告 | **不生成实例用例**，转业务输入待办（BA backlog） |

这样 NC 不再混进"已生成用例"，而是被前置拦截或显式转成待办。

**Needs-Confirmation 生命周期（NC 是待办，不是交付物）：**

```text
raised（含 owner / raised_date / SLA）
  -> answered          -> 升级为 Confirmed（补证据或补输入后生成 L2）
  -> unanswerable      -> Rejected 或 转外部依赖待办
```

NC 字段补充：`owner`、`raised_date`、`due`、`blocking_inputs`、`exit_status`。

**交付口径（修订第 8 节）：** "可交付用例"分母 = Confirmed 用例；NC 单独统计为待澄清项，不计入覆盖率的分子或分母。

## 16. 对标既有用例库：金标准基准（解决 G6）

这是 PoC 最有说服力、却恰恰缺失的一步：用既有人工用例库做基准。项目已有 CTCS 竞价模块 1,363 条真实用例，应作为 gold standard 量化 AI 的真实价值。

**三个对标指标：**

| 指标 | 定义 | 回答的问题 |
|---|---|---|
| 召回率 Recall | AI 重现的人工用例 / 同范围人工用例 | AI 能顶多少现有人工用例 |
| 新增有效率 Novelty | AI 提出且评审接受的新用例 / AI 用例 | AI 补了多少漏（呼应论文 ~15% 补漏结论） |
| 评审通过率 Precision | 评审接受的 AI 用例 / AI 全部用例 | AI 产物的"含金量" |

**方法：** 先按功能点/规则把 AI 用例与人工用例语义对齐（人工或半自动），再抽样评审；范围须可比（同一业务子域、同一版本）。注意 CTCS 是既有系统历史库、QME 是新规则，二者非一一对应，应在"可比子集"上计算，不做全库相减。

**产出：** `benchmark/gold_mapping.xlsx`（AI ↔ 人工用例映射）、`benchmark/recall_report.md`（三指标 + 漏项 / 新增项清单）。这份报告是给管理层的核心证据。

## 17. 分层指标体系（诚实口径，修订第 8 节）

第 8 节的指标本身合理（Validity 目标 ≥ 80%、Traceability 100%），问题出在 PoC 汇报时把"100% 覆盖 / 可追溯"当成了成熟度结论，而真正的 Validity 还标着"待人工评审"。修正：把指标显式分两层，并定一条汇报红线。

| 层 | 指标 | 采集 | 能否单独作为成熟度结论 |
|---|---|---|---|
| 一层：过程 / 一致性 | Traceability、Unsupported-ER、文档需求引用率、Redundancy | 自动 | 否（只证明流程自洽） |
| 二层：结果 / 有效性 | Validity、系统覆盖度(=对标 Recall)、Technique Coverage、Gap Discovery、Review Cost、Automation Pass、Defect Detection | 人工 / 系统 / 对标 | 是（才是真正价值） |

**汇报红线：** 不得在缺少 Validity 与 系统覆盖度(Recall) 的情况下，单独用一层指标的"100%"对外汇报或作为通过标准。一层指标命名也应去歧义——例如把"Requirement Coverage"改称"文档需求引用率"，避免被读成"系统已覆盖"。

## 18. 产物治理与版本规范（解决 G5）

PoC 出现 QME / QH 两套需求/证据/用例 ID 并存、`test_cases_v1` 与 `merged` / `merged_v2` 等 4+ 份重叠 Excel、以及残留锁文件，没有单一事实来源。规范如下：

- **单一 ID 命名空间：** 一个项目一套前缀（如 `QME-REQ` / `QME-E` / `QME-TC`），禁止并存第二套；跨运行 ID 稳定、只增不改写。
- **单一权威产物：** `/approved` 只保留一份当前权威工作簿；历史与中间态一律放 `/generated` 或 `/archive`，不进 approved。
- **运行清单 run manifest：** 每次生成记录 `run_id`、源文件及哈希、模型、prompt 版本、时间、产出文件，便于复现与审计。
- **不可变证据库：** `evidence_index` 一旦建立，不被生成内容回写（呼应 7.3）。
- **卫生规范：** 提交前清理 `~$` 锁文件与临时文件；命名含版本与日期；用一份 README 指明"哪份是权威"。

## 19. 团队交付 schema 映射与优先级对齐（解决 G6 的规范侧）

AI 的追溯型 schema 与公司 CTCS 列不一致，评审需心算对照。做法：**双 schema 并存，交付时映射到团队列并叠加追溯列。**

| 内部追溯字段（保留） | 团队交付列（CTCS） |
|---|---|
| feature | 所属模块 / 功能点 |
| precondition | 前置条件 |
| steps | 测试步骤 |
| expected_result | 预期 |
| test_type / design_technique | 案例类型 |
| risk_level | 优先级（见下方映射） |
| req_ids、evidence_ids | （追加为追溯列，置于末列） |

**优先级对齐：** 内部 P0/P1/P2 → 团队 1/2/3/4（建议 P0→1、P1→2、P2→3，4 留给纯 UI / 查询类）；并补齐 CTCS 的"案例类型"标签（核心流程 / 异常分支 / 权限控制类 / 边界值 / 并发 / 安全要求等运营视角）。最终交付 Excel = 团队列 + 末尾追溯列，既可直接评审入库，又不丢可追溯性。

## 20. 落地实施路线图（阶段闸门 P0–P5）

把第 9 节的"两周 PoC"扩展为带准入 / 退出闸门的完整路线；其中 Phase 0–2 可直接复用已完成的两周 PoC 资产。

| 阶段 | 目标 | 关键动作 | 退出标准 | 工作量 |
|---|---|---|---|---|
| P0 治理基线 | 单一事实来源 | 统一 ID、收敛 approved、run manifest、清理锁文件 | 仅一套 ID、一份权威产物 | 小 |
| P1 指标诚实化 | 正确预期 | 指标分层、改名、定汇报红线、NC 出分母 | 看板含两层且标注口径 | 小 |
| P2 对标金标准 | 证明价值 | 与 CTCS 可比子集做 Recall / Novelty / Precision | recall_report 通过评审 | 中 |
| P3 用例可执行化 | 可被执行 | 接入系统输入资产，L1→L2 实例化；NC 进闸门 | 抽样 L2 可被 QA 直接执行 | 大 |
| P4 规范化输出 | 可直接入库 | 映射到 CTCS 列 + 追溯列，单份交付 Excel | 评审一次性接收 | 中 |
| P5 自动化试点 | 真实通过率 | 选约 10 条可断言 P0 写可执行断言 | 跑出真实 Pass Rate | 中 |

闸门原则：每阶段未达退出标准不进入下一阶段；P3 以"系统输入资产是否到位"为前置（呼应 §15）。

## 21. 更新的目录、人机分工与最终建议（增量）

**目录结构（在第 10 节基础上新增 3 个目录）：**

```text
ai-testcase-workbench/
  sources/ ...                # 同前
  system_inputs/              # 新增：系统输入资产（解锁 L2 可执行用例）
    field_catalog.md          #   字段清单 / 枚举 / 约束
    announcement_template.md  #   公告模板必填项
    page_api_map.md           #   页面入口与接口地图
  parsed/ requirements/ design/ generated/ validation/   # 同前
  benchmark/                  # 新增：金标准对标
    gold_mapping.xlsx         #   AI <-> 人工用例映射
    recall_report.md          #   Recall / Novelty / Precision
  governance/                 # 新增：治理
    run_manifest.jsonl        #   每次运行的可复现记录
    README.md                 #   指明权威产物
  approved/
    test_cases_v1.xlsx        # 仅一份权威（团队列 + 追溯列）
  automation/ ...             # 同前
```

**人机分工（在第 11 节基础上新增三项）：**

| 活动 | AI 负责 | 人负责（owner） |
|---|---|---|
| 输入充分性闸门 | 标注证据充分度、列出待注入 / 外部依赖 | BA 确认并补系统输入 |
| 金标准对标 | 语义对齐、初算三指标 | QA Lead 评审 Recall / Novelty 判定 |
| 产物治理 | 生成 run manifest、查重 | 测试经理裁定权威产物 |

**最终建议（修订第 12 节）：** 定位仍是"测试设计增强系统"。在原三条底线（每条用例有证据、每个预期可验证、每个输出有审查状态）之上，再加三条来自 PoC 复盘的工程底线：

1. **输入闸门**：系统输入未到位的需求只产 L1，不假装可执行。
2. **金标准对标**：用既有人工用例库量化 Recall / Novelty 作为价值证据，而非自报"100%"。
3. **单一事实来源**：一套 ID、一份权威产物、可复现的 run manifest。

做到这六条，AI 才是可信赖的测试"放大器"——提速、补漏、统一格式、让需求歧义更早暴露；做不到，则生成越多，测试资产债务越重。
