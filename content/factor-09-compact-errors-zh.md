[← 返回 README](https://github.com/humanlayer/12-factor-agents/blob/main/README.md)

### 9. 将错误压缩到上下文窗口中

这一条篇幅较短，但值得一说。智能体的优势之一是"自愈"功能——对于短期任务，LLM可能会调用一个失败的工具。优秀的LLM有相当大的概率阅读错误消息或堆栈跟踪，并找出在后续工具调用中需要更改的内容。

大多数框架都实现了这一点，但你可以在不采用其他11个因素的情况下单独实现这一功能。例如：

```python
thread = {"events": [initial_message]}

while True:
  next_step = await determine_next_step(thread_to_prompt(thread))
  thread["events"].append({
    "type": next_step.intent,
    "data": next_step,
  })
  try:
    result = await handle_next_step(thread, next_step) # 我们的 switch 语句
  except Exception as e:
    # 如果遇到错误，可以将其添加到上下文窗口并重试
    thread["events"].append({
      "type": 'error',
      "data": format_error(e),
    })
    # 循环，或者在这里执行其他恢复逻辑
```

你可能需要为特定的工具调用实现一个 errorCounter，限制单个工具的重试次数约为3次，或者根据你的用例实现其他合理的逻辑。

```python
consecutive_errors = 0

while True:

  # ... 现有代码 ...

  try:
    result = await handle_next_step(thread, next_step)
    thread["events"].append({
      "type": next_step.intent + '_result',
      data: result,
    })
    # 成功！重置错误计数器
    consecutive_errors = 0
  except Exception as e:
    consecutive_errors += 1
    if consecutive_errors < 3:
      # 执行循环并重试
      thread["events"].append({
        "type": 'error',
        "data": format_error(e),
      })
    else:
      # 跳出循环，重置上下文窗口的部分内容，升级给人工处理，或者执行其他你想要的操作
      break
  }
}
```

达到某个连续错误阈值可能是一个很好的[升级给人工](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools-zh.md)的时机，无论是通过模型决策还是通过确定性接管控制流。

[![195-factor-09-errors](https://github.com/humanlayer/12-factor-agents/blob/main/img/195-factor-09-errors.gif)](https://github.com/user-attachments/assets/cd7ed814-8309-4baf-81a5-9502f91d4043)


<details>
<summary>[GIF 版本](https://github.com/humanlayer/12-factor-agents/blob/main/img/195-factor-09-errors.gif)</summary>

![195-factor-09-errors](https://github.com/humanlayer/12-factor-agents/blob/main/img/195-factor-09-errors.gif)

</details>

优势：

1. **自愈能力**：LLM可以阅读错误消息并找出在后续工具调用中需要更改的内容
2. **持久性**：即使某个工具调用失败，智能体也可以继续运行

我敢肯定你会发现，如果你这样做太多次，你的智能体就会开始失控并可能一次又一次地重复相同的错误。

这正是[因素8 - 掌控你的控制流](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-08-own-your-control-flow-zh.md)和[因素3 - 掌控你的上下文构建](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window-zh.md)的用武之地——你不需要只是把原始错误放回去，你可以完全重构它的表示方式，从上下文窗口中删除之前的事件，或者执行任何你发现有效的确定性操作来让智能体重新走上正轨。

但防止错误失控的首要方法是采用[因素10 - 小而专注的智能体](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-10-small-focused-agents-zh.md)。

[← 掌控你的控制流](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-08-own-your-control-flow-zh.md) | [小而专注的智能体 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-10-small-focused-agents-zh.md)
