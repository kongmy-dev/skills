---
name: agentic-rag-audit
description: Audit an AI search or Agentic RAG platform's performance and behavior. Use this skill when the user asks to audit, test, or evaluate an AI Search engine, a RAG application, or wants to check how their brand performs across different AI research tools (e.g., Perplexity, ChatGPT). This provides a step-by-step observable audit framework.
---

# Agentic RAG Audit Skill

This skill provides step-by-step instructions for performing an "Observable Agentic RAG Audit". Because modern AI Search platforms use Agentic RAG (incorporating hidden steps like planning, routing, and grading), content creators often only see the final result. This audit helps identify where content is found, where it is ignored, and where it loses out to competitors during intermediate research steps.

> **Credit:** This audit methodology is based on the "Observable Agentic RAG Audit" developed by Mike King and [iPullRank](https://ipullrank.com/).

## Audit Methodology

When tasked with auditing a RAG application or testing brand performance on an AI Search platform, follow these steps meticulously:

### Step 1: Define the Query Set
Identify a diverse set of queries relevant to the brand or application. These should include:
- **Informational Queries:** "How to...", "What is..."
- **Navigational/Branded Queries:** Specific to the company or product.
- **Commercial Investigation:** "Best tools for...", "Compare X vs Y".

### Step 2: Establish the Golden Dataset
For each query, document what the *ideal* response should be.
- What key facts must be included?
- What sources or URLs should ideally be cited?

### Step 3: Run the Queries (Execution Phase)
Run the queries against the target AI Search Platform or Agentic RAG pipeline. If auditing a custom-built pipeline, ensure logging/observability is turned on to see intermediate steps (e.g., LangSmith traces).

### Step 4: Record Observable Outputs
For each query execution, systematically record the following in a spreadsheet or JSON format:
- **Final Output Quality:** Did it answer the query accurately?
- **Citations / References:** Which URLs or documents were cited in the final output?
- **Brand Visibility:** Was the brand mentioned favorably, unfavorably, or omitted?

### Step 5: Analyze Intermediate Steps (The "Black Box")
If you have access to the agent's intermediate logs (the "Observable" part of the audit), record:
- **Planner Output:** Did the agent decompose the query correctly?
- **Tool Routing:** Which tools did it select? Did it search the web, query a database, or read internal docs?
- **Iterative Retrieval:** Did it perform multiple searches? What were the exact search queries it formulated?
- **Grading Rejections:** Did the agent draft a response citing the brand, but the self-correction/grading step rejected it? Why?

### Step 6: Identify the Gap
Synthesize the findings to answer the core audit questions:
1. **Eligibility Gap:** Is the content simply not being retrieved (e.g., crawler blocks, poor traditional SEO, timeout issues)?
2. **Relevance Gap:** Is the content retrieved but ignored by the grading step because it lacks specific answers or structure?
3. **Competitive Gap:** Is the content retrieved but discarded in favor of a competitor's content during the synthesis phase?

### Step 7: Recommend Improvements
Based on the gaps identified, output actionable recommendations. For instance:
- If suffering from an *Eligibility Gap*, recommend technical SEO fixes or structured data improvements.
- If suffering from a *Relevance Gap*, recommend restructuring the content to directly answer the agent's intermediate search queries.
