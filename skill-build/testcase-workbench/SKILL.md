---
name: testcase-workbench
description: >-
  End-to-end, anti-hallucination workflow for turning project documents (PRD, SRS, 业务规则/业务细则, 接口文档, 用户故事, 验收标准) into traceable, reviewable software test cases. Use this skill whenever the user wants to generate, design, audit, or improve test cases or test scenarios from requirements or specifications — including phrases like 测试用例, 用例生成, 从需求/文档生成用例, test case generation, test design, coverage analysis, or benchmarking AI-generated cases against an existing test library. It runs a staged pipeline (evidence index, requirement cards, coverage model, two-layer cases, validation) with strict traceability, honest two-tier metrics, gold-standard benchmarking, and single-source-of-truth governance. Generic skeleton plus a trading/finance adaptation layer. Prefer this skill over ad-hoc one-shot generation whenever project documents are the input and reviewable, non-hallucinated test cases are the goal.
license: For internal/team use.
---

# Testcase Workbench

把项目文档转化为**可追溯、可评审、尽量可执行**的测试用例。这个 skill 的目的不是"一个提示词生成全部用例"，而是把生成过程拆成有证据、有闸门、有审查、可对标、可治理的流水线，让幻觉被限制在**可发现、可追踪、可拒绝**的范围内。

它是一个**通用骨架**：核心流程与任何软件项目无关；对交易/金融/交易所类系统，额外加载 `references/finance.md` 适配层。

## 何时使用

输入是需求/规格/业务文档、目标是成体系的测试用例或测试设计时使用。典型触发：从 PRD/SRS/业务细则/接口文档生成用例、做测试设计与覆盖分析、评审或改进已有 AI 用例、把 AI 用例与既有用例库对标。

## 六条底线（不可妥协）

任何一条做不到，生成越多、测试资产债务越重。开始前先认这六条：

1. **有证据**：每条用例可追溯到至少一个证据 ID（文档条款/页码、代码、或人工确认）。
2. **可验证**：每条预期结果可判定真假；做不到就别写成确认事实。
3. **有审查状态**：每条用例标 `Confirmed / Needs Confirmation / Rejected`。
4. **输入闸门**：系统输入（字段/页面/接口/取值）未到位的需求只产 L1 抽象测试点，不假装可执行（见"两层模型"）。
5. **金标准对标**：有既有用例库时，用 Recall/Novelty/Precision 量化价值，而不是自报"100% 覆盖"。
6. **单一事实来源**：一个项目一套 ID 前缀、一份权威产物、可复现的 run manifest。

## 核心概念：两层用例模型 + 系统输入注入

业务规则文档不含系统页面/字段/接口，只能产出"规则级测试点"。强行让它"可执行"就会编造页面与按钮。解决办法是**显式分两层**，并在两层之间插入"系统输入注入"：

| 层 | 名称 | 来源 | 可执行 | 用途 |
|---|---|---|---|---|
| L1 | 规则测试点 | 需求卡 + 证据 | 否 | 稳定、可追溯，覆盖"测什么、预期意图" |
| L2 | 可执行用例 | L1 + 系统输入资产 | 是 | 含具体步骤/数据/可判定 oracle |

只有当项目方提供**系统输入资产**（字段清单、公告/表单模板、页面与 API 地图、配置与规则参数）时，才把 L1 实例化为 L2。否则停在 L1 并登记"待注入"。详见 `references/pipeline.md`。

## 工作流水线（A–G）

| 阶段 | 动作 | 关键产物 | 闸门 |
|---|---|---|---|
| A 摄取 | 解析文档，按条/款/字段切分，建证据索引（含哈希、页码、原文） | `evidence_index.jsonl`、`source_conflicts.md` | 证据不可被生成内容回写 |
| B 需求卡 | 抽原子需求卡（一卡一规则），标歧义 | `requirement_cards.yaml`、`ambiguity_list.md` | **输入充分性闸门**（见下） |
| C 测试设计 | 为每张卡选测试技术，定风险等级与覆盖维度 | `coverage_model.md`、`risk_matrix.md` | 资金/权限/合规自动升 P0 |
| D 用例生成 | 先 L1 规则测试点；有系统输入再注入为 L2 | `test_cases_draft.md` | 无证据→Needs Confirmation |
| E 验证 | 逐条反幻觉、冲突、冗余、漏测、自一致性、CoVe | `validation_report.md`、`gap_report.md` | NC 不进交付集 |
| F 对标 | 与既有用例库做 Recall/Novelty/Precision | `benchmark/recall_report.md` | 有既有库时必做 |
| G 交付 | 映射到团队规范列 + 追溯列，单份权威产物 | `approved/test_cases_v1.xlsx`、`run_manifest.jsonl` | 单一事实来源 |

每阶段的详细做法、字段与示例见 `references/pipeline.md`；现成 prompt 见 `references/prompts.md`。

## 新项目接入清单（每个项目复用时照做）

这一节是"通用方案"的可复用入口。新项目按此初始化：

1. **定 ID 命名空间**：选一个项目前缀（如 `ACME`），全程用 `ACME-REQ-*`、`ACME-E-*`、`ACME-TC-*`，禁止并存第二套。
2. **盘点输入**：按 `references/pipeline.md` 的输入资产表收集文档；标注哪些是"系统输入资产"（决定能否产 L2）。
3. **选风格基准**：找到团队既有用例库的列与写法（house style），作为交付格式与对标金标准。
4. **建目录**：按 `assets/directory_layout.txt` 建工作区。
5. **套模板**：从 `assets/` 复制需求卡/覆盖/用例/验证/对标/run manifest 模板。
6. **判断领域**：若是交易/金融/交易所系统，先读 `references/finance.md`。

## 闸门与红线

- **输入充分性闸门（阶段 B 后）**：对每张需求卡判证据充分度并路由——
  - 完整（规则+输入+输出 oracle 都有证据）→ 直接产 L2。
  - 部分（规则清楚但缺系统字段/取值）→ 产 L1，挂"待注入"。
  - 外部依赖（明确 defer 到外部办法/公告/另一系统）→ **不产实例用例**，转业务输入待办。
- **Needs-Confirmation 是待办，不是交付物**：带 owner/raised_date/SLA/exit；可交付分母只算 Confirmed。
- **汇报红线**：缺少 Validity 与 系统覆盖度(对标 Recall) 时，**不得**单用 Traceability/需求引用率的"100%"对外汇报或作为通过标准。

## 何时读哪个参考文件

- `references/pipeline.md` — 每个阶段的详细做法、字段、两层注入示例、闸门。**生成前先读。**
- `references/prompts.md` — 需求抽取 / 测试设计 / 用例生成 / 反幻觉审查 四个现成 prompt。
- `references/metrics-and-governance.md` — 分层指标体系、汇报红线、ID/run manifest/单一事实来源。
- `references/benchmarking.md` — 与既有用例库对标的 Recall/Novelty/Precision 方法。
- `references/finance.md` — **交易/金融项目必读**：领域风险清单、规则模式→测试技术映射、house-style 列映射。
- `references/example-qme.md` — 一个真实填好的完整示例（前海现货竞买竞卖），看"做对了长什么样"。

## 交付物

最终只产**一份权威工作簿**：列采用团队规范（所属模块/功能点/前置/步骤/预期/优先级/案例类型…），末尾追加追溯列（需求 ID / 证据 ID）。配套 `validation_report.md`、`benchmark/recall_report.md`、`governance/run_manifest.jsonl`。历史与中间态进 `generated/` 或 `archive/`，不进 `approved/`。

## 常见失败模式（务必规避）

来自首次落地复盘，新项目最容易踩的七个坑：

1. 用例停在复述规则、不可执行 → 用两层模型 + 系统输入注入。
2. 用"100% 覆盖"误导 → 指标分层 + 汇报红线。
3. 只测文档、不测系统 → 对标既有库，量化真实覆盖。
4. 大量 Needs Confirmation 计入产物 → 输入闸门 + NC 生命周期。
5. 多套 ID / 多份重叠产物 → 单一命名空间 + 单一权威产物。
6. 不符合团队规范、无对标 → house-style 映射 + 金标准对标。
7. 单一文档输入封顶质量 → 输入资产盘点，缺则只产 L1 并登记待办。
