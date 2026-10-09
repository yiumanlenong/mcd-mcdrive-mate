# Changelog — 麦麦路上 McDrive Mate

本项目遵循 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/) 格式。

## [Unreleased]

### 计划中
- P1：出发前速览卡（积分过期提醒 + 券有效期 + 当月活动聚合）
- P1：常用路线记忆（"通勤路线 A 老样子"一键复购，基于 `order-list`）

## [0.2.1] - 2026-10-09

### Added
- `SKILL.md` 补充 Agent Skill 规范的 YAML frontmatter（name + 触发式 description）
- `LICENSE`（MIT）
- `.gitignore`（保护 `.env`、真实 `mcp-config.json` 等凭证文件不入库）
- `README.md` 新增 FAQ 章节（自动下单 / Token 安全 / 限流 / 会员依赖 / 设计取舍）
- workbuddy.md 真实实测实录回填并完成脱敏（订单号/门牌/小区名打码）
- `docs/` 架构说明、演示脚本、示例对话实录（examples/）

### Fixed
- examples 与 demo-scenarios 的实录状态标记同步更新

## [0.2.0] - 2026-10-09

### Added
- `docs/architecture.md`：决策架构图（mermaid）与三条安全护栏说明
- `docs/demo-scenarios.md`：三个端到端演示脚本（通勤早餐/深夜得来速/反悔改约）
- `examples/`：三份示例对话实录
- `CONTEST_DECLARATION.md` 填入参赛者信息

## [0.1.0] - 2026-10-09

### Added
- **P0 核心功能**：
  - 场景感知点餐（`now-time-info` + 时段→品类心智映射）
  - 选店决策卡（到店取/得来速/麦乐送按预计总耗时排序）
  - 预约单下单（预计到达 + 5 分钟缓冲，基于 MCP v1.0.4+ 预约能力）
  - 账单确认流程（`calculate-price` → 账单卡 → 用户确认 → `create-order`）
- Skill 主提示词 `skills/mcdrive-mate/SKILL.md`（6 条决策原则）
- 三个分场景策略片段：`time-slot.md` / `store-pick.md` / `order-flow.md`
- 集成文档：`README.md`、`MCP_INTEGRATION.md`、`workbuddy.md`、`mcp-config.example.json`
- 接入腾讯 WorkBuddy 智能体（Connector + 知识文件）

### 设计决策
- 纯 Skill 形态，不自建后端：零部署、对话即用即走、逻辑全开源可审计
- "比时间，不比价格"：路上场景的核心指标是总耗时确定性
- "改单 = 取消重下"：MCP 无改单能力，重下是最快最稳路径
- 不做营养赛道（已饱和），热量问答仅顺带回复

[Unreleased]: https://github.com/yiumanlenong/mcd-mcdrive-mate/compare/v0.2.1...HEAD
[0.2.1]: https://github.com/yiumanlenong/mcd-mcdrive-mate/compare/v0.2.0...v0.2.1
[0.2.0]: https://github.com/yiumanlenong/mcd-mcdrive-mate/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/yiumanlenong/mcd-mcdrive-mate/releases/tag/v0.1.0
