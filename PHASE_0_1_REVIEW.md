# Phase 0/1 设计评审补充与最终审核入口

日期：2026-11-24（Asia/Shanghai）。协议 gacp/v1；wire schema_version=1.0.0；规范草案 0.1.0-draft。状态：架构方向已获用户认可，本轮补充等待最终设计批准；**不构成 Phase 2 开发授权**。

## 已收到的评审意见与落实

用户独立审核后认可“GitHub 原生对象 + 薄协议层 + 现成 GitHub 插件/MCP”，不要求重做竞品调研或大改架构。本轮保留权威来源与组件边界，完成以下补充：

- 完整 JSON Schema：project/context/task/approval/decision/result/claim/event；15 类事件的分支 payload、必填字段、枚举与测试证据限制。
- 完整 definition_sha256 纳入/排除表、JCS UTF-8 字节边界及固定期望哈希向量。
- Git Ref 并发/超时/崩溃/旧 attempt 验证方案；明确 API 推断不等于实测。
- 既有 README、SPEC、能力矩阵、架构、调研及附属验收/验证文档一致性修正。
- 将 ChatGPT 真实写入设为独立早期 E0 实验，先于正式工具开发；不能用本地模拟替代。
- 所有文件提交到 design/phase-0-1 供审核；不向 main 提交、不自动合并。

## 文档一致性核对记录

| 发现的歧义/矛盾 | 本轮统一规则 | 规范位置 |
|---|---|---|
| protocol、schema 与草案版本容易混用 | gacp/v1 / 1.0.0 / 0.1.0-draft 各自独立；尚无发布版，不假装兼容已发布工具 | SPEC §1、Schema 标题 |
| 哈希字段表尚未固定且 extensions 不明确 | 固定 15 字段；仅 extensions 缺省 {}；完整五项排除；其他未知字段拒绝 | DEFINITION_HASH |
| approval 没有显式 attempt | approval/task 批准事件/claim/result 同 attempt；reapproved 才递增 | SPEC §4/10、Schema |
| 草案任务尚无 Issue 回执 | URL/node ID 同 null；创建回读后同非 null；哈希排除二者 | schemas/README |
| task 状态/执行者是否重复存储 | 不写进 task 定义；有效事件折叠 status、claim 推导 executor | SPEC §2/4、Schema |
| “所有关联带 project_id”与 UUID 标量冲突 | 本地标量继承 envelope；跨对象列表使用 qualified object_ref，artifact_ref 明确项目与定位 | SPEC §3 |
| 观测事件是否推进前序链 | 四类观测事件指向所见状态链头，状态/revision 不变，不作为后继状态事件的前序 | SPEC §4 |
| decision.commit 自引用不可计算 | 文件不含自身 SHA；decision.updated 的外部 artifact_ref 固定 commit/path/文件哈希 | SPEC §3、Schema |
| 领取字段不全、Node ID 可能不适于 ref | 增加 execution_id/claim_ref；issue-key 明确 base64url 编码规则 | SPEC §6、并发方案 |
| 创建 ref 后未发 claimed 就崩溃无恢复路径 | 审批者携 orphan_claim 先 ready→blocked，再 reapproved；不删除/抢占旧 ref | SPEC §4/6、Schema |
| 同持有者双进程可能重复执行 | ref 只限制不同 claimant；execution_id 活跃排他/人工接管另外验证 | 并发方案 C07 |
| ChatGPT 写入原先可能被排到后期 | E0 独立早期真实 ChatGPT→GitHub→Codex 回读，不能本地替代 | CHATGPT_WRITE_EXPERIMENT、验收计划 |
| README 链接指向尚未落盘文件 | 统一链接到本评审文件及实际保存的静态材料 | README、本地链接检查 |

没有对既有竞品选择做重新调研。RESEARCH 保留原调查日期和来源；本次临时安装的是离线设计校验库，不是项目运行依赖或 GACP 功能实现。

## 实际验证与未验证

已执行：Schema 2020-12 meta-schema 检查；结构正反例；JCS/哈希向量生成与独立 Node 字节/摘要交叉核对；本地 JSON/文档链接及版本引用检查。精确数量和工具版本以 [DESIGN_CHECK_RESULTS.json](docs/DESIGN_CHECK_RESULTS.json) 为准。初次工具探测、临时目录权限导致的检查中断已修复后重跑；不将中断当通过。

未执行：领取 Ref 竞争、执行器幂等/故障恢复、ChatGPT 真实读取/写入、两个测试项目、全新会话恢复、Hooks 或 Actions。本次普通设计分支发布只证明设计材料能上 GitHub，不证明任务同步或双向闭环。

## 未解决风险与正式实施门槛

| 风险 | 当前证据 | 后续门槛 |
|---|---|---|
| ChatGPT 实际账号可否写及批准 provenance | 官方入口依据；未跨端实测 | 先执行独立 E0，失败明确降级/阻塞并重新决定投入 |
| 同名 ref 并发排他、规则权限兼容 | API 层推断 | 专用授权仓库执行 C01–C13，保存失败轮和实际工作计数 |
| ref 不是服务端执行 fencing | 已明确设计限制 | 旧 attempt 校验、阶段边界授权复查、同身份双活拒绝；不承诺瞬时撤销 |
| Issue/comment 与 Git commit 非事务 | 已有 partial_publish/outbox 设计 | 故障注入及真实远程回读，不静默覆盖或丢任务 |
| 评论可编辑/删除、管理员可篡改 | GitHub 原生对象边界 | 检测链/回执不一致就冻结；不声称不可篡改 |
| Schema 不证明真实测试/授权/时序 | 本地结构检查已做 | 实现完整语义校验；真实证据与指定 reviewer 验收 |
| JCS 语言互通与 scope glob | 本轮固定向量；未有生产实现 | 成熟库固定版本、上游一致性测试、路径逃逸负例 |
| 多项目、跨会话、自动触发 | 只有协议设计 | 两项目/新会话独立验收，Hooks 与事件执行器分别授权验证 |

实施许可顺序：本轮最终设计批准 → 独立 E0 与并发方案所需测试仓库/授权确认 → 根据 E0 结果确认可接受范围 → 再按用户许可进入 Phase 2。现有架构认可不等于批准本轮立即开发 CLI、Skill、Hooks 或自动化。

## 要求的能力状态

协议：设计与 Schema 已补齐，运行实现未开始。Codex 自动同步：未实现。ChatGPT 读取/写入：用户实际产品均未验收，写入路径独立 E0 待授权。跨会话恢复、多项目：设计覆盖，实测未执行。真正事件驱动：未配置。用户操作仍包括本轮最终审核、账号授权与跨端实验；不能把“git push 成功”写成“实时双向同步成功”。

设计从空仓库独立分支开始，main 没有本轮提交；没有可用 base 时不为生成 PR 擅自初始化 main。最终交付回复提供远程分支 URL、commit SHA 和完整清单；本文件不嵌入自身所在 commit SHA 以避免自引用。

完成本轮提交后停止，等待用户明确最终批准。
