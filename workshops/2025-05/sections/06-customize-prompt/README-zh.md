# 第六章 - 使用推理自定义提示词

在本节中，我们将探索如何自定义具有推理步骤的智能体提示词。

这是[第二要素 - 掌控你的提示词](https://github.com/humanlayer/12-factor-agents/blob/main/content/factor-2-own-your-prompts-zh.md)的核心内容。

关于推理的深入探讨，请参阅 AI That Works 上的文章[推理模型与推理步骤](https://github.com/hellovai/ai-that-works/tree/main/2025-04-07-reasoning-models-vs-prompts)。


在本节中，保留 baml 日志启用状态会很有帮助

    export BAML_LOG=debug

更新智能体提示词以包含推理步骤


```diff
baml_src/agent.baml
 
         {{ ctx.output_format }}
+
+        首先，始终规划下一步要做什么，例如：
+
+        - ...
+        - ...
+        - ...
+
+        {...} // schema
     "#
 }
   @@assert(b, {{this.a == 3}})
 }
-        
-
```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/06-agent.baml baml_src/agent.baml

</details>

生成更新后的客户端

    npx baml-cli generate

现在你可以用简单的提示词尝试一下


    npx tsx src/index.ts 'can you multiply 3 and 4'

你应该会看到 baml 日志输出显示推理步骤

#### 可选挑战

在工具输出格式中添加一个字段，将推理步骤包含在输出中！
