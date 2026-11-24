# 能力矩阵

调研日期：2026-11-24；当前产品、账号、版本和工作区均可能改变能力。本文件不代表任意 ChatGPT 账号的授权清单。

状态只有四种：**已验证**（注明本地实测或官方文档核实）、**理论可行**（有接口依据但集成未运行）、**需要用户授权**（账号/安装/权限/人工跨端操作尚缺）、**当前不可行**（在所列边界内不能保证）。文档核实不是端到端通过。

| 能力 | 状态 | 证据与范围 | 未完成项/限制 |
|---|---|---|---|
| 读取当前目录及 Git 状态 | 已验证 | 本地实测：初始目录为空，无 .git；随后克隆空仓库并创建独立分支 | 见验证记录 |
| 远程仓库读取 | 已验证 | 本会话 GitHub get_repo/fetch 返回 Public、repo ID 1469509149、空分支列表 | 仅 Codex 当前连接 |
| 当前连接声明 push 权限 | 已验证 | get_repo.permissions.push=true | 权限声明不等于写入实测；发布见远程交付记录 |
| Git clone 公共仓库 | 已验证 | 本地 clone 成功，提示 empty repository | 不证明 Git push 鉴权 |
| GitHub Issue/评论/PR/refs API | 已验证 | 官方接口文档已核实；当前工具清单含 create_issue/update_issue/create_pull_request | 本阶段未创建测试任务，未验证协议状态机 |
| ChatGPT 按需读取 GitHub | 需要用户授权 | [O1] 官方支持按需访问已授权仓库；套餐/产品入口有差异 | 需要用户在实际 ChatGPT 会话读取最新远程标记，不能用本会话代替 |
| ChatGPT 读取 Issue 与结果评论 | 需要用户授权 | 需检查实际工具是否能 fetch issue/comments；代码检索不保证覆盖任务评论 | 若只能读文件，读取已发布结果文件及带水位摘要；实时状态须直读 Issue |
| ChatGPT 原生 GitHub 入口发布任务 | 需要用户授权 | 当前 Codex 插件暴露 create_issue，并不能证明 ChatGPT 同样暴露；[O1] 未建立普适写权限保证 | 实际工具发现→明确确认→创建测试 Issue→独立远程读取 |
| ChatGPT 自定义远程 MCP 写工具 | 需要用户授权 | [O2] 文档支持适用工作区写工具，当前页面列 Business/Enterprise/Edu；Pro 自定义 MCP 为 read/fetch | 账号资格、管理员策略、OAuth/工具开关和实际写入均待验证 |
| GitHub 官方托管 MCP | 已验证 | [G1/G2] 核实托管端点和工具集配置存在 | 尚未在用户 ChatGPT 中连接；不能把 MCP 存在等同写通 |
| 免自建服务器的 ChatGPT 写入路线 | 理论可行 | 先检查原生插件，再测试官方托管 GitHub MCP [O2/G2] | OAuth host 支持、工作区权限未验证；不承诺直接兼容 |
| 自建轻量 MCP 适配器 | 理论可行 | 只在已有入口确实无法满足、且目标账号具备 MCP 写能力时考虑 | 不是默认组件；不能突破套餐或平台限制；本阶段不实现 |
| 任意个人账号零配置 ChatGPT 写入 | 当前不可行 | 没有账号/入口级实测依据，且 [O2] 存在明确资格限制 | 降级为 GitHub 新建 Issue 页面，或已授权 Codex 发布 |
| 仓库更新自动改变已有 ChatGPT 对话 | 当前不可行 | [O1] 按需获取，不代表推送聊天上下文 | 用户再提问并真实取数；附 fetched_at/source revision |
| 全局复用 Codex Skill | 理论可行 | [O3] 支持用户级技能目录；插件可复用分发 | 未编写、安装或在两个项目验证 |
| 仅安装 Skill 就保证每次任务后同步 | 当前不可行 | [O3] Skill 由显式或模型选择触发 | 必须有已验证的执行包装器或生命周期触发 |
| Codex 非交互执行/结构化输出 | 已验证 | 本地 0.171.0-alpha.13 的 exec --help 暴露 --json/--output-schema；[O5] 有官方执行路径 | 仅帮助接口核实；未启动实际编码任务 |
| Codex SessionStart/Stop Hooks | 已验证 | [O4] 官方文档存在、需信任；Windows 有命令覆盖 | 未安装 Hook，未在本版本运行回调 |
| 任务后自动发布结果 | 理论可行 | Stop Hook 或显式执行包装器读取结构化报告、调用可靠写工具 | Hook 不是验收；断网需持久重试；进程崩溃不能保证立即触发 |
| 不启动 Codex 也持续监听所有 Issues | 当前不可行 | 本阶段没有常驻消费者；GitHub 事件不会自动唤醒任意本地聊天 | 用户主动同步；可选未来 Actions 执行器 |
| GitHub 事件触发校验 | 理论可行 | [G5] issues/issue_comment/workflow_dispatch | 未配置 workflow；不自动执行外部 Issue 内容 |
| GitHub 事件触发 Codex 云端执行 | 需要用户授权 | [O5] codex-action 有 CI 执行路线；[O1] Work 支持部分 PR 事件任务 | 分清 Work PR 事件与任意 Issue 执行；需授权、凭证、费用和安全隔离 |
| Windows 本地通用工具 | 理论可行 | 设计采用 Node/TypeScript、无 bash/tmux 强依赖 | 本机 node v24.22.3；工具尚未实现 |
| 完整 Schema 与固定哈希向量 | 已验证 | 仅离线 meta-schema、结构正反例及 JCS/摘要字节核对；见 DESIGN_CHECK_RESULTS | 不证明授权、远程引用、并发或跨端通过 |
| Git Ref 并发领取 | 理论可行 | 同名 ref 排他为 API 层推断，见 GIT_REF_CONCURRENCY_PLAN C01–C13 | 真实竞争/超时/双活/接管均未实测 |
| 两项目隔离、幂等、冲突恢复 | 理论可行 | SPEC 定义 repository ID、operation ID、追加事件及领取凭据 | 所有故障及多项目测试尚未执行 |
| 新会话仅凭 GitHub 恢复 | 理论可行 | 仓库上下文 + Issue + 决策 + 结果，不依赖 transcript | 需真正全新会话验证 |
| 敏感信息零泄漏的无条件保证 | 当前不可行 | 无检测器可对任意内容保证零泄漏 | 最小公开内容、白名单、人工审查、拒绝上传凭证/权重/原始聊天 |

官方来源的完整链接及适用边界见 [调研报告](docs/RESEARCH.md)。本矩阵没有任何一项把完整 ChatGPT ↔ GitHub ↔ Codex 闭环标记通过。

ChatGPT 写入必须先做 [独立早期 E0 实验](docs/CHATGPT_WRITE_EXPERIMENT.md)，不能由本地模拟或 Codex 代写替代。最终设计批准不自动等于允许正式开发。
