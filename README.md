# GitHub Agent Collaboration Protocol (GACP)

状态：Phase 0/1 架构方向已获认可，本轮补充等待最终设计审核。协议标识 `gacp/v1`，草案版本 `0.1.0-draft`。调研日期：2026-11-24（Asia/Shanghai）。

GACP 让 ChatGPT、Codex 和未来其他 Agent 通过 GitHub 共享项目背景、经确认的任务、决策与可追溯结果。每个业务项目保存自己的数据；本 Public 仓库只发布通用协议、设计与虚构示例。

本阶段没有 CLI、Skill、同步程序、Actions workflow 或可安装发行版。不能宣称基础版本、自动同步或跨端闭环完成。两个测试项目尚未创建，跨端测试尚未执行。

## 审核入口

- [能力矩阵](CAPABILITY_MATRIX.md)：四类能力状态、证据层级和适用范围。
- [协议草案](SPEC.md)：权威来源、数据对象、状态机、审批、冲突与错误规则。
- [架构设计](docs/ARCHITECTURE.md)：组件边界、技术选型与可选触发方式。
- [调研报告](docs/RESEARCH.md)：官方资料、开源复用选择与版本快照。
- [环境与验证记录](docs/VERIFICATION.md)：真实执行记录及未测试事项。
- [后续验收计划](docs/ACCEPTANCE_PLAN.md)：两个独立项目及真实跨端验证。
- [阶段审核报告](PHASE_0_1_REVIEW.md)：停止点和下一阶段入口。
- [设计 Schema](schemas/gacp-v1.schema.json)：JSON Schema 2020-12 草案，不是已实现校验器。
- [哈希规范与向量](docs/DEFINITION_HASH.md)：完整字段表、字节边界和固定摘要。
- [Git Ref 并发验证方案](docs/GIT_REF_CONCURRENCY_PLAN.md)：API 推断与未实测项目分离。
- [ChatGPT 独立早期实验](docs/CHATGPT_WRITE_EXPERIMENT.md)：真实写入准入，不以本地模拟代替。
- [离线设计检查结果](docs/DESIGN_CHECK_RESULTS.json)：结构与哈希检查，不是端到端通过。
- [虚构协议示例](docs/PROTOCOL_EXAMPLE.md)：仅供审阅数据格式。

## 设计原则

Protocol first；GitHub 原生对象优先；单一权威来源；显式用户授权；无独立数据库或 Dashboard；Windows 优先；不依赖任何特定工程或聊天历史。

本次仓库从空仓库开始，设计分支为 `design/phase-0-1`。不直接推送 main，不强制推送，不自动合并。空仓库没有可用于 PR 的基准提交：先审核该分支，批准后再确定 main 的初始化方式。

## 后续目录规划（尚未实现）

`cli/`、`skills/`、`templates/`、`examples/`、`tests/` 仅在设计获批后按需要创建。最终设计批准后，先做独立 E0 真实 ChatGPT 写入口实验，再决定正式开发投入；实现先从最小协议读写与手动触发开始，再验证 Hooks，最后考虑事件驱动。
