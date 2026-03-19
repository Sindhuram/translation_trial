flowchart TD
  U["Instructor / Test Runner"] -->|"Natural-language question"| O["Orchestrator\neduchat/agents/orchestrator.py\nanswer_question()"]

  O --> KB["Load Knowledge Base\neduchat/rag/schema_rag.py\nload_knowledge_base()"]
  KB --> KBD["Markdown KB Files\ndata/schema_descriptions/*\ndata/pedagogy_corpus/*"]

  O --> DB["Load DB into Memory\neduchat/database/executor.py\nload_disk_db_into_memory()"]
  DB --> DBFILE["SQLite on disk\ndata/analytics.db"]

  O --> SP["SQL Planner\neduchat/agents/sql_planner.py\ngenerate_sql()"]
  SP --> CFG["Config\neduchat/config.py\nload_dotenv + client/model"]
  CFG --> ENV[".env / env vars\nANTHROPIC_API_KEY\nCLAUDE_MODEL"]
  SP -->|"API call"| LLM["Claude (Anthropic API)"]
  LLM -->|"SQL in code block"| SP

  O --> EX["Query Executor\neduchat/database/executor.py\nrun_query()"]
  EX --> ROWS["rows: list[dict]"]

  O --> VIZ["Viz Router\neduchat/viz/__init__.py\nvisualize()"]
  VIZ -->|"CHART_RENDERER=plotly"| PLOT["Plotly Engine\neduchat/viz/chart_engine.py"]
  VIZ -->|"CHART_RENDERER=altair"| ALT["Altair Engine\neduchat/viz/altair_engine.py"]

  PLOT --> FIG["figure + summary"]
  ALT --> FIG

  O --> RES["Result dict\nquestion + sql + rows + viz"]
  RES -->|"tests/test_full_pipeline.py saves HTML"| OUT["output/chart_*.html"]
