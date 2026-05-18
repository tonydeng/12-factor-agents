# 第4章 - 为agent.baml添加测试

让我们为BAML agent添加一些测试。

首先，启用baml日志。

    export BAML_LOG=debug

接下来，让我们为agent添加一些测试。

我们将从检查agent处理基本计算能力的简单测试开始。

```diff
baml_src/agent.baml
     "#
   }
+
+test MathOperation {
+  functions [DetermineNextStep]
+  args {
+    thread #"
+      {
+        "type": "user_input",
+        "data": "can you multiply 3 and 4?"
+      }
+    "#
+  }
+}
+
```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/04-agent.baml baml_src/agent.baml

</details>

运行测试。

    npx baml-cli test

现在，让我们用断言来改进测试！

断言是确保agent按预期工作的好方法，
可以轻松扩展以检查更复杂的行为。

```diff
baml_src/agent.baml
     "#
   }
+  @@assert(hello, {{this.intent == "done_for_now"}})
 }
 
     "#
   }
+  @@assert(math_operation, {{this.intent == "multiply"}})
 }
 
```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/04b-agent.baml baml_src/agent.baml

</details>

运行测试。

    npx baml-cli test

当你添加更多测试时，可以禁用日志以保持输出整洁。
你可能希望在迭代特定测试时打开它们。

    export BAML_LOG=off

现在，让我们添加一些更复杂的测试用例，
我们在进行中的agentic上下文窗口的中间恢复。

```diff
baml_src/agent.baml
     "#
   }
-  @@assert(hello, {{this.intent == "done_for_now"}})
+  @@assert(intent, {{this.intent == "done_for_now"}})
 }
 
     "#
   }
-  @@assert(math_operation, {{this.intent == "multiply"}})
+  @@assert(intent, {{this.intent == "multiply"}})
 }
 
+test LongMath {
+  functions [DetermineNextStep]
+  args {
+    thread #"
+      [
+        {
+          "type": "user_input",
+          "data": "can you multiply 3 and 4, then divide the result by 2 and then add 12 to that result?"
+        },
+        {
+          "type": "tool_call",
+          "data": {
+            "intent": "multiply",
+            "a": 3,
+            "b": 4
+          }
+        },
+        {
+          "type": "tool_response",
+          "data": 12
+        },
+        {
+          "type": "tool_call", 
+          "data": {
+            "intent": "divide",
+            "a": 12,
+            "b": 2
+          }
+        },
+        {
+          "type": "tool_response",
+          "data": 6
+        },
+        {
+          "type": "tool_call",
+          "data": {
+            "intent": "add", 
+            "a": 6,
+            "b": 12
+          }
+        },
+        {
+          "type": "tool_response",
+          "data": 18
+        }
+      ]
+    "#
+  }
+  @@assert(intent, {{this.intent == "done_for_now"}})
+  @@assert(answer, {{"18" in this.message}})
+}
+
```

<details>
<summary>跳过此步骤</summary>

    cp ./walkthrough/04c-agent.baml baml_src/agent.baml

</details>

让我们尝试运行它。

    npx baml-cli test
