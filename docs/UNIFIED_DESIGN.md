# Unified Design — Kimi AI + Hermes Agent Merger

## Purpose

Kimi Agent (existing in `Kimi_Agent_Telegram/`) and Hermes Agent both produced design documents for the Telegram Signal Copier. This document merges the **best ideas from both**, resolves differences, and creates a single unified roadmap.

---

## Comparison Matrix

| Aspect | Hermes Agent (Our Doc) | Kimi Agent (Existing Doc) | ✅ Unified Choice |
|--------|----------------------|--------------------------|-------------------|
| **MT5 Connection** | Direct Python `MetaTrader5` API | ZeroMQ + MQL5 EA | **Both**: ZeroMQ primary + direct API fallback |
| **Database** | SQLite (simple) | PostgreSQL (production) | **SQLite for dev**, **PostgreSQL for production** |
| **State/Queue** | In-process | Redis + Celery | **Redis + Celery** for production |
| **API Framework** | Flask | FastAPI | **FastAPI** (async-native, better) |
| **PDF Reports** | ❌ Not included | reportlab | **✅ Add** from Kimi |
| **Trade Lifecycle** | Basic (open/close) | Detailed events (tp_hit, sl_hit, be, etc.) | **✅ Detailed** from Kimi |
| **R-Multiple** | ❌ | ✅ | **✅ Add** |
| **Provider Analytics** | ❌ | ✅ `provider_performance` table | **✅ Add** |
| **NLP Parsing** | Regex-only | Regex + NLP | **Hybrid** (regex primary + optional NLP) |
| **Report Charts** | Text only | matplotlib + plotly | **✅ Add** matplotlib charts |
| **Docker** | ❌ | ✅ docker-compose.yml | **✅ Add** |
| **Alembic Migrations** | ❌ | ✅ | **✅ Add** |
| **Deployment** | Windows/VPS | Docker + VPS | **Both** (Windows native + Docker) |

---

## Key Enhancements to Add (from Kimi)

### 1. ZeroMQ MT5 Connection (Better than Direct API)

Kimi proposes a **ZeroMQ REQ/REP socket** between Python and MQL5 EA. This is more robust than the direct `MetaTrader5` Python API because:

- Survives MT5 restarts (auto-reconnects)
- No Python→MT5 binding issues
- Can run on separate machines (Python on VPS, MT5 on trading PC)
- Handle concurrent orders without blocking
- Bidirectional communication (order confirmations back)

**Our approach**: Support both — ZeroMQ as primary, direct Python API as fallback.

### 2. Redis + Celery for Production Queue

Instead of in-process processing, use:
- **Redis**: Message queue, state cache, pub/sub for real-time updates
- **Celery**: Background workers for parsing, validation, execution, reporting

### 3. Trade Lifecycle Events

Track every event in a trade's life:
- `entry` → `tp1_hit` → `tp2_hit` → `sl_hit` → `breakeven` → `partial_close` → `modified`

### 4. R-Multiple Analytics

```python
R = (close_price - entry_price) / (entry_price - stop_loss)
```

### 5. Provider Performance Table

Track signal provider performance over time:
```sql
CREATE TABLE provider_performance (
    provider_name VARCHAR(255),
    total_signals INT, winning_trades INT,
    total_pnl DECIMAL, win_rate DECIMAL, average_rr DECIMAL
);
```

---

## Unified Project Structure

```
telegram-signal-copier/
├── src/
│   ├── __init__.py
│   ├── main.py                     # Application entry point
│   ├── config/
│   │   ├── __init__.py
│   │   ├── settings.py             # Pydantic settings (v2)
│   │   └── logging.yaml
│   │
│   ├── telegram/                   # LAYER 1: Input
│   │   ├── __init__.py
│   │   ├── listener.py             # Telethon async client
│   │   ├── bot.py                  # python-telegram-bot commands
│   │   └── notifications.py        # Alert/report sender
│   │
│   ├── parser/                     # LAYER 2: Processing — Parser
│   │   ├── __init__.py
│   │   ├── preprocessor.py         # Text cleaning, normalization
│   │   ├── patterns.py             # Keyword dictionary matcher
│   │   ├── extractor.py            # Regex price/symbol/lot extraction
│   │   ├── validator.py            # Signal validation & dedup
│   │   ├── schemas.py              # Pydantic models for signals
│   │   └── nlp_extractor.py        # Optional NLP enhancement
│   │
│   ├── risk/                       # LAYER 2: Processing — Risk
│   │   ├── __init__.py
│   │   ├── position_sizer.py       # Lot size calculation
│   │   └── limits.py               # Exposure/loss limits
│   │
│   ├── executors/                  # LAYER 3: Execution
│   │   ├── __init__.py
│   │   ├── mt5/
│   │   │   ├── __init__.py
│   │   │   ├── zeromq.py           # ZeroMQ client ↔ MQL5 EA
│   │   │   ├── direct_api.py       # Direct MetaTrader5 API (fallback)
│   │   │   └── mql_bridge.py       # File-based (secondary fallback)
│   │   └── tradingview/
│   │       ├── __init__.py
│   │       └── webhook.py          # FastAPI webhook endpoint
│   │
│   ├── journal/                    # LAYER 4: Reporting
│   │   ├── __init__.py
│   │   ├── database.py             # SQLAlchemy models + connection
│   │   ├── trade_logger.py         # Event-driven trade logging
│   │   ├── analytics.py            # Statistics (R-multiple, etc.)
│   │   └── migrations/             # Alembic migration scripts
│   │
│   ├── reports/                    # Report generation
│   │   ├── __init__.py
│   │   ├── daily.py                # Daily report logic
│   │   ├── weekly.py               # Weekly report logic
│   │   ├── charts.py               # matplotlib chart functions
│   │   ├── pdf_generator.py        # reportlab PDF builder
│   │   └── templates.py            # Telegram message templates
│   │
│   └── core/                       # Shared infrastructure
│       ├── __init__.py
│       ├── redis_client.py         # Redis connection
│       ├── celery_app.py           # Celery app configuration
│       ├── state_manager.py        # In-memory state tracking
│       └── exceptions.py           # Custom exceptions
│
├── mql5/                           # MQL5 Expert Advisors
│   └── SignalExecutorEA/
│       ├── SignalExecutor.mq5
│       └── SignalExecutor.ex5
│
├── pinescript/                     # TradingView Pine Scripts
│   └── SignalConnector.pine
│
├── config/                         # Runtime configurations
│   ├── config.yaml
│   └── keywords/
│       ├── default.json
│       └── example_advanced.json
│
├── data/                           # Data storage (dev SQLite)
│   ├── signals.db
│   ├── symbols.txt
│   └── keywords.json
│
├── reports/                        # Generated report output
│   └── .gitkeep
│
├── migrations/                     # Alembic migrations
│   └── versions/
│
├── docker/
│   ├── Dockerfile
│   ├── docker-compose.yml
│   └── .dockerignore
│
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── fixtures/
│   └── __init__.py
│
├── docs/
│   ├── IMPLEMENTATION_PLAN.md      # Merged plan
│   ├── SYSTEM_ARCHITECTURE.md      # Merged architecture
│   ├── JOURNAL_DESIGN.md           # Merged journal
│   ├── SKILL.md                    # Reusable skill doc
│   └── REFERENCE_ANALYSIS.md       # Reference repo analysis
│
├── logs/
├── requirements.txt
├── pyproject.toml
├── config.yaml
├── README.md
└── .env.example
```

---

## Unified DB Schema (PostgreSQL + SQLite Compatible)

```sql
-- 1. Signals: Raw + parsed signal storage
CREATE TABLE signals (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    raw_message TEXT NOT NULL,
    parsed_signal JSONB NOT NULL,
    source_channel VARCHAR(255),
    source_channel_id BIGINT,
    message_id BIGINT UNIQUE,
    signal_provider VARCHAR(255),
    status VARCHAR(50) DEFAULT 'pending',
        -- pending, parsed, validated, queued, executed, failed, ignored
    error_message TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    executed_at TIMESTAMPTZ
);

-- 2. Trades: Executed orders
CREATE TABLE trades (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    signal_id UUID REFERENCES signals(id),
    platform VARCHAR(50) NOT NULL,       -- mt5, tradingview
    account_id VARCHAR(255),
    symbol VARCHAR(20) NOT NULL,
    direction VARCHAR(10) NOT NULL,      -- BUY, SELL
    order_type VARCHAR(20) NOT NULL,     -- MARKET, LIMIT, STOP
    entry_price DECIMAL(15,5),
    stop_loss DECIMAL(15,5),
    take_profits DECIMAL(15,5)[],
    lot_size DECIMAL(10,2),
    risk_percent DECIMAL(5,2),
    status VARCHAR(50) DEFAULT 'open',
        -- open, closed, partial, cancelled
    opened_at TIMESTAMPTZ,
    closed_at TIMESTAMPTZ,
    close_price DECIMAL(15,5),
    realized_pnl DECIMAL(15,2),
    pips DECIMAL(10,2),
    r_multiple DECIMAL(5,2),
    ticket_number VARCHAR(100),
    magic_number BIGINT,
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 3. Trade Events: Lifecycle tracking
CREATE TABLE trade_events (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    trade_id UUID REFERENCES trades(id),
    event_type VARCHAR(50) NOT NULL,
        -- entry, tp1_hit, tp2_hit, sl_hit, breakeven,
        -- partial_close, modified, closed
    description TEXT,
    pnl_at_event DECIMAL(15,2),
    price_at_event DECIMAL(15,5),
    data JSONB DEFAULT '{}',
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 4. Daily Summaries
CREATE TABLE daily_summaries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    date DATE NOT NULL,
    account_id VARCHAR(255),
    total_trades INT DEFAULT 0,
    winning_trades INT DEFAULT 0,
    losing_trades INT DEFAULT 0,
    breakeven_trades INT DEFAULT 0,
    total_pnl DECIMAL(15,2) DEFAULT 0,
    gross_profit DECIMAL(15,2) DEFAULT 0,
    gross_loss DECIMAL(15,2) DEFAULT 0,
    profit_factor DECIMAL(8,2),
    average_win DECIMAL(10,2),
    average_loss DECIMAL(10,2),
    average_r DECIMAL(5,2),
    max_drawdown DECIMAL(10,2),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(date, account_id)
);

-- 5. Provider Performance
CREATE TABLE provider_performance (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    provider_name VARCHAR(255) NOT NULL,
    channel_id BIGINT,
    total_signals INT DEFAULT 0,
    executed_signals INT DEFAULT 0,
    winning_trades INT DEFAULT 0,
    losing_trades INT DEFAULT 0,
    total_pnl DECIMAL(15,2) DEFAULT 0,
    win_rate DECIMAL(5,2),
    profit_factor DECIMAL(8,2),
    UNIQUE(provider_name, channel_id)
);
```

---

## Unified Implementation Roadmap

| Phase | Name | Duration | Key Deliverables |
|-------|------|----------|-----------------|
| **1** | Foundation & Parser | Week 1-2 | Config, Telethon listener, regex parser, keywords |
| **2** | MT5 Integration | Week 3-4 | ZeroMQ client, MQL5 EA, direct API fallback, risk manager |
| **3** | TradingView | Week 5 | FastAPI webhook, PineScript template, signature verification |
| **4** | Database & Journal | Week 6 | PostgreSQL/SQLite, trade lifecycle, provider analytics |
| **5** | Reporting | Week 7 | Daily/weekly reports, charts (matplotlib), PDF (reportlab) |
| **6** | Bot & UI | Week 8 | Telegram bot commands, /status, /journal, admin panel |
| **7** | Deployment & Test | Week 9-10 | Docker, demo testing, bug fixes, performance optimization |

---

## What to Keep from Each Source

### Keep from Hermes Agent (Our Doc):
✅ **PLAN.md** — Complete code examples for every module (copy-pasteable)
✅ **architecture-telegram-signal-copier.html** — Visual SVG architecture diagram
✅ `signal_parser.py` — Working regex parser for GOLD BUY/SELL format
✅ `risk_manager.py` — Practical lot size calculation
✅ `database.py` — Simple SQLite CRUD for development
✅ **Practical, build-it-now approach** — You can start coding today

### Keep from Kimi Agent:
✅ `SYSTEM_ARCHITECTURE.md` — ZeroMQ, Redis/Celery, FastAPI, PostgreSQL, Docker
✅ `JOURNAL_DESIGN.md` — Trade lifecycle events, R-multiple, PDF reports, provider analytics
✅ `REFERENCE_ANALYSIS.md` — Thorough comparison of both reference repos
✅ **Production-grade design** — For when you need scalability

### Next Action
The **PLAN.md** already has complete working code. I recommend:
1. Start implementing **Phase 1-2** immediately using the code from our PLAN.md
2. Add ZeroMQ + Redis/Celery in **Phase 4-5** when scaling up
3. Add PDF reports + provider analytics in **Phase 5**
4. Dockerize in **Phase 7**
