# Harness Engineering（Agent Harness 工程）综述报告

**调研日期**：2026-03-27
**调研范围**：学术文献、工业实践、开源社区、未来趋势

---

## Executive Summary

**Harness Engineering** 是 2026 年 AI Agent 领域的核心概念，指设计、构建和运维**围绕 AI 模型的基础设施**，以确保 agent 在生产环境中可靠、一致、大规模运行的工程学科。核心洞察：

1. **概念起源**：由 Mitchell Hashimoto（HashiCorp 联合创始人）于 2026 年 2 月正式命名，OpenAI 百万行代码实验（零人工编写）是催化剂
2. **核心区分**：Agent = 模型本身（神经网络），Harness = 工具+知识+观察+权限+动作接口（让模型运作的环境）
3. **学术进展**：清华+哈工大于 2026/03/26 发表 arXiv 论文 "Natural-Language Agent Harnesses"，形式化 harness 逻辑
4. **工业采用**：OpenAI、Anthropic 官方使用术语；Codex App Server、Claude Agent SDK 是代表性实现
5. **对比边界**：Framework 是构建工具（LangChain/CrewAI），Harness 是运行时环境；MLOps 管理模型生命周期，Harness 管理 agent 执行
6. **组成要素**：7大核心组件（上下文管理、工具编排、验证循环、记忆持久化、可观测性、安全控制、架构模式）
7. **未来趋势**：2026-2027 从临时脚本走向标准化平台（Semantic Graphs、AaaS 基础设施），可靠性增益 2-5x
8. **时间线**：2024 Prompt Engineering → 2025 Context Engineering → 2026 Harness Engineering

---

## 1. 概念起源与历史

### 1.1 术语正式化（2026年2月）

**"Harness Engineering"** 这个术语由 **Mitchell Hashimoto**（Terraform 创建者、HashiCorp 联合创始人）在 2026 年 2 月的博客文章中正式提出，标题为 **"Engineer the Harness"**[^1][^5]。

**关键时间节点**：
- **2025年末**：隐式实践阶段——AutoGPT、Codex 等系统中已存在 harness 组件，但未形成统一术语
- **2026年2月初**：OpenAI 发布 **"Harness engineering: leveraging Codex in an agent-first world"** 报告，详述内部实验[^1][^4]
- **2026年2月**：Mitchell Hashimoto 博客文章将实践命名为 "Harness Engineering"[^5]
- **2026年3月**：业界快速采用，Martin Fowler 撰文讨论，清华+哈工大发表 arXiv 论文[^3][^5]
- **2026年3月**：SmartScope 评估："从概念引入阶段过渡到实现模式系统化阶段"[^10]

### 1.2 催化剂：OpenAI 百万行代码实验

最关键的事件是 OpenAI 2026 年初的内部实验[^1][^4][^11]：
- 5个月内用 Codex 生成了 **约 100 万行代码**
- **零行人工编写的代码**（zero manually typed code）
- 连初始的 `AGENTS.md` 指导文件本身都是 Codex 生成的
- 仓库完全为 agent 可读性优化，而非人类工程师

**核心发现**：
> "Because models like GPT-4, Claude, and Gemini perform within a narrow band of each other, **the model is no longer the competitive advantage**. The system surrounding the model (the harness) determines whether an agent succeeds or fails in production."[^1][^2]

### 1.3 术语隐喻

"Harness" 借用马具（reins, saddle, bridle）的隐喻[^2]：
- **模型 = 马**：强大且快，但不可预测
- **Harness = 缰绳系统**：引导马的力量做有用的工作，防止失控
- **工程师 = 骑手**：提供方向，但不直接驱动

---

## 2. 核心定义与本质区别

### 2.1 Harness 的定义

**Harness Engineering** 是设计、构建和运维 AI agent 生产基础设施的工程学科，包含以下要素[^2][^13]：

$$
\text{Harness} = \text{Tools} + \text{Knowledge} + \text{Observation} + \text{Action Interfaces} + \text{Permissions}
$$

**完整定义**（综合多个来源）[^2][^7][^13]：
> Harness 是**除模型本身外，用户请求到 agent 最终输出之间的一切基础设施**，包括上下文组装、工具编排、验证循环、状态持久化、可观测性和治理机制，确保 agent 在约束、告知、验证和纠正下可靠工作。

### 2.2 Agent Framework vs Agent Harness

| 维度 | Agent Framework | Agent Harness |
|------|----------------|---------------|
| **主要角色** | 提供构建 agent 的库、抽象和工具（工具接口、记忆抽象、prompt 模板）| 提供执行 agent 的**运行时基础设施**，管理生命周期、状态、安全和上下文 |
| **类比** | 蓝图 / 建筑材料 / 脚手架 | 工厂车间 / 操作系统 / 运行时环境 |
| **关注点** | "What" 和 "Why"（agent 的逻辑和推理）| "How" 和 "Where"（执行、持久化、护栏）|
| **设计哲学** | 高度模块化，用户组装组件 | 强主张（opinionated），内置决策 |
| **例子** | LangChain, CrewAI, AutoGen, Semantic Kernel | Deep Agents SDK, Claude Agent SDK, OpenAI Codex harness, Salesforce Agentforce |
| **状态管理** | 通常无状态或轻量状态 | 持久化状态，跨会话恢复 |
| **安全控制** | 提供工具，但不强制治理 | 内置策略强制、工具访问中介、安全边界 |

**关键区别**[^3][^6][^7][^9]：
1. **构建 vs 执行**：Framework 帮你**组装** agent loop；Harness 帮你**运行** agent loop
2. **可选 vs 必需**：Framework 提供选项，Harness 提供默认决策
3. **依赖关系**：Harness 通常**使用** Framework（如 DeepAgents harness 使用 LangChain）[^4][^7]
4. **产品定位**：LangChain 自己的术语——LangChain = framework, LangGraph = runtime, DeepAgents = harness[^3]

### 2.3 概念边界对比

| 概念 | 定义 | 范围 | 与 Harness Engineering 的关系 |
|------|------|------|------------------------------|
| **Harness Engineering** | Agent 运行时基础设施工程 | 单个 agent 的执行环境、工具、记忆、验证 | 核心概念 |
| **Agent Engineering** | 构建 autonomous agents 的工程实践 | Agent 设计、训练、部署全流程 | Harness Engineering 是其基础设施子集 |
| **MLOps / LLM Ops** | 模型生命周期管理 | 模型训练、版本控制、监控、部署、A/B测试 | 关注模型本身，Harness 关注 agent 执行 |
| **AI Engineering** | 构建 AI 系统的通用工程学科 | 数据管道、模型开发、评估、生产化 | Harness Engineering 是其针对 autonomous agents 的分支 |
| **Prompt Engineering** | 优化输入提示词 | 单次交互的 prompt 设计 | 2024 主流范式，被 Harness 超越 |
| **Context Engineering** | 管理 agent 看到的上下文 | RAG、MCP、记忆、动态注入 | 2025 主流范式，现为 Harness 的一个组件 |

**演进时间线**[^7][^10]：
- **2024年**：Prompt Engineering（优化单次输入）
- **2025年**：Context Engineering（管理 agent 看到什么）
- **2026年**：Harness Engineering（管理 agent 如何工作）

---

## 3. 学术层面进展

### 3.1 核心论文

#### arXiv:2603.25723 - Natural-Language Agent Harnesses（2026/03/26）[^3][^5]

**作者**：Linyue Pan, Lexiao Zou, Shuo Guo, Jingchen Ni, Hai-Tao Zheng（清华大学 + 哈尔滨工业大学）
**主题**：cs.CL (计算与语言), cs.AI (人工智能)

**核心贡献**：
1. **Natural-Language Agent Harnesses (NLAHs)**：用可编辑的自然语言表达 harness 行为，将高层控制逻辑外化为可移植的可执行工件
2. **Intelligent Harness Runtime (IHR)**：通过显式契约、持久化工件和轻量级适配器执行 NLAHs 的共享运行时
3. **研究发现**：在编码和计算机使用 benchmark 上证明 NLAHs 的可操作性、模块消融能力、代码到文本的迁移保真度

**研究动机**[^3]：
> "Agent harness design is typically buried in controller code and runtime-specific conventions, making it difficult to transfer, compare, or study as a scientific object."

**论文影响**：将从业者术语形式化为学术研究对象，Anthropic 和 OpenAI 的工程博客被论文多次引用[^3]。

#### 其他相关论文
- **ICML 2025**："General Modular Harness for LLM Agents in Multi-Turn Gaming Environments"（在 GPT-4-class 模型上测试可分离的 perception、memory、reasoning 模块）[^10]
- **2026 年 Vasilopoulos 论文**：在 283 个开发会话、108K 行代码上验证三层上下文基础设施的有效性[^8]

### 3.2 Agent 工程化的学术定义

学术界对 **Agent** 的定义与 Harness Engineering 核心哲学一致[^4]：
- **Agent = 训练好的神经网络模型**（感知、推理、行动的能力）
- **Framework / Harness = 工程师构建的环境**（让模型有效运作）

这与业界实践（OpenAI、Anthropic）的术语使用完全吻合。

---

## 4. 工业实践：Anthropic、OpenAI、Google

### 4.1 Anthropic 的 Harness 实践

**官方术语使用**：
- **"Effective harnesses for long-running agents"**（2025/11/26 工程博客）[^3][^6]
- **"Effective context engineering for ai agents"**（2025/09/29 工程博客）[^3]

**Claude Agent SDK 架构**[^6]：
- **Model Context Protocol (MCP)**：连接 AI 应用到数据源、工具、工作流的协议
- **自动编排**：SDK 自动处理工具编排，Claude 根据任务需求决定何时使用工具
- **配置方式**：通过 `query()` 函数的 `options` 参数或 `.mcp.json` 文件配置 MCP 服务器
- **工具权限**：MCP 工具需显式权限，命名模式 `mcp__server_name__tool_name`
- **错误处理**：SDK 在查询开始时发出 `init` 子类型的系统消息，包含每个 MCP 服务器的连接状态

**Initializer/Coding-Agent 模式**[^2][^6]：
- **问题**：单个 agent 在超过上下文窗口的项目中失去连贯性
- **解决方案**：Initializer agent 负责项目初始化，Coding agent 执行具体任务，通过 `claude-progress.txt` 日志在会话间交接
- **效果**：让 Claude 在长期项目中保持连贯工作

### 4.2 OpenAI 的 Harness 架构

**Codex Harness**[^4][^11]：
- **定义**：驱动所有 Codex 体验（web app、CLI、IDE 扩展、macOS app）的核心逻辑和 agent loop
- **Codex App Server**：双向 JSON-RPC 1.0 API，连接客户端和 Codex harness[^4]
- **Conversation Primitives**[^4]：
  - **Item**：原子输入输出单元（用户消息、diff、审批请求），生命周期 `started` → `delta` (streaming) → `completed`
  - **Turn**：用户输入触发的 agent 工作单元，包含一系列 items
  - **Thread**：持久化会话容器，支持创建、恢复、分支、归档

**MCP Server 模式**[^4]：
- Codex 可作为 MCP server 被 OpenAI Agents SDK 编排
- 暴露两个主要工具：
  - `codex()`：启动新 Codex 会话，配置参数（`approval-policy`、`sandbox` 设置）
  - `codex-reply()`：使用 `threadId` 继续现有会话
- Agents SDK 处理编排，实现 agent 间交接、护栏、全流程可追溯

**Harness 定义**（OpenAI 官方）[^1][^11]：
> "将脚手架、反馈循环、文档和架构约束编码为机器可读的工件，Codex agents 使用这些工件执行跨开发工作流的任务，包括代码生成、测试、调试和部署。"

### 4.3 Google DeepMind

搜索结果中未发现 Google DeepMind 官方使用 "Harness Engineering" 术语的直接证据，但其 Agent 产品（如 Bard Code、Gemini Code Assist）隐含了类似的基础设施层设计。

### 4.4 Martin Fowler 评论

著名软件工程师 Martin Fowler 在 2026 年初撰文讨论 Harness Engineering[^4][^8]：
> "The article is titled 'Harness engineering: leveraging Codex in an agent-first world', but only mentions 'harness' once in the text. Maybe the term was an afterthought inspired by Mitchell Hashimoto's recent blog post. Either way, I like 'harness' as a word to describe the tooling and practices we can use to keep AI agents in check."

**关键洞察**[^4]：
- OpenAI 团队的 harness 组件混合了**确定性方法**（linters、CI/CD）和 **LLM 驱动方法**（agents 审查 agents）
- 分为 3 类：Context Engineering（持续增强知识库 + 动态上下文）、Tool Orchestration、Verification

---

## 5. 社区认知与讨论

### 5.1 Reddit / Hacker News 活跃讨论

**Reddit 关键帖子**（2025/11 - 2026/03）[^12]：
- **r/ClaudeCode**: "Harness engineering is the next big thing, so I started a newsletter about it"（2天前）
- **r/AI_Agents**: "The Agent Harness: defining the behaviors frameworks leave undefined"（2025/12/20）
- **r/ClaudeCode**: "I built an open-source harness that gives coding agents persistent memory across sessions"（1个月前）
- **r/ClaudeCode**: "I turned Anthropic's 'long-running agent harness' research into a Claude Code plugin"（2025/11/28）
- **r/ClaudeCode**: "Learnings from building an agent harness that now keeps agents improving code w/ few errors for days on end"（1个月前）
- **r/programming**: "Harness engineering: leveraging Codex in an agent-first world"（2026/02/13）："This is going to be the next paradigm shift in engineering"
- **r/ExperiencedDevs**: "Harness Engineering vs Enjoyment of Coding"（2026/02/23，资深开发者对"0行人工代码"的担忧）

**Hacker News**[^12]：
- **Item 46081704**: "Effective harnesses for long-running agents"（2025/11/30）讨论

### 5.2 社区共识

**时间线共识**[^7][^10][^12]：
- **2024 = Prompt Engineering**：优化单次输入
- **2025 = Context Engineering**：管理 agent 看到什么
- **2026 = Harness Engineering**：管理 agent 如何工作

**核心理念**[^12][^13]：
> "Agents aren't hard; the Harness is hard."（OpenAI + Stripe 验证）
> "Your agent isn't broken. Your harness is."

### 5.3 开源工具生态

| 工具 | 功能 | 来源 |
|------|------|------|
| **Lore** | 持久化、可搜索的知识库，跨会话工作，支持 Claude Code/Cursor/OpenCode | 社区开源 [^12] |
| **Desloppify** | 保持 agent 持续改进代码质量数天的 harness，强制宏观结构和质量分数 | 社区开源 [^12] |
| **Altimate Code** | 数据工程专用 harness，SQL 反模式检测 + 本地验证，防止表 schema 幻觉 | 商业产品 [^12] |
| **Agent Foreman** | 实现 Anthropic 长期 agent 研究的插件，使用外部记忆文件（`feature_list.json`, `progress.md`）持久化状态 | 社区开源 [^12] |
| **Agent Harness Bootstrap** | 扫描仓库生成定制 harness 文件（`CLAUDE.md`, `ARCHITECTURE.md`, lint configs, pre-commit hooks, CI pipeline）的轻量级工具 | 社区开源 [^12] |

### 5.4 社区心智模型：三层循环

Reddit 社区广泛采用的 harness 设计心智模型[^12][^13]：

1. **Outer Loop (Project Level)**：捕捉意图（规格、架构文档、知识库）+ 治理（防止代码库腐化）
2. **Orchestration Loop (Feature Level)**：每个 feature 需要完整计划（需求、设计、任务拆解），验证后再进入下一任务
3. **Inner Loop (Task Level)**：写代码 → 验证 → 错误反馈 → 重试

> "The structure of this cycle determines whether the agent produces working software or 'confident garbage'."

---

## 6. shareAI-lab/learn-claude-code：教学实现 vs 原创概念

### 6.1 仓库定位

**shareAI-lab/learn-claude-code**[^15] 是一个 **0-to-1 学习项目**，专注于教授 harness engineering 原则：
- **最后更新**：2026/03/18（9天前）
- **教学标的**：使用 Claude Code 作为案例，展示适用于任何领域、任何 agent 的 harness 工程通用原则
- **核心哲学**：Agent 是模型本身，Harness 是让模型运作的环境（工具+知识+观察+权限）

### 6.2 课程结构（12节）

| Session | 主题 | 口号 | 机制 |
|---------|------|------|------|
| **s01** | The Agent Loop | "One loop & Bash is all you need" | 最小循环：模型调用工具，代码执行工具 |
| **s02** | Tool Use | "Adding a tool means adding one handler" | 工具通过 dispatch map 注册 handler |
| **s03** | TodoWrite | "An agent without a plan drifts" | Agent 执行前列出步骤提高完成率 |
| **s04** | Subagents | "Break big tasks down; each subtask gets a clean context" | 子 agent 使用独立消息数组保持父上下文清洁 |
| **s05** | Skills | "Load knowledge when you need it, not upfront" | 知识通过 `tool_result` 注入，而非系统提示词 |
| **s06** | Context Compact | "Context will fill up; you need a way to make room" | 实现压缩策略管理增长的上下文窗口 |
| **s07** | Tasks | "Break big goals into small tasks, order them, persist to disk" | 使用文件持久化任务图进行多 agent 协作 |
| **s08** | Background Tasks | "Run slow operations in the background; the agent keeps thinking" | 在线程中执行慢速命令，agent 继续推理 |
| **s09** | Agent Teams | "When the task is too big for one, delegate to teammates" | 引入持久化队友和异步 mailbox |
| **s10** | Team Protocols | "Teammates need shared communication rules" | 实现请求-响应握手（shutdown/plan approval）使用共享状态机 |
| **s11** | Autonomous Agents | "Teammates scan the board and claim tasks themselves" | 启用自组织，agents 自主认领任务 |
| **s12** | Worktree + Task Isolation | "Each works in its own directory, no interference" | 在独立目录中隔离 agents，防止文件冲突 |

### 6.3 结论：教学传播，非原创术语

**shareAI-lab/learn-claude-code 并非原创** "Harness Engineering" 概念，而是[^15]：
1. **教学实现**：逆向工程 Claude Code 架构中的 harness 机制，用 Python 从零实现
2. **概念传播**：通过中英日三语文档、12节渐进式课程，传播 harness engineering 的核心原则
3. **时间线证据**：
   - 仓库最后更新 2026/03/18
   - Mitchell Hashimoto 博客 + OpenAI 报告在 2026/02 已形成术语
   - 仓库明确引用 Claude Code 作为教学标本，而非声称原创

**定位准确性**：
> "本仓库的每一个课程（s01-s12）都在逆向工程 Claude Code 架构中的一个 harness 机制。学完之后，你理解的不只是 Claude Code 怎么工作，而是适用于任何领域、任何 agent 的 harness 工程通用原则。"[^15]

---

## 7. 实践组成：Harness 的7大核心组件

基于学术论文、工业实践和社区共识，现代 agent harness 包含以下 7 大核心组件[^13][^14]：

### 7.1 Context Management and Engineering（上下文管理与工程）

- **Working Context**：当前 turn 的即时 prompt（临时）
- **Session State**：当前任务的持久化日志（任务完成后重置）
- **Long-Term Memory**：跨任务/会话的向量存储或结构化文件
- **Compaction and Optimization**：总结历史、压缩低价值上下文、定位关键信息到 prompt 边界（避免 "Lost in the Middle"）
- **External Memory**：使用文件系统（如 `AGENTS.md`、`claude-progress.txt`）存储大输出和进度追踪

**最高杠杆投资**[^14]：Context engineering 被认为是 agent 系统中投资回报率最高的领域。

### 7.2 Tool Orchestration（工具编排）

- **Controlled Access**：定义文件系统、API、shell 命令、数据库查询的边界
- **Validation and Sandboxing**：拦截工具调用（如 `search`、`bash`），验证参数、沙箱执行、净化输出
- **Atomic Design**：工具设计为原子操作而非复杂工作流，保持控制和可预测性
- **Web Access**：专用工具（如 Firecrawl）处理现代 Web 挑战（JavaScript 渲染、反爬机制）

**核心理念**[^14]：Agents 的能力来自工具而非纯推理。

### 7.3 Verification Loops and Guardrails（验证循环与护栏）

- **Pre-Completion Checks**：任务完成前运行测试、linters、类型检查
- **Deterministic Rules**：结合规则检查（linting、schema 验证）和 AI 驱动验证（agents 审查 agents）
- **Safety Filters**：防止有害、错误或不符合品牌的响应，充当模型边界
- **Error Recovery**：将 agent 失败视为系统问题永久修复，而非仅重试 prompt

**混合验证趋势**[^4][^14]：确定性工具（linters、CI/CD）+ LLM 驱动验证成为标准实践。

### 7.4 Memory and State Persistence（记忆与状态持久化）

- **Progress Tracking**：使用文件（通常 JSON 格式，可靠性高）追踪 todo 列表和已完成子任务
- **Session Recovery**：存储结构化记忆，中断后（如网络超时）下一会话可从上次位置恢复
- **Context Retrieval**：使用 RAG 模式，仅获取当前步骤相关文档，而非加载整个知识库

**解决的问题**[^14]：原始 AI 模型和 framework 通常无状态，每次请求从头开始（主要成本来源）。

### 7.5 Observability and Cost Control（可观测性与成本控制）

- **Logging**：追踪每次工具调用、决策和 token 使用
- **Failure Analysis**：聚类错误模式识别系统问题，设计永久性修正
- **Cost Efficiency**：监控 token 使用，实现 KV-cache 优化等策略（可降低 10x 成本）
- **Metrics**：测量任务完成率、验证成功率、循环频率，隔离 harness 变更的影响

**核心理念**[^14]："You cannot improve what you cannot measure."（可观测性将 agents 从黑盒转为可检查系统）

### 7.6 Safety and Human-in-the-Loop（安全与人机交互）

- **Risk Classification**：操作分类为安全（自动批准）、中等（条件批准）、危险（需显式确认）
- **Approval Gates**：关键操作需人类批准后执行
- **Escalation**：受控机制将复杂或高风险问题升级给人类操作员

**原则**：并非所有操作都应自主。

### 7.7 Architectural Patterns（架构模式）

不同 harness 设计适配不同需求[^14]：
- **Single-Threaded Master Loop**：简单、高度可控，一个循环运行：model → tool → feedback → repeat
- **Middleware-Based Harness**：添加模块化控制在操作前后注入逻辑，适合实验
- **Protocol-Based Harness**：分离 agent 逻辑与接口，同一 agent 可跨 CLI、IDE、Web 工作（如 OpenAI Codex App Server）
- **Long-Running Architecture**：使用 initializer 和 worker agents 维护跨多小时或多天工作流的进度（如 Anthropic 的 initializer/coding-agent 模式）

**2026 年共识**[^14]：
> "可靠性来自约束和系统设计，而非仅靠模型智能。投资 harness engineering 的团队相比仅做 prompt engineering，可靠性提升 50-80% vs 5-15%。"

---

## 8. 未来走向：标准化与趋势（2026-2027）

### 8.1 从临时到标准化

**当前问题**（2026 年初）[^6][^10]：
- Bespoke harness = 临时脚本集合（linters、自定义规则、CI rules）
- 被描述为"手摇引擎"——正确理念，但脆弱且不可扩展

**标准化趋势**[^6][^10]：
1. **Semantic Graphs 取代临时文件系统**：
   - **Epsilla** 等平台提供 Semantic Graphs 作为标准基础设施
   - 图本身充当"操作系统"，提供结构约束、长期记忆、架构 linting
   - Agents 无法探索死胡同，因为这些路径在图 schema 中不存在
   - 支持 Agent-as-a-Service (AaaS) 部署

2. **"No Manual Code" 成为新标准**[^1][^11]：
   - OpenAI 实验确立：Harness 是主要工程工件，模型仅是执行引擎
   - 预期更多团队采用"零人工编写代码"作为强制函数驱动 harness 设计

### 8.2 架构模式形式化

**三层循环模型成为标准心智模型**[^12][^13]：
- Outer Loop（项目级）+ Orchestration Loop（功能级）+ Inner Loop（任务级）
- 社区广泛采用，多个博客和教程引用

**确定性约束优于 Prompts**[^4][^14]：
- OpenAI + Stripe 验证的核心哲学："Agents aren't hard; the Harness is hard"
- 可靠性来自约束解空间，而非改进模型指令
- Deterministic linters 和 type checkers 强制架构边界，而非依赖 LLM prompts

**混合验证成为最佳实践**[^4][^14]：
- 确定性工具（linters、CI/CD）+ AI 驱动验证（agents 审查 agents）
- 管理长期项目中的"熵"（代码漂移和不一致）

### 8.3 "熵管理"形式化

**问题**[^11][^14]：Agents 产出代码速度 > 人类审查速度 → 技术债累积

**解决方案**（预期成为 CI/CD 强制流程）[^11][^14]：
- **重构 agents**：周期性审计代码质量，自动清理漂移
- **垃圾回收 agents**：自动化"清洁"agents，定期清理不一致
- **质量分数**：Composite score（机械+主观）作为"北极星"指标，agents 朝此优化

### 8.4 Agent-as-a-Service (AaaS) 基础设施

**趋势**[^6][^10]：
- 标准化平台提供内置记忆、访问控制、反馈循环的"shell"
- 减少团队从零构建 foundational harness 的需求
- 类似 PaaS（Platform-as-a-Service）模式，但专为 autonomous agents 设计

**代表性平台**：
- Epsilla（Semantic Graphs for AaaS）
- Salesforce Agentforce（企业级 agent harness）
- LangChain DeepAgents（agent harness 内置在 LangChain 生态）

### 8.5 "AI-Friendliness" 成为技术栈选型标准

**Martin Fowler 预测**（2026年初）[^4]：
> "There is a trend toward choosing tech stacks and codebase structures based on 'AI-friendliness' and the availability of robust harnesses, rather than developer preference."

**影响**：
- 提供良好 harness 支持的框架/SDK 将成为事实标准
- 技术选型考量：模型可读性、harness 生态、社区工具支持

### 8.6 模型商品化 + Harness 差异化

**核心趋势**[^1][^2][^11]：
- **Claude / GPT-4 / Gemini / 开源模型性能窄带差异**（标准 benchmark 差异 < 10%）
- **模型不再是竞争优势，Harness 是主要产品**
- **模型成为 harness 内的商品组件**

**量化证据**（Can.ac + LangChain 实验）[^1]：
- 仅改进 harness 设计（不改变模型权重）
- 编码 benchmark 分数从 30th percentile → 5th percentile
- 某些模型准确率从 6.7% → 68.3%

### 8.7 可靠性增益量化

**2026 年案例研究**（OpenAI + 独立 benchmark）[^7][^14]：
- 遵循 Harness Engineering：**2-5x 可靠性增益**
- 仅 Prompt Engineering：1.05-1.15x 可靠性增益
- 投资回报率：Harness Engineering > Context Engineering > Prompt Engineering

### 8.8 时间线预测

| 时期 | 状态 | 特征 |
|------|------|------|
| **2025年末** | 隐式实践 | AutoGPT、Codex 中存在 harness 组件，无统一术语 |
| **2026年2月** | 概念形成 | Mitchell Hashimoto + OpenAI 正式化，快速传播 |
| **2026年3月** | 实现模式系统化 | arXiv 论文、社区工具生态、企业采用 |
| **2026年下半年** | 标准化平台涌现 | AaaS 基础设施、Semantic Graphs、混合验证成为标准 |
| **2027年** | 主导范式 | Harness 成为 AI Agent 开发的核心工程学科，模型完全商品化 |

---

## 9. 关键洞察与争议

### 9.1 核心洞察

1. **"Agent 不是框架，是模型本身"**[^4][^15]
   → 工程师造的是 Harness，不是 Agent

2. **"Agents aren't hard; the Harness is hard"**[^4][^14]
   → 可靠性瓶颈在环境，不在模型

3. **"不是调 prompt，是造缰绳"**
   → 范式转变：从优化输入到设计环境

4. **"上下文中的大部分 token 是'结构性浪费'"**
   → 系统提示词、工具定义、历史重发 vs 信息性消耗

5. **"模型商品化，Harness 差异化"**[^1][^2]
   → Claude / GPT-4 / Gemini 窄带差异，Harness 是竞争优势

### 9.2 社区争议

**资深开发者的担忧**（r/ExperiencedDevs）[^12]：
- **"0 lines of manually-written code"** 承诺是否过于激进？
- Harness engineering 是否会降低编码乐趣？
- 人类工程师的角色如何定义？

**回应**（社区共识）[^12][^14]：
- Harness engineering 是"设计环境"而非"编写代码"
- 人类从"码农"变为"系统架构师"
- 类比：从马车夫变为汽车工程师（更高层次的控制）

### 9.3 术语演化

**早期混淆**（2026年2月）[^4]：
- OpenAI 文章标题提到 "harness engineering"，但正文仅提及一次
- Martin Fowler 评论：可能是 Mitchell Hashimoto 博客的"事后灵感"
- 说明术语在快速传播中逐渐统一

**当前状态**（2026年3月）[^10]：
- OpenAI、Anthropic 官方使用
- 学术论文形式化
- 社区广泛采纳
- 正从"概念引入"过渡到"实现模式系统化"

---

## 10. 参考文献与资源

### 10.1 学术论文

[^3]: Pan, L., Zou, L., Guo, S., Ni, J., & Zheng, H.-T. (2026). Natural-Language Agent Harnesses. *arXiv:2603.25723*. https://arxiv.org/abs/2603.25723

### 10.2 工业博客与报告

[^1]: OpenAI. (2026). Harness engineering: leveraging Codex in an agent-first world. https://openai.com/index/harness-engineering/

[^4]: OpenAI. (2026). Unlocking the Codex harness: how we built the App Server. https://openai.com/index/unlocking-the-codex-harness/

[^6]: Anthropic. (2025). Effective harnesses for long-running agents. *Engineering Blog*. Retrieved 2026-03-12.

### 10.3 社区资源

[^2]: Firecrawl. (2026). What Is an Agent Harness? The Infrastructure That Makes AI Agents Actually Work. https://www.firecrawl.dev/blog/what-is-an-agent-harness

[^5]: The Epsilla Blog. (2026). The Third Evolution: Why Harness Engineering Replaced Prompting in 2026. https://www.epsilla.com/blogs/harness-engineering-evolution-prompt-context-autonomous-agents

[^7]: Agent-Engineering.dev. (2026). Harness Engineering in 2026: The Discipline That Makes AI Agents Production-Ready. https://www.agent-engineering.dev/article/harness-engineering-in-2026

[^8]: Parallel Web Systems. (2026). What is an agent harness in the context of large-language models? https://parallel.ai/articles/what-is-an-agent-harness

[^9]: Inngest Blog. (2026). Your Agent Needs a Harness, Not a Framework. https://www.inngest.com/blog/your-agent-needs-a-harness-not-a-framework

[^10]: SmartScope Blog. (2026). What Is Harness Engineering: A New Concept Defining the 'Outside' of Context Engineering. https://smartscope.blog/en/blog/harness-engineering-overview/

[^11]: InfoQ. (2026). OpenAI Introduces Harness Engineering: Codex Agents Power Large‑Scale Software Development. https://www.infoq.com/news/2026/02/openai-harness-engineering-codex/

[^12]: Reddit r/ClaudeCode, r/AI_Agents, r/LLMDevs, r/programming, r/ExperiencedDevs. (2025-2026). Community discussions on harness engineering.

[^13]: LangChain Blog. (2026). The Anatomy of an Agent Harness. https://blog.langchain.com/the-anatomy-of-an-agent-harness/

[^14]: Multiple sources including NxCode, Harness-Engineering.ai, HumanLayer Blog, Generative Inc. (2026). Industry guides on harness engineering.

[^15]: shareAI-lab. (2026). learn-claude-code: Bash is all you need - A nano claude code–like agent harness, built from 0 to 1. GitHub repository. https://github.com/shareAI-lab/learn-claude-code

---

## 附录：Harness Engineering 快速参考

### 概念起源
- **时间**：2026年2月（Mitchell Hashimoto 博客 + OpenAI 报告）
- **催化剂**：OpenAI 百万行代码实验（零人工编写）

### 核心定义
- **Agent = 模型本身**（训练好的神经网络）
- **Harness = 工具+知识+观察+权限+动作接口**（让模型运作的环境）

### Framework vs Harness
- **Framework**：构建工具/蓝图（LangChain/CrewAI）
- **Harness**：运行时环境/操作系统（Claude Agent SDK/Codex harness）

### 7大核心组件
1. Context Management（上下文管理）
2. Tool Orchestration（工具编排）
3. Verification Loops（验证循环）
4. Memory Persistence（记忆持久化）
5. Observability（可观测性）
6. Safety Control（安全控制）
7. Architectural Patterns（架构模式）

### 时间线
- **2024**：Prompt Engineering
- **2025**：Context Engineering
- **2026**：Harness Engineering

### 量化效果
- **Harness Engineering**：50-80% 可靠性提升，2-5x 增益
- **Prompt Engineering**：5-15% 可靠性提升

### 未来趋势（2026-2027）
- 标准化平台（Semantic Graphs、AaaS）
- 模型商品化，Harness 差异化
- "AI-Friendliness" 成为技术栈选型标准

---

*报告完成日期：2026-03-27*
*调研深度：8 轮 Web 搜索 + 学术文献 + 工业实践 + 社区讨论*
*总搜索结果：100+ 条*
*核心来源：arXiv 论文、OpenAI/Anthropic 官方博客、Reddit/HN 社区、开源项目文档*
