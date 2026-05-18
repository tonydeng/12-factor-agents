# 第八章 - 添加 API 端点

添加一个 Express 服务器来通过 HTTP 暴露智能体。

在本节中，我们将禁用 baml 日志。如果你想查看更多详细信息，可以选择启用它们。

    export BAML_LOG=off

安装 Express 和类型定义

    npm install express && npm install --save-dev @types/express supertest

添加服务器实现

    cp ./walkthrough/08-server.ts src/server.ts

<details>
<summary>显示文件</summary>

```ts
// ./walkthrough/08-server.ts
import express from 'express';
import { Thread, agentLoop } from '../src/agent';

const app = express();
app.use(express.json());
app.set('json spaces', 2);

// POST /thread - 启动新线程
app.post('/thread', async (req, res) => {
    const thread = new Thread([{
        type: "user_input",
        data: req.body.message
    }]);
    const result = await agentLoop(thread);
    res.json(result);
});

// GET /thread/:id - 获取线程状态
app.get('/thread/:id', (req, res) => {
    // 可选 - 添加状态
    res.status(404).json({ error: "Not implemented yet" });
});

const port = process.env.PORT || 3000;
app.listen(port, () => {
    console.log(`Server running on port ${port}`);
});

export { app };
```

</details>

启动服务器

    npx tsx src/server.ts

使用 curl 测试（在另一个终端中）

    curl -X POST http://localhost:3000/thread \
  -H "Content-Type: application/json" \
  -d '{"message":"can you add 3 and 4"}'

你应该会收到来自智能体的回复，包括
智能体追踪，最终消息类似：


    {"intent":"done_for_now","message":"The sum of 3 and 4 is 7."}
