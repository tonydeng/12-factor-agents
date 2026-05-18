# 12-Factor Agents - 构建可靠LLM应用的原则

<div align="center">
<a href="https://www.apache.org/licenses/LICENSE-2.0">
        <img src="https://img.shields.io/badge/Code-Apache%202.0-blue.svg" alt="Code License: Apache 2.0"></a>
<a href="https://creativecommons.org/licenses/by-sa/4.0/">
        <img src="https://img.shields.io/badge/Content-CC%20BY--SA%204.0-lightgrey.svg" alt="Content License: CC BY-SA 4.0"></a>
<a href="https://humanlayer.dev/discord">
    <img src="https://img.shields.io/badge/chat-discord-5865F2" alt="Discord Server"></a>
<a href="https://www.youtube.com/watch?v=8kMaTybvDUw">
    <img src="https://img.shields.io/badge/aidotengineer-conf_talk_(17m)-white" alt="YouTube
Deep Dive"></a>
<a href="https://www.youtube.com/watch?v=yxJDyQ8v6P0">
    <img src="https://img.shields.io/badge/youtube-deep_dive-crimson" alt="YouTube
Deep Dive"></a>

</div>

<p></p>

*秉承[12 Factor Apps](https://12factor.net/)的精神。本项目源码公开于 https://github.com/humanlayer/12-factor-agents，欢迎您的反馈和贡献。让我们一起探索！*

> [!提示]
> 错过了AI Engineer World's Fair？[点击观看演讲](https://www.youtube.com/watch?v=8kMaTybvDUw)
>
> 想了解上下文工程？[直接跳转到Factor 3](./content/factor-03-own-your-context-window-zh.md)
>
> 想参与`npx/uvx create-12-factor-agent`的贡献？查看[讨论帖](https://github.com/humanlayer/12-factor-agents/discussions/61)


<img referrerpolicy="no-referrer-when-downgrade" src="https://static.scarf.sh/a.png?x-pxid=2acad99a-c2d9-48df-86f5-9ca8061b7bf9" />

<a href="#visual-nav"><img width="907" alt="截图 2025-04-03 下午2:49:07" src="https://github.com/user-attachments/assets/23286ad8-7bef-4902-b371-88ff6a22e998" /></a>


大家好，我是Dex。我从事[AI agent](https://theouterloop.substack.com)相关工作已经[有一段时间](https://humanlayer.dev)了。


**我尝试过所有的agent框架**，从即插即用的crew/langchains到"极简主义"的smolagents，再到"生产级别"的langraph、griptape等。

**我和很多优秀的创始人交流过**，无论是否在YC，他们都在用AI构建令人印象深刻的产品。他们大多数都在自己搭建技术栈。在生产级面向客户的agent中，我很少看到框架的身影。

**我惊讶地发现**，大多数标榜自己是"AI Agents"的产品并不那么agentic。它们大多是确定性代码，只是在合适的时机穿插LLM步骤，让体验变得真正神奇。

Agent，至少是好的Agent，并不会遵循["给你一个提示词，给你一堆工具，循环直到达到目标"](https://www.anthropic.com/engineering/building-effective-agents#agents)的模式。相反，它们主要由软件组成。

所以，我开始回答这个问题：

> ### **我们可以使用什么原则来构建真正能够交付给生产客户的LLM驱动软件？**

欢迎来到12-Factor Agents。就像芝加哥自Daley以来的每一位市长都在该市主要机场贴满标语一样，我们很高兴您能来到这里。

*特别感谢[@iantbutler01](https://github.com/iantbutler01)、[@tnm](https://github.com/tnm)、[@hellovai](https://www.github.com/hellovai)、[@stantonk](https://www.github.com/stantonk)、[@balanceiskey](https://www.github.com/balanceiskey)、[@AdjectiveAllison](https://www.github.com/AdjectiveAllison)、[@pfbyjy](https://www.github.com/pfbyjy)、[@a-churchill](https://www.github.com/a-churchill)，以及SF MLOps社区对本指南的早期反馈。*

## 简短版：12个Factor

即使LLM[继续变得指数级强大](./content/factor-10-small-focused-agents-zh.md#what-if-llms-get-smarter)，仍然会有一些核心工程技巧使LLM驱动的软件更加可靠、可扩展和易于维护。

- [我们是如何走到这里的：软件简史](./content/brief-history-of-software-zh.md)
- [Factor 1: 自然语言转工具调用](./content/factor-01-natural-language-to-tool-calls-zh.md)
- [Factor 2: 拥有你的提示词](./content/factor-02-own-your-prompts-zh.md)
- [Factor 3: 拥有你的上下文窗口](./content/factor-03-own-your-context-window-zh.md)
- [Factor 4: 工具只是结构化输出](./content/factor-04-tools-are-structured-outputs-zh.md)
- [Factor 5: 统一执行状态和业务状态](./content/factor-05-unify-execution-state-zh.md)
- [Factor 6: 使用简单API启动/暂停/恢复](./content/factor-06-launch-pause-resume-zh.md)
- [Factor 7: 使用工具调用联系人类](./content/factor-07-contact-humans-with-tools-zh.md)
- [Factor 8: 拥有你的控制流](./content/factor-08-own-your-control-flow-zh.md)
- [Factor 9: 将错误压缩到上下文窗口](./content/factor-09-compact-errors-zh.md)
- [Factor 10: 小而专注的Agent](./content/factor-10-small-focused-agents-zh.md)
- [Factor 11: 从任何地方触发，在用户所在的地方与他们相遇](./content/factor-11-trigger-from-anywhere-zh.md)
- [Factor 12: 让你的Agent成为一个无状态Reducer](./content/factor-12-stateless-reducer-zh.md)

### 可视化导航

|    |    |    |
|----|----|-----|
|[![factor 1](./img/110-natural-language-tool-calls.png)](./content/factor-01-natural-language-to-tool-calls-zh.md) | [![factor 2](./img/120-own-your-prompts.png)](./content/factor-02-own-your-prompts-zh.md) | [![factor 3](./img/130-own-your-context-building.png)](./content/factor-03-own-your-context-window-zh.md) |
|[![factor 4](./img/140-tools-are-just-structured-outputs.png)](./content/factor-04-tools-are-structured-outputs-zh.md) | [![factor 5](./img/150-unify-state.png)](./content/factor-05-unify-execution-state-zh.md) | [![factor 6](./img/160-pause-resume-with-simple-apis.png)](./content/factor-06-launch-pause-resume-zh.md) |
| [![factor 7](./img/170-contact-humans-with-tools.png)](./content/factor-07-contact-humans-with-tools-zh.md) | [![factor 8](./img/180-control-flow.png)](./content/factor-08-own-your-control-flow-zh.md) | [![factor 9](./img/190-factor-9-errors-static.png)](./content/factor-09-compact-errors-zh.md) |
| [![factor 10](./img/1a0-small-focused-agents.png)](./content/factor-10-small-focused-agents-zh.md) | [![factor 11](./img/1b0-trigger-from-anywhere.png)](./content/factor-11-trigger-from-anywhere-zh.md) | [![factor 12](./img/1c0-stateless-reducer.png)](./content/factor-12-stateless-reducer-zh.md) |

## 我们是如何走到这里的

要深入了解我的agent之旅以及是什么引导我们走到这里，请查看[软件简史](./content/brief-history-of-software-zh.md)——下面是简要总结：

### Agent的承诺

我们将大量讨论有向图（DG）及其无环朋友DAG。我首先要指出的是……软件本身就是一个有向图。我们过去用流程图来表示程序是有原因的。

![010-software-dag](https://github.com/humanlayer/12-factor-agents/blob/main/img/010-software-dag.png)

### 从代码到DAG

大约20年前，我们开始看到DAG编排器变得流行。经典的有[Airflow](https://airflow.apache.org/)、[Prefect](https://www.prefect.io/)，还有一些前辈，以及一些更新的如（[dagster](https://dagster.io/)、[inngest](https://www.inngest.com/)、[windmill](https://www.windmill.dev/)）。它们遵循相同的图形模式，并增加了可观测性、模块化、重试、管理等好处。

![015-dag-orchestrators](https://github.com/humanlayer/12-factor-agents/blob/main/img/015-dag-orchestrators.png)

### Agent的承诺

我不是[第一个说这个的人](https://youtu.be/Dc99-zTMyMg?si=bcT0hIwWij2mR-40&t=73)，但当我开始学习agent时，我最大的收获是，你可以把DAG扔掉了。软件开发人员不需要编写每个步骤和边界情况，而是给agent一个目标和一组转换：

![025-agent-dag](https://github.com/humanlayer/12-factor-agents/blob/main/img/025-agent-dag.png)

让LLM实时做决策来确定路径

![026-agent-dag-lines](https://github.com/humanlayer/12-factor-agents/blob/main/img/026-agent-dag-lines.png)

这里的承诺是你写更少的软件，你只需要给LLM图的"边"，让它来弄清楚节点。你可以恢复错误，你可以写更少的代码，你可能会发现LLM能找到问题的创新解决方案。


### Agent即循环

正如我们稍后将看到的，事实证明这并不完全有效。

让我们再深入一步——对于agent，你有一个由3个步骤组成的循环：

1. LLM决定工作流中的下一步，输出结构化JSON（"工具调用"）
2. 确定性代码执行工具调用
3. 结果被追加到上下文窗口
4. 重复直到下一步被确定为"完成"

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

我们的初始上下文只是起始事件（可能是用户消息，可能是cron触发，可能是webhook等），我们让LLM选择下一步（工具）或确定我们已经完成。

这是一个多步骤示例：

[![027-agent-loop-animation](https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-animation.gif)](https://github.com/user-attachments/assets/3beb0966-fdb1-4c12-a47f-ed4e8240f8fd)

<details>
<summary><a href="https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-animation.gif">GIF版本</a></summary>

![027-agent-loop-animation](https://github.com/humanlayer/12-factor-agents/blob/main/img/027-agent-loop-animation.gif)

</details>

## 为什么要12-Factor Agents？

归根结底，这种方法的效果并不如我们想要的那么好。

在构建HumanLayer的过程中，我与至少100位SaaS构建者（主要是技术创始人）交谈过，他们希望让现有产品更具agentic特性。旅程通常是这样的：

1. 决定要构建一个agent
2. 产品设计、用户体验映射、要解决的问题
3. 想快速前进，所以选择$FRAMEWORK并开始构建
4. 达到70-80%的质量标准
5. 意识到80%对于大多数面向客户的功能来说还不够好
6. 意识到要超过80%需要反向工程框架、提示词、流程等
7. 从头开始

<details>
<summary>随机免责声明</summary>

**免责声明**：我不确定在什么地方说这些最合适，但这里似乎是个好地方：**这绝不是在批评现有的众多框架或在它们上面工作的非常聪明的人**。它们实现了令人难以置信的事情，加速了AI生态系统。

我希望这篇文章的一个成果是，agent框架构建者可以从我和他人旅程中学习，让框架变得更好。特别是对于那些想要快速前进但需要深度控制的构建者。

**免责声明2**：我不打算讨论MCP。我相信你能看到它在哪里适用。

**免责声明3**：我主要使用TypeScript，[原因](https://www.linkedin.com/posts/dexterihorthy_llms-typescript-aiagents-activity-7290858296679313408-Lh9e?utm_source=share&utm_medium=member_desktop&rcm=ACoAAA4oHTkByAiD-wZjnGsMBUL_JT6nyyhOh30)，但所有这些在Python或你喜欢的任何其他语言中都行得通。

总之，回到正题...

</details>

### 构建出色LLM应用的设计模式

在深入研究数百个AI库并与数十位创始人合作后，我的直觉是：

1. 有些核心东西让agent变得出色
2. 全力投入框架并构建本质上全新的重写可能适得其反
3. 有些核心原则让agent变得出色，如果你引入框架，你会得到大部分/全部
4. 但是，我见过的让构建者将高质量AI软件交付给客户的最快方式，是从agent构建中提取小的、模块化的概念，并将它们融入现有产品
5. 这些来自agent的模块化概念可以被大多数熟练的软件工程师定义和应用，即使他们没有AI背景

> #### 我见过的让构建者将好的AI软件交付给客户的最快方式，是从agent构建中提取小的、模块化的概念，并将它们融入现有产品


## 12个Factor（再次）


- [我们是如何走到这里的：软件简史](./content/brief-history-of-software-zh.md)
- [Factor 1: 自然语言转工具调用](./content/factor-01-natural-language-to-tool-calls-zh.md)
- [Factor 2: 拥有你的提示词](./content/factor-02-own-your-prompts-zh.md)
- [Factor 3: 拥有你的上下文窗口](./content/factor-03-own-your-context-window-zh.md)
- [Factor 4: 工具只是结构化输出](./content/factor-04-tools-are-structured-outputs-zh.md)
- [Factor 5: 统一执行状态和业务状态](./content/factor-05-unify-execution-state-zh.md)
- [Factor 6: 使用简单API启动/暂停/恢复](./content/factor-06-launch-pause-resume-zh.md)
- [Factor 7: 使用工具调用联系人类](./content/factor-07-contact-humans-with-tools-zh.md)
- [Factor 8: 拥有你的控制流](./content/factor-08-own-your-control-flow-zh.md)
- [Factor 9: 将错误压缩到上下文窗口](./content/factor-09-compact-errors-zh.md)
- [Factor 10: 小而专注的Agent](./content/factor-10-small-focused-agents-zh.md)
- [Factor 11: 从任何地方触发，在用户所在的地方与他们相遇](./content/factor-11-trigger-from-anywhere-zh.md)
- [Factor 12: 让你的Agent成为一个无状态Reducer](./content/factor-12-stateless-reducer-zh.md)

## 荣誉提及 / 其他建议

- [Factor 13: 预取你可能需要的所有上下文](./content/appendix-13-pre-fetch-zh.md)

## 相关资源

- 在[这里](https://github.com/humanlayer/12-factor-agents)为本指南做贡献
- 2025年3月我在Tool Use播客节目中[谈了很多相关内容](https://youtu.be/8bIHcttkOTE)
- 我在[The Outer Loop](https://theouterloop.substack.com)写一些这方面的内容
- 我与[@hellovai](https://github.com/hellovai)合作做关于[最大化LLM性能的网络研讨会](https://github.com/hellovai/ai-that-works/tree/main)
- 我们使用这种方法论在[got-agents/agents](https://github.com/got-agents/agents)下构建开源agent
- 我们无视自己的建议，构建了一个[在kubernetes中运行分布式agent的框架](https://github.com/humanlayer/kubechain)
- 本指南中的其他链接：
  - [12 Factor Apps](https://12factor.net)
  - [构建有效的Agent（Anthropic）](https://www.anthropic.com/engineering/building-effective-agents#agents)
  - [提示词即函数](https://thedataexchange.media/baml-revolution-in-ai-engineering/)
  - [库模式：为什么框架是邪恶的](https://tomasp.net/blog/2015/library-frameworks/)
  - [错误的抽象](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction)
  - [Mailcrew Agent](https://github.com/dexhorthy/mailcrew)
  - [Mailcrew演示视频](https://www.youtube.com/watch?v=f_cKnoPC_Oo)
  - [Chainlit演示](https://x.com/chainlit_io/status/1858613325921480922)
  - [用于LLM的TypeScript](https://www.linkedin.com/posts/dexterihorthy_llms-typescript-aiagents-activity-7290858296679313408-Lh9e)
  - [模式对齐解析](https://www.boundaryml.com/blog/schema-aligned-parsing)
  - [函数调用 vs 结构化输出 vs JSON模式](https://www.vellum.ai/blog/when-should-i-use-function-calling-structured-outputs-or-json-mode)
  - [GitHub上的BAML](https://github.com/boundaryml/baml)
  - [OpenAI JSON vs 函数调用](https://docs.llamaindex.ai/en/stable/examples/llm/openai_json_vs_function_calling/)
  - [外循环Agent](https://theouterloop.substack.com/p/openais-realtime-api-is-a-step-towards)
  - [Airflow](https://airflow.apache.org/)
  - [Prefect](https://www.prefect.io/)
  - [Dagster](https://dagster.io/)
  - [Inngest](https://www.inngest.com/)
  - [Windmill](https://www.windmill.dev/)
  - [AI Agent指数（MIT）](https://aiagentindex.mit.edu/)
  - [NotebookLM关于发现模型能力边界](https://open.substack.com/pub/swyx/p/notebooklm?selection=08e1187c-cfee-4c63-93c9-71216640a5f8)

## 贡献者

感谢每一位为12-factor agents做出贡献的人！

[<img src="https://avatars.githubusercontent.com/u/3730605?v=4&s=80" width="80px" alt="dexhorthy" />](https://github.com/dexhorthy) [<img src="https://avatars.githubusercontent.com/u/50557586?v=4&s=80" width="80px" alt="Sypherd" />](https://github.com/Sypherd) [<img src="https://avatars.githubusercontent.com/u/66259401?v=4&s=80" width="80px" alt="tofaramususa" />](https://github.com/tofaramususa) [<img src="https://avatars.githubusercontent.com/u/18105223?v=4&s=80" width="80px" alt="a-churchill" />](https://github.com/a-churchill) [<img src="https://avatars.githubusercontent.com/u/4084885?v=4&s=80" width="80px" alt="Elijas" />](https://github.com/Elijas) [<img src="https://avatars.githubusercontent.com/u/39267118?v=4&s=80" width="80px" alt="hugolmn" />](https://github.com/hugolmn) [<img src="https://avatars.githubusercontent.com/u/1882972?v=4&s=80" width="80px" alt="jeremypeters" />](https://github.com/jeremypeters)

[<img src="https://avatars.githubusercontent.com/u/380402?v=4&s=80" width="80px" alt="kndl" />](https://github.com/kndl) [<img src="https://avatars.githubusercontent.com/u/16674643?v=4&s=80" width="80px" alt="maciejkos" />](https://github.com/maciejkos) [<img src="https://avatars.githubusercontent.com/u/85041180?v=4&s=80" width="80px" alt="pfbyjy" />](https://github.com/pfbyjy) [<img src="https://avatars.githubusercontent.com/u/36044389?v=4&s=80" width="80px" alt="0xRaduan" />](https://github.com/0xRaduan) [<img src="https://avatars.githubusercontent.com/u/7169731?v=4&s=80" width="80px" alt="zyuanlim" />](https://github.com/zyuanlim) [<img src="https://avatars.githubusercontent.com/u/15862501?v=4&s=80" width="80px" alt="lombardo-chcg" />](https://github.com/lombardo-chcg) [<img src="https://avatars.githubusercontent.com/u/160066852?v=4&s=80" width="80px" alt="sahanatvessel" />](https://github.com/sahanatvessel)

## 许可证

所有内容和图片采用<a href="https://creativecommons.org/licenses/by-sa/4.0/">CC BY-SA 4.0许可证</a>

代码采用<a href="https://www.apache.org/licenses/LICENSE-2.0">Apache 2.0许可证</a>
