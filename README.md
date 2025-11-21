# DOSM Dataset Chatbot — RAG + Workflow Automation (Dify)
This project implements a Retrieval-Augmented Generation (RAG) chatbot using the Dify workflow builder to answer demographic questions using DOSM (Department of Statistics Malaysia) datasets. The system includes automated data cleaning, structured RAG, evaluation, monitoring, and responsible AI safeguards.

## Quickstart
1. Clone the repository
   git clone https://github.com/YOUR_REPO/dosm-rag-chatbot.git
   cd dosm-rag-chatbot

2. Start Dify
   Follow setup: https://docs.dify.ai

3. Import the workflow
   Go to Workflows → Import
   Upload: flows/dosm_rag_workflow.json

4. Load datasets
   Create Knowledge Bases:
    DOSM-Population
    DOSM-Marriages
   Upload CSVs from /data.

5. Use the chatbot
   Try:
    “What was the Bumi Malay male population aged 20–24 in 2025?”
    “How many marriages were recorded for females aged 25–29 in 2017?”

## Tool Choice & Model Provider
| Component      | Tool                              |
| -------------- | --------------------------------- |
| Orchestration  | Dify Workflow                     |
| RAG            | Dify Vector Store                 |
| Preprocessing  | Python code nodes                 |
| LLM            | OpenAI GPT-3.5 (API key redacted) |
| Extra Features | CSV export node                   |
| Monitoring     | Dify built-in analytics           |

## Data Card
### Source
    Department of Statistics Malaysia
    https://open.dosm.gov.my
    Last updated: 2024

## Datasets
    DOSM-Population
    DOSM-Marriages
    
## RAG Design
### Chunking
    Each CSV row = 1 chunk
    No text chunking required

### Embeddings

    Model: text-embedding-3-small
    Dim: 1536

### Retrieval Settings
    k = 1 (top-1 strict retrieval)
    cosine similarity

## Limitations

    Only national-level DOSM data

    No forecasting

    Some ethnic group labels differ across years

    No state/district breakdowns

    Cannot infer missing numbers

## Future Work

    Add charts/visualization nodes

    Add more DOSM datasets (CPI, labour, income)

    Add comparison agent using LangGraph

    Add dataset freshness alerts

    Improve ethnic category normalization

# Alternative Approach (Without Dify)

If not using Dify, the system could be implemented with a fully custom stack using LangChain, LangGraph, and tool-enabled LLM agents. The high-level design is:

## Architecture Overview (LangGraph)
  
1. Assistant Node (Query Classifier)

      Role: Determine which LLM node should handle the query.

      Input: User query

      Output: query_type (e.g., "population", "marriage", "faq", "refusal")

## Example system prompt for classifier LLM:
      You are a query router. Classify the user's query into one of the following types: 
      - "population": questions about population datasets
      - "marriage": questions about marriage datasets
      - "faq": general DOSM FAQs
      - "refusal": off-topic or unclear queries

      Respond ONLY with the type as a single string.
2. Population LLM Node → Tools Node

      Input: User query + cleaned/retrieved population docs

## Process:

      Retrieve top-k vector documents (RAG)

      Preprocess and format context

      Use LLM to answer using the retrieved documents

      Tools: Optional, e.g., numeric computation, aggregation

      Output: Answer

3. Marriage LLM Node → Tools Node

      Same as population, but tailored for marriage dataset.

      Tools could calculate total marriages or generate summaries.

4. FAQ Node

      Handles generic questions about DOSM datasets.

      Uses a smaller LLM or simple retrieval from static FAQ docs.

5. Refusal / Clarifying Node

      Triggered if query cannot be classified or is off-topic

      Example output: "I’m sorry, I cannot answer this question. Can you clarify or ask about population/marriage data?"

6. End Node

      Returns final output to user.

## Python-Style Flow for LangGraph / Conditional Edges
```
from langgraph.graph import StateGraph

graph = StateGraph()

# Nodes
graph.add_node("assistant", classify_query)
graph.add_node("population_llm", population_llm)
graph.add_node("population_tools", population_tools)
graph.add_node("marriage_llm", marriage_llm)
graph.add_node("marriage_tools", marriage_tools)
graph.add_node("faq_llm", faq_llm)
graph.add_node("refusal", refusal_node)
graph.add_node("end", lambda x: x)

# Conditional edges based on assistant classification
graph.add_edge("assistant", "population_llm", condition="query_type=='population'")
graph.add_edge("population_llm", "population_tools")
graph.add_edge("population_tools", "end")

graph.add_edge("assistant", "marriage_llm", condition="query_type=='marriage'")
graph.add_edge("marriage_llm", "marriage_tools")
graph.add_edge("marriage_tools", "end")

graph.add_edge("assistant", "faq_llm", condition="query_type=='faq'")
graph.add_edge("faq_llm", "end")

graph.add_edge("assistant", "refusal", condition="query_type=='refusal'")
graph.add_edge("refusal", "end")

app = graph.compile()

```
