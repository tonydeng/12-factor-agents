# 第0章 - Hello World

让我们从一个基本的TypeScript设置和一个hello world程序开始。

本指南使用TypeScript编写（是的，Python版本即将推出）

在workshop步骤的每个文件编辑之间都有许多检查点，
因此即使您不太熟悉TypeScript，
也应该能够跟上并运行每个示例。

要运行本指南，您需要安装相对较新版本的nodejs和npm

您可以使用任何您想要的nodejs版本管理器，[homebrew](https://formulae.brew.sh/formula/node)就可以。

    brew install node@20

您应该会看到node版本

    node --version

复制初始package.json

    cp ./walkthrough/00-package.json package.json

安装依赖

    npm install

复制tsconfig.json

    cp ./walkthrough/00-tsconfig.json tsconfig.json

添加.gitignore

    cp ./walkthrough/00-.gitignore .gitignore

创建src文件夹

    mkdir -p src

添加一个简单的hello world index.ts

    cp ./walkthrough/00-index.ts src/index.ts

运行它来验证

    npx tsx src/index.ts

您应该会看到：

    hello, world!


# 第1章 - CLI和Agent循环

现在让我们添加BAML并创建一个带有CLI界面的第一个agent。

首先，我们需要安装[BAML](https://github.com/boundaryml/baml)，
这是一个用于提示和结构化输出的工具。


    npm install @boundaryml/baml

初始化BAML

    npx baml-cli init

删除默认的resume.baml

    rm baml_src/resume.baml

添加我们的入门agent，一个我们将逐步构建的单一baml提示


    cp ./walkthrough/01-agent.baml baml_src/agent.baml

生成BAML客户端代码

    npx baml-cli generate

为本节启用BAML日志记录

    export BAML_LOG=debug

添加CLI界面

    cp ./walkthrough/01-cli.ts src/cli.ts

更新index.ts以使用CLI

    cp ./walkthrough/01-index.ts src/index.ts

添加agent实现

    cp ./walkthrough/01-agent.ts src/agent.ts

BAML代码配置为默认使用BASETEN_API_KEY

要获取Baseten API密钥和URL，请在[baseten.co](https://baseten.co)创建账户，
然后从模型库部署[Qwen3 32B](https://www.baseten.co/library/qwen-3-32b/)。

```rust 
  function DetermineNextStep(thread: string) -> DoneForNow {
      client Qwen3
      // ...
```

如果您想不做任何更改地运行示例，可以将BASETEN_API_KEY环境变量设置为任何有效的baseten密钥。

如果您想尝试更换模型，可以更改`client`行。

[BAML客户端文档在这里](https://docs.boundaryml.com/guide/baml-basics/switching-llms)

例如，您可以配置[gemini](https://docs.boundaryml.com/ref/llm-client-providers/google-ai-gemini) 
或[anthropic](https://docs.boundaryml.com/ref/llm-client-providers/anthropic)作为您的模型提供商。

例如，要使用带有OPENAI_API_KEY的openai，您可以这样做：

    client "openai/gpt-4o"


设置您的环境变量

    export BASETEN_API_KEY=...
export BASETEN_BASE_URL=...

试试看

    npx tsx src/index.ts hello

您应该会看到来自模型的熟悉响应

    {
      intent: 'done_for_now',
      message: 'Hello! How can I assist you today?'
    }


# 第2章 - 添加计算器工具

让我们为agent添加一些计算器工具。

让我们从添加计算器的工具定义开始

这些是我们将要求模型返回为"下一步"的简单结构化输出
在agent循环中。


    cp ./walkthrough/02-tool_calculator.baml baml_src/tool_calculator.baml

现在，让我们更新agent的DetermineNextStep方法以
将计算器工具作为潜在的下一步公开


    cp ./walkthrough/02-agent.baml baml_src/agent.baml

生成更新的BAML客户端

    npx baml-cli generate

试试计算器

    npx tsx src/index.ts 'can you add 3 and 4'

您应该会看到对计算器的工具调用

    {
      intent: 'add',
      a: 3,
      b: 4
    }


# 第3章 - 在循环中处理工具调用

现在让我们添加一个真正的agent循环，可以运行工具并从LLM获取最终答案。

首先，让我们更新agent以处理工具调用


    cp ./walkthrough/03-agent.ts src/agent.ts

现在，让我们试试看


    npx tsx src/index.ts 'can you add 3 and 4'

您应该会看到agent调用工具然后返回结果

    {
      intent: 'done_for_now',
      message: 'The sum of 3 and 4 is 7.'
    }

对于下一步，我们将做一个更复杂的计算，让我们关闭baml日志以获得更简洁的输出

    export BAML_LOG=off

尝试多步骤计算

    npx tsx src/index.ts 'can you add 3 and 4, then add 6 to that result'

您会注意到乘法和除法等工具不可用

    npx tsx src/index.ts 'can you multiply 3 and 4'

接下来，让我们为其余的计算器工具添加处理程序


    cp ./walkthrough/03b-agent.ts src/agent.ts

测试减法

    npx tsx src/index.ts 'can you subtract 3 from 4'

现在，让我们测试乘法工具


    npx tsx src/index.ts 'can you multiply 3 and 4'

最后，让我们测试一个具有多个操作的更复杂计算


    npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'

恭喜，您已经迈出了手工制作agent循环的第一步。

从这里开始，我们将开始整合12-factor agents的一些更中级和高级的概念。




# 第4章 - 向agent.baml添加测试

让我们为BAML agent添加一些测试。

首先，启用baml日志

    export BAML_LOG=debug

接下来，让我们向agent添加一些测试

我们将从一个简单的测试开始，检查agent处理
基本计算的能力。


    cp ./walkthrough/04-agent.baml baml_src/agent.baml

运行测试

    npx baml-cli test

现在，让我们用断言来改进测试！

断言是确保agent按预期工作的好方法，
并且可以轻松扩展以检查更复杂的行为。


    cp ./walkthrough/04b-agent.baml baml_src/agent.baml

运行测试

    npx baml-cli test

当您添加更多测试时，可以禁用日志以保持输出整洁。
在迭代特定测试时您可能希望打开它们。


    export BAML_LOG=off

现在，让我们添加一些更复杂的测试用例，
其中我们从一个进行中的
agent上下文窗口的中间恢复


    cp ./walkthrough/04c-agent.baml baml_src/agent.baml

让我们尝试运行它


    npx baml-cli test


# 第5章 - 多个Human工具

在本节中，我们将添加支持多种用于
联系人类的工具。


对于本节，我们将禁用baml日志。如果您想查看更多细节，可以选择启用它们。

    export BAML_LOG=off

首先，让我们添加一个可以向人类请求澄清的工具

这将与"done_for_now"工具不同，
可以更灵活地处理agent中不同类型的人类交互


    cp ./walkthrough/05-agent.baml baml_src/agent.baml

接下来，让我们重新生成客户端代码

注意 - 如果您使用BAML的VSCode扩展，
当您在编辑器中保存文件时，客户端将自动重新生成。


    npx baml-cli generate

现在，让我们更新agent以使用新工具


    cp ./walkthrough/05-agent.ts src/agent.ts

接下来，让我们更新CLI以处理澄清请求
通过在CLI上向用户请求输入


    cp ./walkthrough/05-cli.ts src/cli.ts

让我们试试看


    npx tsx src/index.ts 'can you multiply 3 and FD*(#F&& '

接下来，让我们添加一个测试，检查agent处理
澄清请求的能力


    cp ./walkthrough/05b-agent.baml baml_src/agent.baml

现在我们可以再次运行测试


    npx baml-cli test

您会注意到新测试通过了，但hello world测试失败了

这是因为agent的默认行为是返回"done_for_now"


    cp ./walkthrough/05c-agent.baml baml_src/agent.baml

验证测试通过

    npx baml-cli test


# 第6章 - 使用推理自定义您的提示词

在本节中，我们将探索如何使用推理步骤自定义agent的提示词。

这是[factor 2 - 拥有你的提示词](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-2-own-your-prompts.md)的核心

关于推理有一个深入探讨在AI That Works [推理模型与推理步骤](https://github.com/hellovai/ai-that-works/tree/main/2025-04-07-reasoning-models-vs-prompts)


对于本节，启用baml日志会很有帮助

    export BAML_LOG=debug

更新agent提示以包含推理步骤


    cp ./walkthrough/06-agent.baml baml_src/agent.baml

生成更新的客户端

    npx baml-cli generate

现在，您可以使用简单的提示词试试看


    npx tsx src/index.ts 'can you multiply 3 and 4'

您应该会从baml日志中看到推理步骤的输出

#### 可选挑战

在工具输出格式中添加一个字段，包含输出中的推理步骤！



# 第7章 - 自定义您的上下文窗口

在本节中，我们将探索如何自定义agent的上下文窗口。

这是[factor 3 - 拥有你的上下文窗口](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-3-own-your-context-window-zh.md)的核心


更新agent以美化为模型格式化上下文窗口


    cp ./walkthrough/07-agent.ts src/agent.ts

测试格式化

    BAML_LOG=info npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'

接下来，让我们更新agent以使用XML格式化

这是一种向模型传递数据的非常流行的格式，

除其他原因外，因为XML的token效率。


    cp ./walkthrough/07b-agent.ts src/agent.ts

让我们试试看


    BAML_LOG=info npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'

让我们更新测试以匹配新的输出格式


    cp ./walkthrough/07c-agent.baml baml_src/agent.baml

查看更新的测试


    npx baml-cli test


# 第8章 - 添加API端点

添加Express服务器以通过HTTP公开agent。

对于本节，我们将禁用baml日志。如果您想查看更多细节，可以选择启用它们。

    export BAML_LOG=off

安装Express和类型

    npm install express && npm install --save-dev @types/express supertest

添加服务器实现

    cp ./walkthrough/08-server.ts src/server.ts

启动服务器

    npx tsx src/server.ts

用curl测试（在另一个终端中）

    curl -X POST http://localhost:3000/thread \
  -H "Content-Type: application/json" \
  -d '{"message":"can you add 3 and 4"}'

您应该会从agent获得答案，其中包括
agent跟踪，以类似这样的消息结尾：


    {"intent":"done_for_now","message":"The sum of 3 and 4 is 7."}


# 第9章 - 内存状态和异步澄清

添加状态管理和异步澄清支持。

对于本节，我们将禁用baml日志。如果您想查看更多细节，可以选择启用它们。

    export BAML_LOG=off

为线程添加一些简单的内存状态管理

    cp ./walkthrough/09-state.ts src/state.ts

更新服务器以使用状态管理

* 使用`ThreadStore`添加线程状态管理
* 从/thread端点返回线程ID和响应URL
* 实现GET /thread/:id
* 实现POST /thread/:id/response


    cp ./walkthrough/09-server.ts src/server.ts

启动服务器

    npx tsx src/server.ts

测试澄清流程

    curl -X POST http://localhost:3000/thread \
  -H "Content-Type: application/json" \
  -d '{"message":"can you multiply 3 and xyz"}'


# 第10章 - 添加人类审批

添加对操作的人类审批支持。

对于本节，我们将禁用baml日志。如果您想查看更多细节，可以选择启用它们。

    export BAML_LOG=off

更新服务器以处理人类审批

* 导入`handleNextStep`以执行批准的操作
* 添加两种有效载荷类型以区分审批和响应
* 在端点中不同地处理响应和审批
* 当出现问题时显示更好的错误消息


    cp ./walkthrough/10-server.ts src/server.ts

向agent添加一些方法以处理审批和响应

    cp ./walkthrough/10-agent.ts src/agent.ts

启动服务器

    npx tsx src/server.ts

测试带审批的除法

    curl -X POST http://localhost:3000/thread \
  -H "Content-Type: application/json" \
  -d '{"message":"can you divide 3 by 4"}'

您应该会看到：

    {
      "thread_id": "2b243b66-215a-4f37-8bc6-9ace3849043b",
      "events": [
        {
          "type": "user_input",
          "data": "can you divide 3 by 4"
        },
        {
          "type": "tool_call",
          "data": {
            "intent": "divide",
            "a": 3,
            "b": 4,
            "response_url": "/thread/2b243b66-215a-4f37-8bc6-9ace3849043b/response"
          }
        }
      ]
    }

用另一个curl调用拒绝请求，更改线程ID

    curl -X POST 'http://localhost:3000/thread/{thread_id}/response' \
  -H "Content-Type: application/json" \
  -d '{"type": "approval", "approved": false, "comment": "I dont think thats right, use 5 instead of 4"}'

您应该会看到：最后一次工具调用现在是`"intent":"divide","a":3,"b":5`

    {
      "events": [
        {
          "type": "user_input",
          "data": "can you divide 3 by 4"
        },
        {
          "type": "tool_call",
          "data": {
            "intent": "divide",
            "a": 3,
            "b": 4,
            "response_url": "/thread/2b243b66-215a-4f37-8bc6-9ace3849043b/response"
          }
        },
        {
          "type": "tool_response",
          "data": "user denied the operation with feedback: \"I dont think thats right, use 5 instead of 4\""
        },
        {
          "type": "tool_call",
          "data": {
            "intent": "divide",
            "a": 3,
            "b": 5,
            "response_url": "/thread/1f1f5ff5-20d7-4114-97b4-3fc52d5e0816/response"
          }
        }
      ]
    }

现在您可以批准该操作

    curl -X POST 'http://localhost:3000/thread/{thread_id}/response' \
  -H "Content-Type: application/json" \
  -d '{"type": "approval", "approved": true}'

您应该会看到最终消息包含工具响应和最终结果！

    ...
    {
      "type": "tool_response",
      "data": 0.5
    },
    {
      "type": "done_for_now",
      "message": "I divided 3 by 6 and the result is 0.5. If you have any more operations or queries, feel free to ask!",
      "response_url": "/thread/2b469403-c497-4797-b253-043aae830209/response"
    }


# 第11章 - 通过电子邮件进行人类审批

在本节中，我们将添加通过电子邮件进行人类审批的支持。

这将开始有点人为，只是为了掌握概念——

我们将首先从CLI调用工作流，但对于`divide`
和`request_more_information`的审批将通过电子邮件处理，
然后最终的`done_for_now`答案将打印回CLI

虽然有点人为，但这是一个很好的例子，展示了[factor 7 - 用工具联系人类](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-7-contact-humans-with-tools.md)的灵活性带来的灵活性


对于本节，我们将禁用baml日志。如果您想查看更多细节，可以选择启用它们。

    export BAML_LOG=off

安装HumanLayer

    npm install humanlayer

更新CLI以通过电子邮件将`divide`和`request_more_information`发送给人类

    cp ./walkthrough/11-cli.ts src/cli.ts

运行CLI

    npx tsx src/index.ts 'can you divide 4 by 5'

程序的最后一行应该提到人工审查步骤

    nextStep { intent: 'divide', a: 4, b: 5 }
    HumanLayer: Requested human approval from HumanLayer cloud

继续并回复电子邮件并提供一些反馈：

![reject-email](https://github.com/humanlayer/12-factor-agents/blob/main/workshops/2025-05/walkthrough/11-email-reject.png?raw=true)


您应该会收到另一封基于您的反馈的更新尝试的电子邮件！

您可以继续批准这一个：

![approve-email](https://github.com/humanlayer/12-factor-agents/blob/main/workshops/2025-05/walkthrough/11-email-approve.png?raw=true)


您的最终输出将如下所示

    nextStep {
     intent: 'done_for_now',
     message: 'The division of 4 by 5 is 0.8. If you have any other calculations or questions, feel free to ask!'
    }
    The division of 4 by 5 is 0.8. If you have any other calculations or questions, feel free to ask!

让我们也实现`request_more_information`流程


    cp ./walkthrough/11b-cli.ts src/cli.ts

让我们测试require_approval流程，通过询问带有乱码输入的计算


    npx tsx src/index.ts 'can you multiply 4 and xyz'

您应该会收到一封请求澄清的电子邮件

    Can you clarify what 'xyz' represents in this context? Is it a specific number, variable, or something else?

您可以回复类似这样的内容

    use 8 instead of xyz

您应该在CLI上看到最终结果

    I have multiplied 4 and xyz, using the value 8 for xyz, resulting in 32.

作为最后一步，让我们探索使用自定义html模板进行电子邮件


    cp ./walkthrough/11c-cli.ts src/cli.ts

首先尝试divide：


    npx tsx src/index.ts 'can you divide 4 by 5'

您应该会看到一封略有不同的电子邮件，包含自定义模板

![custom-template-email](https://github.com/humanlayer/12-factor-agents/blob/main/workshops/2025-05/walkthrough/11-email-custom.png?raw=true)

随意使用该流程，然后您可以尝试根据自己的喜好更新模板

（如果您使用的是cursor，简单地突出显示模板并要求"让它更好"
应该可以做到）

也尝试触发"request_more_information"！


就是这样 - 在下一章中，我们将构建一个完全由电子邮件驱动的工作流agent，使用webhooks进行人类审批



# 第XX章 - HumanLayer Webhook集成

前面的章节以"同步模式"使用humanlayer SDK - 这意味着
每次我们等待人类审批时，我们都会坐在一个循环中
轮询直到收到人类响应。

这显然不理想，特别是对于生产工作负载，
所以在本节中，我们将通过更新服务器以在联系人类后结束处理，并使用webhooks接收结果来实现[factor 6 - 使用简单API启动/暂停/恢复](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-6-launch-pause-resume-zh.md)。


添加代码以在服务器中初始化humanlayer


    cp ./walkthrough/12-1-server-init.ts src/server.ts

接下来，让我们更新/thread端点以

1. 异步处理请求，立即返回
2. 在request_more_information和done_for_now调用上创建人类联系人


更新服务器以能够处理request_clarification响应

- 删除旧的/response端点和类型
- 更新/thread端点以异步运行处理，立即返回
- 在请求人类响应时发送state.threadId
- 添加handleHumanResponse函数来处理人类响应
- 添加/webhook端点来处理webhook响应


    cp ./walkthrough/12a-server.ts src/server.ts

在另一个终端中启动服务器

    npx tsx src/server.ts

现在服务器正在运行，向'/thread'端点发送有效载荷


__ 做响应步骤

__ 现在处理divide的审批

__ 现在也处理done_for_now
