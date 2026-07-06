# 架构介绍（主 agent + 多个子 agent）

概述
- 架构类型：集中式协调（Coordinator — 主 agent）+ 专职执行的子 agent（Domain specialists）。
- 基于仓库配置要点：agents.list 中定义了一个 id 为 `main` 的主 agent（允许子 agent 列表 `pm`/`frontend`/`backend`/`qa`），`agents.defaults` 包含 `subagents` 配置项，且 `tools.agentToAgent.enabled = true`（允许 agent 间通信），表明主-从协调与 agent 间直接通信并存的混合多 agent 架构。

关键组件
- 网关（Gateway）
  - 负责外部接入、认证与路由（配置项示例：gateway.port、mode、auth.token、trustedProxies）。
- 主 agent（main）
  - 主要职责：会话路由与分发、起始决策、聚合子 agent 结果、流控与安全边界（在 agents.list 中 `main` 配置 `subagents.allowAgents`）。
  - 拥有协调权限：可创建/唤醒子 agent，会话管理（dmScope、session.maintenance 等）。
- 子 agent（pm、frontend、backend、qa）
  - 各自为角色型专家：在 agents.list 中每个子 agent 有独立 `workspace` 与 `agentDir`，使用同一或各自模型（model.primary）。
  - 职责划分：按角色处理具体任务（例如 pm 做需求/排期，frontend 处理界面实现建议，backend 处理接口/架构，qa 负责测试策略与用例）。
- SafeRoom（saferoom）
  - 专用安全上下文 agent：配置了单独 workspace、agentDir 与安全模型提供者（kubeplex-maas-saferoom），可用于处理敏感数据或合规受限的请求。
- 模型提供与内存（Models & Memory）
  - 远端模型提供者（providers，示例：kubeplex-maas），共享主模型或为特定 agent 提供差异化模型。
  - `memorySearch`、`catpaw-memory` 插件：用于会话记忆、检索与长期存储（agents.defaults.memorySearch、plugins.entries.catpaw-memory）。
- 插件与技能（Plugins & Skills）
  - 插件（message-logger-plugin、catpaw-memory、catclawsec 等）为 agent 提供日志、记忆与安全钩子。
  - skills 目录支持按需加载自定义技能（/app/skills）。
- 通道桥接（Channels）
  - 外部消息桥接（channels.daxiang 等）支持将外部消息转发到主 agent，配置了 `safeRoomAgentId` 用于敏感消息路由到 saferoom。

通信与协调模式
- 主 agent 作为协调中心：接收外部请求后依据策略将任务分配给一个或多个子 agent；主 agent 负责合并结果与最终回复。
- 中央会话/记忆存储：`memorySearch.sources` 包含 `memory` 与 `sessions`，插件（catpaw-memory）为跨 agent 的上下文共享提供支持。
- 排队与中断：`messages.queue.mode = interrupt`，允许新高优先级消息中断当前处理。
- 循环检测与断路：`loopDetection.enabled = true` 且存在 `globalCircuitBreakerThreshold`，防止 agent 间出现无限循环调用。

数据流（简化）
1. 外部用户 -> 网关（认证与路由）
2. 网关 -> 主 agent（主 agent 根据消息类型/策略决定路由）
3. 主 agent -> 一个或多个子 agent（并行或串行调用）
4. 子 agent -> 模型提供者（生成响应）、/ 插件（记忆/日志/安全检查）
5. 子 agent 返回结果 -> 主 agent 汇总 -> 网关 -> 用户
（敏感数据路径：若消息标记或通道配置要求，路由到 `saferoom` agent 处理后返回）

并发、伸缩与限流
- 单机并发配置：`agents.defaults.maxConcurrent`（默认 4），`subagents.maxConcurrent`（默认 8），结合 `timeoutSeconds`（1800s）控制并发 & 超时。
- 心跳与空闲回收：`heartbeat.every`（48h）、`llm.idleTimeoutSeconds` 等用于检测与回收空闲资源。
- 伸缩策略：可以通过调整 `maxConcurrent`、为模型提供者增加并发实例、或水平扩展 agent 运行实例来扩展处理能力。

容错与稳定性
- 循环检测（loopDetection）和全局断路器限制防止递归调用耗尽资源。
- 会话保留策略（cron.sessionRetention）与 compaction（压缩/记忆 flush）减少长期会话数据占用。
- 日志和告警插件（message-logger-plugin）用于检测 agent 卡顿并触发告警 webhook。
- 网关 `trustedProxies` 与 `auth.token` 提供基本接入保护。

安全与合规
- 安全 agent（saferoom）与专用模型 provider（kubeplex-maas-saferoom）用于处理敏感/合规请求；headers 中可包含 `X-SafeRoom-Token`。
- 网关 token 验证（`gateway.auth.mode = token`）与 `trustedProxies` 控制访问来源。
- 插件钩子（`allowConversationAccess`）与 message-logger/security 插件对会话访问进行保护或审计。

角色分工建议（基于仓库配置）
- main（主 agent）
  - 接受外部请求、路由给适当子 agent、负责结果聚合与最终决策、管理会话生命周期、处理策略（比如哪些场景交给 saferoom）。
- pm（项目经理 agent）
  - 需求澄清、优先级评估、与其他 agent 协商交付计划。
- frontend（前端 agent）
  - 提供界面实现建议、样式/组件推荐、审查 UI 变更的可行性。
- backend（后端 agent）
  - 设计 API、数据库方案、性能建议、接口契约与示例代码。
- qa（测试 agent）
  - 生成测试用例、自动化测试脚本、回归测试策略与质量评估。
- saferoom（安全/合规 agent）
  - 专门处理包含敏感信息或需要严格审计的对话，使用受控模型与存储。

示例交互场景（简短）
- 用例：用户在外部通道请求“实现某接口”。
  1. 网关 -> main（发现是实现任务）；
  2. main 并行派发给 backend（接口实现细节）与 qa（测试用例）；
  3. backend 调用模型生成代码片段并写入 workspace；qa 生成测试用例；
  4. main 汇总并返回最终建议；若请求涉及敏感数据，主 agent 将该会话路由到 saferoom 处理。

实施建议（基于当前配置的可行改进）
- 为主 agent 明确“编排策略”模块：定义何时广播任务、何时串行执行、超时 & 回退策略（fallback）。
- 增强监控：利用 message-logger-plugin 的 agentStuck webhook + 心跳监控，建立告警面板。
- 权限与审计：对 channels.bridge 与 safeRoom 的权限进行更细粒度控制（限制 `allowFrom` 而非 "*"）。
- 测试与回放：使用 sessionRetention 和会话快照功能做端到端回放，便于调试 agent 协同问题。
- 文档化 prompts：在 `agents/dev-team/prompts` 中为每个角色提供标准化 prompt 模板（例如：pm-onboard.md、backend-implement-api.md），以保证子 agent 输出可预测且一致。

ASCII 架构图
User -> Gateway -> Main Agent
                 ├─> Subagent: pm
                 ├─> Subagent: frontend
                 ├─> Subagent: backend
                 ├─> Subagent: qa
                 └─> Saferoom (for sensitive)
Agents <-> Model Provider(s)
Agents <-> Plugins (memory/logger/security)

---

（基于仓库的 agents/openclaw.md 中的配置项撰写）
