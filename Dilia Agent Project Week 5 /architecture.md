KAM types question in Slack
    │
    ▼
n8n webhook trigger (Slack event)
    │
    ▼
POST /ask  →  Flask server  →  LangGraph agent
                                      │
                          ┌───────────┼───────────┐
                          ▼           ▼            ▼
                    [Node 1]      [Node 2]     [Node 3]
                  understand    fetch client  retrieve schema
                  question      from          from ChromaDB
                                Salesforce    RAG
                                API (2)
                          │           │            │
                          └───────────┼────────────┘
                                      ▼
                                  [Node 4]
                                generate SQL
                                (with schema
                                 context)
                                      │
                                      ▼
                                  [Node 5]
                                execute SQL on
                                Supabase API (3)
                                      │
                           ┌──────────┴──────────┐
                           ▼                     ▼
                      SQL succeeds           SQL error
                           │                → retry Node 4
                           ▼                  with error msg
                        [Node 6]
                      combine Salesforce
                      + Supabase results
                      format final answer
                      OpenAI API (1)
                           │
                           ▼
                  n8n posts answer to Slack