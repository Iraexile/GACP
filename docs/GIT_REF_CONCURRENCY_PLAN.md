# Git Ref 领取并发验证方案

状态：**理论可行，真实并发未实测**。本次只设计实验，没有创建领取 ref、测试 Issue 或运行任务。普通设计分支的推送不算领取机制测试。

## 证据分层

| 结论 | 证据等级 | 本次状态 |
|---|---|---|
| GitHub 提供创建/精确读取 ref 的 REST 接口，创建有 201/409/422 等响应 | 官方 API 文档核实 | [Git refs](https://docs.github.com/en/rest/git/refs#create-a-reference) 已读取 |
| 空仓库不能直接用该 API 创建首个 ref | 官方接口边界；本仓库初始确为空 | 不采用空库 claim，启用项目前需基准提交 |
| 两个不同 commit 同时创建同名 ref，最多一个成功 | API 层推断：单命名空间唯一性 | **尚未对真实 GitHub 竞争请求实测** |
| 只有拿到合法 claim 的执行者才开始工作 | 客户端协议约束 | 尚无执行器，未验证 |
| 读到 updated_at/ETag 后 PATCH Issue 可当原子锁 | 没有可靠依据 | 禁止使用该假设 |
| 超时重试不会重复执行、崩溃恢复和旧 attempt 被拒绝 | 协议设计预期 | 全部待实测 |
| 非协作写者或仓库管理员无法修改/删除 claim | 不成立 | 协议不是权限隔离；检测篡改后冻结 |

接口的某个非 2xx 不能一律解释为“别人已领取”：422 也可能是输入/规则/滥用限制，404 可能不可见，403 可能权限/策略/限流。必须精确 GET 目标 ref 和 claim 内容，不能用前缀 matching refs 返回任意同前缀记录当成功。

## 命名与生命周期

ref = refs/heads/gacp/claims/<issue-key>/<attempt>。issue-key 是 GitHub 返回 issue_node_id 字符串原始 UTF-8 字节的 RFC4648 base64url 编码（无 padding）；不是先解码 Node ID。attempt 是无前导零的正整数，与批准绑定。

claim.json 符合 Schema claim，含 task_id/project_id/attempt/definition_sha256/approval_id/operation_id/actor/execution_id/claim_ref。claim_ref 的名称与内容必须一致；其独立 commit 在创建 ref 前准备好，上传 blob/tree/commit 不等于领取成功。成功创建后回读确切 commit 与文件，再检查批准及状态，再追加 claimed 事件，回读事件确认才可执行。

已有 ref 不 update、不 delete、不 force-push。相同 operation_id 与内容可恢复回执，但还要确认当前 execution_id 无活跃竞争实例。operation_id 是幂等身份，不是两个进程共享后可并发执行的许可；单机需排他进程锁，跨主机不明时人工接管，不能声称 ref 本身解决同持有者双活。

无自动 lease 超时、GC 或抢占。接管须先确认旧进程停止；若远程只有 ready 但已有未发 claimed 的孤儿 claim，审批者必须先发布 task.blocked（ready→blocked 的专用 orphan-claim 情况，携 claim 证据），再 reapproved 新 attempt。旧 ref 长期保留；旧 attempt 不能改变新任务状态。这不是服务端 fencing，已在运行的副作用不能被瞬时撤销。

## 实验前提与方法（设计获批后）

选择用户批准的专用测试仓库，至少一个基准 commit；记录仓库 ID、可见性、GitHub API 版本、运行平台、token 类型/最小权限及规则。不记录 token 值，不修改真实项目。用两个独立执行进程/operation_id/不同 claim commit；预先准备后在同一屏障释放 POST create-ref，请求到达次序不可控，不能只测顺序请求。

每组 fresh ref 至少 50 轮，两方各自可能获胜；统计结果必须列出完整计数，不能只挑成功轮。后续以 10 客户端、延迟和受控网络错误扩展，不当作压力攻击。所有请求有界重试，遵守限流；测试 ref 不自动删除，清理另经许可。

| 用例 | 注入方式 | 期望及验收证据 |
|---|---|---|
| C01 不同持有者同 attempt | 同时发两个不同 SHA 的 create-ref | 每轮唯一 holder；记录状态码、GitHub request ID、目标 ref 和 SHA，loser 实际工作计数=0 |
| C02 已存在 ref | 相同与不同 operation 再请求 | 精确读取；相同仅恢复，不再执行；不同报告冲突 |
| C03 成功但响应丢失 | 客户端屏蔽写响应后恢复 | GET 判定原 operation；不换 attempt、不重复执行 |
| C04 POST 前断网/远程不可达 | 在发送前阻断 | 无任务执行；outbox 保留；重连验证 |
| C05 上传 claim commit 后崩溃 | 尚未创建 ref 即终止 | 无 claimed/执行；孤立 Git 对象不被当作锁 |
| C06 创建 ref 后、claimed 前崩溃 | 终止 holder | 其他执行者拒绝领取；人工核实停止，经 orphan-blocked 与 reapproved 新 attempt 恢复 |
| C07 claimed 后两进程同身份恢复 | 复制同 operation/execution ID 的恢复状态 | 第二进程拒绝；无法证明唯一实例则停机，不能双执行 |
| C08 定义修改/取消/批准到期并发 | 在 ref 创建与 started 之间改有效批准 | 开始前复查拒绝；运行中阶段边界停止；记录不可瞬时撤销窗口 |
| C09 接管后旧 holder 恢复 | 新 attempt 生效后旧实例发布 | 旧输出只作 orphan evidence，不产生有效 done |
| C10 权限/规则拒绝 | Contents 只读、拒绝该分支命名、受限 token | 明确错误、无工作副作用；不升权/改规则/降级成 assignee 锁 |
| C11 ref 被删除/修改 | 经许可由测试管理员篡改 | 本地/远程回执不符即 INTEGRITY_ERROR，冻结；不得静默重新领取 |
| C12 两项目同 Issue 编号 | A/B 都为 #1 同 attempt | repo ID/精确 ref/队列隔离，互不污染 |
| C13 事件链并发 | 同前序两个受信状态事件 | 保留两者，冻结、人工解冲突；不最后写入获胜 |

## 通过标准与实施门槛

不仅是“只有一个 HTTP 201”：还必须唯一实际执行、重试无副作用、旧 attempt 被拒绝、授权撤销复查有效、身份不明时失败关闭。保存每轮脱敏请求回执、执行计数、claim 内容和事件链，并独立远程读取；记录失败轮，不伪造 100% 成功。

C01–C13 未完成前，能力矩阵维持理论可行。若规则/权限/操作成本不合适，提交窄范围替代提案（如受信单写执行入口）重新审核；不默认引入复杂锁服务或数据库。
