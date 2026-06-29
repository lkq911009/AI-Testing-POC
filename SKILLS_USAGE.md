# 项目 Skills 使用说明

## 1. 当前已有 Skill

本项目目前包含 1 个自定义 Skill：

| Skill | 位置 | 用途 |
|---|---|---|
| `testcase-workbench` | `skill-build/testcase-workbench/` | 把 PRD、SRS、业务规则、接口文档、用户故事等项目资料，转换为可追溯、可评审的软件测试用例 |

它不是“一次提示词批量生成用例”，而是一套 A–G 分阶段工作流：

1. 文档摄取与证据索引
2. 原子需求卡提取
3. 测试设计与风险分析
4. L1/L2 分层用例生成
5. 反幻觉与覆盖验证
6. 与既有用例库对标
7. 规范化交付与运行治理

仓库中的 `testcase-workbench.skill` 是该 Skill 的压缩打包文件；开发和维护时，以 `skill-build/testcase-workbench/` 下的源码为准。

## 2. 安装到 Codex

Codex 会从仓库的 `.agents/skills/` 目录发现项目级 Skill。当前源码位于 `skill-build/`，首次使用前需要复制或建立软链接。

### 方式一：软链接，推荐用于本仓库开发

在仓库根目录执行：

```bash
mkdir -p .agents/skills
ln -s ../../skill-build/testcase-workbench .agents/skills/testcase-workbench
```

优点是修改 `skill-build/testcase-workbench/` 后无需再次复制。

### 方式二：复制为仓库级 Skill

```bash
mkdir -p .agents/skills
cp -R skill-build/testcase-workbench .agents/skills/testcase-workbench
```

复制后，如果继续修改 `skill-build/` 中的源码，需要重新同步到 `.agents/skills/`。

### 方式三：安装为个人 Skill

如果希望在其他项目中也能使用：

```bash
mkdir -p ~/.agents/skills
cp -R skill-build/testcase-workbench ~/.agents/skills/testcase-workbench
```

Codex 通常会自动发现新增或修改的 Skill；如果没有显示，重启 Codex。可在 CLI 或 IDE 中通过 `/skills` 查看是否存在 `testcase-workbench`。

## 3. 如何触发

### 显式触发，推荐

在提示词中直接写 `$testcase-workbench`：

```text
请使用 $testcase-workbench，根据 sources/docs 中的需求文档生成测试用例。
项目代号使用 ACME，先执行阶段 A 和阶段 B，不要生成用例。
```

显式触发适合正式生产测试资产，可以避免 Codex 误用普通的一次性生成方式。

### 隐式触发

当任务明显包含以下意图时，Codex也可能自动选择该 Skill：

- 从需求文档生成测试用例
- 测试设计或覆盖分析
- 审查、改进已有 AI 测试用例
- 对比 AI 用例与人工回归用例库
- 检查测试用例中的幻觉、冲突、重复或漏测

示例：

```text
请根据这份 PRD 设计一套可追溯的测试用例，并标出缺少依据的预期结果。
```

## 4. 使用前准备

### 最低输入

至少提供一种需求证据：

- PRD、SRS、业务细则
- 用户故事、验收标准
- 接口文档
- 流程说明
- 代码或配置中的明确规则

只有业务规则文档时，Skill 默认生成 L1 规则测试点，不会臆造页面、按钮、字段或接口。

### 推荐补充的系统输入资产

以下资料用于把 L1 实例化为可执行的 L2 用例：

- 页面与 API 地图
- 字段清单、类型、枚举和约束
- 表单或公告模板
- 业务配置与规则参数
- 明确的错误码、提示和状态变化
- 测试账号、角色和权限矩阵

### 可选输入

- 团队既有测试用例库：用于统一用例写法并计算 Recall、Novelty、Precision
- 历史缺陷：用于错误推测和回归风险分析
- 交易、金融、支付、清结算项目资料：应同时启用金融适配规则

建议按以下结构准备新项目：

```text
ai-testcase-workbench/
  sources/
    docs/
    api/
    code_refs/
    defects/
  system_inputs/
    field_catalog.md
    form_or_announcement_template.md
    page_api_map.md
  parsed/
  requirements/
  design/
  generated/
  validation/
  benchmark/
  governance/
  approved/
  automation/
```

完整目录模板见：

```text
skill-build/testcase-workbench/assets/directory_layout.txt
```

## 5. 快速开始

### 场景一：只有需求文档

```text
请使用 $testcase-workbench。

项目代号：ACME
需求文档：sources/docs/
当前没有页面、字段和接口资料。

请按 A–E 阶段执行：
1. 建立证据索引；
2. 提取原子需求卡并标记歧义；
3. 建立覆盖模型和风险矩阵；
4. 只生成 L1 规则测试点，不要猜测系统页面、按钮、接口和提示；
5. 执行反幻觉审查。

将中间产物写入 ai-testcase-workbench 对应目录，并汇总仍需业务确认的问题。
```

预期结果：生成可追溯的 L1 测试点，并把缺失的系统信息登记为待注入项。

### 场景二：需求文档和系统输入都已齐备

```text
请使用 $testcase-workbench。

项目代号：ACME
需求文档：sources/docs/
接口文档：sources/api/
系统字段和页面资料：system_inputs/

请执行 A–E 阶段，并将满足以下条件的测试点实例化为 L2：
- 前置条件明确；
- 测试数据有字段约束依据；
- 操作步骤有页面或接口依据；
- 预期结果具有可判定的 oracle。

无法满足条件的内容保留为 L1 或 Needs Confirmation，禁止补写无证据细节。
```

### 场景三：与既有人工用例库对标

```text
请使用 $testcase-workbench，对比本次 AI 用例和 approved/ 中的人工回归用例库。

先划定相同业务范围和版本的可比子集，再输出：
- Recall：AI 覆盖了多少人工用例；
- Novelty：AI 新增且评审有效的用例比例；
- Precision：AI 用例的评审通过率；
- 人工有但 AI 遗漏的用例；
- AI 新发现且建议纳入回归库的用例。

不要直接对两份全量文件按数量做差。
```

### 场景四：审查已有 AI 用例

```text
请使用 $testcase-workbench 审查 generated/test_cases_draft.md。

逐条检查：
- 预期结果是否有 evidence 支撑；
- 是否臆造页面、按钮、接口、字段或数值；
- L2 是否真的具备可执行步骤和可判定 oracle；
- 是否存在冲突、重复和覆盖空洞；
- Needs Confirmation 是否被错误放入正式交付集。

请更新 validation/ 下的验证报告、漏测报告、拒绝用例和开放问题。
```

### 场景五：交易或金融项目

```text
请使用 $testcase-workbench 处理该交易业务需求，并加载金融适配规则。

涉及金额、报价、撮合、担保、权限、合规、结算和外部规则引用的需求默认按 P0 评估。
文档未给出的费率、金额、罚则和结算细节不得臆造。
```

金融适配规则位于：

```text
skill-build/testcase-workbench/references/finance.md
```

## 6. 推荐的分阶段执行方式

对于正式项目，建议分阶段调用并在每个阶段评审后继续，不建议一条提示词直接跑完全部流程。

| 调用 | 执行内容 | 主要检查点 |
|---|---|---|
| 第 1 次 | A：证据索引 | 来源、页码、哈希、冲突是否正确 |
| 第 2 次 | B：需求卡 | 是否一卡一规则，歧义是否被显式记录 |
| 第 3 次 | C：测试设计 | 风险等级和测试技术是否合理 |
| 第 4 次 | D：用例生成 | L1/L2 分层是否正确，是否存在无依据细节 |
| 第 5 次 | E：验证 | 幻觉、冲突、重复、漏测是否处理 |
| 第 6 次 | F：金标准对标 | 可比范围与三项指标是否可信 |
| 第 7 次 | G：交付 | 是否只有一份权威工作簿 |

阶段式提示词示例：

```text
继续使用 $testcase-workbench。
阶段 A 和 B 已经完成并通过评审。
本次只执行阶段 C，读取 requirements/requirement_cards.yaml，
输出 design/coverage_model.md 和 design/risk_matrix.md，不要提前生成测试用例。
```

Skill 中还提供了四个可复用的阶段 Prompt：

```text
skill-build/testcase-workbench/references/prompts.md
```

## 7. L1 与 L2 的区别

| 类型 | 含义 | 是否可直接执行 |
|---|---|---|
| L1 | 基于业务规则的测试点，只描述测什么和预期意图 | 否 |
| L2 | 注入页面、字段、接口、取值和可判定 oracle 后的实例用例 | 是 |

判断原则：

- 缺字段、页面、接口或具体取值时，只生成 L1。
- L2 必须有具体前置条件、测试数据、步骤和可判定预期。
- 不能因为希望“看起来完整”，就在 L2 中编造系统细节。

## 8. 主要输出

| 目录 | 产物 |
|---|---|
| `parsed/` | 证据索引、来源冲突 |
| `requirements/` | 原子需求卡、歧义清单 |
| `design/` | 覆盖模型、风险矩阵 |
| `generated/` | 测试用例草稿和中间候选 |
| `validation/` | 验证报告、漏测、拒绝用例、开放问题 |
| `benchmark/` | AI 与人工用例映射、Recall 报告 |
| `governance/` | 每次运行的 manifest、权威产物说明 |
| `approved/` | 唯一的当前正式测试用例工作簿 |
| `automation/` | 可自动化候选及后续脚本 |

正式交付时应遵守：

- `approved/` 只保留一份当前权威工作簿。
- 每条正式用例必须带需求 ID 和证据 ID。
- `Needs Confirmation` 是待办，不是已确认用例。
- 中间稿放在 `generated/`，不要与正式交付物混放。

## 9. 评审状态

| 状态 | 含义 | 是否进入正式交付集 |
|---|---|---|
| `Confirmed` | 证据充分，预期可验证 | 是 |
| `Needs Confirmation` | 缺证据、系统输入或业务确认 | 否 |
| `Rejected` | 存在臆造、冲突、重复或不成立 | 否 |

## 10. 指标解读

不要只用“需求引用率 100%”代表测试充分。

建议同时报告：

- Traceability：用例是否引用需求和证据
- Unsupported Expected Result：无证据预期的比例
- Validity：人工评审确认有效的比例
- Recall：对既有人工用例的召回
- Novelty：AI 新发现并通过评审的用例比例
- Precision：AI 用例的评审通过率
- Review Cost：人工审查耗时

缺少 Validity 和 Recall 时，不应对外宣称“系统测试覆盖率 100%”。

## 11. 仓库内现有示例

本项目已经包含一个“前海现货竞买竞卖”PoC，可作为结果参考：

```text
ai-testcase-workbench/
```

重点文件：

```text
ai-testcase-workbench/parsed/evidence_index.jsonl
ai-testcase-workbench/requirements/requirement_cards.yaml
ai-testcase-workbench/design/coverage_model.md
ai-testcase-workbench/generated/test_cases_draft.md
ai-testcase-workbench/validation/validation_report.md
ai-testcase-workbench/approved/test_cases_v1.xlsx
```

完整示例说明见：

```text
skill-build/testcase-workbench/references/example-qme.md
```

`tools/generate_qme_test_artifacts.mjs` 是为当前 QME PoC 编写的专项生成脚本，包含本地文件路径和领域数据，不应直接当作新项目的通用入口。

## 12. 常见问题

### `/skills` 中看不到 `testcase-workbench`

检查 Skill 是否位于以下任一目录：

```text
<仓库根目录>/.agents/skills/testcase-workbench/SKILL.md
~/.agents/skills/testcase-workbench/SKILL.md
```

确认目录名下直接存在 `SKILL.md`。仍未出现时，重启 Codex。

### 为什么生成的都是 L1，不是可执行用例

通常是缺少 `system_inputs/` 中的字段、页面、接口或规则配置。补充这些资料后，再要求 Skill 执行“系统输入注入”，把对应 L1 升级为 L2。

### 为什么有很多 Needs Confirmation

这表示原始文档没有给出足够证据。应补充业务确认、系统输入或外部规则，而不是让模型猜测。补充后重新验证，才能升级为 `Confirmed`。

### 可以一次生成全部用例吗

可以，但正式项目不推荐。分阶段执行更容易发现证据错误、需求歧义和系统输入缺口，也能减少后续返工。

### 可以直接把生成数量当作效果指标吗

不可以。生成数量越多不代表质量越高，应结合人工评审、Recall、Novelty、Precision、重复率和审查成本综合判断。

## 13. 进一步阅读

- Skill 主说明：`skill-build/testcase-workbench/SKILL.md`
- A–G 流程细则：`skill-build/testcase-workbench/references/pipeline.md`
- 可复用 Prompt：`skill-build/testcase-workbench/references/prompts.md`
- 指标与治理：`skill-build/testcase-workbench/references/metrics-and-governance.md`
- 人工用例库对标：`skill-build/testcase-workbench/references/benchmarking.md`
- 金融适配规则：`skill-build/testcase-workbench/references/finance.md`
- QME 完整示例：`skill-build/testcase-workbench/references/example-qme.md`
- Codex Skills 官方文档：https://developers.openai.com/codex/skills
