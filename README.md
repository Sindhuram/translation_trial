flowchart TD
  U[Instructor / Test Runner] -->|Natural-language question| O[Orchestrator<br/>educhat/agents/orchestrator.py<br/>answer_question()]

  O --> KB[Load Knowledge Base<br/>educhat/rag/schema_rag.py<br/>load_knowledge_base()]
  KB --> KBD[(Markdown KB Files<br/>data/schema_descriptions/*<br/>data/pedagogy_corpus/*)]

  O --> DB[Load DB into Memory<br/>educhat/database/executor.py<br/>load_disk_db_into_memory()]
  DB --> DBFILE[(SQLite on disk<br/>data/analytics.db)]

  O -->|Build grounded prompt| SP[SQL Planner<br/>educhat/agents/sql_planner.py<br/>generate_sql()]
  SP --> CFG[Config<br/>educhat/config.py<br/>load_dotenv + client/model]
  CFG --> ENV[(.env / env vars<br/>ANTHROPIC_API_KEY<br/>CLAUDE_MODEL)]
  SP -->|API call| LLM[Claude (Anthropic API)]
  LLM -->|SQL in code block| SP

  O -->|Execute SQL| EX[Query Executor<br/>educhat/database/executor.py<br/>run_query()]
  EX --> ROWS[(rows: list[dict])]

  O -->|Choose + render chart| VIZ[Viz Router<br/>educhat/viz/__init__.py<br/>visualize()]
  VIZ -->|CHART_RENDERER=plotly| PLOT[Plotly Engine<br/>educhat/viz/chart_engine.py]
  VIZ -->|CHART_RENDERER=altair| ALT[Altair Engine<br/>educhat/viz/altair_engine.py]

  PLOT --> FIG[(figure + summary)]
  ALT --> FIG

  O --> RES[(Result dict<br/>question, sql, rows, viz)]

  RES -->|tests/test_full_pipeline.py saves HTML| OUT[(output/chart_*.html)]
