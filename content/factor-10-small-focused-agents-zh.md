[← 返回 README](../README-zh.md)

### 10. 小而专注的智能体

不要构建试图包揽一切的单一巨型智能体，而是要构建小而专注的智能体，每个智能体做好一件事。智能体只是更大的、主要是确定性的系统中的一个构建块。

![1a0-small-focused-agents](https://github.com/humanlayer/12-factor-agents/blob/main/img/1a0-small-focused-agents.png)

这里的关键洞察是关于LLM的局限性：任务越大越复杂，它需要的步骤就越多，这意味着更长的上下文窗口。随着上下文增长，LLM更容易迷失方向或失去专注。通过保持智能体专注于特定领域，最多3-10步，最多20步，我们可以保持上下文窗口可控并保持LLM的高性能。

> #### 随着上下文增长，LLM更容易迷失方向或失去专注

小而专注的智能体的优势：

1. **可控的上下文**：更小的上下文窗口意味着更好的LLM性能
2. **清晰的职责**：每个智能体都有明确定义的范围和目的
3. **更好的可靠性**：在复杂工作流中迷失的可能性更小
4. **更易于测试**：更容易测试和验证特定功能
5. **改进的调试**：当问题发生时更容易识别和修复

### 如果LLM变得更聪明怎么办？

如果LLM足够智能，能够处理100步以上的工作流，我们还需要这样做吗？

简而言之：是的。随着智能体和LLM的改进，它们**可能**自然会扩展到能够处理更长的上下文窗口。这意味着处理更大DAG的更多部分。这种小而专注的方法确保你今天就能获得结果，同时为随着LLM上下文窗口变得更加可靠而逐步扩展智能体范围做好准备。（如果你以前重构过大型确定性代码库，你现在可能正在点头。）

[![gif](https://github.com/humanlayer/12-factor-agents/blob/main/img/1a5-agent-scope-grow.gif)](https://github.com/user-attachments/assets/0cd3f52c-046e-4d5e-bab4-57657157c82f
)

<details>
<summary><a href="https://github.com/humanlayer/12-factor-agents/blob/main/img/1a5-agent-scope-grow.gif">GIF 版本</a></summary>
![gif](https://github.com/humanlayer/12-factor-agents/blob/main/img/1a5-agent-scope-grow.gif)
</details>

在这里，有意识地控制智能体的大小和范围，并且只以能够保持质量的方式增长是关键。正如[构建 NotebookLM 的团队所说](https://open.substack.com/pub/swyx/p/notebooklm?selection=08e1187c-cfee-4c63-93c9-71216640a5f8&utm_campaign=post-share-selection&utm_medium=web)：

> 我觉得一直以来，对我来说，AI构建中最神奇的时刻就是当我真的、真的、真的非常接近模型能力边缘的时候

无论那个边界在哪里，如果你能找到那个边界并持续正确地把握它，你就能构建出神奇的体验。这里有很多护城河可以建立，但一如既往，它们需要一些工程严谨性。

[← 压缩错误](./factor-09-compact-errors-zh.md) | [从任何地方触发 →](./factor-11-trigger-from-anywhere-zh.md)
