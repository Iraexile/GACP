# definition_sha256 规范与一致性向量

协议 gacp/v1；schema_version=1.0.0；规范草案 0.1.0-draft。本文是 SPEC 的规范性附录；与 Schema 同一版本，不另建版本号。

## 完整字段表

只对 task 对象生成定义哈希。先严格解析并校验，再按下表生成一个新的 JSON 对象 P；不直接散列 Issue Markdown 或整个 task。表内路径均为顶层精确字段名，不允许别名。

| 纳入 P 的字段 | 规则 |
|---|---|
| protocol | 必须为 gacp/v1 |
| schema_version | 必须为 1.0.0 |
| kind | 必须为 task |
| id | 任务 UUID，绑定身份 |
| project_id | 稳定 GitHub repository ID |
| created_at | 原始 UTC 字符串，不重写精度 |
| source_agent | 原始字符串 |
| actor | 原始 GitHub 稳定主体标识 |
| title | 原始字符串 |
| description | 原始字符串；任务执行指令只来自此定义及受审上下文 |
| priority | 原枚举 P0/P1/P2/P3 |
| acceptance | 原数组，顺序保留 |
| related_files | 原数组，顺序保留 |
| dependencies | 原数组，每项为含 project_id/id 的 qualified object_ref，顺序保留 |
| extensions | 缺省时向 P 写入空对象 {}；存在时完整递归纳入，不允许删除未知扩展后计算 |

完整排除表：issue_url、issue_node_id（创建后才有的 GitHub 定位回执）；operation_id（发布重试身份）；definition_sha256（避免自引用）；links（非执行性导航，不能携带授权或任务需求）。上述五项修改不改变哈希；issue_url/issue_node_id 必须同时 null 或同时非 null。

除此之外没有可排除的 task 字段。未知顶层字段、错误版本、缺少必填字段先拒绝，不能投影后“忽略错误”。缺省 extensions 与显式 {} 等价。其他必填字段没有默认值，null 不等于缺失。

## 哈希边界

1. 接收 UTF-8 JSON 定义块；拒绝 BOM、重复键、NaN/Infinity、未配对 Unicode surrogate、不可表示为有限 binary64 的数字。整数绝对值超过 2^53-1 必须以扩展内字符串表达，不允许悄悄取整。原始数值 token 必须按 I-JSON/binary64 语义处理；不要先用语言的任意精度数字序列化。
2. 验证 task 结构、版本及所有输入约束。正文中的自报 actor 仍需后续远程身份验证，哈希本身不提供认证。
3. 仅提取上表 15 个字段；extensions 缺省为 {}。
4. 对 P 应用 [RFC8785 JCS](https://www.rfc-editor.org/rfc/rfc8785)。递归按 UTF-16 code units 排序对象键；数字使用 ECMAScript 的 binary64 输出规则（包括 -0 变 0）；数组不排序。
5. 得到的规范化字符串编码为 UTF-8，无 BOM、无末尾换行、无域前缀、无额外空白。definition_sha256 = SHA-256(这些字节) 的 64 位小写十六进制。
6. 写回 task.definition_sha256，读取时重新计算并比较；批准必须绑定同一个哈希、project_id、task_id 和 attempt。

不得 trim、Unicode NFC/NFD 归一化、大小写折叠、替换反斜杠、调整换行（CRLF 与 LF 不同）、去重数组、排序依赖或换算时间字符串。传输层 JSON 缩进/键顺序、字符串等价转义不会改变 P 的 JCS 字节。文件路径的合法性另行检查，不能靠规范化悄悄修复。

GitHub Issue 标题必须与 task.title 一致，Issue 定义块外的讨论不能改变任务范围。正文外新增需求只能重新编辑定义并获得新批准；不能利用未散列 links 注入执行指令。项目策略/上下文不是 task 哈希内容，执行前需单独读取受审 policy_ref 并检查批准 scope 仍有效。

## 静态一致性向量

- [输入](../schemas/vectors/hash-inputs.json)：24 个正例、7 个拒绝例，保留原始 input_json 以测试重复键和非法输入。
- [完整期望结果](../schemas/vectors/definition-sha256.json)：每正例包括 input_json、canonical_utf8、utf8_hex、sha256；拒绝例包含期望错误。
- [Schema 正反例](../schemas/vectors/schema-cases.json)：50 例，结构验证与哈希语义分开。
- [实际离线检查记录](DESIGN_CHECK_RESULTS.json)。

基准 H01-base 的 SHA-256：`876f4a5398771cd5dfa1fccbebf3ede28b909fbfcce12df63d78137478030504`。向量使用虚构项目，不能当作待执行任务。

覆盖范围包括：排除字段稳定性、缺省扩展、每个可变纳入字段、数组顺序、Unicode NFC/NFD 差异、中文/emoji/控制字符、数字和数字样式对象键、输入键顺序与缩进；版本常量变更属于拒绝输入。初始 definition_sha256 使用占位字符是为了计算；验证完整 task 的哈希相等属于下一步语义校验，不能因 Schema 检查成功就认为占位值有效。

## 本次真实检查及限度

2026-11-24 使用临时目录中的 jsonschema 4.25.1（Draft202012Validator，显式日期/HTTPS URI format 检查）核对 Schema meta-schema 和 50 例；使用 rfc8785 0.1.4 + Python hashlib 生成并检查 24/7 例；再用一次性独立 ECMAScript 排序序列化与 Node crypto 核对全部 24 例规范化字节及摘要，均匹配。

这是设计材料离线检查，不是可分发 GACP validator、同步脚本或完整 RFC8785 一致性认证。临时工具未纳入仓库。实现阶段应直接消费这些固定向量，并额外运行所选成熟 JCS 库的上游测试集及跨语言边界测试。不能用本结果替代 GitHub 或 ChatGPT E2E。
