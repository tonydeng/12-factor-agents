[← 返回 README](../README-zh.md)

### 3. 掌控你的上下文窗口

你不必使用标准的基于消息的格式来向LLM传递上下文。

> #### 在任何给定时刻，你对LLM的输入是"到目前为止发生了什么，下一步是什么"

<!-- todo syntax highlighting -->
<!-- ![130-own-your-context-building](https://github.com/humanlayer/12-factor-agents/blob/main/img/130-own-your-context-building.png) -->

一切都是上下文工程。[LLM是无状态函数](https://thedataexchange.media/baml-revolution-in-ai-engineering/)，将输入转换为输出。要获得最佳输出，你需要提供最佳输入。

创建优秀的上下文意味着：

- 给模型的提示词和指令
- 你检索的任何文档或外部数据（如RAG）
- 任何过去的状态、工具调用、结果或其他历史
- 来自相关但独立的历史/对话的任何过去消息或事件（记忆）
- 关于输出何种结构化数据的指令

![image](https://github.com/user-attachments/assets/0f1f193f-8e94-4044-a276-576bd7764fd0)


### 关于上下文工程

本指南全是关于从当今模型中获取尽可能多的内容。值得注意的是未提及的：

- 修改模型参数如temperature、top_p、frequency_penalty、presence_penalty等
- 训练你自己的补全或嵌入模型
- 微调现有模型

再次强调，我不知道向LLM传递上下文的最佳方式，但我知道你希望有灵活性能够尝试一切。

#### 标准与自定义上下文格式

大多数LLM客户端使用标准的基于消息的格式，如下：

```yaml
[
  {
    "role": "system",
    "content": "You are a helpful assistant..."
  },
  {
    "role": "user",
    "content": "Can you deploy the backend?"
  },
  {
    "role": "assistant",
    "content": null,
    "tool_calls": [
      {
        "id": "1",
        "name": "list_git_tags",
        "arguments": "{}"
      }
    ]
  },
  {
    "role": "tool",
    "name": "list_git_tags",
    "content": "{\"tags\": [{\"name\": \"v1.2.3\", \"commit\": \"abc123\", \"date\": \"2024-03-15T10:00:00Z\"}, {\"name\": \"v1.2.2\", \"commit\": \"def456\", \"date\": \"2024-03-14T15:30:00Z\"}, {\"name\": \"v1.2.1\", \"commit\": \"abe033d\", \"date\": \"2024-03-13T09:15:00Z\"}]}",
    "tool_call_id": "1"
  }
]
```

虽然这在大多数用例中效果很好，但如果你真的想从当今的LLM中获得最大收益，你需要以最节省令牌和最有效利用注意力的方式将上下文传递给LLM。

作为标准基于消息格式的替代方案，你可以构建针对你的用例优化的自定义上下文格式。例如，你可以使用自定义对象并将它们打包到一个或多个用户、系统、助手或工具消息中。

这是一个将整个上下文窗口放入单个用户消息的示例：
```yaml

[
  {
    "role": "system",
    "content": "You are a helpful assistant..."
  },
  {
    "role": "user",
    "content": |
            Here's everything that happened so far:

        <slack_message>
            From: @alex
            Channel: #deployments
            Text: Can you deploy the backend?
        </slack_message>

        <list_git_tags>
            intent: "list_git_tags"
        </list_git_tags>

        <list_git_tags_result>
            tags:
              - name: "v1.2.3"
                commit: "abc123"
                date: "2024-03-15T10:00:00Z"
              - name: "v1.2.2"
                commit: "def456"
                date: "2024-03-14T15:30:00Z"
              - name: "v1.2.1"
                commit: "ghi789"
                date: "2024-03-13T09:15:00Z"
        </list_git_tags_result>

        what's the next step?
    }
]
```

模型可能通过你提供的工具模式推断你在问它"下一步是什么"，但将其融入你的提示词模板中并无坏处。

### 代码示例

我们可以用类似的方式构建：

```python

class Thread:
  events: List[Event]

class Event:
  # 可以只是字符串，也可以是显式的——由你决定
  type: Literal["list_git_tags", "deploy_backend", "deploy_frontend", "request_more_information", "done_for_now", "list_git_tags_result", "deploy_backend_result", "deploy_frontend_result", "request_more_information_result", "done_for_now_result", "error"]
  data: ListGitTags | DeployBackend | DeployFrontend | RequestMoreInformation |
        ListGitTagsResult | DeployBackendResult | DeployFrontendResult | RequestMoreInformationResult | string

def event_to_prompt(event: Event) -> str:
    data = event.data if isinstance(event.data, str) \
           else stringifyToYaml(event.data)

    return f"<{event.type}>\n{data}\n</{event.type}>"


def thread_to_prompt(thread: Thread) -> str:
  return '\n\n'.join(event_to_prompt(event) for event in thread.events)
```

#### 示例上下文窗口

以下是这个方法中上下文窗口可能的样子：

**初始Slack请求：**
```xml
<slack_message>
    From: @alex
    Channel: #deployments
    Text: Can you deploy the latest backend to production?
</slack_message>
```

**列出Git标签后：**
```xml
<slack_message>
    From: @alex
    Channel: #deployments
    Text: Can you deploy the latest backend to production?
    Thread: []
</slack_message>

<list_git_tags>
    intent: "list_git_tags"
</list_git_tags>

<list_git_tags_result>
    tags:
      - name: "v1.2.3"
        commit: "abc123"
        date: "2024-03-15T10:00:00Z"
      - name: "v1.2.2"
        commit: "def456"
        date: "2024-03-14T15:30:00Z"
      - name: "v1.2.1"
        commit: "ghi789"
        date: "2024-03-13T09:15:00Z"
</list_git_tags_result>
```

**错误和恢复后：**
```xml
<slack_message>
    From: @alex
    Channel: #deployments
    Text: Can you deploy the latest backend to production?
    Thread: []
</slack_message>

<deploy_backend>
    intent: "deploy_backend"
    tag: "v1.2.3"
    environment: "production"
</deploy_backend>

<error>
    error running deploy_backend: Failed to connect to deployment service
</error>

<request_more_information>
    intent: "request_more_information_from_human"
    question: "I had trouble connecting to the deployment service, can you provide more details and/or check on the status of the service?"
</request_more_information>

<human_response>
    data:
      response: "I'm not sure what's going on, can you check on the status of the latest workflow?"
</human_response>
```

从这里你的下一步可能是：

```python
nextStep = await determine_next_step(thread_to_prompt(thread))
```

```python
{
  "intent": "get_workflow_status",
  "workflow_name": "tag_push_prod.yaml",
}
```

XML风格格式只是一个例子——重点是你可以构建对自己的应用程序有意义格式。如果你有灵活性尝试不同的上下文结构以及你存储什么vs传递给LLM什么，你会获得更好的质量。

掌控上下文窗口的关键好处：

1. **信息密度**：以最大化LLM理解的方式构建信息
2. **错误处理**：以帮助LLM恢复的格式包含错误信息。考虑一旦解决就从上下文窗口中隐藏错误和失败的调用。
3. **安全性**：控制传递给LLM的信息，过滤敏感数据
4. **灵活性**：根据你对用例的了解最佳调整格式
5. **令牌效率**：优化上下文格式以提高令牌效率和LLM理解

上下文包括：提示词、指令、RAG文档、历史、工具调用、记忆

记住：上下文窗口是你与LLM的主要接口。控制你构建和呈现信息的方式可以显著提高智能体的性能。

示例——信息密度——相同的消息，更少的令牌：

![Loom Screenshot 2025-04-22 at 09 00 56](https://github.com/user-attachments/assets/5cf041c6-72da-4943-be8a-99c73162b12a)


### 不要只听我说

12因素智能体发布约2个月后，上下文工程开始成为一个相当流行的术语。

<a href="https://x.com/karpathy/status/1937902205765607626"><img width="378" alt="Screenshot 2025-06-25 at 4 11 45 PM" src="https://github.com/user-attachments/assets/97e6e667-c35f-4855-8233-af40f05d6bce" /></a> <a href="https://x.com/tobi/status/1935533422589399127"><img width="378" alt="Screenshot 2025-06-25 at 4 12 59 PM" src="https://github.com/user-attachments/assets/7e6f5738-0d38-4910-82d1-7f5785b82b99" /></a>

[@lenadroid](https://x.com/lenadroid)在2025年7月还发布了一份相当不错的[上下文工程速查表](https://x.com/lenadroid/status/1943685060785524824)。

<a href="https://x.com/lenadroid/status/1943685060785524824"><img width="256" alt="image" src="https://github.com/user-attachments/assets/cac88aa3-8faf-440b-9736-cab95a9de477" /></a>



这里反复出现的主题：我不知道什么是最好的方法，但我知道你希望有灵活性能够尝试一切。


[← 掌控你的提示词](./factor-02-own-your-prompts-zh.md) | [工具是结构化输出 →](./factor-04-tools-are-structured-outputs-zh.md)
