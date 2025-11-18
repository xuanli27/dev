---
title: 深入Agent核心：ReAct框架与工具调用详解
published: 2025-11-18
description: 揭示AI Agent“思考”与“行动”的秘密——ReAct框架。本文深入解析其工作原理，并教你如何为Agent赋予调用外部API和数据库的“工具”，实现真正强大的自主能力。
image: ''
tags: [AI, Agent, ReAct, LangChain, 工具调用, 核心原理]
category: AI技术
draft: false
lang: zh_CN
---

## 超越简单的问答

在上一篇教程中，我们构建了一个RAG问答Agent，它能根据已有知识回答问题。但真正的Agent远不止于此，它们需要能够为了实现目标而主动采取行动，例如：
-   查询最新的天气信息。
-   在数据库中下单。
-   给用户发送邮件。
-   执行一段代码来计算结果。

为了实现这些功能，Agent需要一个强大的推理框架来驱动它进行思考和行动。目前，业界最主流的框架之一就是 **ReAct**。

## ReAct：协同推理与行动

ReAct（**Re**asoning and **Act**ing）是一种巧妙的Prompting框架，它指导大语言模型（LLM）通过一个交错的“**思考 -> 行动 -> 观察**”循环来解决问题。

这个过程非常像人类解决问题的模式：
1.  **思考（Thought）**：面对一个任务，LLM首先会分析任务，并思考下一步应该做什么。
2.  **行动（Action）**：基于思考，LLM决定调用一个具体的“工具”（Tool），并确定调用这个工具需要的输入参数。
3.  **观察（Observation）**：Agent执行这个行动（即调用工具），并将工具返回的结果作为“观察”信息。

然后，Agent会将这个观察结果作为新一轮循环的输入，再次进行“思考 -> 行动 -> 观察”，直到任务最终完成。

## 工具（Tools）：Agent能力的延伸

“工具”是ReAct框架的核心概念。**工具是Agent可以调用的任何外部函数或API**。它可以是一个简单的数学计算器，一个网络搜索引擎，一个数据库查询接口，甚至是另一个Agent。

通过为Agent提供一系列工具，我们极大地扩展了它的能力边界，使其不再局限于LLM内部的知识，而是能够与真实世界进行交互。

## 实战：构建一个会使用搜索引擎的Agent

下面，我们将使用LangChain构建一个简单的ReAct Agent，并为它配备一个“网络搜索”工具。这里我们使用Tavily作为搜索引擎，你需要先获取一个API密钥。

### 1. 准备工作

首先，安装必要的库并设置环境变量。

```bash
pip install langchain langchain_community langchain_experimental tavily-python
export TAVILY_API_KEY="你的Tavily API密钥"
```

### 2. 定义工具

LangChain让定义工具变得非常简单。我们直接从库中导入`TavilySearchResults`工具。

```python
# 1. 定义工具
from langchain_community.tools.tavily_search import TavilySearchResults

tools = [TavilySearchResults(max_results=1)]
```

### 3. 创建Agent

接下来，我们使用LangChain提供的`create_react_agent`工厂函数来快速创建一个遵循ReAct范式的Agent。这需要一个LLM、我们定义的工具集和一个特殊的Prompt模板。

这个Prompt模板至关重要，它明确地指示了LLM应该如何进行思考、如何选择工具、以及如何格式化它的输出。

```python
# 2. 创建Agent
from langchain import hub
from langchain.agents import AgentExecutor, create_react_agent
from langchain_community.llms import Ollama

# 获取官方的ReAct Prompt模板
prompt = hub.pull("hwchase17/react")

# 定义LLM
llm = Ollama(model="llama3")

# 创建Agent
agent = create_react_agent(llm, tools, prompt)
```

### 4. 创建Agent执行器并运行

最后，我们创建一个`AgentExecutor`来运行我们的Agent，并观察它的行动过程。

```python
# 3. 创建并运行Agent
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

response = agent_executor.invoke({
    "input": "LangChain框架的最新版本是多少？它是在什么时候发布的？"
})

print(response["output"])
```

当你运行这段代码时，将会在控制台看到`verbose=True`打印出的完整思考链：

```
> Entering new AgentExecutor chain...

Thought: 用户在问LangChain的最新版本号和发布日期。我需要使用网络搜索工具来查找这个信息。
Action:
{
  "action": "tavily_search_results_json",
  "action_input": {
    "query": "LangChain latest version release date"
  }
}
Observation: [{"url": "...", "content": "LangChain v0.1.20 was released on May 28, 2024..."}]
Thought: 我已经找到了答案。最新版本是v0.1.20，发布于2024年5月28日。现在我可以回答用户的问题了。
Final Answer: LangChain框架的最新版本是v0.1.20，它是在2024年5月28日发布的。

> Finished chain.
```

## 总结

通过本教程，你已经掌握了驱动现代AI Agent的核心引擎——ReAct框架。你理解了“思考-行动-观察”循环的工作原理，并学会了如何通过定义“工具”来赋予Agent与外部世界交互的超能力。

这是从一个简单的信息处理器到一个真正的自主问题解决者的关键一步。在接下来的文章中，我们将探索更高级的主题：多智能体协作。

---

**参考资源：**
- [ReAct 原始论文: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629)
- [LangChain Agents 官方文档](https://python.langchain.com/docs/modules/agents/)