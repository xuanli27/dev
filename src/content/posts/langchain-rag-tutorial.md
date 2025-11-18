---
title: 实战教程：用LangChain构建一个知识库问答Agent
published: 2025-11-18
description: 手把手教你使用LangChain构建一个基于私有文档的RAG（检索增强生成）问答Agent。从加载数据到创建检索链，一步步实现你的专属知识库。
image: ''
tags: [AI, Agent, LangChain, RAG, 实战教程, 问答机器人]
category: AI技术
draft: false
lang: zh_CN
---

## 前言：什么是RAG？

在构建问答Agent时，我们常常希望它能基于我们提供的特定文档（如产品手册、公司规定、学习笔记等）来回答问题，而不是仅仅依赖其训练数据中的通用知识。实现这一目标的核心技术就是**RAG（Retrieval-Augmented Generation，检索增强生成）**。

RAG的工作流程可以概括为两步：
1.  **检索（Retrieval）**：当用户提出问题时，系统首先从你的知识库（文档）中检索出与问题最相关的片段。
2.  **生成（Generation）**：系统将用户的问题和检索到的相关文段一起发送给大语言模型（LLM），让LLM基于这些上下文信息，生成精准的回答。

LangChain为实现RAG流程提供了强大的支持。本教程将带你一步步构建一个完整的RAG问答Agent。

## 准备工作

在开始之前，请确保你已经安装了必要的Python库。我们将使用`langchain`来构建核心逻辑，`ollama`来本地运行大模型，`chromadb`作为向量数据库，以及`bs4`来解析HTML文档。

```bash
pip install langchain langchain_community ollama chromadb beautifulsoup4
```

同时，请确保你已经通过Ollama在本地运行了一个大语言模型（如`llama3`）。

## 步骤一：加载文档

首先，我们需要一个数据源。这里我们以加载一个网页内容为例。LangChain提供了丰富的`DocumentLoader`，可以加载各种类型的数据。

```python
# 1. 加载文档
from langchain_community.document_loaders import WebBaseLoader

loader = WebBaseLoader(
    web_paths=("https://lilianweng.github.io/posts/2023-06-23-agent/",),
    bs_kwargs=dict(
        parse_only=("article",),
    ),
)
docs = loader.load()
```

## 步骤二：分割文档

直接将整篇长文档交给LLM处理，效率低且效果差。我们需要将其分割成更小的、语义完整的文本块（Chunks）。

```python
# 2. 分割文档
from langchain.text_splitter import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
splits = text_splitter.split_documents(docs)
```

## 步骤三：创建并存储Embeddings

为了让计算机能够理解文本并判断相关性，我们需要将文本块转化为向量（Embeddings）。然后，将这些向量存储在向量数据库（Vector Store）中，以便后续快速检索。

```python
# 3. 创建并存储Embeddings
from langchain_community.embeddings import OllamaEmbeddings
from langchain_community.vectorstores import Chroma

# 创建Embeddings模型
embeddings = OllamaEmbeddings(model="llama3")

# 创建向量数据库并存入分割后的文档
vectorstore = Chroma.from_documents(documents=splits, embedding=embeddings)
```

## 步骤四：创建检索器

向量数据库本身只是一个存储。我们需要创建一个`retriever`（检索器），它定义了如何根据用户查询从数据库中检索相关文档。

```python
# 4. 创建检索器
retriever = vectorstore.as_retriever()
```

## 步骤五：创建并运行检索链（Retrieval Chain）

现在，万事俱备。我们可以创建一个“检索链”，将前面所有的步骤串联起来：接收用户问题 -> 格式化Prompt -> 调用检索器获取相关文档 -> 将问题和文档传给LLM -> 生成最终答案。

LangChain的表达式语言（LCEL）让这个过程变得异常简单。

```python
# 5. 创建并运行检索链
from langchain.chains import create_retrieval_chain
from langchain.chains.combine_documents import create_stuff_documents_chain
from langchain_core.prompts import ChatPromptTemplate
from langchain_community.llms import Ollama

# 定义LLM
llm = Ollama(model="llama3")

# 定义Prompt模板
prompt = ChatPromptTemplate.from_template("""Answer the following question based only on the provided context:

<context>
{context}
</context>

Question: {input}""")

# 创建文档处理链
document_chain = create_stuff_documents_chain(llm, prompt)

# 创建检索链
retrieval_chain = create_retrieval_chain(retriever, document_chain)

# 发起调用
response = retrieval_chain.invoke({"input": "What is Task Decomposition?"})
print(response["answer"])
```

运行代码后，你将看到LLM基于你提供的网页内容，给出了关于“Task Decomposition”的精准回答。

## 总结

恭喜你！你已经成功构建了一个功能完备的知识库问答Agent。通过本教程，你掌握了RAG的核心流程，并学会了如何使用LangChain将文档加载、分割、向量化、检索和生成等步骤有机地结合起来。

这只是一个开始。你可以尝试替换不同的文档加载器、分割策略、向量数据库和LLM模型，来满足更复杂的业务需求。在下一篇文章中，我们将更深入地探讨Agent的核心机制：ReAct框架与工具调用。

---

**参考资源：**
- [LangChain RAG 官方教程](https://python.langchain.com/docs/use_cases/question_answering/)