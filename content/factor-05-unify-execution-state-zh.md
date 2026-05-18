[← 返回 README](https://github.com/humanlayer/12-factor-agents/blob/main/README-zh.md)

### 5. 统一执行状态和业务状态

即使在AI领域之外，许多基础设施系统也会尝试将"执行状态"和"业务状态"分开。对于AI应用，这可能涉及复杂的抽象来跟踪诸如当前步骤、下一步、等待状态、重试次数等信息。这种分离可能创造了值得的复杂性，但对你的用例来说可能过于复杂。

一如既往，是否分离取决于你的应用需要什么。但不要认为你*必须*将它们分开管理。

更清晰地说：

- **执行状态**：当前步骤、下一步、等待状态、重试次数等。
- **业务状态**：智能体工作流中到目前为止发生的情况（例如，OpenAI消息列表、工具调用和结果列表等）

如果可能，请简化——尽可能统一这些状态。

[![155-unify-state](https://github.com/humanlayer/12-factor-agents/blob/main/img/155-unify-state-animation.gif)](https://github.com/user-attachments/assets/e5a851db-f58f-43d8-8b0c-1926c99fc68d)

<details>
<summary><a href="https://github.com/humanlayer/12-factor-agents/blob/main/img/155-unify-state-animation.gif">GIF 版本</a></summary>

![155-unify-state](https://github.com/humanlayer/12-factor-agents/blob/main/img/155-unify-state-animation.gif)

</details>

实际上，你可以设计你的应用，使得可以从上下文窗口推断所有执行状态。在许多情况下，执行状态（当前步骤、等待状态等）只是关于到目前为止发生的事件的元数据。

你可能会有无法放入上下文窗口的内容，比如会话ID、密码上下文等，但你应该尽量减少这些东西。通过拥抱[第3条原则](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-03-own-your-context-window-zh.md)，你可以控制哪些内容真正进入LLM。

这种方法有几个好处：

1. **简单性**：所有状态单一数据源
2. **可序列化**：线程可以轻松地序列化/反序列化
3. **可调试**：整个历史记录在一个地方可见
4. **灵活性**：通过添加新的事件类型轻松添加新状态
5. **可恢复**：通过加载线程可以从任意点恢复
6. **可分叉**：通过将线程的某个子集复制到新的上下文/状态ID，可以在任意点分叉线程
7. **人机界面和可观测性**：将线程转换为人类可读的markdown或富Web应用UI非常简单

[← 工具是结构化输出](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-04-tools-are-structured-outputs-zh.md) | [启动/暂停/恢复 →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-06-launch-pause-resume-zh.md)
