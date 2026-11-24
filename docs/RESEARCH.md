# 官方能力与开源复用调研

日期：2026-11-24（Asia/Shanghai）。只使用官方产品文档、原项目仓库和当前会话工具结果。文档支持与账户实际能力分开；没有安装被调研项目，没有运行其测试，不声称其在本机通过。

## 官方能力依据

| 编号 | 一手来源 | 已核实内容与边界 |
|---|---|---|
| O1 | [Connecting GitHub to ChatGPT](https://help.openai.com/en/articles/11145903-connecting-github-to-chatgpt) | 按需访问授权仓库，产品/套餐有差异；当前页面还描述适用 Work 的部分 PR webhook 任务，不等于所有 Issue 都能触发本地 Codex。文档不是用户账户写入实测。 |
| O2 | [Connecting custom MCP servers in ChatGPT](https://help.openai.com/en/articles/12584461-connecting-custom-mcp-servers-in-chatgpt) | 适用工作区可使用写工具；当前文档列 Business/Enterprise/Edu，Pro 自定义 MCP 为 read/fetch。远程连接、工具快照更新和管理员审批有边界；deep research 自定义工具只读。 |
| O3 | [Build skills](https://learn.chatgpt.com/docs/build-skills) | 用户级发现目录、显式/隐式选择、插件分发；不提供必定调用技能的保证。 |
| O4 | [Hooks](https://learn.chatgpt.com/docs/hooks) | 生命周期事件、信任检查、Windows 命令、Stop 递归字段；本机尚未运行回调。 |
| O4a | [Advanced configuration](https://learn.chatgpt.com/docs/config-file/config-advanced) | 受信项目配置与云协调 Hook 范围不同；本地 command hooks 不能随意套用于云编排。 |
| O5 | [Codex GitHub Action](https://learn.chatgpt.com/docs/github-action) | 官方 CI 执行路径，需凭证和权限；文档 Windows runner 有 unsafe 策略约束，GACP 不默认开启此策略。 |
| G1 | [GitHub MCP Server](https://github.com/github/github-mcp-server) | GitHub 官方维护，工具/权限/host 配置必须分别确认。 |
| G2 | [Remote server guide](https://github.com/github/github-mcp-server/blob/main/docs/remote-server.md) | 官方托管 https://api.githubcopilot.com/mcp/ 和分工具集/readonly 路径，可避免自建运行时；不证明用户 ChatGPT OAuth 兼容。 |
| G3 | [Issues REST](https://docs.github.com/en/rest/issues/issues) | 任务读写 API；列表可能含 PR，需排除；未建立通用 Issue CAS 假设。 |
| G3a | [Issue comments REST](https://docs.github.com/en/rest/issues/comments) | 可追加任务事件评论，需分页与身份核实；评论不是不可篡改日志。 |
| G4 | [Git references REST](https://docs.github.com/en/rest/git/refs) | 创建 ref 与非强制更新；空仓库/已有 ref/权限受接口约束，领取设计仍需竞争测试。 |
| G5 | [Actions event triggers](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows) | 原生事件触发，但某些事件要求默认分支上存在 workflow。 |
| G6 | [GITHUB_TOKEN](https://docs.github.com/en/actions/concepts/security/github_token) | job token 权限限本仓库，多数 token 触发事件不再递归触发 workflow，存在明确例外；不假定所有机器人写入会触发后续流水线。 |
| G7 | [Webhook deliveries](https://docs.github.com/en/webhooks/using-webhooks/handling-webhook-deliveries) | webhook 需要消费者/接收端；不是向任意 ChatGPT 对话推送。 |
| J1 | [RFC8785](https://www.rfc-editor.org/rfc/rfc8785) | 设计选用 JSON 规范化；具体库与互通测试后续核实。 |

OpenAI 的旧 developers.openai.com/codex 链接在本次访问中跳转到 learn.chatgpt.com。以实际返回页面为证据，不沿用旧文章的套餐/只读结论。本会话实际 GitHub 插件工具比“GitHub 永远只能读”的泛化说法更丰富，但这些工具处于 Codex，不能外推到 ChatGPT。

## 开源项目快照

以下 SHA 由当前 GitHub 连接读取 commits/main；页面与 README 亦已读取。不是声称这些版本已部署或测试。许可证均实际读取 LICENSE，为 MIT；复用代码时仍需保留原许可与版权，不能只凭许可证标签。

| 项目 | 本次 main SHA | 核心定位 | 采用决定 |
|---|---|---|---|
| [GitHub MCP Server](https://github.com/github/github-mcp-server) | 9a72d8d38fec18c207cd924b33aa0eb3e186252c | 官方 GitHub 工具传输 | 优先直接复用托管服务；不复制整个 server |
| [Beads](https://github.com/gastownhall/beads)（旧 steveyegge 地址跳转） | 78624109904655d7a4609f27d34eb0982cd8963f | Agent 持久任务依赖图，当前 Dolt 存储 | 不作为 v1 核心依赖；可选外部系统适配 |
| [Gas Town](https://github.com/gastownhall/gastown)（旧 steveyegge 地址跳转） | 94cc82289d2ead275a62e1db4ef1d4ada4a1f98e | 多 Agent 工作区、身份和运行编排 | 不作为默认运行时；不重新实现其编排能力 |
| [Gas City](https://github.com/gastownhall/gascity) | 9285cfb3ed95f20bb4dea3ec085b4f36ff6b3a6c | 可配置多 Agent 编排/软件工厂 | 保留未来执行后端；不引入完整平台 |

### Beads

本次 README 明确 Dolt 为存储、支持 Windows，导出 JSONL 不再是权威数据。其持久化/ready/依赖/claim 值得借鉴，但不能把旧版“纯 Git JSONL”印象套到当前架构。若默认同时使用 Beads 和 Issue 作为任务权威，就必须维护双向映射、身份、冲突与两套同步。因此本设计复用现有 GitHub 任务功能，不新建 Beads 替代品，也不默认导入 Dolt。

未来用户已经使用 Beads 时可选一个方向：Beads 保持该系统内部权威，GACP Issue 仅作为显式发布边界；或者 GACP Issue 权威、Beads 只作派生执行视图。不能同时声明双主；具体映射另开版本提案。

### Gas Town / Gas City

Gas Town 已提供跨工作区运行与多 Agent 协调；README 给出完整 tmux 工作流与 Windows 最小 CLI/WSL 边界。Gas City 当前文档强调通过配置构建编排，默认安装链涉及 bd、dolt、tmux、jq、flock，快速路径为 macOS/Linux Homebrew。没有实际验证其原生 Windows 完整运行。两者解决的范围大于轻量的两个产品共享协议；它们也不凭自身解决 ChatGPT 账号写入口。

复用的是可选执行后端和可追踪任务交接的接口思想，不能为“研究过”而强行作为必装项。需要多 Agent 自动编排时应优先接它们，不能在 GACP 内再开发 Mayor、车队调度、Dashboard 或消息平台。

### 官方 GitHub MCP Server

最适合直接复用的部分是 GitHub 授权工具层。托管模式免自建 server，但 MCP host 的认证能力、工具集与仓库权限仍需验证；本地 server 也不会让 ChatGPT 自动拥有本机 stdio 连接。不要要求用户把 PAT 粘贴进聊天或公开配置。只有实测现有入口不足，才考虑窄适配器。

## 选择结论

GitHub 原生存储 + 薄协议层 + 现成 GitHub 传输是当前最小方案。新增代码只应覆盖稳定对象、授权校验、幂等回执、跨项目定位及失败恢复。复用成本不能只看安装行数：额外数据库迁移、Windows 运行时、升级及双主冲突都是长期成本。当前不据 stars 宣称“成熟且可靠”，实施前应固定版本、核实依赖与漏洞，并完成实际兼容测试。
