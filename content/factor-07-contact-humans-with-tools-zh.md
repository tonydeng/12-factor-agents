[← 返回 README](https://github.com/humanlayer/12-factor-agents/blob/main/README-zh.md)

### 7. 通过工具调用联系人类

默认情况下，LLM API依赖于一个基本的高风险token选择：我们是返回纯文本内容，还是返回结构化数据？

![170-contact-humans-with-tools](https://github.com/humanlayer/12-factor-agents/blob/main/img/170-contact-humans-with-tools.png)

你在第一个token的选择上承担了很大的权重，在"the weather in tokyo"的情况下，它是

> "the"

但在"fetch_weather"的情况下，它是一些特殊token来表示JSON对象的开始。

> |JSON>

通过让LLM*始终*输出JSON，然后用一些自然语言token声明其意图，比如`request_human_input`或`done_for_now`（而不是"proper"工具如`check_weather_in_city`），你可能会得到更好的结果。

同样，你可能不会从中获得任何性能提升，但你应该尝试，并确保你可以自由尝试奇怪的东西以获得最佳结果。

```python

class Options:
  urgency: Literal["low", "medium", "high"]
  format: Literal["free_text", "yes_no", "multiple_choice"]
  choices: List[str]

# 用于人类交互的工具定义
class RequestHumanInput:
  intent: "request_human_input"
  question: str
  context: str
  options: Options

# 智能体循环中的示例用法
if nextStep.intent == 'request_human_input':
  thread.events.append({
    type: 'human_input_requested',
    data: nextStep
  })
  thread_id = await save_state(thread)
  await notify_human(nextStep, thread_id)
  return # 跳出循环并等待响应返回线程ID
else:
  # ... 其他情况
```

稍后，你可能会收到来自处理slack、email、sms或其他事件系统的webhook。

```python

@app.post('/webhook')
def webhook(req: Request):
  thread_id = req.body.threadId
  thread = await load_state(thread_id)
  thread.events.push({
    type: 'response_from_human',
    data: req.body
  })
  # ... 为简洁而简化，你可能不想在这里阻塞web worker
  next_step = await determine_next_step(thread_to_prompt(thread))
  thread.events.append(next_step)
  result = await handle_next_step(thread, next_step)
  # todo - 循环或中断或任何你想要的

  return {"status": "ok"}
```

以上包含了来自[第5条原则 - 统一执行状态和业务状态](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-05-unify-execution-state.md)、[第8条原则 - 掌控你的控制流](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-08-own-your-control-flow.md)、[第3条原则 - 掌控你的上下文窗口](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)和[第4条原则 - 工具只是结构化输出](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs.md)的模式，以及其他几个原则。

如果我们使用[第3条原则 - 掌控你的上下文窗口](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)中的XML格式，在几轮对话后我们的上下文窗口可能看起来像这样：

```xml

（为简洁而省略）

<slack_message>
    From: @alex
    Channel: #deployments
    Text: Can you deploy backend v1.2.3 to production?
    Thread: []
</slack_message>

<request_human_input>
    intent: "request_human_input"
    question: "Would you like to proceed with deploying v1.2.3 to production?"
    context: "This is a production deployment that will affect live users."
    options: {
        urgency: "high"
        format: "yes_no"
    }
</request_human_input>

<human_response>
    response: "yes please proceed"
    approved: true
    timestamp: "2024-03-15T10:30:00Z"
    user: "alex@company.com"
</human_response>

<deploy_backend>
    intent: "deploy_backend"
    tag: "v1.2.3"
    environment: "production"
</deploy_backend>

<deploy_backend_result>
    status: "success"
    message: "Deployment v1.2.3 to production completed successfully."
    timestamp: "2024-03-15T10:30:00Z"
</deploy_backend_result>
```

好处：

1. **清晰的指令**：不同类型人类联系的工具允许LLM更具体地表达
2. **内循环vs外循环**：支持传统chatGPT风格界面**之外**的智能体工作流，其中控制流和上下文初始化可能是`智能体→人类`而不是`人类→智能体`（想象一下，由cron或事件触发的智能体）
3. **多人访问**：可以通过结构化事件轻松跟踪和协调来自不同人类的输入
4. **多智能体**：简单的抽象可以轻松扩展以支持`智能体→智能体`请求和响应
5. **持久性**：结合[第6条原则 - 使用简单API进行启动/暂停/恢复](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-06-launch-pause-resume-zh.md)，这可以创建持久、可靠和可内省的多人工作流

[更多关于外循环智能体的内容](https://theouterloop.substack.com/p/openais-realtime-api-is-a-step-towards)

![175-outer-loop-agents](https://github.com/humanlayer/12-factor-agents/blob/main/img/175-outer-loop-agents.png)

与[第11条原则 - 从任何地方触发，在用户所在的地方相遇](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-11-trigger-from-anywhere-zh.md)配合使用效果很好。

[← 启动/暂停/恢复](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-06-launch-pause-resume-zh.md) | [掌控你的控制流 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-08-own-your-control-flow-zh.md)
