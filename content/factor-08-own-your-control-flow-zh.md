[← 返回 README](../README-zh.md)

### 8. 掌控你的控制流

如果你掌控了你的控制流，你可以做很多有趣的事情。

![180-control-flow](../img/180-control-flow.png)

构建你自己的控制结构，使其适合你的特定用例。具体来说，某些类型的工具调用可能是跳出循环并等待人类或另一个长时间运行任务（如训练管道）响应的理由。你可能还想加入以下自定义实现：

- 工具调用结果的摘要或缓存
- 结构化输出的LLM评判
- 上下文窗口压缩或其他[内存管理](./factor-03-own-your-context-window-zh.md)
- 日志、追踪和指标
- 客户端速率限制
- 持久睡眠/暂停/"等待事件"

下面的示例展示了三种可能的控制流模式：

- request_clarification：模型请求更多信息，打破循环并等待人类响应
- fetch_git_tags：模型请求git标签列表，获取标签，追加到上下文窗口，然后直接传回给模型
- deploy_backend：模型请求部署后端，这是一个高风险操作，所以打破循环并等待人类批准

```python
def handle_next_step(thread: Thread):

  while True:
    next_step = await determine_next_step(thread_to_prompt(thread))

    # 为清晰而内联——实际上你可以把它放在方法中，
    # 使用异常进行控制流，或任何你想要的方式
    if next_step.intent == 'request_clarification':
      thread.events.append({
        type: 'request_clarification',
          data: nextStep,
        })

      await send_message_to_human(next_step)
      await db.save_thread(thread)
      # 异步步骤——跳出循环，稍后会收到webhook
      break
    elif next_step.intent == 'fetch_open_issues':
      thread.events.append({
        type: 'fetch_open_issues',
        data: next_step,
      })

      issues = await linear_client.issues()

      thread.events.append({
        type: 'fetch_open_issues_result',
        data: issues,
      })
      # 同步步骤——将新上下文传给LLM以确定下一个下一步
      continue
    elif next_step.intent == 'create_issue':
      thread.events.append({
        type: 'create_issue',
        data: next_step,
      })

      await request_human_approval(next_step)
      await db.save_thread(thread)
      # 异步步骤——跳出循环，稍后会收到webhook
      break
```

这种模式允许你在需要时中断和恢复智能体的流程，创建更自然的对话和工作流。

**示例**——我对每个AI框架的头号功能请求是，我们需要能够中断一个正在工作的智能体并在稍后恢复，特别是在工具**选择**时刻和工具**调用**时刻之间。

如果没有这种级别的可恢复性/细粒度，就无法在工具运行前进行审查/批准，这意味着你被迫：

1. 在等待长时间运行的事情完成时将任务暂停在内存中（想象`while...sleep`），如果进程被中断则从头重启
2. 将智能体限制为仅低风险、低风险的调用，如研究和摘要
3. 给智能体访问更大、更有用的东西，然后yolo希望它不会搞砸

你可能会注意到这与[第5条原则 - 统一执行状态和业务状态](./factor-05-unify-execution-state-zh.md)和[第6条原则 - 使用简单API进行启动/暂停/恢复](./factor-06-launch-pause-resume-zh.md)密切相关，但可以独立实现。

[← 通过工具联系人类](./factor-07-contact-humans-with-tools-zh.md) | [压缩错误 →](./factor-09-compact-errors-zh.md)
