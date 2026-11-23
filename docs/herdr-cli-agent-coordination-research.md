# 独立 Herdr CLI：内置了 agent 协作吗？

**结论：v0.9.5 内置了足够搭建协作的终端和 agent 控制原语，但没有找到 `hc` 式的任务委派闭环。** 一个 agent 可以启动另一个 agent、向它提交提示、等待状态并读取输出；首任务何时发、谁该收回复、何时收齐结果、何时清理，则需要调用者自己编排。

## 定版和范围

2026-11-21 UTC 查询 [GitHub 官方 `/releases/latest` API](https://api.github.com/repos/herdrdev/herdr/releases/latest)：[v0.9.5](https://github.com/herdrdev/herdr/releases/tag/v0.9.5)，release ID `412918418`，`published_at=2026-10-14T16:07:28Z`，`draft=false`、`prerelease=false`。在临时目录检出该 tag，解析到 commit [`2ecf19a92fc3a7fff75b6ce3655ef04f4625ec01`](https://github.com/herdrdev/herdr/tree/2ecf19a92fc3a7fff75b6ce3655ef04f4625ec01)；[Cargo.toml](https://github.com/herdrdev/herdr/blob/v0.9.5/Cargo.toml#L1-L8) 也标为 0.9.5。只调查 `herdrdev/herdr` 的 `herdr`；`hc` 对照来自本机 `hc 0.1.21` 的 `--help` / `--skill`，不把 Core 或 Desktop 的能力归到独立 CLI。以下链接均固定在发布 tag；`docs/next` 是该 tag 内的目录名，不引用滚动站点。静态核对源码和测试断言，未安装、未运行上游测试或开启实验会话。

## 逐项核对

### 首任务：有启动就绪检查，没有绑定到启动的任务投递

官方[自动化示例](https://github.com/herdrdev/herdr/blob/v0.9.5/docs/next/website/src/content/docs/agent-automation.mdx#L100-L118) 明确分为 `pane split` → `agent start` → `agent prompt … --wait` → `agent read`。`agent start` 要有可用 shell pane；它启动指定 kind，支持透传 agent 自身 argv，CLI 等待同一 pane 中预期 agent 可交互，遇启动阻塞则报错并留名供检查。见[启动 CLI](https://github.com/herdrdev/herdr/blob/v0.9.5/src/cli/agent.rs#L370-L434)、[启动实现](https://github.com/herdrdev/herdr/blob/v0.9.5/src/app/agents.rs#L145-L236)和[启动阻塞测试](https://github.com/herdrdev/herdr/blob/v0.9.5/tests/cli/agent_transport.rs#L4)。透传某个 agent 自己的初始提示参数，即使可用，也不是 Herdr 提供的通用就绪后投递。

`agent prompt` 是独立请求：对目标 pane 写入文本和 Enter，已知 `blocked` 时拒绝写入；`--wait` 将提交和等待合为一次请求，不是把启动和首任务合为一次请求。见[实现及测试](https://github.com/herdrdev/herdr/blob/v0.9.5/src/app/api/agents.rs#L599)和[传输测试](https://github.com/herdrdev/herdr/blob/v0.9.5/tests/cli/agent_transport.rs#L220)。对照 `hc run --bg … <task>` 可随启动托管首任务、等待就绪后尝试投递，模板可拼接角色指令，并用 `first_message.sent`/`reason` 记录结果；未找到 Herdr 对等的启动任务字段、投递记录或失败状态。

### 发送方、回复和报告：可传文本，不等于消息路由

`agent prompt <target> <text>` 可作为 agent 对 agent 的输入通道；也有更底层的 `pane send-text`、`send-keys` / `send-input`。但 [AgentStart/Prompt/Info schema](https://github.com/herdrdev/herdr/blob/v0.9.5/src/api/schema/agents.rs#L167) 中，prompt 只有目标、正文和可选等待；目标名称或 pane ID 是**收件地址**，不是发送方、父任务或回复地址。调用者可以在正文里自填回报命令，或约定结果文件；没有找到原生发送方标记、回复路由或与任务关联的消息收件箱。历史命令名也不能作为证据：[CLI 测试明确拒绝已移除的 `agent send`](https://github.com/herdrdev/herdr/blob/v0.9.5/tests/cli/surface.rs#L352)。`hc send` 则有 `<hc from="…">`，首任务可带 `reach="hc send … when done or stuck"`；实际结果仍由子 agent 主动发送，不是 core 替它总结。

Herdr 的 [状态报告和事件 API](https://github.com/herdrdev/herdr/blob/v0.9.5/docs/next/website/src/content/docs/socket-api.mdx#L656) 能用 `pane.report_agent` 更新语义状态、订阅生命周期事件，也能通过 `pane.report_agent_session` 上报会话引用，或用 `pane.report_metadata` 展示 token。这里的 `source` 是集成/状态来源，展示 token 也不是发给父 agent 的结果报告。[hook 测试](https://github.com/herdrdev/herdr/blob/v0.9.5/tests/cli/hooks.rs#L131) 还明确覆盖忽略 Claude 原生 subagent 完成报告。可以 `agent read` 获取界面输出、用 `pane wait-output` 匹配标记，或让外部事件消费者处理状态；未找到像 `hc` 那样在子 agent blocked/error/exited 时向父 agent 的输入自动送达通知。

### 等待多个 agent：有单目标状态等待和通用事件，没有子任务聚合

[Socket API](https://github.com/herdrdev/herdr/blob/v0.9.5/docs/next/website/src/content/docs/socket-api.mdx#L103-L124) 的 `agent.wait` 是服务端事件驱动的单目标等待，绑定解析后的 pane occupant，不会因同名替换就改等新 agent；[测试也覆盖别的 pane 状态不触发该等待，以及目标 pane 关闭时报错](https://github.com/herdrdev/herdr/blob/v0.9.5/tests/cli/agents.rs#L1298)。重复 `--until` 是**同一目标**的多个候选状态，任一匹配即返回，[并非多目标](https://github.com/herdrdev/herdr/blob/v0.9.5/tests/cli/agent_options.rs#L4)。`prompt --wait` 从非 working 状态发起时，还要先观察到活动，之后默认等 `idle`、`done` 或 `blocked`；[随包说明明确说它不是逐 turn 跟踪](https://github.com/herdrdev/herdr/blob/v0.9.5/skills/herdr/SKILL.md#L137-L171)。状态匹配不是收到任务结果；`blocked` 也可能只是需要人工处理。

外部程序可以并发发起单目标等待，或[订阅事件](https://github.com/herdrdev/herdr/blob/v0.9.5/docs/next/website/src/content/docs/socket-api.mdx#L775)自行维护多个目标、实现 any/all；未找到 Herdr 内建的 `hc wait --children any|all`、来信中断等待并返回 `reason: message`，或用首任务投递进度对该聚合做判定。

### 子生命周期：pane 布局归属不是委派关系

Herdr 管理 session/workspace/tab/pane，agent 占用 pane；可以显式关闭 pane 或更大的布局容器（[命令/API 面](https://github.com/herdrdev/herdr/blob/v0.9.5/docs/next/website/src/content/docs/socket-api.mdx#L99-L114)，[关闭操作注意事项](https://github.com/herdrdev/herdr/blob/v0.9.5/skills/herdr/SKILL.md#L209)）。分屏和指定目标 pane 并不登记发起 agent 为任务父级；没有找到关闭/退出发起 agent 时按委派关系清理子 agent，或 `--outlive-parent` 一类例外。`hc` 则把调用者启动的 terminal 视为 children，默认随父 terminal 结束，可显式脱离；这不同于关闭共同容器导致容器内 pane 一并关闭，也不等于清扫所有脱离 terminal 的 OS 进程。

## 未找到的边界

对照了发布版的 CLI 分发、Agent/Pane/Events schema、服务端启动/输入/等待路径、自动化文档、随包 skill 及相关测试；以上“未找到”限于该稳定版内建接口及其语义，不外推到其他 release、主分支、插件、agent 厂商自己的 subagents，或第三方编排器。选型上，如果只需在多个 pane 里发提示、看状态、抓输出，独立 Herdr 已有原语；若依赖自动首任务、带回报地址的消息、父级通知、子任务聚合等待和父子退出约束，不应把这些原语当成 `hc` 的直接替代。
