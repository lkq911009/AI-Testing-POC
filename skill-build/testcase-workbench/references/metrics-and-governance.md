# Metrics & Governance — 分层指标、汇报红线、产物治理

## 1. 分层指标体系（诚实口径）

把指标分两层。一层只证明流程自洽，**不能**单独当成熟度结论；二层才是真正价值。

| 层 | 指标 | 定义 | 采集 | 目标 |
|---|---|---|---|---|
| 一层（过程/一致性） | Traceability | 有 req/evidence 引用的用例比例 | 自动 | 100% |
| 一层 | Unsupported Expected Result | 预期无证据比例 | 自动 | 正式集 0% |
| 一层 | 文档需求引用率 | 需求卡被至少 1 条用例覆盖比例（**注意：不是系统覆盖**） | 自动 | P0/P1 100% |
| 一层 | Redundancy Rate | 重复/等价用例比例 | 自动 | ≤ 10% |
| 二层（结果/有效性） | Validity | 人工确认有效用例比例 | 人工 | PoC ≥ 80% |
| 二层 | 系统覆盖度 (Recall) | 对标既有用例库的召回（见 benchmarking.md） | 对标 | 记录并提升 |
| 二层 | Technique Coverage | 测试设计技术覆盖完整度 | 半自动 | 高风险模块 ≥ 90% |
| 二层 | Gap Discovery | AI 发现的人工遗漏有效用例数 | 人工 | 记录复盘 |
| 二层 | Review Cost | 人工审查耗时 | 计时 | 比纯人工下降 |
| 二层 | Automation Pass Rate | 生成脚本编译/运行通过比例 | 自动 | 逐步提高 |

**汇报红线**：缺少 Validity 与 系统覆盖度(Recall) 时，不得单用一层指标的"100%"对外汇报或当通过标准。一层指标命名要去歧义——把"Requirement Coverage"叫"文档需求引用率"，避免被读成"系统已覆盖"。

**交付口径**：可交付分母 = Confirmed 用例；Needs Confirmation 单独统计为待澄清项，不进分子分母。

## 2. 产物治理与版本规范

来自落地复盘，最容易失控的就是产物。规则：

- **单一 ID 命名空间**：一个项目一套前缀（`{{PROJECT}}-REQ/-E/-TC`），禁止并存第二套；跨运行 ID 稳定、只增不改写。
- **单一权威产物**：`approved/` 只保留一份当前权威工作簿；历史与中间态进 `generated/` 或 `archive/`，绝不进 `approved/`。
- **运行清单 run manifest**：每次生成写一行，记 `run_id`、源文件及哈希、模型、prompt 版本、时间、产出文件，便于复现与审计。模板见 `assets/run_manifest.template.jsonl`。
- **不可变证据库**：`evidence_index` 一旦建立不被生成内容回写。
- **卫生**：提交前清理锁文件/临时文件；命名含版本与日期；用一份 README 指明"哪份是权威"。

## 3. 团队交付 schema 映射（house style）

内部用追溯型 schema，交付时映射到团队列并叠加追溯列：

| 内部追溯字段 | 团队交付列（按各团队规范替换） |
|---|---|
| feature | 所属模块 / 功能点 |
| precondition | 前置条件 |
| steps | 测试步骤 |
| expected_result | 预期 |
| test_type / design_technique | 案例类型 |
| risk_level (P0/P1/P2) | 优先级（如 1/2/3/4，约定映射） |
| req_ids, evidence_ids | 追溯列（置于末列） |

最终交付 = 团队列 + 末尾追溯列：既能直接评审入库，又不丢可追溯性。
