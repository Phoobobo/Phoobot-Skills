---
name: herdr-agent-comm
description: 仅在独立 Herdr CLI（herdr）托管的会话中，用于向另一个同环境 agent 临时通知、提问或回复。只适用于 HERDR_ENV=1 且 HC_TERMINAL_ID 为空；Herdr Core（hc）、Herdr Desktop 会话工具和其他载体不命中。不负责派单或会话生命周期。
---

# Herdr Agent Comm（Herdr CLI only）

## 发送前

先检查 `test -z "${HC_TERMINAL_ID:-}" && test "${HERDR_ENV:-}" = 1 && command -v herdr >/dev/null`。任一条件不满足就停止本 skill，不用 `herdr` 探查聚焦会话，也不回退到其他工具。Herdr Core 由其自身的 `hc --skill` 指导，与本 skill 无关。

读取 `herdr` skill，按当前 `herdr --help` 和 `herdr agent` 帮助操作。用 `agent get` 核验实际发送者及接收者路由；不猜路由、不创建或改名会话。只发送用户授权范围内的消息。

## 每条消息都有来回对应

所有通信首行写 `[Prompt by <env> <provider> agent] <from> → <to>`，可附话题；通知、提问、中途澄清、等待期间的本会话回答及最终回报都遵守。首次联系时简短提醒对方也使用此格式。

- `env` 是本次消息使用的通信环境，此处固定为 Herdr，而不是操作系统或进程宿主。
- `provider` 是实际 agent 产品，如 Codex、Claude、Gemini；`from` / `to` 填实际双方名称。
- 通常使用同一通信环境的工具向 `from` 回复；只有名称不能定位时才补真实 ID。回复头填写自身的 env/provider，并交换 from/to，不照抄对方来源。

回复引用原话题或简短原问题摘要，例如“关于你问的‘配置从哪里读取’：……”。有多个相似问题时才加 `Q1` 等短标识，双方沿用即可。使用自然语言，不加 JSON 信封、持久台账、接单 ACK 或状态机。

## 普通通知

直接用 `herdr agent prompt <目标> <消息>` 发送，不带 `--wait`，不要求确认收到。消息文本须正确 shell quoting 或通过结构化参数传递。

## 需要答案：先短等询问

这里将“询问是否可以等待”解释为：**问接收消息的 agent 能否短时间答复、是否值得继续等**。用户另有明确指代时按用户指示。

将原问题和等待询问放在同一条消息，用 `agent prompt <目标> <消息> --wait --timeout 10000` 做一次短等。例如（假设 caller 使用 Codex、helper 使用 Claude，均通过 Herdr 通信；使用时替换为实际来源和双方名称）：

> [Prompt by Herdr Codex agent] caller → helper
>
> 配置从哪里读取？你能在约 30 秒内回答，值得我继续等吗？若需更久，请说明，并在答案准备好后通过 agent prompt 主动回报 caller，不带 --wait。你的回复首行请写 [Prompt by Herdr Claude agent] helper → caller，并引用本问题。

短等后用 `agent read <目标> --source recent-unwrapped --lines 120` 读取实际答复，再选择：

- 已给答案：核对它回答的是本问题，直接采用。
- 对方明确可很快回答：在约定的短时间内用 `agent wait <目标> --timeout 30000` 等待并读取；对方在其本会话输出答案即可，无需另发主动回报。
- 对方预计较久：明确约定答案准备好后向已核验的发送者名称/路由主动回报，然后继续其他工作。首条消息已提出该约定且对方接受时，无需重复发送。

`wait` 是等待工具检测到 idle/done/blocked 等状态，**不是答案，也不证明本问题完成**。尤其对正在工作的 agent，上一轮结束可能满足 wait；必须读内容并核对原问题。超时或状态不明先 `agent get` / `agent read`，不盲目重发、不把沉默当成同意，不无限续等。无法确认回报约定时说明尚未确认，稍后读取；没有可用回报路由时也明确采用稍后读取。

## 长问题的主动回报

预期耗时较长时，短等只用于确认答复方式，不原地等完整答案。完成后用不带 `--wait` 的 `agent prompt` 向约定目标发送答案；受阻也主动说明原问题、阻塞及缺少的信息。不等待对方再确认，避免互等。

> [Prompt by Herdr Claude agent] helper → caller
>
> 关于“配置从哪里读取”：入口是……，依据是……。

中途追问仍遵循同样的署名与问题呼应规则；需要答案时也先短等询问。发送成功只证明提交了消息；读取到与原问题对应的内容才算获得答案。
