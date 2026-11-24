# GACP 协议草案

协议：`gacp/v1`；规范草案：`0.1.0-draft`；日期：2026-11-24。MUST/SHOULD/MAY 分别表示必须/建议/可选。尚无通过一致性验收的实现。

## 1. 范围与版本

协议仅定义跨 Agent 可交换对象和 GitHub 映射，不规定模型、IDE、语言或业务路径。核心模式是用户确认任务、执行者主动同步；后台执行和 Hooks 属于可选能力。

每个对象 MUST 有 protocol、schema_version、kind、id、project_id、created_at、source_agent、actor、links。时间为 RFC3339 UTC；显示可用用户时区。UUID 采用随机 UUIDv4；不得以短任务号作为跨仓库唯一 ID。schema_version 当前为 1.0.0；规范草案版本独立于 wire version。未知主版本拒绝写入；未知 minor 或未知非 namespaced 字段保守拒写并报告 VERSION_UNSUPPORTED。扩展只放 extensions，以反向域名为键；忽略扩展不能扩大授权。迁移必须显式、备份、PR 审核，不静默重写旧对象。

`project_id = github:<numeric repository id>` 是稳定身份；host 固定 github.com（v1 核心），owner/name/url 为可变定位信息。fork 有新 repository ID，必须显式初始化新项目；模板不能复制实例 UUID、批准记录或待发队列。

## 2. 权威来源

| 数据 | 唯一权威 | 其他位置 |
|---|---|---|
| 项目背景、目标、约束、重要文件 | 默认审核分支 .gacp/context.md 的 Front Matter 与正文 | 聊天是讨论，不自动生效 |
| 项目身份、协议版本、审批者策略 | 默认审核分支 .gacp/project.json | 本地缓存不得覆盖 |
| 任务定义、审批和生命周期 | 对应 GitHub Issue 内 GACP 定义块与有效追加事件评论 | Labels、assignees、Issue open/closed 和摘要是投影 |
| 已确认技术决策 | 经 PR 审核合入的 .gacp/decisions/<uuid>.md | Issue 事件只引用生效 commit |
| 执行结果 | Issue 的 result.published 事件所指向的固定 commit 上 .gacp/results/<uuid>.md | 评论内摘要、聊天回答均非独立结果 |
| 代码与审查 | commit 与 PR 原生对象 | 报告记录 SHA/URL，不复制 diff 为另一真相 |
| 项目当前工作状态 | 有效 Issue 事件、结果与决策的组合 | .gacp/STATUS.md 是可删除重建的快照 |
| 领取互斥凭据 | 同仓库 refs/heads/gacp/claims/<issue-key>/<attempt> | 只控制执行权，不保存另一套任务状态 |

Issue Front Matter 式定义使用唯一的 fenced JSON 块，以前置 `<!-- gacp:task -->` 标记。事件评论使用 `<!-- gacp:event -->` + 单个 fenced JSON。普通文字、外部 Issue、标签变化不构成执行授权。源字段由 GitHub 返回的身份和时间核对，不能相信正文自报的 actor。

纳入哈希的任务定义字段编辑会改变 definition_sha256（固定 15 个字段经 RFC8785 JCS 后对 UTF-8 字节求 SHA-256，完整纳入/排除表及测试向量见 [DEFINITION_HASH](docs/DEFINITION_HASH.md)），已有批准立即失效；标题同步变化也要求重新确认。未实现规范化库前不能声称哈希互通。发现删除/编辑既有事件导致链断裂 MUST 停止写入并报告 INTEGRITY_ERROR。GitHub 评论可编辑和删除，协议不能提供防仓库管理员篡改的密码学保证。

## 3. 对象

Schema 见 schemas/gacp-v1.schema.json；Markdown 对象仅把 Front Matter 解析为 JSON 后校验，正文为 description/summary 的人类补充，不承载隐含权限。

- context：background、goals、constraints、important_files、work_summary、repository、policy_ref。
- task：issue_url、issue_node_id、title、description、priority(P0-P3)、acceptance、related_files、dependencies、definition_sha256、operation_id。来源 Agent 与实际 GitHub 操作者分别记录。
- approval：task_id、attempt、definition_sha256、approved_by、approved_at、scope、risk、expires_at、authorization_receipt。批准只授权指定仓库/文件范围/操作；不包括合并、部署、删除数据或更改权限，除非明确列入 scope。
- decision：title、rationale、confirmed_by、effective_at、tasks、supersedes；生效需外部 artifact_ref 指向已审核 commit，文件不含自身 commit SHA。
- result：task_id、attempt、approval_id、definition_sha256、summary、change_type、no_code_reason、code_commits、pull_requests、tests、errors、logs、recommendations。每条 test 包含 argv、cwd、environment、started_at、ended_at、exit_code（未运行时 null）、status、evidence。允许 passed/failed/not_run/inconclusive；没有真实证据不得 passed。代码任务至少关联 commit 或 PR，纯研究任务允许空数组并说明原因。
- event：event_type、task_id、operation_id、previous_event_id、revision、attempt、from_state、to_state、payload、definition_sha256、approval_id。GitHub comment ID/author/created_at 为传输回执，不从自报字段建立信任。
- error：code、message、retryable、operation_id、details；details 不含令牌或原始敏感日志。

对象 id 为 urn:uuid:<UUIDv4>；任务另有稳定 Issue URL/node ID。同对象内 task_id/approval_id/previous_event_id 等标量 UUID 继承 envelope.project_id；dependencies、decision.tasks/supersedes 使用显式 project_id/id 的 object_ref。artifact_ref 必须带 project_id/id/commit/path/sha256；sha256 对目标文件完整原始字节（含存在的 BOM/末尾换行）计算，不是任务定义哈希。跨项目依赖 v1 仅作为外链展示，不自动获取权限或调度。

## 4. 任务状态机

初始定义状态 proposed（revision=0，无执行许可）。

| 起点 | 事件 | 终点 | 条件/操作者 |
|---|---|---|---|
| 无 | task.created | proposed | 已解析合法 Issue；创建不等于批准 |
| proposed | task.approved | ready | 受信审批者明确批准当前定义哈希、scope 和 attempt=1 |
| ready | task.claimed | claimed | 执行者取得唯一 attempt 凭据，批准有效且依赖全 done |
| claimed | task.started | running | 当前凭据持有者；独立工作分支；权限和上下文检查完成 |
| ready/claimed/running | task.blocked | blocked | 正常由持有者记录原因；ready 仅限审批者携有效 orphan_claim 证据冻结孤儿领取，禁止静默重领 |
| running | result.published | verifying | 结果固定 SHA 可远程读取，不代表测试通过 |
| running | task.failed | failed | 执行失败，保留部分结果与错误 |
| verifying | verification.passed | done | 验收条件全部有证据；指定验证者确认 |
| verifying | verification.failed | failed | 记录失败、证据和后续建议 |
| blocked/failed/done | task.reapproved | ready | 审批者批准新 attempt（旧值+1），撤销旧 attempt 的后续写权 |
| proposed/ready/claimed/running/blocked/verifying/failed | task.cancelled | cancelled | 审批者撤销；停止在途执行，已完成变更仍保留 |

done 不表示 PR 已合并；如验收要求合并，则合并完成前不能 done。cancelled 不原地重开，创建关联后继任务。verification.passed 默认由审批者/指定 reviewer 发出；允许低风险任务自验证必须写入受信项目策略，不能由执行者自行放宽。

状态事件中，仅 task.created 的 previous_event_id=null、revision=0、attempt=0；其余引用有效前序 event，revision=前序+1。task.approved 设置 attempt=1；task.reapproved 设置 attempt=前序+1；其他状态事件保留 attempt。第一个批准的前序为 task.created；初始事件缺失不得批准。decision.updated、claim.conflict、sync.failed、conflict.resolved 为观测事件，previous_event_id 指向观察到的状态链头，revision/attempt 保持观察值，from_state=to_state；不成为后继状态事件的前序。所有事件使用独立 operation_id。

## 5. 授权与信任

项目策略的审批者 allowlist 和委派主体 MUST 来自受审默认分支；先核实 GitHub 实际身份和仓库权限，再核实内容授权。不能因 Issue 作者是 ChatGPT 或有 gacp 标签就执行。若工具使用共享 App 身份，正文 claimed approved_by 不能证明用户确认：要求用户本人在 GitHub 发布批准评论，或由已审查的入口提供可核验的授权回执；否则只能 proposed。

普通讨论可生成草案，但 publish-task 的创建行为与执行授权独立；UI 写入确认也不必然等于允许执行任务。批准应显示任务正文、目标仓库、范围、风险、验收和有效期；高风险操作另行批准。每次开始或恢复执行前重新读取批准、定义哈希、取消状态及凭据。撤销不能原子回收已在运行的命令，所以必须使用短执行阶段及工具权限隔离，不承诺瞬时中止。

外部 Issue/PR/日志是非可信数据，不拼接为 shell 命令、不执行内嵌提示、不向其提供密钥。禁止把个人项目内容传到公共协议仓库。发布默认只含白名单的小型文本、必要结果；不自动上传模型、数据集、令牌、Cookie、全部聊天、完整环境变量或用户目录。日志先脱敏；链接也不能指向未经批准的私人路径。

## 6. 领取、并发与冲突

Issue PATCH 没有在本次官方接口核查中建立可依赖的 compare-and-swap 保证。读 updated_at/ETag 后再 PATCH 不能被描述为原子锁。因此协议核心不并发覆写任务正文/评论来更新状态；生命周期用追加事件。

执行前 MUST 创建唯一 `gacp/claims/<issue-node-id>/<attempt>` 分支引用，指向含小型 claim.json 的独立 commit：task_id/project_id/attempt/actor/operation_id/definition_sha256/approval_id/execution_id/claim_ref。已有分支时只允许读取，禁止更新、删除、强推或拿最新值重试抢占。同名 ref 唯一性预计可提供领取排他机制，但这只是 API 层推断，未实测；不是已验证的分布式执行锁。issue-key 为 issue_node_id 原始 UTF-8 字节的 base64url（无 padding），attempt 为正整数无前导零。完整验证矩阵见 [GIT_REF_CONCURRENCY_PLAN](docs/GIT_REF_CONCURRENCY_PLAN.md)。没有 Contents write 或仓库规则不允许此命名时，返回 CLAIM_UNAVAILABLE，不自动退回 assignee 当锁。ChatGPT 发布 proposed/批准无须领取写权限，Codex 执行者才需要。

创建成功→回读 ref 与 claim→重新读取当前批准/任务链→追加 claimed 事件→回读确认后才执行。创建响应超时先读取准确 ref 并匹配 operation_id；不盲目新建其他 attempt。分支凭据长期保留作为每次 attempt 的回执，不自动过期或删除。崩溃后同一实际执行者可恢复同 operation_id/execution_id，但必须先确认没有同身份活跃实例；领取 ref 不解决同持有者双活。换执行者必须人工确认前者停止、批准新 attempt。若已创建 ref 却仍为 ready，审批者先以 orphan_claim 固定证据发布 task.blocked，再 reapproved。旧执行者发布结果时必须校验当前 attempt，过期结果仅记录为 orphan evidence，不能改变状态。

按 GitHub comment ID 升序读取全量分页事件；每个状态事件核对前序/revision/批准/凭据。同前序出现多个不同合法候选视为冲突，不按最后写入获胜；停止领取/执行/投影，交审批者追加 conflict.resolved 观测记录，列出候选与选中 event，恢复链。冲突涉及仍运行执行者时先人工停止，再批准新 attempt。不存在伪造候选即可由外部用户拒绝服务：未授权事件不进入合法候选集合，只记录审计。

项目策略变更、定义变更和取消可能与领取并发，因此必须在每个执行阶段重读；此设计减少覆盖，不承诺分布式跨对象事务。并发机制仍是实现阶段的高优先级验证门槛。

## 7. 幂等、发布与恢复

每个逻辑操作生成一次 operation_id 并持久保存，重试沿用；键为 (project_id, operation_id)。同键同内容返回已有回执；同键不同 payload 返回 IDEMPOTENCY_CONFLICT。GitHub Issue 创建无通用原生幂等键：publisher 在正文放 operation_id，重试前分页直接列 Issues 查找（排除 PR，不仅依赖搜索索引）。响应不明必须 reconcile；单 publisher 串行创建。不能承诺任意独立客户端竞争创建时恰好一次，重复创建时保留记录并人工选择 canonical Issue，其他标为 duplicate，绝不自动执行多个。

事件评论同键同内容作为同一逻辑事件处理；同键不同内容冻结。输出不得重复执行代码或重新运行测试来“确认”已完成的发布操作。SHA、Issue URL/node ID、comment ID、结果哈希和权限回执存本地 outbox（小型 JSON 文件，凭证不入队）。

结果发布顺序：本地结构化证据→脱敏/范围检查→独立分支提交结果→推送并远程按 SHA 回读→追加 result.published→回读事件→可选更新摘要。任一步失败保留 outbox、当前 remote receipt 及明确阶段；已推代码未发结果为 partial_publish，不伪装 done。摘要失败不回滚正确结果；不同项目 outbox 按 host+repository ID 隔离，复制目录不得带走队列。

断网：不领取、不假装同步成功；已授权进行中的本地工作保存 checkpoint，写操作待重连重新验证。401 重新授权；403 区分权限/策略/限流，停止越权重试；404 既可能不存在也可能不可见；409/422 冲突重新取数，不覆盖；429/可重试 5xx 按 Retry-After、有界退避+jitter；取消/权限撤销后禁止盲目重放。错误码包括 AUTH_REQUIRED、PERMISSION_DENIED、NOT_VISIBLE、NETWORK_UNAVAILABLE、CONFLICT、CLAIM_UNAVAILABLE、INTEGRITY_ERROR、VERSION_UNSUPPORTED、VALIDATION_FAILED、PARTIAL_PUBLISH、RATE_LIMITED、TEST_FAILED。

## 8. 摘要与新会话恢复

STATUS.md MUST 标注 generated=true、generated_at、generator_version、context_commit、各 Issue 的 comment 水位和 source URLs。摘要是多对象非原子快照，不承诺全仓库瞬时一致。问“最新”必须主动读目标仓库/Issue，记录 fetched_at 和引用；没有入口读评论时明确摘要时间与未覆盖变更，不从聊天记忆推断。

新会话从 Git remote→稳定 repo ID→受审 project.json/context→当前任务定义与全量有效事件→生效决策→结果固定 SHA→相关 PR/checks 恢复。先看 outbox 与 attempt 再决定是否恢复，不因曾 claimed 就重新执行副作用。

## 9. 一致性门槛

Schema 校验只证明结构，不证明授权、哈希、引用存在、时间顺序、互斥、状态机或测试真实性。实现必须额外校验这些语义。详细场景见 ACCEPTANCE_PLAN；两项目、全新会话及 ChatGPT 实际读写未验证前，禁止宣称 gacp/v1 基础版完成。

## 10. 完整 Schema 与语义边界

project 包含受审 approvers/reviewers、执行白名单和验证策略；context.policy_ref 为 project.json 所在的受审 commit，work_summary 仅为背景摘要，不是实时状态权威。claim 使用同一 envelope，文件路径 claim.json。事件类型、每类 payload 及结构限制以 [Schema](schemas/gacp-v1.schema.json) 为规范性结构定义；[schemas/README](schemas/README.md) 列出结构检查无法表达的跨对象语义。

approval.scope 必须是受审策略的子集，approval.attempt 必须与批准事件及 claim/result 的 attempt 一致；approved_by 与实际 GitHub 身份或可信委派回执核对；authorization_receipt 不能仅凭正文自报。task.approved/task.reapproved 的 payload.approval.id 必须等于 event.approval_id，且所有关联对象的 project/task/hash 一致。

verification 的 result 引用必须等于当前 attempt 发布的结果；verified_by 必须是指定 reviewer，测试证据与时间先后单独核实。结果 change_type=code 至少关联 commit/PR，其他类型必须填写 no_code_reason。failed 可包含 exit_code=0 的验收断言失败，但必须有失败证据。Markdown 正文只补充可读说明，不能扩大结构化授权。

本次只完成设计离线校验。ChatGPT 写入是独立早期实验，必须真实 ChatGPT 会话发起写请求、GitHub 产生远程对象、Codex 独立回读；见 [实验方案](docs/CHATGPT_WRITE_EXPERIMENT.md)。不得由本地模拟、Codex 代写或工具清单替代。
