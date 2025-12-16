
+++
title = "Productionizing RAG: Beyond the Naive Loop"
date = 2023-10-25
description = "Naive RAG is easy. Production RAG requires query routing, re-ranking, and agentic reasoning. A look at LlamaIndex and LangGraph."
[taxonomies]
categories = ["ETC/CONF", "BIN/CONF"]
tags = ["llm", "rag", "langchain", "architecture"]
+++

Retrieval Augmented Generation (RAG) has moved past the "Hello World" phase of dumping PDFs into a vector database. The industry is now grappling with **Context Window Pollution** and **Retrieval Latency**.

## The Architecture of Advanced RAG

We are seeing a shift from linear chains to **Agentic Graphs**. Instead of `Retrieve -> Generate`, we now have `Route -> Retrieve -> Grade -> Generate`.

### 1. Query Routing

Not every query needs a vector search. Some are summarization tasks; others are factual lookups. A router decides the path.

```python
from typing import Literal
from langchain_core.pydantic_v1 import BaseModel

class RouteQuery(BaseModel):
    """Route a user query to the most relevant datasource."""
    datasource: Literal["vectorstore", "web_search"]

llm = ChatOpenAI(model="gpt-4", temperature=0)
structured_llm_router = llm.with_structured_output(RouteQuery)

# The LLM acts as a classifier before doing any heavy lifting
result = structured_llm_router.invoke("How do I configure a Zola theme?")
print(result.datasource) # 'vectorstore'
2. Re-ranking (The Secret Sauce)
Vector similarity (Cosine/Euclidean) is often structurally correct but semantically vague. Introducing a Cross-Encoder re-ranker step after retrieval but before LLM context injection typically boosts accuracy by 15-20%.

Performance Note: Re-ranking is computationally expensive. Use it on the top 10-20 retrieved chunks, not the whole dataset.

The Shift to Graph-Based Flows
Libraries like LangGraph represent state as a graph edge. This allows loops—essential for "Agentic RAG" where the model can decide the retrieved context is insufficient and re-write its own search query to try again.

The Python AI landscape is moving away from "Scripting" LLMs to "Architecting" cognitive flows.