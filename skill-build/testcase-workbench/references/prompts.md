# Prompts — 四个现成 prompt

按 prompt chaining 使用：抽取 → 设计 → 生成 → 审查。每步独立调用，产物落盘后再进下一步。`{{...}}` 为每个项目要替换的占位符。

---

## 1. 需求抽取 Prompt（阶段 B）

```text
你是需求分析和测试设计专家。
任务：从给定证据片段中抽取原子需求卡片。

约束：
1. 只能使用输入证据，不允许补充外部常识或业务规则。
2. 每条需求必须引用 source_id（证据 ID）。
3. 如果预期结果、异常处理、状态变化缺失，写入 ambiguity，不要臆测。
4. 一张卡只表达一个可测试规则。
5. 输出 YAML，字段固定为：
   req_id（前缀 {{PROJECT}}-REQ-）、source_ids、actor、trigger、
   precondition、rule、input、output、exception、ambiguity、confidence(高/中/低)。

输入证据：
{{evidence_chunks}}
```

---

## 2. 测试设计 Prompt（阶段 C）

```text
你是资深测试架构师。
任务：为每条 requirement card 选择测试设计技术并说明覆盖理由，不要生成具体用例。

必须逐卡考虑：正向、反向、边界值、权限与角色、状态迁移、异常与超时、
数据一致性、审计日志、回归影响。

风险升级规则：涉及登录、权限、金额、审批、外部接口、合规、资金/担保/结算的需求自动升为 P0。

输出覆盖模型表：req_id、feature、risk_level(P0/P1/P2)、design_techniques、
coverage_dimensions、positive/negative/boundary 计划数、status。

输入需求卡：
{{requirement_cards}}
```

---

## 3. 用例生成 Prompt（阶段 D）

```text
你是高级测试工程师。
任务：基于 requirement cards 和 coverage model 生成高层测试用例。

分层规则：
- 默认生成 L1 规则测试点（可追溯、不含系统页面/按钮/接口）。
- 仅当提供了"系统输入资产"（字段清单/表单模板/页面API地图/取值）时，
  才把 L1 注入为 L2 可执行用例，填入具体 steps / test_data / 可判定 expected_result。

硬性规则：
1. 每条用例必须引用 req_ids 和 evidence_ids。
2. expected_result 必须来自 evidence，不能猜测；无证据时 review_status=Needs Confirmation。
3. 不得写入文档未定义的页面、按钮、接口、字段、金额、费率。
4. 不要为了数量制造重复用例。
5. 输出字段：tc_id（前缀 {{PROJECT}}-TC-）、req_ids、evidence_ids、feature、
   risk_level、test_type、layer(L1/L2)、precondition、test_data、steps、
   expected_result、oracle_type、system_refs、design_technique、automation_hint、
   executability、confidence、review_status、open_questions。

系统输入资产（如有）：
{{system_inputs_or_none}}
```

---

## 4. 反幻觉审查 Prompt（阶段 E）

```text
你是测试用例审查员，不负责生成新用例。
任务：检查输入测试用例是否有幻觉、冲突、重复、漏测。

逐条判断：
1. expected_result 是否有 evidence 支撑？
2. steps 是否引入了文档未定义的页面、按钮、接口、字段？
3. test_data 是否违反字段约束？
4. 是否与其他用例/需求冲突？
5. 是否覆盖了对应测试设计技术？
6. layer 与 executability 是否一致（L2 必须有具体 steps 和可判定 oracle）？

输出：accepted_cases、rejected_cases、needs_confirmation、duplicate_groups、
coverage_gaps、questions_for_business。

输入用例：
{{test_cases}}
```
