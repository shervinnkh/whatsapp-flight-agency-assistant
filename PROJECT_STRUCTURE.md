# Project Structure

```
ticket_bot/
├── .env.example                      # Environment settings template
├── requirements.txt                  # Python dependencies
├── app/
│   ├── main.py                       # FastAPI entry point (message endpoints, monitoring, /chat test page)
│   │
│   ├── core/
│   │   ├── config.py                 # All settings, loaded from .env
│   │   ├── context.py                # Per-request context (trace_id, conversation_id, worker)
│   │   └── logging_config.py         # Central logging setup (console + file + JSON)
│   │
│   ├── db/
│   │   ├── base.py                   # Shared SQLAlchemy declarative base
│   │   ├── session.py                # Async engine/session, init_db, get_db
│   │   └── models/
│   │       ├── conversation.py       # Conversation (rolling summary, worker claims, escalation flag)
│   │       └── message.py            # Message + SenderType / AnswerStatus / SendStatus enums
│   │
│   ├── schemas/
│   │   ├── conversation.py           # Pydantic schemas for conversations
│   │   └── message.py                # Pydantic schemas for messages
│   │
│   ├── services/
│   │   ├── conversation_service.py   # Conversation CRUD
│   │   ├── message_service.py        # Message CRUD
│   │   ├── chat_service.py           # API-level orchestration
│   │   └── worker_service.py         # Queries and atomic claims for the three workers
│   │
│   ├── ai/
│   │   ├── client_factory.py         # LLM client singleton (OpenAI-compatible)
│   │   ├── registry.py               # Central instantiation of all AI models
│   │   ├── message_formatter.py      # DB messages -> LLM message format
│   │   │
│   │   ├── models/                   # One class per task, each with its own model setting
│   │   │   ├── base.py               # BaseAIModel (shared call, token and latency tracking)
│   │   │   ├── router.py             # Router: extracts intent, route and dates as JSON
│   │   │   ├── text_generation.py    # Final answer generation
│   │   │   ├── summarization.py      # Rolling conversation summary
│   │   │   ├── intent_detection.py   # Intent detection model
│   │   │   └── escalation.py         # Escalation detection model
│   │   │
│   │   ├── graph/                    # LangGraph decision layer
│   │   │   ├── state.py              # Graph state definition
│   │   │   ├── nodes.py              # All graph nodes
│   │   │   ├── builder.py            # Graph assembly + decide_route
│   │   │   └── runner.py             # run_ticket_graph(): the only external entry point
│   │   │
│   │   └── prompts/                  # Versioned system prompts
│   │       ├── __init__.py           # Loader: picks the highest version or a pinned one
│   │       ├── v1/ ... v7/           # Each version: router.py, answer.py, static.py
│   │
│   ├── summary/
│   │   ├── summary_control.py        # Builds model context (summary + last M exchanges)
│   │   └── formatting.py             # Message batch -> plain text for summarization
│   │
│   ├── messaging/
│   │   ├── whatsapp_client.py        # WhatsApp message delivery
│   │   └── escalation.py             # Builds and sends escalation notices to management
│   │
│   ├── scrapers/
│   │   ├── flight_data.py            # Three data methods: summary days / full day / full days
│   │   ├── airports.py               # City name -> IATA code dictionary and resolver
│   │   └── flight_source/            # Clients for the external flight data source
│   │       ├── flight_search.py      # Flight search client
│   │       └── flight_calendar.py    # Price calendar client
│   │
│   ├── utils/
│   │   ├── jalali.py                 # Today's Jalali date block, Gregorian -> Jalali
│   │   └── date_resolver.py          # Deterministic date resolution and conversion
│   │
│   ├── monitoring/
│   │   ├── events.py                 # Event types and shared envelope
│   │   ├── store.py                  # JSON Lines writer/reader + live counters
│   │   ├── recorder.py               # record_event / track / token_scope
│   │   ├── aggregator.py             # Raw events -> dashboard-ready stats
│   │   ├── api.py                    # Read-only GET /monitoring/* endpoints
│   │   └── middleware.py             # Automatic HTTP request logging
│   │
│   ├── workers/
│   │   ├── base.py                   # BaseWorker (polling loop + timeout)
│   │   ├── manager.py                # Runs N instances of each worker
│   │   ├── answer_generation_worker.py
│   │   ├── answer_delivery_worker.py
│   │   ├── summarization_worker.py
│   │   └── runner.py                 # Standalone worker process entry point
│   │
│   └── static/
│       └── index.html                # WhatsApp-style local test chat page
│
└── scripts/
    ├── monitoring_report.py          # Terminal monitoring report
    ├── test_graph_routes.py          # Tests all graph routes (no network, no API key)
    └── test_flight_data.py           # Tests data methods with fake clients
```
