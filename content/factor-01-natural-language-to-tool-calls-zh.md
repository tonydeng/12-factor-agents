[← 返回 README](https://github.com/humanlayer/12-factor-agents/blob/main/README-zh.md)

### 1. 自然语言到工具调用

在智能体构建中最常见的模式之一是将自然语言转换为结构化的工具调用。这是一个强大的模式，允许你构建能够推理任务并执行任务的智能体。

![110-natural-language-tool-calls](https://github.com/humanlayer/12-factor-agents/blob/main/img/110-natural-language-tool-calls.png)

当原子化应用时，这个模式就是简单地将如下短语

> 你能创建一个750美元的支付链接给Terri，用于赞助二月份的AI技术爱好者聚会吗？

转换为描述Stripe API调用的结构化对象

```json
{
  "function": {
    "name": "create_payment_link",
    "parameters": {
      "amount": 750,
      "customer": "cust_128934ddasf9",
      "product": "prod_8675309",
      "price": "prc_09874329fds",
      "quantity": 1,
      "memo": "Hey Jeff - see below for the payment link for the february ai tinkerers meetup"
    }
  }
}
```

**注意**：实际上Stripe API稍微复杂一些，一个[真正这样做的智能体](https://github.com/dexhorthy/mailcrew)（[视频](https://www.youtube.com/watch?v=f_cKnoPC_Oo)）会列出客户、列出产品、列出价格等来构建这个带有正确ID的有效载荷，或者在提示/上下文窗口中包含那些ID（我们将在下面看到这些实际上是类似的东西！）

从这里开始，确定性代码可以获取有效载荷并对其执行操作。（更多内容见[因素3](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window.md)）

```python
# LLM接收自然语言并返回结构化对象
nextStep = await llm.determineNextStep(
  """
  create a payment link for $750 to Jeff 
  for sponsoring the february AI tinkerers meetup
  """
  )

# 根据其函数处理结构化输出
if nextStep.function == 'create_payment_link':
    stripe.paymentlinks.create(nextStep.parameters)
    return  # 或者你想要的任何操作，见下文
elif nextStep.function == 'something_else':
    # ... 更多情况
    pass
else:  # 模型调用了我们不知道的工具
    # 做其他事情
    pass
```

**注意**：虽然一个完整的智能体会接收API调用结果并与之循环，最终返回类似

> 我已成功为Terri创建了一个750美元的支付链接，用于赞助二月份的AI技术爱好者聚会。链接如下：https://buy.stripe.com/test_1234567890

**但实际上**，我们在这里要跳过这一步，将其留到另一个因素中，你可能想也可能不想包含（由你决定！）

[← 我们是如何走到这里的](https://github.com/humanlayer/12-factor-agents/blob/main/content/brief-history-of-software-zh.md) | [掌控你的提示词 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-02-own-your-prompts-zh.md)
