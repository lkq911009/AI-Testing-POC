# Coverage Model

测试设计技术选择规则：字段类需求使用等价类/边界值；多条件规则使用决策表；流程类需求使用状态迁移/场景法；涉及资质、担保、金额、成交、合同、违规、权限和信息保护的需求提升为 P0。

| req_id | feature | risk_level | design_techniques | coverage_dimensions | positive_cases | negative_cases | boundary_cases | case_ids | status |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| REQ-QME-001 | 交易定义与合同要素 | P0 | 场景法 | 正向/场景 | 2 | 0 | 0 | TC-QME-001, TC-QME-002 | Covered |
| REQ-QME-002 | 交易模式分类 | P0 | 状态迁移, 决策表, 权限测试, 安全测试 | 正向/状态, 权限/安全 | 1 | 1 | 0 | TC-QME-003, TC-QME-004 | Covered |
| REQ-QME-003 | 报价方式与成交方数量 | P0 | 等价类, 边界值, 错误推测 | 正向/等价类, 正向/边界, 反向/规则 | 2 | 1 | 2 | TC-QME-005, TC-QME-006, TC-QME-007 | Covered |
| REQ-QME-004 | 参与角色 | P2 | 等价类 | 正向/等价类 | 2 | 0 | 0 | TC-QME-008, TC-QME-009 | Covered |
| REQ-QME-005 | 发起方资质 | P0 | 等价类 | 正向/配置 | 1 | 0 | 0 | TC-QME-010 | Covered |
| REQ-QME-006 | 竞价方资质 | P0 | 场景法 | 正向/流程 | 1 | 0 | 0 | TC-QME-011 | Covered |
| REQ-QME-007 | 单向竞价方向 | P2 | 状态迁移, 决策表 | 正向/状态 | 1 | 0 | 0 | TC-QME-003 | Covered |
| REQ-QME-008 | 加价竞买 | P0 | 边界值, 等价类, 错误推测 | 正向/边界, 反向/边界 | 1 | 1 | 2 | TC-QME-012, TC-QME-013 | Covered |
| REQ-QME-009 | 减价竞买方式一 | P0 | 状态迁移, 边界值 | 正向/状态, 反向/边界, 状态/边界 | 2 | 1 | 3 | TC-QME-014, TC-QME-015, TC-QME-016 | Covered |
| REQ-QME-010 | 减价竞买方式二 | P0 | 场景法, 状态迁移 | 正向/流程, 正向/业务 | 2 | 0 | 0 | TC-QME-017, TC-QME-018 | Covered |
| REQ-QME-011 | 单向竞卖 | P0 | 边界值, 等价类, 错误推测 | 正向/边界, 反向/边界 | 1 | 1 | 2 | TC-QME-019, TC-QME-020 | Covered |
| REQ-QME-012 | 单向竞价公告 | P0 | 等价类, 字段校验, 边界值, 决策表, 组合测试 | 正向/字段完整性, 正向/决策表, 兼容/回归 | 4 | 0 | 1 | TC-QME-021, TC-QME-022, TC-QME-023, TC-QME-024, TC-QME-065 | Covered |
| REQ-QME-013 | 成交原则 | P0 | 决策表 | 正向/决策表 | 1 | 0 | 0 | TC-QME-025 | Covered |
| REQ-QME-014 | 最小成交数量门槛 | P0 | 边界值, 状态迁移 | 边界/正向, 边界/反向, 状态/反向 | 2 | 2 | 3 | TC-QME-026, TC-QME-027, TC-QME-066 | Covered |
| REQ-QME-015 | 非公开竞价 | P0 | 权限测试, 安全测试, 场景法, 决策表 | 权限/安全, 正向/场景 | 1 | 1 | 0 | TC-QME-004, TC-QME-028 | Covered |
| REQ-QME-016 | 非公开竞价公告 | P0 | 字段校验, 组合测试 | 正向/字段完整性, 兼容/回归 | 1 | 0 | 0 | TC-QME-029, TC-QME-065 | Covered |
| REQ-QME-017 | 发起方申请 | P0 | 字段校验, 场景法, 错误推测 | 正向/字段完整性, 反向/业务规则 | 1 | 1 | 0 | TC-QME-030, TC-QME-031 | Covered |
| REQ-QME-018 | 交易中心审核 | P0 | 场景法, 错误推测 | 正向/流程 | 1 | 0 | 0 | TC-QME-032 | Covered |
| REQ-QME-019 | 竞价方资格确认 | P2 | 权限测试, 决策表 | 权限/正向, 权限/反向 | 1 | 2 | 0 | TC-QME-033, TC-QME-034 | Covered |
| REQ-QME-020 | 发起方担保 | P0 | 决策表, 状态迁移 | 流程/反向, 流程/正向 | 2 | 1 | 0 | TC-QME-035, TC-QME-036 | Covered |
| REQ-QME-021 | 公告发布条件 | P0 | 决策表, 状态迁移 | 流程/反向, 流程/正向 | 2 | 1 | 0 | TC-QME-035, TC-QME-036 | Covered |
| REQ-QME-022 | 竞价方担保 | P0 | 决策表, 权限测试 | 权限/反向, 权限/正向 | 1 | 2 | 0 | TC-QME-037, TC-QME-038 | Covered |
| REQ-QME-023 | 回应不可撤回 | P0 | 状态迁移, 错误推测 | 反向/状态 | 1 | 1 | 0 | TC-QME-039 | Covered |
| REQ-QME-024 | 交易结束与合同 | P0 | 场景法, 状态迁移, 边界值 | 正向/流程, 状态/反向 | 3 | 1 | 1 | TC-QME-040, TC-QME-041, TC-QME-066 | Covered |
| REQ-QME-025 | 拒签违约 | P0 | 决策表, 状态迁移 | 反向/业务规则 | 0 | 1 | 0 | TC-QME-042 | Covered |
| REQ-QME-026 | 服务费与手续费 | P0 | 场景法 | 正向/业务规则 | 1 | 0 | 0 | TC-QME-043 | Covered |
| REQ-QME-027 | 结算交收入口 | P0 | 状态迁移 | 状态/正向 | 1 | 0 | 0 | TC-QME-044 | Covered |
| REQ-QME-028 | 结算交收方式 | P0 | 场景法, 配置校验 | 正向/配置 | 1 | 0 | 0 | TC-QME-045 | Covered |
| REQ-QME-029 | 结算交收违规违约 | P0 | 决策表, 错误推测 | 反向/业务规则 | 0 | 1 | 0 | TC-QME-046 | Covered |
| REQ-QME-030 | 争议协商与调解 | P1 | 场景法, 状态迁移 | 正向/流程 | 2 | 0 | 0 | TC-QME-047, TC-QME-048 | Covered |
| REQ-QME-031 | 诉讼仲裁与法律文书 | P1 | 决策表, 场景法 | 正向/决策表, 正向/流程 | 2 | 0 | 0 | TC-QME-049, TC-QME-050 | Covered |
| REQ-QME-032 | 监督检查 | P1 | 场景法 | 正向/流程 | 1 | 0 | 0 | TC-QME-051 | Covered |
| REQ-QME-033 | 公开公平公正 | P1 | 审计测试, 错误推测 | 审计/合规 | 0 | 1 | 0 | TC-QME-052 | Covered |
| REQ-QME-034 | 发起方自参与限制 | P0 | 权限测试, 错误推测 | 权限/反向 | 0 | 2 | 0 | TC-QME-053, TC-QME-054 | Covered |
| REQ-QME-035 | 关联方限制 | P0 | 权限测试, 决策表 | 权限/反向, 权限/正向 | 1 | 2 | 0 | TC-QME-055, TC-QME-056 | Covered |
| REQ-QME-036 | 串通操纵禁止 | P0 | 错误推测, 安全测试 | 合规/反向 | 0 | 1 | 0 | TC-QME-057 | Covered |
| REQ-QME-037 | 保密与竞价信息保护 | P2 | 权限测试, 安全测试, 错误推测, 合规测试 | 权限/安全, 合规/反向 | 0 | 2 | 0 | TC-QME-058, TC-QME-059 | Covered |
| REQ-QME-038 | 违规违约责任 | P0 | 错误推测, 场景法 | 反向/业务规则 | 0 | 1 | 0 | TC-QME-060 | Covered |
| REQ-QME-039 | 违规违约处理权 | P0 | 场景法, 错误推测 | 反向/流程 | 1 | 1 | 0 | TC-QME-061 | Covered |
| REQ-QME-040 | 未尽事宜与解释 | P2 | 场景法 | 流程/回归 | 1 | 0 | 0 | TC-QME-062 | Covered |
| REQ-QME-041 | 规则修改与生效 | P1 | 变更测试, 回归, 边界值 | 变更/回归, 边界/回归 | 0 | 0 | 1 | TC-QME-063, TC-QME-064 | Covered |
