# Pipeline — 阶段 A–G 详细做法

读这份文件来执行每个阶段。通用流程，任何项目适用；交易/金融项目同时参考 `finance.md`。

## 目录

- A 文档摄取与证据索引
- B 原子需求卡 + 输入充分性闸门
- C 测试设计技术选择
- D 两层用例生成（L1 → L2 注入）
- E 验证与反幻觉
- F 金标准对标（见 `benchmarking.md`）
- G 规范化交付（见 `metrics-and-governance.md`）

---

## A 文档摄取与证据索引

目标：让模型只能在可追溯证据上工作。

1. 解析 PDF/Word/Excel/Markdown/接口规范/代码注释；若存在 `sources/screenshots/`，同步解析软件页面截图。
2. 按章节、表格、字段、流程步骤结构化切分；截图按页面名、页面状态、可见字段、按钮、提示文案、截图文件名与哈希建索引。
3. 每个片段生成稳定 ID，例如 `ACME-E-001`，并记录来源文件、页码/行号、文件哈希(SHA256)、原文片段、text_hash。
4. 对版本冲突/重复建立优先级：最新签字版 > 当前代码 > 历史需求 > 会议纪要。
5. 输出 `parsed/evidence_index.jsonl` 与 `parsed/source_conflicts.md`，并显式列出"被引用但未提供"的外部文件（它们是质量天花板）。

**铁律**：后续任何预期结果必须引用至少一个证据 ID，否则标 `Needs Confirmation`。截图只能证明 UI 元素、页面状态和提示文案存在，不能单独证明业务规则成立。证据库不可被生成内容回写。

---

## B 原子需求卡 + 输入充分性闸门

把长文档拆成"一卡一规则"的需求卡。字段：`req_id, source_ids, actor, trigger, precondition, rule, input, output, exception, ambiguity, confidence`。模板见 `assets/requirement_cards.template.yaml`。

抽取原则：只抽文档中明确出现的规则；不要用常识补业务规则；缺预期/状态/异常时写入 `ambiguity`；每个字段引用 `source_ids`。

**输入充分性闸门**（本流程相对原始方法论的关键增强）——抽完卡后，对每张卡判证据充分度并路由：

| 充分度 | 判据 | 路由 |
|---|---|---|
| 完整 | 规则、输入、输出/oracle 均有证据 | 直接产 L2 可执行用例 |
| 部分 | 规则清楚但缺系统字段/取值 | 产 L1，挂 `pending-injection` |
| 外部依赖 | 明确 defer 到外部办法/公告/另一系统 | 不产实例用例，转 `open_questions.md` 业务输入待办 |

闸门执行时检查 `sources/screenshots/`。若有截图，先产出 `system_inputs/ui_screenshot_inventory.md`（页面、状态、可见控件、提示文案、截图引用），再与字段清单、页面/API 地图、需求证据交叉验证。截图中出现但未被需求或系统输入定义的字段/按钮，进入 `open_questions.md`，不要直接写成 Confirmed 用例。

这一步把"开放问题"前置拦截，避免它们伪装成用例混进交付集。

---

## C 测试设计技术选择

先让模型决定"用什么方法"，再生成用例。技术 → 适用对象：

| 技术 | 适用 |
|---|---|
| 等价类 | 输入字段、枚举、类型 |
| 边界值 | 金额、日期、长度、数量、阈值 |
| 决策表 | 多条件业务规则 |
| 状态迁移 | 订单/审批/支付/竞价/任务流 |
| 场景法 | 端到端流程 |
| 错误推测 | 历史缺陷与高风险点 |
| 组合测试 | 多参数（pairwise） |
| 安全测试 | 登录、权限、敏感数据 |
| 兼容/回归 | 升级、接口版本、迁移 |
| 非功能 | 性能、并发、幂等、审计 |

风险升级规则：涉及登录、权限、金额、审批、外部接口、合规、资金/担保/结算时自动升 P0。输出 `design/coverage_model.md` 与 `design/risk_matrix.md`（模板见 `assets/`）。

---

## D 两层用例生成（L1 → L2 注入）

先产 L1 规则测试点（自然语言，不写脚本），再视系统输入注入为 L2。若有软件页面截图，用截图校准页面入口、按钮/字段名称、可见状态和错误提示文案，但预期结果仍必须来自需求、接口、规则参数或人工确认。

L2 steps 必须尽量原子化，便于后续直接映射为自动化脚本动作：

- 每个 step 只包含一个用户操作、系统操作或 API 调用。
- 使用稳定编号（`S1/S2/S3`）；不要只用一串 `->`。
- 避免"并且/然后/同时"把多个动作塞进同一步。
- 观察/读取/等待也可以是独立 step；具体断言仍放 `expected_result`。

用例字段（在基础 schema 上新增 `layer / system_refs / oracle_type / executability`）：
`tc_id, req_ids, evidence_ids, feature, risk_level, test_type, layer(L1|L2), precondition, test_data, steps, expected_result, oracle_type, system_refs, design_technique, automation_hint, executability, confidence, review_status, open_questions`

**注入示例（通用，以"加价步长"类规则为例）：**

L1（注入前，复述规则，不可判定）：
```
预期：报价应以初始价为起点，按最小加价幅度的整数倍递增。
```
L2（注入"报价表单字段：初始价/最小加价幅度/最大加价幅度"后，可判定）：
```
前置：场次已开始，初始价=100，最小加价幅度=5，最大加价幅度=20
数据：105（正向，一倍步长）；103（反向，非整倍数）；121（边界，超最大幅度）
步骤：
S1 打开报价页面
S2 在报价字段输入 <报价>
S3 点击提交报价按钮
S4 读取系统提示或报价状态
预期(oracle)：105 接受；103 拒绝并提示"须为最小加价幅度的整数倍"；121 拒绝（>初始价+最大幅度）
```
L2 的粒度应对齐团队既有用例库（house style）。模板见 `assets/test_cases.template.csv`。

---

## E 验证与反幻觉

每条用例生成后立即校验，而不是全部生成完再查。

| 校验项 | 规则 |
|---|---|
| 证据校验 | `expected_result` 必须可追溯到文档/代码/人工确认 |
| UI 证据校验 | steps 中的页面、按钮、字段、提示文案必须可追溯到页面/API 地图、字段清单或软件截图 |
| 步骤粒度校验 | L2 每个 step 尽量只含一个操作；复合 step 退回拆分 |
| 需求覆盖 | 每个 P0/P1 需求至少有正向、反向、边界或异常覆盖 |
| 冲突检测 | 同一状态/字段/接口的预期不得互相矛盾 |
| 冗余检测 | 语义近似且覆盖相同 req 的用例合并 |
| 漏测检测 | 按角色/状态/字段/异常/权限/接口逐维扫空洞 |
| 自一致性 | P0/P1 需求独立生成 3 次，分歧进人工复核 |
| Chain-of-Verification | 高风险用例生成验证问题并独立回答（证据/角色/状态/数据/冲突），全过才 Confirmed |

输出 `validation/validation_report.md`、`gap_report.md`、`open_questions.md`、`rejected_cases.md`。

**Needs-Confirmation 生命周期**（NC 是待办，不是交付物）：
```
raised(owner, raised_date, SLA, blocking_inputs)
  -> answered      -> 补证据/补输入 -> 升级 Confirmed（可产 L2）
  -> unanswerable  -> Rejected 或 转外部依赖待办
```

---

## F / G

对标见 `benchmarking.md`；指标分层、ID 命名空间、run manifest、单一事实来源、house-style 列映射见 `metrics-and-governance.md`。
