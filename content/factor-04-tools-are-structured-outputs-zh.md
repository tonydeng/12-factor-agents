[← 返回 README](../README-zh.md)

### 4. 工具只是结构化输出

工具不一定很复杂。从本质上讲，它们只是来自LLM的结构化输出，触发确定性代码执行。

![140-tools-are-just-structured-outputs](https://github.com/humanlayer/12-factor-agents/blob/main/img/140-tools-are-just-structured-outputs.png)

例如，假设你有两个工具`CreateIssue`和`SearchIssues`。要求LLM"使用多个工具之一"就是要求它输出我们可以解析为代表这些工具的对象的JSON。

```python

class Issue:
  title: str
  description: str
  team_id: str
  assignee_id: str

class CreateIssue:
  intent: "create_issue"
  issue: Issue

class SearchIssues:
  intent: "search_issues"
  query: str
  what_youre_looking_for: str
```

这个模式很简单：
1. LLM输出结构化JSON
2. 确定性代码执行适当的操作（如调用外部API）
3. 结果被捕获并反馈到上下文中

这在LLM的决策和应用程序的操作之间创建了清晰的分离。LLM决定做什么，但你的代码控制如何做。仅仅因为LLM"调用了工具"并不意味着你每次都必须以相同的方式执行相应的函数。

如果你回想我们上面的switch语句

```python
if nextStep.intent == 'create_payment_link':
    stripe.paymentlinks.create(nextStep.parameters)
    return # 或者你想要的任何操作，见下文
elif nextStep.intent == 'wait_for_a_while':
    # 做某种单子式的操作，我不知道
else: #... 模型调用了我们不知道的工具
    # 做其他事情
```

**注意**：关于"纯提示"vs"工具调用"vs"JSON模式"的好处已经有很多讨论，以及每个的性能权衡。我们很快会链接一些相关资源，但在这里不会深入讨论。参见[Prompting vs JSON Mode vs Function Calling vs Constrained Generation vs SAP](https://www.boundaryml.com/blog/schema-aligned-parsing)、[何时应该使用函数调用、结构化输出或JSON模式？](https://www.vellum.ai/blog/when-should-i-use-function-calling-structured-outputs-or-json-mode#:~:text=We%20don%27t%20recommend%20using%20JSON,always%20use%20Structured%20Outputs%20instead)和[OpenAI JSON vs函数调用](https://docs.llamaindex.ai/en/stable/examples/llm/openai_json_vs_function_calling/)。

"下一步"可能不像"运行一个纯函数并返回结果"那么简单。当你把"工具调用"想象成模型输出描述确定性代码应该做什么的JSON时，你解锁了很多灵活性。将这个与[因素8掌控你的控制流](./factor-08-own-your-control-flow-zh.md)结合起来。

[← 掌控你的上下文窗口](./factor-03-own-your-context-window-zh.md) | [统一执行状态 →](./factor-05-unify-execution-state-zh.md)
