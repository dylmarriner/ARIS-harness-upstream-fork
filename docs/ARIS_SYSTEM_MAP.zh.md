# ARIS 系统地图：各部分归属何处

[English](ARIS_SYSTEM_MAP.md) | 中文

- **状态：** 持续运行的 ARIS 运行时的规范归属指南
- **规范副本：** `dylmarriner/ARIS/docs/ARIS_SYSTEM_MAP.md`
- **镜像位置：** `ARIS-harness`、`ARIS-intelligence`、`scos-memory`（均位于 `docs/ARIS_SYSTEM_MAP.md`）
- **上级文档：** [`ARIS_MASTER_TECHNICAL_BLUEPRINT.md`](https://github.com/dylmarriner/ARIS/blob/main/docs/ARIS_MASTER_TECHNICAL_BLUEPRINT.md)
- **最近决定：** 2026-09-28

本页只回答一个问题：**ARIS 的某个部分应该放在哪个仓库？** 它不取代总体蓝图。两者冲突时，在总体蓝图更新之前以总体蓝图为准。先编辑规范副本，再在同一组变更中把它原样复制到三个镜像。ARIS-harness 的镜像另外带有该仓库的语言切换行和中文对侧文件。

## 1. 本地图描述的转变

ARIS 不是无状态的请求/响应式助手。旧模型是：

```text
user -> build request -> HTTP POST -> model -> response -> destroy execution context
```

ARIS 是一台持续运行的状态机，具备感知、记忆、认知和行动。持续维护的状态是系统的中心；推理是 ARIS 针对该状态发起的调用。小型原生模型不需要记住数周的运行过程，只需要基于从 ARIS 持续更新的世界状态中抽取的几千个 token 进行推理：

```text
current relevant state + relevant history + current objective + available actions
```

由于 ARIS 掌控操作系统，大部分感知是语义化的（`process.exited`、`unit.failed`、`network.changed`），而不是像素或音频。当语义信息不足时，才使用摄像头、麦克风和屏幕捕获。

## 2. 四个仓库

| 仓库 | 负责 | 不负责 |
| --- | --- | --- |
| `ARIS` | 操作系统与产品：镜像、安装器、shell、感知适配器、事件总线、主机状态（第一层）、特权 System Executor、节点 agent（智能体）、网关、打包、整机测试 | 目标与任务生命周期、认知世界模型、模型训练、持久记忆 |
| `ARIS-harness` | Executive：持续认知循环、目标/任务/计划、世界模型与注意力（第二层）、上下文组装、模型与 agent 路由、工具注册表、授权排序、验证、skill（技能） | 操作系统集成与特权执行、模型权重与训练、持久记忆存储 |
| `ARIS-intelligence` | 原生大脑及其求解 agent：模型训练、推理、评估、发布，以及 resourceful agent 运行时（有界工具循环、模型知识库、专家升级、自我改进循环） | ARIS Executive、全局目标与任务状态、特权操作系统执行、用户与系统的持久记忆 |
| `scos-memory` | 持久记忆：日志、情景时间线、语义记忆、来源追溯、受治理的晋升、检索、整合、OKF、记忆审计 | Executive、任务/工具编排、提示词构建、模型训练 |

**不再计划拆分更多仓库。** 只有当某个组件需要独立的发布节奏、不同的语言/运行时或独立的安全边界时，才值得新建仓库。在此之前，新工作以服务或包的形式放进这四个仓库之一。

## 3. 持续运行时流水线

```text
                         ARIS                                         ARIS-harness
 ┌──────────────────────────────────────────────┐   ┌──────────────────────────────────────────┐
 │ PERCEPTION ADAPTERS                          │   │ TIER 2: WORLD MODEL                      │
 │ procfs systemd journald udev sysfs mounts    │   │ entities, relationships, activities,     │
 │ NetworkManager PipeWire KDE/KWin filesystem  │   │ intentions, confidence, provenance       │
 │ microphone camera screen sensors             │   │                                          │
 │ Home Assistant                               │   │ ATTENTION / SALIENCE                     │
 │        │                                     │   │ goal relevance, urgency, novelty, risk   │
 │        ▼                                     │   │        │                                 │
 │ EVENT FABRIC  (NATS, aris.v1.*)              │   │   matters?──no──▶ update state, continue │
 │ SystemEventEnvelope                          │   │        │yes                              │
 │        │                                     │   │        ▼                                 │
 │        ▼                                     │   │ EXECUTIVE / COGNITIVE LOOP               │
 │ TIER 1: HOST STATE        (aris-eventd)      │   │ orient ▶ retrieve ▶ reason ▶ decide      │
 │ current OS state, raw ring buffers,          │──▶│        │                                 │
 │ cheap filter/coalesce: leave the host?       │   │        ├──▶ speak ──▶ ARIS Shell         │
 └──────────────────────────────────────────────┘   │        ├──▶ act ──▶ policy ▶ ARIS        │
                                                    │        │            System Executor      │
       ARIS-intelligence              scos-memory   │        └──▶ wait / remain silent         │
 ┌───────────────────────────┐ ┌──────────────────┐ └───────┬──────────────────┬───────────────┘
 │ native model inference    │ │ journal          │         │ cognition calls  │ memory calls
 │ /v1/generate              │◀┼──────────────────┼─────────┘                  │
 │ solver agent  /v1/solve   │ │ episodic,        │◀─────────────────────────────┘
 │ training, evals, releases │ │ semantic,        │
 │ model knowledge store     │ │ retrieval,       │
 └───────────────────────────┘ │ promotion        │
                               └──────────────────┘
```

流水线规则：

1. 大多数事件永远不会到达模型。第一层丢弃、合并事件或将其折叠进主机状态；第二层将其折叠进世界模型。只有注意力判定某事重要时才运行认知。
2. **等待是一等结果。** 只观察而不回应是正确行为，而不是失败。
3. 模型提出建议；Harness Executive 授权；ARIS System Executor 执行并独立重新校验。模型输出从不携带权限。
4. 所有应跨会话保留的内容都记入 SCOS Memory 的日志，并经其受治理的流水线晋升。

## 4. 归属表

| 部分 | 负责方 | 当前实现 | 状态 |
| --- | --- | --- | --- |
| Linux 感知（procfs、systemd、journald、udev、sysfs、挂载、NetworkManager、登录会话） | ARIS | `packages/domain` 数据源、`services/eventd` | 部分完成，已实机运行 |
| KDE/KWin 窗口观察 | ARIS | `shell/kwin/aris-window-observer` | 部分完成 |
| 麦克风、摄像头、屏幕捕获 | ARIS（发布 `observation.*` 的感知服务） | 无 | 计划中。所用的语音与视觉模型由 ARIS-intelligence 训练和发布 |
| Home Assistant 与智能家居设备 | ARIS（感知与执行器适配器） | 无 | 计划中 |
| 事件总线 | ARIS | `packages/events`（NATS）、`SystemEventEnvelope` | 已实现；持久 JetStream 消费待完成 |
| 第一层主机状态、原始环形缓冲区、低成本过滤 | ARIS | `aris-eventd`、`RecentEventBuffer` | 部分完成 |
| 第二层世界模型（信念图） | ARIS-harness | `packages/aris/runtime` 中的 `BeliefGraph` | 仅进程内，集成分支 |
| 注意力与显著性 | ARIS-harness | `AttentionGate` | 仅进程内，集成分支 |
| Executive、目标、任务、计划 | ARIS-harness | `ARISExecutive` | 集成分支 |
| 模型调用的上下文组装 | ARIS-harness | `<ARIS_STATE_CONTEXT>` 状态打包 | 部分完成 |
| 权限、策略、审批 | ARIS-harness（决定）、ARIS（重新校验） | `ApprovalPolicyEngine` | 部分完成 |
| 特权执行 | ARIS | 仅有 `services/system-agent` 的 README | 计划中 |
| 说话（语音、通知、界面） | ARIS Shell | 无 | 计划中 |
| 原生模型推理 | ARIS-intelligence | llama.cpp 后端、提供方 `/v1/generate`、独立服务 | 已实现 |
| 求解 agent（工具循环、模型知识库、专家升级） | ARIS-intelligence | `agent/resourceful.py`、`runtime/resourceful.py`、提供方 `/v1/solve` | 已实现，支持独立模式与提供方模式 |
| 自适应测试时计算、验证器、搜索 | ARIS-intelligence | `orchestrator/controller.py`、`verifier/`、`search/`、`reasoning/` | 已实现 |
| 模型训练、评估、冠军/挑战者 | ARIS-intelligence | `learning/`、`evaluation/`、`benchmarks/` | 已实现，已在 2B 开发模型上实机运行 |
| 持久记忆、情景时间线、检索 | scos-memory | `memory-gateway` 之后的 Go 服务 | 已实现 |
| skill（过程式学习） | ARIS-harness 负责运行；ARIS-intelligence 评估原生模型策略 | 两处均有 WikiSkill | 部分完成 |

## 5. 2026-09-28 记录的决定

### 5.1 世界状态分为两层

**第一层，ARIS（`aris-eventd`）：** 贴近硬件的原始主机状态和环形缓冲区。它用低成本、确定性的过滤与合并回答“这个事件是否应该离开主机？”，并发布规范化事件。

**第二层，ARIS-harness：** 认知世界模型与注意力。它回答“这件事是否关系到某个目标，ARIS 应该说话、行动还是等待？”它保存实体、关系、活动、意图、置信度和来源追溯。

第一层从不决定 ARIS 应该做什么。第二层从不直接读取硬件。

### 5.2 ARIS-intelligence 负责求解 agent 运行时

原生大脑自带一个有界 agent：resourceful 求解器。它在非特权工具（精确计算器、沙箱化 Python、网页搜索与页面阅读、`command_help` 手册查询与命令检查）上运行工具循环；当原生模型力不能及时升级到外部专家；验证答案；并从已验证的结果中学习。它维护一个由已验证事实和策略组成的**模型知识库**，用于改进原生模型。它可以独立运行（`aris-intelligence ask`/`chat`），也可以作为提供方运行（`POST /v1/solve`）。

这不会产生第二个 Executive。边界如下：

- Harness 决定解决**什么**问题，以及结果**是否**可以付诸行动。Intelligence 决定**如何**解决一个被委派的问题。
- 求解器的工具都是非特权的。任何会改变系统的操作都作为提议返回给 Harness Executive，由其授权；再由 ARIS System Executor 执行。
- 模型知识库不是用户或系统的持久记忆。生活历史、任务历史以及用户期望 ARIS 记住的任何内容都属于 SCOS Memory。

### 5.3 多模态感知位于 ARIS

麦克风、摄像头和屏幕捕获是 ARIS 的感知适配器，像其他传感器一样向事件总线发布 `observation.*` 事件。它们使用的语音与视觉模型由 ARIS-intelligence 训练、评估和发布，由 ARIS 部署。

## 6. 我的改动应该放在哪里？

按顺序提问，遇到第一个“是”即停止：

1. 是否涉及硬件、操作系统、特权、打包或 shell？**ARIS。**
2. 是否跨时间决定优先级、负责目标或任务、维护世界模型或授权行动？**ARIS-harness。**
3. 是否训练、运行、评估或发布模型，或使用非特权工具解决一个被委派的问题？**ARIS-intelligence。**
4. 是否必须带着来源追溯和治理跨会话持久记住？**scos-memory。**

如果似乎有两个答案都适用，那么这个组件很可能其实是两个组件：应在接口处拆分，而不是把它复制到两个仓库。

## 7. 仍待解决的已知重叠

| 重叠 | 仓库 | 方向 |
| --- | --- | --- |
| Harness 通过临时的 OpenAI `chat/completions` 调用原生模型，而不是 ARIS-intelligence 提供方协议 | ARIS-harness、ARIS-intelligence | Harness 采用 `/v1/generate` 和 `/v1/solve`（协议 v1.0） |
| SCOS 暴露了 `/v1/context/compile`，但提示词构建归 Harness 负责 | scos-memory、ARIS-harness | SCOS 返回排序后的证据；Harness 构建提示词 |
| 求解器的模型知识库与 SCOS Memory 之间没有定义交换方式 | ARIS-intelligence、scos-memory | 定义对 SCOS 证据的读取访问，以及从已验证求解事实出发的候选晋升路径 |
| `docs/architecture/overview.md` 仍描述仓库内的“ARIS Core”编排器 | ARIS | 改写为指向 Harness Executive |
| `aris-eventd` 只发布已升级的事件 | ARIS | 发布规范化事件流，使第二层能够维护世界模型 |
