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


