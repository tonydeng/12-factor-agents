[← 返回 README](https://github.com/humanlayer/12-factor-agents/blob/main/README-zh.md)

### 11. 从任何地方触发，在用户所在之处与他们相遇

如果你在等 [humanlayer](https://humanlayer.dev) 的宣传，那你想对了。如果你正在实践[因素6 - 使用简单API启动/暂停/恢复](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-06-launch-pause-resume-zh.md)和[因素7 - 通过工具调用联系人工](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-07-contact-humans-with-tools-zh.md)，你就准备好结合这个因素了。

![1b0-trigger-from-anywhere](https://github.com/humanlayer/12-factor-agents/blob/main/img/1b0-trigger-from-anywhere.png)

让用户能够从 Slack、电子邮件、短信或任何其他他们想要的渠道触发智能体。让智能体能够通过相同的渠道进行响应。

优势：

- **在用户所在之处与他们相遇**：这帮助你构建感觉像真正人类的AI应用程序，或者至少是数字同事
- **外环智能体**：使智能体能够被非人类触发，例如事件、定时任务、故障等。它们可能运行5分钟、20分钟、90分钟，但当它们到达关键点时，可以联系人工寻求帮助、反馈或批准
- **高风险工具**：如果你能够快速联系各种人工，你可以给智能体访问更高风险操作的权限，如发送外部电子邮件、更新生产数据等。保持清晰的标准可以让你获得可审计性和对智能体的信心，从而[执行更大更好的事情](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md#what-if-llms-get-smarter)

[← 小而专注的智能体](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-10-small-focused-agents.md) | [无状态 Reducer →](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-12-stateless-reducer.md)
