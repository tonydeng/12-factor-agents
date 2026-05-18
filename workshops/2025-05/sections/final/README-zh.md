# 第零章 - Hello World

让我们从一个基本的 TypeScript 设置和一个 hello world 程序开始。

本指南使用 TypeScript 编写（是的，Python 版本即将推出）

本指南中的每个文件编辑之间都有很多检查点，
所以即使你对 TypeScript 不是特别熟悉，
也应该能够跟上并运行每个示例。

要运行本指南，你需要安装较新版本的 nodejs 和 npm

你可以使用任何你想要的 nodejs 版本管理器，[homebrew](https://formulae.brew.sh/formula/node) 就可以


    brew install node@20

你应该会看到 node 版本

    node --version

复制初始 package.json

    cp ./walkthrough/00-package.json package.json

安装依赖

    npm install

复制 tsconfig.json

    cp ./walkthrough/00-tsconfig.json tsconfig.json

添加 .gitignore

    cp ./walkthrough/00-.gitignore .gitignore

创建 src 文件夹

    mkdir -p src

添加一个简单的 hello world index.ts

    cp ./walkthrough/00-index.ts src/index.ts

运行它以验证

    npx tsx src/index.ts

你应该看到：

    hello, world!


# 第一章 - CLI 和智能体循环

现在让我们添加 BAML 并创建我们的第一个带有 CLI 接口的智能体。

首先，我们需要安装 [BAML](https://github.com/boundaryml/baml)，
这是一个用于提示和结构化输出的工具。


    npm install @boundaryml/baml

初始化 BAML

    npx baml-cli init

删除默认的 resume.baml

    rm baml_src/resume.baml

添加我们的起始智能体，一个我们将逐步构建的单个 baml 提示词

    cp ./walkthrough/01-agent.baml baml_src/agent.baml

生成 BAML 客户端代码

    npx baml-cli generate

为本节启用 BAML 日志

    export BAML_LOG=debug

添加 CLI 接口

    cp ./walkthrough/01-cli.ts src/cli.ts

更新 index.ts 以使用 CLI

    cp ./walkthrough/01-index.ts src/index.ts

添加智能体实现

    cp ./walkthrough/01-agent.ts src/agent.ts

BAML 代码配置为默认使用 OPENAI_API_KEY

在测试时，你可以根据需要更改模型/提供商

        client "openai/gpt-4o"

[BAML 客户端文档可以在这里找到](https://docs.boundaryml.com/guide/baml-basics/switching-llms)

例如，你可以配置 [gemini](https://docs.boundaryml.com/ref/llm-client-providers/google-ai-gemini) 
或 [anthropic](https://docs.boundaryml.com/ref/llm-client-providers/anthropic) 作为你的模型提供商。

如果你想不做更改直接运行示例，可以将 OPENAI_API_KEY 环境变量设置为任何有效的 openai 密钥。


    export OPENAI_API_KEY=...

试试看

    npx tsx src/index.ts hello

你应该会看到来自模型的熟悉响应

    {
  intent: 'done_for_now',
  message: 'Hello! How can I assist you today?'
}


# 第二章 - 添加计算器工具

让我们为智能体添加一些计算器工具。

首先，让我们为计算器添加一个工具定义

这些是我们将要求模型返回为智能体循环中"下一步"的简单结构化输出。


    cp ./walkthrough/02-tool_calculator.baml baml_src/tool_calculator.baml

现在，让我们更新智能体的 DetermineNextStep 方法以
将计算器工具作为潜在的下一步暴露


    cp ./walkthrough/02-agent.baml baml_src/agent.baml

生成更新后的 BAML 客户端

    npx baml-cli generate

尝试计算器

    npx tsx src/index.ts 'can you add 3 and 4'

你应该会看到对计算器的工具调用

    {
  intent: 'add',
  a: 3,
  b: 4
}


# 第三章 - 在循环中处理工具调用

现在让我们添加一个真正的智能体循环，可以运行工具并从 LLM 获取最终答案。

首先，让我们更新智能体以处理工具调用


    cp ./walkthrough/03-agent.ts src/agent.ts

现在，让我们试试看


    npx tsx src/index.ts 'can you add 3 and 4'

你应该会看到智能体调用工具然后返回结果

    {
  intent: 'done_for_now',
  message: 'The sum of 3 and 4 is 7.'
}

对于下一步，我们将进行更复杂的计算，让我们关闭 baml 日志以获得更简洁的输出

    export BAML_LOG=off

尝试多步计算

    npx tsx src/index.ts 'can you add 3 and 4, then add 6 to that result'

你会注意到乘法和除法等工具不可用

    npx tsx src/index.ts 'can you multiply 3 and 4'

接下来，让我们为其他计算器工具添加处理程序


    cp ./walkthrough/03b-agent.ts src/agent.ts

测试减法

    npx tsx src/index.ts 'can you subtract 3 from 4'

现在，让我们测试乘法工具


    npx tsx src/index.ts 'can you multiply 3 and 4'

最后，让我们测试一个包含多个操作的更复杂计算


    npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'


# 第四章 - 向 agent.baml 添加测试

让我们向 BAML 智能体添加一些测试。

首先，保留 baml 日志启用状态

    export BAML_LOG=debug

接下来，让我们向智能体添加一些测试

我们将从一个简单的测试开始，检查智能体处理基本计算的能力。


    cp ./walkthrough/04-agent.baml baml_src/agent.baml

运行测试

    npx baml-cli test

现在，让我们用断言改进测试！

断言是确保智能体按预期工作的好方法，
可以轻松扩展以检查更复杂的行为。


    cp ./walkthrough/04b-agent.baml baml_src/agent.baml

运行测试

    npx baml-cli test

随着你添加更多测试，你可以禁用日志以保持输出整洁。
在迭代特定测试时可能需要打开它们。


    export BAML_LOG=off

现在，让我们添加一些更复杂的测试用例，
我们在智能体上下文窗口进行中恢复的地方


    cp ./walkthrough/04c-agent.baml baml_src/agent.baml

让我们试着运行它


    npx baml-cli test


# 第五章 - 多种人工工具

在本节中，我们将添加支持多种用于联系人工的工具。


在本节中，我们将禁用 baml 日志。如果你想查看更多详细信息，可以选择启用它们。

    export BAML_LOG=off

首先，让我们添加一个可以向人工请求澄清的工具

这将与"done_for_now"工具不同，
可以用来更灵活地处理智能体中不同类型的人工交互


    cp ./walkthrough/05-agent.baml baml_src/agent.baml

接下来，让我们重新生成客户端代码

注意——如果你使用 BAML 的 VSCode 扩展，
客户端会在你保存文件时自动重新生成。


    npx baml-cli generate

现在，让我们更新智能体以使用新工具


    cp ./walkthrough/05-agent.ts src/agent.ts

接下来，让我们更新 CLI 以通过在 CLI 上请求用户输入来处理澄清请求


    cp ./walkthrough/05-cli.ts src/cli.ts

让我们试试看


    npx tsx src/index.ts 'can you multiply 3 and FD*(#F&& '

接下来，让我们添加一个测试来检查智能体处理澄清请求的能力


    cp ./walkthrough/05b-agent.baml baml_src/agent.baml

现在我们可以再次运行测试


    npx baml-cli test

你会注意到新测试通过了，但 hello world 测试失败了

这是因为智能体的默认行为是返回"done_for_now"


    cp ./walkthrough/05c-agent.baml baml_src/agent.baml

验证测试通过

    npx baml-cli test


# 第六章 - 使用推理自定义提示词

在本节中，我们将探索如何自定义具有推理步骤的智能体提示词。

这是[第二要素 - 掌控你的提示词](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-2-own-your-prompts.md)的核心内容。

关于推理的深入探讨，请参阅 AI That Works 上的文章[推理模型与推理步骤](https://github.com/hellovai/ai-that-works/tree/main/2025-04-07-reasoning-models-vs-prompts)。


在本节中，保留 baml 日志启用状态会很有帮助

    export BAML_LOG=debug

更新智能体提示词以包含推理步骤


    cp ./walkthrough/06-agent.baml baml_src/agent.baml

生成更新后的客户端

    npx baml-cli generate

现在你可以用简单的提示词尝试一下


    npx tsx src/index.ts 'can you multiply 3 and 4'

你应该会看到 baml 日志输出显示推理步骤

#### 可选挑战

在工具输出格式中添加一个字段，将推理步骤包含在输出中！



# 第七章 - 自定义上下文窗口

在本节中，我们将探索如何自定义智能体的上下文窗口。

这是[第三要素 - 掌控你的上下文窗口](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-3-own-your-context-window.md)的核心内容。


更新智能体以美化模型的上下文窗口输出


    cp ./walkthrough/07-agent.ts src/agent.ts

测试格式化效果

    BAML_LOG=info npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'

接下来，让我们更新智能体使用 XML 格式化

这是一种非常流行的向模型传递数据的格式，

原因之一是 XML 的 token 效率较高。


    cp ./walkthrough/07b-agent.ts src/agent.ts

让我们测试一下


    BAML_LOG=info npx tsx src/index.ts 'can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result'

更新测试以匹配新的输出格式


    cp ./walkthrough/07c-agent.baml baml_src/agent.baml

查看更新后的测试


    npx baml-cli test


# 第八章 - 添加 API 端点

添加一个 Express 服务器来通过 HTTP 暴露智能体。

在本节中，我们将禁用 baml 日志。如果你想查看更多详细信息，可以选择启用它们。

    export BAML_LOG=off

安装 Express 和类型定义

    npm install express && npm install --save-dev @types/express supertest

添加服务器实现

    cp ./walkthrough/08-server.ts src/server.ts

启动服务器

    npx tsx src/server.ts

使用 curl 测试（在另一个终端中）

    curl -X POST http://localhost:3000/thread \
  -H "Content-Type: application/json" \
  -d '{"message":"can you add 3 and 4"}'

你应该会收到来自智能体的回复，包括
智能体追踪，最终消息类似：


    {"intent":"done_for_now","message":"The sum of 3 and 4 is 7."}


# 第九章 - 内存状态和异步澄清

添加状态管理和异步澄清支持。

在本节中，我们将禁用 baml 日志。如果你想查看更多详细信息，可以选择启用它们。

    export BAML_LOG=off

添加一些简单的线程内存状态管理

    cp ./walkthrough/09-state.ts src/state.ts

更新服务器以使用状态管理

- 使用 `ThreadStore` 添加线程状态管理
- 从 /thread 端点返回线程 ID 和响应 URL
- 实现 GET /thread/:id
- 实现 POST /thread/:id/response


    cp ./walkthrough/09-server.ts src/server.ts

启动服务器

    npx tsx src/server.ts

测试澄清流程

    curl -X POST http://localhost:3000/thread \
  -H "Content-Type: application/json" \
  -d '{"message":"can you multiply 3 and xyz"}'


# 第十章 - 添加人工审批

添加对操作人工审批的支持。

在本节中，我们将禁用 baml 日志。如果你想查看更多详细信息，可以选择启用它们。

    export BAML_LOG=off

更新服务器以处理人工审批

- 导入 `handleNextStep` 以执行已批准的操作
- 添加两种负载类型以区分审批和响应
- 在端点中不同处理响应和审批
- 出错时显示更好的错误消息


    cp ./walkthrough/10-server.ts src/server.ts

向智能体添加一些方法来处理审批和响应

    cp ./walkthrough/10-agent.ts src/agent.ts

启动服务器

    npx tsx src/server.ts

测试带审批的除法

    curl -X POST http://localhost:3000/thread \
  -H "Content-Type: application/json" \
  -d '{"message":"can you divide 3 by 4"}'

你应该看到：

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

用另一个 curl 调用拒绝请求，更改线程 ID

    curl -X POST 'http://localhost:3000/thread/{thread_id}/response' \
  -H "Content-Type: application/json" \
  -d '{"type": "approval", "approved": false, "comment": "I dont think thats right, use 5 instead of 4"}'

你应该看到：最后的工具调用现在是 `"intent":"divide","a":3,"b":5`

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

现在你可以批准操作

    curl -X POST 'http://localhost:3000/thread/{thread_id}/response' \
  -H "Content-Type: application/json" \
  -d '{"type": "approval", "approved": true}'

你应该看到最终消息包含工具响应和最终结果！

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


# 第十一章 - 通过邮件进行人工审批

在本节中，我们将添加通过邮件进行人工审批的支持。

这开始会稍微有些刻意，只是为了掌握概念——

我们将首先从 CLI 调用工作流，但对 `divide`
和 `request_more_information` 的审批将通过邮件处理，
然后最终的 `done_for_now` 答案将打印回 CLI

虽然有些刻意，但这很好地展示了[第七要素 - 使用工具联系人类](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-7-contact-humans-with-tools.md)的灵活性。


在本节中，我们将禁用 baml 日志。如果你想查看更多详细信息，可以选择启用它们。

    export BAML_LOG=off

安装 HumanLayer

    npm install humanlayer

更新 CLI 以通过邮件将 `divide` 和 `request_more_information` 发送给人工

    cp ./walkthrough/11-cli.ts src/cli.ts

运行 CLI

    npx tsx src/index.ts 'can you divide 4 by 5'

程序的最后一行应该提到人工审查步骤

    nextStep { intent: 'divide', a: 4, b: 5 }
HumanLayer: Requested human approval from HumanLayer cloud

继续回复邮件并提供一些反馈：

![reject-email](https://github.com/humanlayer/12-factor-agents/blob/main/workshops/2025-05/walkthrough/11-email-reject.png?raw=true)


你应该会收到另一封邮件，其中包含基于你反馈的更新尝试！

你可以批准这一个：

![approve-email](https://github.com/humanlayer/12-factor-agents/blob/main/workshops/2025-05/walkthrough/11-email-approve.png?raw=true)


你的最终输出应该类似于

    nextStep {
 intent: 'done_for_now',
 message: 'The division of 4 by 5 is 0.8. If you have any other calculations or questions, feel free to ask!'
}
The division of 4 by 5 is 0.8. If you have any other calculations or questions, feel free to ask!

让我们也实现 `request_more_information` 流程


    cp ./walkthrough/11b-cli.ts src/cli.ts

让我们通过请求一个包含乱码输入的计算来测试 require_approval 流程：


    npx tsx src/index.ts 'can you multiply 4 and xyz'

你应该会收到一封请求澄清的邮件

    Can you clarify what 'xyz' represents in this context? Is it a specific number, variable, or something else?

你可以回复类似这样的内容

    use 8 instead of xyz

你应该会在 CLI 上看到最终结果

    I have multiplied 4 and xyz, using the value 8 for xyz, resulting in 32.

作为最后一步，让我们探索使用自定义 HTML 模板发送邮件


    cp ./walkthrough/11c-cli.ts src/cli.ts

先用除法试试：


    npx tsx src/index.ts 'can you divide 4 by 5'

你应该会看到一封使用自定义模板的略有不同的邮件

![custom-template-email](https://github.com/humanlayer/12-factor-agents/blob/main/workshops/2025-05/walkthrough/11-email-custom.png?raw=true)

可以按照流程运行，然后随意更新模板使其更符合你的需求

（如果你使用 cursor，简单到只需选中模板然后要求"让它更好"
就能达到效果）

也试试触发 "request_more_information"！


这就是全部内容——在下一章中，我们将构建一个完全由邮件驱动的
工作流智能体，使用 webhook 进行人工审批



# 第XX章 - HumanLayer Webhook 集成

前面的章节中使用 humanlayer SDK 的"同步模式"——这意味着
每次我们等待人工审批时，我们都会在一个循环中轮询
直到收到人工响应。

这显然不太理想，特别是对于生产工作负载，
所以在本节中我们将通过更新服务器来实现[第六要素 - 使用简单 API 启动/暂停/恢复](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-6-launch-pause-resume-zh.md)，
在联系人工后结束处理，并使用 webhook 接收结果。


在服务器中添加初始化 humanlayer 的代码


    cp ./walkthrough/12-1-server-init.ts src/server.ts

接下来，让我们更新 /thread 端点以
  
1. 异步处理请求，立即返回
2. 在 request_more_information 和 done_for_now 调用时创建人工联系


更新服务器以处理 request_clarification 响应

- 删除旧的 /response 端点和类型
- 更新 /thread 端点异步运行处理，立即返回
- 在请求人工响应时发送 state.threadId
- 添加 handleHumanResponse 函数来处理人工响应
- 添加 /webhook 端点来处理 webhook 响应


    cp ./walkthrough/12a-server.ts src/server.ts

在另一个终端启动服务器

    npx tsx src/server.ts

现在服务器正在运行，向 '/thread' 端点发送请求


__ 执行响应步骤

__ 现在处理除法的审批

__ 现在也处理 done_for_now
