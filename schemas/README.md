# GACP v1 JSON Schema

规范：gacp/v1；wire schema_version：1.0.0；设计草案：0.1.0-draft；JSON Schema 方言：2020-12。schema 的 $id 使用保留 .invalid 域作为离线标识，不是部署承诺；所有 $ref 均为文件内部引用，不需要网络解析。

## 覆盖范围

gacp-v1.schema.json 根 oneOf 支持 project、context、task、approval、decision、result、claim、event 共 8 类对象。$defs 包含 qualified object_ref、固定 commit 的 artifact_ref、scope、test、evidence、error。所有对象默认拒绝额外字段；扩展只能进入命名空间 extensions。

event 以 event_type 区分 15 个分支：11 个状态事件与 4 个观测事件。每类 payload 有明确字段；状态事件校验允许的 from_state/to_state、初始 revision/attempt、非空前序和必要批准。观测事件的“前后状态相等”和 revision 不递增属于跨字段语义校验。

task 不保存 status/executor；状态与当前执行者由 Issue 有效事件及 claim 推导。待发布 task 的 issue_url/issue_node_id 均为 null，创建回读后两者均非空；task.created 语义上要求已取得远程回执。批准绑定 attempt、定义哈希、scope 与可核验 authorization_receipt；receipt URL 本身不构成认证。

result 分 code/research/documentation；code 至少关联 commit 或 PR；非 code 提供 no_code_reason。passed 测试要求真实时间、exit_code=0 和至少一条 evidence；failed 允许命令 exit_code=0 但验收断言失败，仍必须有失败证据；not_run 的时间/退出码为 null、evidence 为空并有 notes；inconclusive 必须解释不确定性。argv 保留重复参数，不把 shell 命令字符串当作可直接执行内容。

decision 文件不保存自身所在 commit（避免自引用）；外部 artifact_ref 固定其 commit/path/字节哈希。claim 文件也通过外部引用定位自己的 commit，不把自有 SHA 写入文件。

## 必需的语义检查

JSON Schema 不表达或不证明以下事项：

- GitHub 实际作者/授权回执是否可信、scope 是否在受审项目策略内、批准是否到期。
- approval_id 与嵌入 approval.id 相等，task/project/hash/attempt 在关联对象间一致。
- 状态事件 previous_event_id 存在、revision 连续、attempt 仅在 reapproved 时递增。
- 观测事件 from_state=to_state、previous_event_id/revision 指向观察到的链头；不推进状态链。
- dependency 只能同项目自动调度，当前全 done；不能用未授权跨项目引用获取权限。
- 哈希与规范化、固定 SHA 引用/文件/测试证据是否真实、时序是否合理。
- 同一 execution_id 是否有两个活跃进程、claim ref 是否唯一且未被修改。
- 路径必须是项目相对 POSIX 路径；拒绝绝对路径、..、越出根目录、隐藏的凭证路径及符号链接逃逸。scope.paths 使用显式路径或 / 分隔的 *、** glob；实现阶段必须固定匹配库与负例，不准扩大 scope。
- 允许的 extensions 可以被读取/保存，但不能改变授权、状态机或执行范围。

日期/URI format 必须启用 assertion，不允许只按注解忽略。UTC 只接受大写 Z。对象及引用的 created_at 是生产者时间；排序和身份由 GitHub 的回执元数据验证，不要求生产者时间与服务端时间完全相同。errors.details/environment 仅存脱敏白名单，不得导出全量环境。

## 哈希与数据样例

完整字段表和字节边界见 [DEFINITION_HASH](../docs/DEFINITION_HASH.md)。vectors 下仅有静态 JSON 测试数据，不是 CLI、测试运行器或同步程序。schema-cases 的 positive 仅表示结构合法，不表示其中占位 SHA、日期、身份、远程 URL 已通过语义或执行验收。完整静态检查结果见 [DESIGN_CHECK_RESULTS](../docs/DESIGN_CHECK_RESULTS.json)。

正式实现必须实现全部语义校验后才能执行任务；当前没有可安装工具。
