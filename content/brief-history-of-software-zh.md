[← 返回 README](https://github.com/humanlayer/12-factor-agents/blob/main/README.md)

## 完整版本：我们是如何走到这里的

### 你不必听我说

无论你是 AI Agent 的新手，还是像我一样顽固的老兵，我都要说服你抛弃你对 AI Agent 的大部分认知，退后一步，从第一性原理重新思考它们。（如果你没有注意到几周前 OpenAI 发布的响应，但往 API 背后推更多 Agent 逻辑并不是解决方案）

## Agent 就是软件，以及软件的简史

让我们聊聊我们是如何走到这里的

### 60 年前

我们会经常讨论有向图（DGs）及其无环的朋友——DAGs。我首先要指出的是……软件本质上就是一个有向图。我们过去用流程图来表示程序是有原因的。

![010-software-dag](https://github.com/humanlayer/12-factor-agents/blob/main/img/010-software-dag.png)

### 20 年前

大约 20 年前，我们开始看到 DAG 编排器变得流行。我们谈论的是经典作品如 [Airflow](https://airflow.apache.org/)、[Prefect](https://www.prefect.io/)，以及一些前辈和更新的如（[dagster](https://dagster.io/)、[inngest](https://www.inngest.com/)、[windmill](https://www.windmill.dev/)）。这些遵循相同的图模式，并增加了可观测性、模块化、重试、管理等功能。

![015-dag-orchestrators](https://github.com/humanlayer/12-factor-agents/blob/main/img/015-dag-orchestrators.png)

### 10-15 年前

当机器学习模型开始变得足够好、能够派上用场时，我们开始看到 DAG 中嵌入了 ML 模型。你可能会想象这样的步骤："将这列中的文本总结到新列"或"按严重程度或情感对支持问题进行分类"。

![020-dags-with-ml](https://github.com/humanlayer/12-factor-agents/blob/main/img/020-dags-with-ml.png)

但归根结底，这仍然大多是相同的老式确定性软件。

### Agent 的前景

我不是第一个[这样说的人](https://youtu.be/Dc99-zTMyMg?si=bcT0hIwWij2mR-40&t=73)，但当我开始了解 Agent 时，我最大的收获是——你可以把 DAG 扔掉了。不再需要软件工程师编码每个步骤和边界情况，你可以给 Agent 一个目标和一组转换：

![025-agent-dag](https://github.com/humanlayer/12-factor-agents/blob/main/img/025-agent-dag.png)

然后让 LLM 实时做出决策，找出路径

![026-agent-dag-lines](https://github.com/humanlayer/12-factor-agents/blob/main/img/026-agent-dag-lines.png)

这里的前景是：你写的软件更少了，你只需要给 LLM 图的"边"，让它自己找出节点。你可以从中恢复错误，可以写更少的代码，而且你可能会发现 LLM 能找到解决问题的全新方案。

### 作为循环的 Agent

换句话说，你有一个由 3 个步骤组成的循环：

1. LLM 决定工作流的下一个步骤，输出结构化 JSON（"工具调用"）
2. 确定性代码执行工具调用
3. 结果被追加到上下文窗口
4. 重复直到下一个步骤被判定为"完成"

```python
initial_event = {"message": "..."}
context = [initial_event]
while True:
  next_step = await llm.determine_next_step(context)
  context.append(next_step)

  if (next_step.intent === "done"):
    return next_step.final_answer

  result = await execute_step(next_step)
  context.append(result)
```

我们的初始上下文只是起始事件（可能是用户消息，可能是 cron 触发，也可能是 webhook 等），然后我们让 LLM 选择下一步（工具）或确定我们已完成了。

这是一个多步骤示例：

[![027-agent-loop-animation](https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-animation.gif)](https://github.com/user-attachments/assets/3beb0966-fdb1-4c12-a47f-ed4e8240f8fd)

<details>
<summary><a href="https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-animation.gif">GIF 版本</a></summary>

![027-agent-loop-animation](https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-animation.gif)

</details>

而生成的"物化"DAG 大致如下：

![027-agent-loop-dag](https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-dag.png)

### 这种"循环直到解决"模式的问题

这个模式最大的问题：

- 当上下文窗口变得太长时，Agent 会迷失方向——它们会一遍又一遍地尝试相同的方法，最终失控
- 说白了就这么多，但这足以让这种方法失效

即使你没有自己动手实现过 Agent，你可能在使用 Agent 编码工具时也见过这种长上下文问题。它们用了一会儿就会迷失，你需要开始一个新的聊天。

我甚至可以提出一些我偶尔听到的观点，你自己可能也在这方面形成了自己的直觉：

> ### **即使模型支持越来越长的上下文窗口，使用简短、专注的提示词和上下文，你总是能得到更好的结果**

我交谈过的大多数构建者**把"工具调用循环"的想法放到了一边**，因为他们意识到超过 10-20 轮对话就会变成一团糟，LLM 无法从中恢复。即使 Agent 有 90% 的正确率，这也离"足以交付给客户使用"差得很远。你能想象一个每 10 次页面加载就崩溃一次的 Web 应用吗？

**2025-06-09 更新** - 我真的很喜欢 [@swyx](https://x.com/swyx/status/1932125643384455237) 对此的表述：

<a href="https://x.com/swyx/status/1932125643384455237"><img width="593" alt="Screenshot 2025-07-02 at 11 50 50 AM" src="https://github.com/user-attachments/assets/c7d94042-e4b9-4b87-87fd-55c7ff94bb3b" /></a>

### 真正有效的方法——微型 Agent

我在实际场景中**确实**见过很多的一个做法是：将 Agent 模式融入到更广泛的确定性 DAG 中。

![micro-agent-dag](https://github.com/humanlayer/12-factor-agents/blob/main/img/028-micro-agent-dag.png)

你可能会问——"在这种情况下为什么还要使用 Agent？"——我们稍后会深入讨论这个问题，但基本上，让语言模型管理范围明确的任务集，可以轻松纳入实时人工反馈，将其转化为工作流步骤，而不会陷入上下文错误循环。

> #### 让语言模型管理范围明确的任务集，可以轻松纳入实时人工反馈……而不会陷入上下文错误循环

### 一个真实的微型 Agent

下面是一个确定性代码如何运行一个负责处理部署中人工介入步骤的微型 Agent 的示例。

![029-deploybot-high-level](https://github.com/humanlayer/12-factor-agents/blob/main/img/029-deploybot-high-level.png)

* **人工** 将 PR 合并到 GitHub main 分支
* **确定性代码** 部署到 staging 环境
* **确定性代码** 针对 staging 运行端到端（e2e）测试
* **确定性代码** 将控制权交给 Agent 进行生产部署，初始上下文："部署 SHA 4af9ec0 到生产环境"
* **Agent** 调用 `deploy_frontend_to_prod(4af9ec0)`
* **确定性代码** 请求人工批准此操作
* **人工** 拒绝该操作并反馈"你能先部署后端吗？"
* **Agent** 调用 `deploy_backend_to_prod(4af9ec0)`
* **确定性代码** 请求人工批准此操作
* **人工** 批准该操作
* **确定性代码** 执行后端部署
* **Agent** 调用 `deploy_frontend_to_prod(4af9ec0)`
* **确定性代码** 请求人工批准此操作
* **人工** 批准该操作
* **确定性代码** 执行前端部署
* **Agent** 确定任务已成功完成，我们完成了！
* **确定性代码** 针对生产环境运行端到端测试
* **确定性代码** 任务完成，或者转交给回滚 Agent 审查失败并可能进行回滚

[![033-deploybot-animation](https://github.com/humanlayer/12-factor-agents/blob/main/img/033-deploybot.gif)](https://github.com/user-attachments/assets/deb356e9-0198-45c2-9767-231cb569ae13)

<details>
<summary><a href="https://github.com/humanlayer/12-factor-agents/blob/main/img/033-deploybot.gif">GIF 版本</a></summary>

![033-deploybot-animation](https://github.com/humanlayer/12-factor-agents/blob/main/img/033-deploybot.gif)

</details>

这个示例基于我们在 Humanlayer 实际交付的[用于管理部署的开源 Agent](https://github.com/got-agents/agents/tree/main/deploybot-ts)——这是我上周与它的一次真实对话：

![035-deploybot-conversation](https://github.com/humanlayer/12-factor-agents/blob/main/img/035-deploybot-conversation.png)

我们没有给这个 Agent 一大堆工具或任务。LLM 的主要价值在于解析人类的纯文本反馈并提出更新的行动方案。我们尽可能隔离任务和上下文，让 LLM 专注于一个小的、5-10 步的工作流。

这是另一个[更经典的客服/聊天机器人演示](https://x.com/chainlit_io/status/1858613325921480922)。

### 那么 Agent 到底是什么？

- **提示词（prompt）** - 告诉 LLM 如何行为，以及它有哪些可用的"工具"。提示词的输出是一个 JSON 对象，描述工作流中的下一个步骤（"工具调用"或"函数调用"）。([因素 2](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-02-own-your-prompts-zh.md))
- **switch 语句** - 根据 LLM 返回的 JSON，决定如何处理它。（属于[因素 8](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-08-own-your-control-flow-zh.md)）
- **累积上下文** - 存储已发生的步骤列表及其结果([因素 3](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window-zh.md))
- **for 循环** - 直到 LLM 发出某种"终端"工具调用（或纯文本响应），将 switch 语句的结果添加到上下文窗口，并让 LLM 选择下一步。([因素 8](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-08-own-your-control-flow-zh.md))

![040-4-components](https://github.com/humanlayer/12-factor-agents/blob/main/img/040-4-components.png)

在"deploybot"示例中，我们通过拥有控制流和上下文累积获得了一些好处：

- 在我们的 **switch 语句** 和 **for 循环** 中，我们可以劫持控制流以暂停等待人工输入或等待长时间运行任务的完成
- 我们可以轻松地序列化 **上下文** 窗口以实现暂停+恢复
- 在我们的 **提示词** 中，我们可以优化如何向 LLM 传递指令和"到目前为止发生了什么"

[第二部分](https://github.com/humanlayer/12-factor-agents/blob/main/README-zh.md#12-factor-agents)将**正式化这些模式**，以便它们可以应用于为任何软件项目添加令人印象深刻的 AI 功能，而无需完全投入传统的"AI Agent"实现/定义。

[因素 1 - 自然语言转工具调用 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-01-natural-language-to-tool-calls-zh.md)
