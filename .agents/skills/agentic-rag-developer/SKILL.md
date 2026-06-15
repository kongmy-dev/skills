---
name: agentic-rag-developer
description: Build and implement advanced Agentic RAG (Retrieval-Augmented Generation) pipelines. Use this skill when the user asks to build an advanced RAG application that requires multi-step reasoning, tool routing, planning, iterative retrieval, or self-correction, or when fixing limitations of a single-shot RAG system. Make sure to use this skill whenever the user mentions Agentic RAG, iPullRank methodology, complex retrieval, or evaluation/grading of retrieved contexts.
---

# Agentic RAG Developer Skill

This skill guides the implementation of Agentic RAG pipelines. Unlike traditional single-shot RAG, Agentic RAG treats the LLM as an active agent capable of planning, using tools, and self-correcting to retrieve the most accurate information.

> **Credit:** This methodology is heavily inspired by the Agentic RAG principles outlined by Mike King and the team at [iPullRank](https://ipullrank.com/).

## The Four Pillars of Agentic RAG

When scaffolding an Agentic RAG system, ensure the architecture supports the following four pillars. These principles are framework-agnostic and should be applied whether using vanilla Python, TypeScript, or an orchestration framework.

### 1. Planning
The system must be able to decompose complex queries into a multi-step research plan before acting.
- **Implementation:** Introduce a planner step or agent that breaks down a user prompt (e.g., "Compare X and Y") into distinct retrieval tasks ("1. Find details on X. 2. Find details on Y. 3. Compare them").

### 2. Tool Routing
The system should have access to multiple retrieval tools (e.g., Vector DB, SQL Database, Web Search, API) and the autonomy to choose the right one for each sub-task.
- **Implementation:** Provide the LLM with tool descriptions and let it emit tool calls to route the query effectively.

### 3. Iterative Retrieval
The system should retrieve context, read it, and determine if it needs more information. If the initial retrieval is insufficient or misses the mark, it should re-retrieve using updated search terms.
- **Implementation:** Implement a loop where the LLM evaluates the retrieved documents against the sub-task. If the information is missing, generate a new query and search again.

### 4. Grading and Self-Correction
The system must grade its drafted response against the retrieved context to prevent hallucinations, and critique its own output before finalizing.
- **Implementation:** Use an "evaluator" LLM call (or self-reflection prompt) to grade the draft. If it fails the check, prompt the generator to fix the draft or trigger a new retrieval cycle.

## Example Code Segments (Framework-Agnostic)

Here are conceptual examples of how to implement the agentic pillars:

**1. Tool Routing Example**
```python
tools = [
    {"name": "vector_search", "description": "Search internal company documents."},
    {"name": "sql_query", "description": "Search user analytics and database records."}
]

# The LLM decides which tool to use based on the query
tool_choice = llm.choose_tool(query="How many users signed up yesterday?", tools=tools)
if tool_choice.name == "sql_query":
    context = execute_sql(tool_choice.arguments)
```

**2. Grading / Self-Correction Example**
```python
draft_response = generate_draft(query, retrieved_context)

# Grading Step
evaluation_prompt = f"""
You are a grader evaluating an answer.
Context: {retrieved_context}
Answer: {draft_response}
Does the answer strictly rely on the context? Answer YES or NO.
"""
grade = llm.generate(evaluation_prompt)

if "NO" in grade:
    # Trigger self-correction or re-retrieval
    draft_response = generate_draft(query, retrieved_context, instructions="Stick strictly to the provided context.")
```

## Guidelines for Developers
- **Framework Agnostic:** Do not force a specific framework unless requested by the user. You can build these loops using `while` loops in vanilla code, or state machines/graphs if using orchestration libraries.
- **Observability:** Ensure each step (planning, routing, grading) is observable. It's critical for debugging to see *why* the agent chose a specific tool or rejected a draft.
- **Fallbacks:** Always implement a fallback mechanism to prevent infinite loops during iterative retrieval. Limit the number of retrieval attempts.
