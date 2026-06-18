# Validation Report

## 自动校验摘要

| metric | result |
| --- | --- |
| 需求卡片数量 | 41 |
| 证据片段数量 | 47 |
| 高层测试用例数量 | 66 |
| Traceability | 100% |
| Requirement Coverage | 100% |
| Confirmed Cases | 45 |
| Needs Confirmation Cases | 21 |
| Rejected Candidate Patterns | 5 |

## Accepted Cases

TC-QME-001, TC-QME-002, TC-QME-003, TC-QME-005, TC-QME-006, TC-QME-007, TC-QME-008, TC-QME-009, TC-QME-011, TC-QME-012, TC-QME-014, TC-QME-016, TC-QME-017, TC-QME-019, TC-QME-021, TC-QME-022, TC-QME-023, TC-QME-024, TC-QME-025, TC-QME-026, TC-QME-027, TC-QME-029, TC-QME-030, TC-QME-032, TC-QME-033, TC-QME-034, TC-QME-035, TC-QME-036, TC-QME-037, TC-QME-038, TC-QME-039, TC-QME-040, TC-QME-041, TC-QME-042, TC-QME-044, TC-QME-047, TC-QME-048, TC-QME-049, TC-QME-053, TC-QME-055, TC-QME-056, TC-QME-063, TC-QME-064, TC-QME-065, TC-QME-066

## Needs Confirmation

TC-QME-004, TC-QME-010, TC-QME-013, TC-QME-015, TC-QME-018, TC-QME-020, TC-QME-028, TC-QME-031, TC-QME-043, TC-QME-045, TC-QME-046, TC-QME-050, TC-QME-051, TC-QME-052, TC-QME-054, TC-QME-057, TC-QME-058, TC-QME-059, TC-QME-060, TC-QME-061, TC-QME-062

## Rejected Candidate Patterns

| id | candidate | reason |
| --- | --- | --- |
| RC-QME-001 | 固定页面按钮名校验 | PDF 未定义系统页面、按钮或提示文案，不能把猜测写入正式用例。 |
| RC-QME-002 | 固定担保金额计算 | 担保收取方式、标准和处理方式由交易中心另行确定或公告，本文档无金额公式。 |
| RC-QME-003 | 固定手续费金额计算 | 手续费收取方式和标准由交易中心另行公布，本文档无具体费率。 |
| RC-QME-004 | 结算/交收详细流程 | 需要结算业务管理办法、交收业务管理办法和具体公告，本文档只给出引用。 |
| RC-QME-005 | 违规违约处罚明细 | 需要违规违约管理办法和公告标准，本文档只给出处理依据。 |

## Chain-of-Verification 抽查

| case | 证据问题 | 角色/状态问题 | 数据问题 | 结论 |
| --- | --- | --- | --- | --- |
| TC-QME-027 | QME-E017 明确低于最小总成交数量无成交。 | 交易结束且门槛未达，状态不应进入成交。 | 99 < 100，边界数据有效。 | 通过 |
| TC-QME-039 | QME-E025 明确回应后不得撤回。 | 单向竞价已开始且已回应。 | 无需额外字段。 | 通过 |
| TC-QME-053 | QME-E038 明确发起方不得参与自己发起的交易。 | 账号身份等同发起方。 | 身份数据需系统提供。 | 通过，准入数据待对接 |
| TC-QME-058 | QME-E041 明确保密和不得打探其他竞价方信息。 | 访问角色需权限矩阵确认。 | 目标信息为竞价信息。 | 业务预期通过，系统权限矩阵待确认 |

## 结论

- 所有测试用例均引用了至少一个 req_id 和 evidence_id。
- 未把页面按钮、接口路径、提示文案、担保金额、费率、违约金金额等未在 PDF 中出现的内容写成确认事实。
- `Needs Confirmation` 用例可作为评审清单，不建议直接进入正式测试集。
