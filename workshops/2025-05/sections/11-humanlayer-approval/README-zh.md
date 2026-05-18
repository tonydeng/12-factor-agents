# 第十一章 - 通过邮件进行人工审批

在本节中，我们将添加通过邮件进行人工审批的支持。

这开始会稍微有些刻意，只是为了掌握概念——

我们将首先从 CLI 调用工作流，但对 `divide`
和 `request_more_information` 的审批将通过邮件处理，
然后最终的 `done_for_now` 答案将打印回 CLI

虽然有些刻意，但这很好地展示了[第七要素 - 使用工具联系人类](../../../content/factor-7-contact-humans-with-tools-zh.md)的灵活性。


在本节中，我们将禁用 baml 日志。如果你想查看更多详细信息，可以选择启用它们。

    export BAML_LOG=off

安装 HumanLayer

    npm install humanlayer

更新 CLI 以通过邮件将 `divide` 和 `request_more_information` 发送给人工

```diff
src/cli.ts
 // cli.ts 让你可以从命令行调用智能体循环
 
+import { humanlayer } from "humanlayer";
 import { agentLoop, Thread, Event } from "../src/agent";
 
 
 
 export async function cli() {
     // 获取命令行参数，跳过前两个（node 和脚本名称）
 
     // 使用线程运行智能体循环
-    const result = await agentLoop(thread);
-    let lastEvent = result.events.slice(-1)[0];
+    let newThread = await agentLoop(thread);
+    let lastEvent = newThread.events.slice(-1)[0];
 
-    while (lastEvent.data.intent === "request_more_information") {
-        const message = await askHuman(lastEvent.data.message);
-        thread.events.push({ type: "human_response", data: message });
-        const result = await agentLoop(thread);
-        lastEvent = result.events.slice(-1)[0];
+    while (lastEvent.data.intent !== "done_for_now") {
+        const responseEvent = await askHuman(lastEvent);
+        thread.events.push(responseEvent);
+        newThread = await agentLoop(thread);
+        lastEvent = newThread.events.slice(-1)[0];
     }
 
     // 打印最终结果
     console.log(lastEvent.data.message);
     process.exit(0);
 }
 
-async function askHuman(message: string) {
+async function askHuman(lastEvent: Event): Promise<Event> {
+    if (process.env.HUMANLAYER_API_KEY) {
+        return await askHumanEmail(lastEvent);
+    } else {
+        return await askHumanCLI(lastEvent.data.message);
+    }
+}
+
+async function askHumanCLI(message: string): Promise<Event> {
     const readline = require('readline').createInterface({
         input: process.stdin,
     return new Promise((resolve) => {
         readline.question(`${message}\n> `, (answer: string) => {
-            resolve(answer);
+            resolve({ type: "human_response", data: answer });
         });
     });
 }
+
+export async function askHumanEmail(lastEvent: Event): Promise<Event> {
+    if (!process.env.HUMANLAYER_EMAIL) {
+        throw new Error("missing or invalid parameters: HUMANLAYER_EMAIL");
+    }
+    const hl = humanlayer({ //从环境变量读取 apiKey
+        // 此智能体的名称
+        runId: "12fa-cli-agent",
+        verbose: true,
+        contactChannel: {
+            // 智能体应通过邮件请求许可
+            email: {
+                address: process.env.HUMANLAYER_EMAIL,
+            }
+        }
+    }) 
+
+    if (lastEvent.data.intent === "divide") {
+        // 同步获取审批 - 这将阻塞直到收到回复
+        const response = await hl.fetchHumanApproval({
+            spec: {
+                fn: "divide",
+                kwargs: {
+                    a: lastEvent.data.a,
+                    b: lastEvent.data.b
+                }
+            }
+        })
+
+        if (response.approved) {
+            const result = lastEvent.data.a / lastEvent.data.b;
+            console.log("tool_response", result);
+            return {
+                "type": "tool_response",
+                "data": result
+            };
+        } else {
+            return {
+                "type": "tool_response",
+                "data": `user denied operation ${lastEvent.data.intent}
+                with feedback: ${response.comment}`
+            };
+        }
+    }
+    throw new Error(`unknown tool: ${lastEvent.data.intent}`)
+}
```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/11-cli.ts src/cli.ts

</details>

运行 CLI

    npx tsx src/index.ts 'can you divide 4 by 5'

程序的最后一行应该提到人工审查步骤

    nextStep { intent: 'divide', a: 4, b: 5 }
HumanLayer: Requested human approval from HumanLayer cloud

继续回复邮件并提供一些反馈：

![reject-email](../walkthrough/11-email-reject.png?raw=true)


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


```diff
src/cli.ts
     }) 
 
+    if (lastEvent.data.intent === "request_more_information") {
+        // 同步获取响应 - 这将阻塞直到收到回复
+        const response = await hl.fetchHumanResponse({
+            spec: {
+                msg: lastEvent.data.message
+            }
+        })
+        return {
+            "type": "tool_response",
+            "data": response
+        }
+    }
+    
     if (lastEvent.data.intent === "divide") {
         // 同步获取审批 - 这将阻塞直到收到回复
```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/11b-cli.ts src/cli.ts

</details>

让我们通过请求一个包含乱码输入的计算来测试 require_approval 流程：


    npx tsx src/index.ts 'can you multiply 4 and xyz'

你应该会收到一封请求澄清的邮件

    Can you clarify what 'xyz' represents in this context? Is it a specific number, variable, or something else?

你可以回复类似这样的内容

    use 8 instead of xyz

你应该会在 CLI 上看到最终结果

    I have multiplied 4 and xyz, using the value 8 for xyz, resulting in 32.

作为最后一步，让我们探索使用自定义 HTML 模板发送邮件


```diff
src/cli.ts
             email: {
                 address: process.env.HUMANLAYER_EMAIL,
+                // 自定义邮件正文 - jinja
+                template: `{% if type == 'request_more_information' %}
+{{ event.spec.msg }}
+{% else %}
+agent {{ event.run_id }} is requesting approval for {{event.spec.fn}}
+with args: {{event.spec.kwargs}}
+<br><br>
+reply to this email to approve
+{% endif %}`
             }
         }
```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/11c-cli.ts src/cli.ts

</details>

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
