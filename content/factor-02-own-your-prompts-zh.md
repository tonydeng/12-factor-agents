[← 返回 README](../README-zh.md)

### 2. 掌控你的提示词

不要将提示词工程外包给框架。

![120-own-your-prompts](https://github.com/humanlayer/12-factor-agents/blob/main/img/120-own-your-prompts.png)

顺便说一句，[这远非新颖的建议：](https://hamel.dev/blog/posts/prompt/)

![image](https://github.com/user-attachments/assets/575bab37-0f96-49fb-9ce3-9a883cdd420b)

一些框架提供"黑盒"方法如下：

```python
agent = Agent(
  role="...",
  goal="...",
  personality="...",
  tools=[tool1, tool2, tool3]
)

task = Task(
  instructions="...",
  expected_output=OutputModel
)

result = agent.run(task)
```

这对于引入一些顶级提示词工程技术来帮助你起步非常有用，但通常很难调整和/或反向工程以获得恰好正确的令牌进入你的模型。

相反，要掌控你的提示词并将其视为一等公民代码：

```rust
function DetermineNextStep(thread: string) -> DoneForNow | ListGitTags | DeployBackend | DeployFrontend | RequestMoreInformation {
  prompt #"
    {{ _.role("system") }}

    You are a helpful assistant that manages deployments for frontend and backend systems.
    You work diligently to ensure safe and successful deployments by following best practices
    and proper deployment procedures.

    Before deploying any system, you should check:
    - The deployment environment (staging vs production)
    - The correct tag/version to deploy
    - The current system status

    You can use tools like deploy_backend, deploy_frontend, and check_deployment_status
    to manage deployments. For sensitive deployments, use request_approval to get
    human verification.

    Always think about what to do first, like:
    - Check current deployment status
    - Verify the deployment tag exists
    - Request approval if needed
    - Deploy to staging before production
    - Monitor deployment progress

    {{ _.role("user") }}

    {{ thread }}

    What should the next step be?
  "#
}
```

（上面的例子使用[BAML](https://github.com/boundaryml/baml)来生成提示词，但你可以用任何你想要的提示词工程工具来做这件事，甚至可以手动模板化）

如果签名看起来有点奇怪，我们将在[因素4 - 工具只是结构化输出](./factor-04-tools-are-structured-outputs-zh.md)中讨论。

```typescript
function DetermineNextStep(thread: string) -> DoneForNow | ListGitTags | DeployBackend | DeployFrontend | RequestMoreInformation {
```

掌控提示词的关键好处：

1. **完全控制**：编写智能体所需的确切指令，没有黑盒抽象
2. **测试和评估**：像对其他代码一样为提示词构建测试和评估
3. **迭代**：根据实际表现快速修改提示词
4. **透明性**：确切知道智能体正在使用的指令
5. **角色黑客**：利用支持非标准用户/助手角色用法的API——例如，现在已弃用的OpenAI"补全"API的非聊天风格。这包括一些所谓的"模型欺骗"技术

记住：你的提示词是应用程序逻辑和LLM之间的主要接口。

完全掌控你的提示词可以为你提供生产级智能体所需的灵活性和提示词控制。

我不知道什么是最好的提示词，但我知道你希望有灵活性能够尝试一切。

[← 自然语言到工具调用](./factor-01-natural-language-to-tool-calls-zh.md) | [掌控你的上下文窗口 →](./factor-03-own-your-context-window-zh.md)
