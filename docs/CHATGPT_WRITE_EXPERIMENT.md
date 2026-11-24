# E0：真实 ChatGPT 写 GitHub 的独立早期实验

状态：**需要用户授权，尚未执行**。这是设计最终批准后的早期准入实验，应先于 CLI/Skill/Hooks 开发投入；不延后到工具链完成后才发现产品入口不可用。本次仅交付实验方案。

## 目标与边界

证明实际用户 ChatGPT 会话能通过受支持、已授权入口创建远程 GitHub Issue，且 Codex 能独立读取该对象。它不证明完整 GACP 任务执行闭环、实时同步、未来账号资格或自动触发。

不使用本地 mock、Codex 代写、浏览器 Cookie、抓取聊天、非公开接口或粘贴令牌。不为实验预先实现自定义 MCP 适配器；先测现成 GitHub 插件，再测适用工作区的 GitHub 官方托管 MCP。若账号没有写 MCP 资格，增加本地 server 也不能证明已解决。

## 前置条件

用户指定一个允许创建无害测试 Issue 的仓库和可见性；不默认创建新仓库或在 Public GACP 上传任何个人背景。用户在 ChatGPT 实际入口完成连接/选库，必要时由管理员批准。记录产品入口（web/desktop）、套餐/工作区策略、工具名/版本、时间和授权仓库 ID；不把本 Codex 连接的权限当作 ChatGPT 连接。

## 操作和证据

1. 用户在真实 ChatGPT 新会话发现可用 GitHub 工具，确认是否存在实际创建 Issue 的写工具。仅出现搜索/fetch 就判定该入口未证明可写，不能套用 Codex 的 create_issue。
2. 在该 ChatGPT 会话内生成一条无害 proposed 任务和新随机公开 nonce；显示准确目标仓库、标题、描述和公开内容。用户明确确认“发布这条测试 Issue”；普通讨论不能代替确认。
3. 由 ChatGPT 通过现成已授权工具实际执行写入，保留脱敏工具回执、Issue URL/node ID、GitHub 实际作者、nonce 与时间。正文写明仅验证发布，不授权代码执行。当前 wire task 需要真实哈希时按已批准规范填写；尚无工具时可先做入口探针，明确该探针不是 GACP schema 一致性测试。
4. 用户仅提供定位 URL 给 Codex；Codex 使用独立 GitHub 连接真实回读 Issue，而不是仅相信聊天截图/复制的回执，核对仓库 ID、标题、nonce、作者及时间。
5. 由已授权 Codex 在同一测试 Issue 添加另一条公开随机回读标记（需实验范围事先授权）；ChatGPT 再次通过实际工具读回。若其入口只能检索代码、不能读 Issue/评论，记录准确限制，不把缓存命中当最新读取。
6. 人工记录 write_path_verified/read_back_verified、产品入口、连接、仓库 ID、工具回执/URL、实际身份、fresh nonce 与 fetched_at。测试 Issue 是否关闭/清理由用户决定，不自动删除证据。

## 成功与失败判定

写入口通过要求步骤 1–4 证据齐全；步骤 5 另行证明双向取数，不能与写入混为一个布尔值。若工具使用共享 App 身份，只证明发布可用；用户执行批准的可信 provenance 必须另测，不以 approved_by 字符串自认证。

无写工具、套餐/管理员限制、OAuth 不兼容、权限不足、写调用被拒绝或 Codex 远程未读到，都记录为 failed/blocked/unverified 及原因；不得标记 passed。GitHub 页面人工提交和用户让 Codex 代写是明确人工降级，不算 ChatGPT 写入口通过。

本地离线 Schema/哈希检查、当前设计分支 git push、官方 MCP README、工具清单均不能作为本实验通过证据。

## 准入决定

若现成入口通过，保持既定架构；若未通过，先报告实际限制及最小人工降级，请用户决定是否接受该能力缺口再投入后续工具实现。只有已证明目标 ChatGPT 账号支持写 MCP 且现成工具确有必要缺口时，才另提可选窄适配器设计，不擅自进入开发。
