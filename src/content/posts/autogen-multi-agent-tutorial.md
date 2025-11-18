---
title: 多智能体协作入门：使用Autogen构建自动化任务小组
published: 2025-11-18
description: 学习如何使用微软的Autogen框架，创建由多个AI Agent组成的“虚拟团队”。本文将通过一个实例，教你定义不同角色的Agent，并让它们通过对话协作，自动化完成复杂任务。
image: ''
tags: [AI, Agent, Autogen, 多智能体, 协作, 自动化]
category: AI技术
draft: false
lang: zh_CN
---

## 单个Agent的局限性

尽管单个ReAct Agent已经非常强大，但在面对需要多种技能、多轮反馈和规划的复杂任务时，它仍会显得力不从心。例如，要完成“调研并撰写一篇关于AI Agent最新进展的博客文章”这个任务，需要：
-   **研究员**：搜索最新的论文和文章。
-   **作者**：根据研究资料撰写初稿。
-   **评论家**：审阅初稿并提出修改意见。
-   **代码执行者**：运行代码示例并验证结果。

让单个Agent扮演所有这些角色，往往会导致逻辑混乱和性能下降。而这，正是多智能体协作框架——**Autogen**——大放异彩的地方。

## Autogen：让Agent像团队一样对话

Autogen的核心思想非常直观：**将复杂的任务分解给一组具有不同角色和能力的专业Agent，然后让它们通过模拟对话来协作完成任务**。

在Autogen中，最核心的两个概念是：
1.  **ConversableAgent**：这是Agent的基础形式，它能够发送和接收消息，并根据收到的消息决定是否回复。
2.  **UserProxyAgent**：这是一种特殊的Agent，它可以代表人类用户。它不仅能发起对话，还能执行代码或等待人类的输入。

通过定义不同的Agent，并设定它们之间的对话规则，我们就能构建出一个自动化的“虚拟团队”。

## 实战：构建一个“研究员-作者”协作小组

下面，我们将构建一个简单的双Agent系统，来自动化完成“查找一篇关于特定主题的arXiv论文，并为其撰写摘要”的任务。

### 1. 准备工作

首先，安装`pyautogen`库，并设置你的LLM API密钥。Autogen支持多种LLM，这里我们以OpenAI为例。

```bash
pip install pyautogen
export OPENAI_API_KEY="你的OpenAI API密钥"
```

### 2. 定义工具

我们需要一个工具来搜索arXiv。我们可以自己编写一个函数，并用`@tool`装饰器将其标记为工具。

```python
# 1. 定义工具
import arxiv
from autogen.agentchat.contrib.agent_builder import tool

@tool
def search_arxiv(query: str) -> str:
    """根据关键词在arXiv上搜索论文，并返回最相关的一篇论文的摘要。"""
    client = arxiv.Client()
    search = arxiv.Search(
        query=query,
        max_results=1,
        sort_by=arxiv.SortCriterion.Relevance
    )
    results = list(client.results(search))
    if not results:
        return "没有找到相关的论文。"
    paper = results[0]
    return f"标题: {paper.title}\n摘要: {paper.summary}"
```

### 3. 配置和创建Agent

现在，我们来创建两个Agent：
-   **Researcher（研究员）**：一个`AssistantAgent`，它的任务是使用`search_arxiv`工具。
-   **user_proxy（用户代理）**：一个`UserProxyAgent`，它负责发起任务，并可以执行代码（虽然本例中用不到）。

```python
# 2. 配置和创建Agent
import autogen

# LLM配置
config_list = autogen.config_list_from_json(
    "OAI_CONFIG_LIST",
    filter_dict={"model": ["gpt-4"]},
)

# 创建研究员Agent
researcher = autogen.AssistantAgent(
    name="Researcher",
    system_message="你是一名专业的研究员。利用提供的工具来查找信息。",
    llm_config={"config_list": config_list},
)

# 创建用户代理Agent
user_proxy = autogen.UserProxyAgent(
    name="user_proxy",
    human_input_mode="TERMINATE", # 任务完成后自动结束
    max_consecutive_auto_reply=10,
    is_termination_msg=lambda x: x.get("content", "").rstrip().endswith("TERMINATE"),
    code_execution_config={"work_dir": "web"},
)

# 将工具注册给Agent
researcher.register_function(
    function_map={"search_arxiv": search_arxiv}
)
user_proxy.register_function(
    function_map={"search_arxiv": search_arxiv}
)
```

### 4. 发起对话

最后，让`user_proxy`向`researcher`发起一个任务，启动它们的协作。

```python
# 3. 发起对话
user_proxy.initiate_chat(
    researcher,
    message="查找一篇关于 'ReAct prompting' 的arXiv论文，并总结其内容。"
)
```

运行后，你将看到两个Agent在控制台开始了自动对话：
1.  `user_proxy`首先发言，提出任务。
2.  `researcher`接收到任务，思考后决定需要调用`search_arxiv`工具。
3.  `researcher`执行工具调用，并将返回的论文摘要作为自己的回复。
4.  `user_proxy`接收到摘要，任务完成，对话结束。

## 总结

你已经成功构建了你的第一个多智能体协作系统！通过Autogen，我们将一个复杂的任务自然地分解给了不同的专业角色，并通过对话实现了任务的自动化流转。

这仅仅是多智能体世界的冰山一角。你可以尝试构建更复杂的团队（如“产品经理-程序员-测试员”小组），设计更精巧的对话流程，甚至让Agent动态地学习和适应。在系列的最后一篇文章中，我们将一起展望Agent技术的未来。

---

**参考资源：**
- [Autogen 官方文档](https://microsoft.github.io/autogen/)
- [Autogen GitHub 仓库](https://github.com/microsoft/autogen)