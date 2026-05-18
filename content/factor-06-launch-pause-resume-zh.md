[← 返回 README](../README-zh.md)

### 6. 使用简单API进行启动/暂停/恢复

智能体只是程序，我们对如何启动、查询、恢复和停止它们有明确的期望。

[![pause-resume animation](https://github.com/humanlayer/12-factor-agents/blob/main/img/165-pause-resume-animation.gif)](https://github.com/user-attachments/assets/feb1a425-cb96-4009-a133-8bd29480f21f)

<details>
<summary><a href="https://github.com/humanlayer/12-factor-agents/blob/main/img/165-pause-resume-animation.gif">GIF 版本</a></summary>

![pause-resume animation](https://github.com/humanlayer/12-factor-agents/blob/main/img/165-pause-resume-animation.gif)

</details>

用户、应用、管道和其他智能体应该能够通过简单API轻松启动智能体。

智能体及其编排的确定性代码应该能够在需要长时间运行操作时暂停智能体。

像webhook这样的外部触发器应该能够使智能体从中断的地方恢复，而无需与智能体编排器深度集成。

这与[第5条原则 - 统一执行状态和业务状态](./factor-05-unify-execution-state-zh.md)和[第8条原则 - 掌控你的控制流](./factor-08-own-your-control-flow-zh.md)密切相关，但可以独立实现。

**注意**——通常AI编排器会允许暂停和恢复，但在工具选择和工具执行之间无法进行。见[第7条原则 - 通过工具调用联系人类](./factor-07-contact-humans-with-tools-zh.md)和[第11条原则 - 从任何地方触发，在用户所在的地方相遇](./factor-11-trigger-from-anywhere-zh.md)。

[← 统一执行状态](./factor-05-unify-execution-state-zh.md) | [通过工具联系人类 →](./factor-07-contact-humans-with-tools-zh.md)
