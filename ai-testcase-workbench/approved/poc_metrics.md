# PoC Metrics

| metric | value | note |
| --- | --- | --- |
| Validity | 待人工评审 | AI 仅能做证据一致性校验，不能替代业务确认。 |
| Traceability | 100% | 每条用例均含 req_ids 和 evidence_ids。 |
| Unsupported Expected Result | 0 个未标注项 | 缺失实现细节的用例均标为 Needs Confirmation。 |
| Requirement Coverage | 100% | 41/41 需求卡被覆盖。 |
| Redundancy Rate | 未发现明显重复 | 同一需求的边界/正反向按目的区分。 |
| Automation Pass Rate | 不适用 | 本阶段未生成可执行脚本。 |
