# 静态协议样例的阅读方式

本阶段的完整数据样例位于 [schema-cases.json](../schemas/vectors/schema-cases.json)。其中所有项目、主体、SHA、URL 和事件都是虚构设计夹具；positive 只表示结构合法，不证明授权、哈希、引用或测试真实性，绝不能当作待执行任务。

覆盖 task、approval、claim、result、project、context、decision 和每类 event；额外覆盖 passed/failed/not_run/inconclusive 记录。result 内的测试状态为夹具数据，并不意味着本项目实际执行了这些测试。

具有真实计算期望值的任务哈希样例单独位于 [definition-sha256.json](../schemas/vectors/definition-sha256.json)。详情见 [规范化字段表](DEFINITION_HASH.md)，不再维护另一份可能漂移的手写 JSON 示例。

Issue 传输格式为唯一的 <!-- gacp:task --> 标记和 fenced JSON 定义；事件为 <!-- gacp:event --> 标记和 fenced JSON。Markdown Front Matter 对象先解析成同一 Schema 的 JSON，正文不承载额外授权。
