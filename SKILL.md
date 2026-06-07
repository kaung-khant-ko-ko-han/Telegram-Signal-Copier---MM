---
name: telegram-signal-copier
description: "Build a Telegram Signal Copier that copies trading signals from Telegram to MT5 and/or TradingView PineScript with automated journal reporting"
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [windows, linux]
metadata:
  tags: [trading, forex, mt5, tradingview, telegram, signals, automation]
---

# Telegram Signal Copier from MT5 or TradingView PineScript

## Overview

Build an event-driven Telegram Signal Copier that:
1. **Ingests** trading signals from Telegram channels/groups in real-time (Telethon)
2. **Parses** unstructured text into structured trade data (Symbol, Action, Entry, SL, TP)
3. **Executes** via **Dual Path**: MT5 Python API (direct) OR TradingView Webhook (PineScript alert)
4. **Reports** Daily/Weekly Journal via Telegram (PnL, Win Rate, Drawdown, Trade History)

## Architecture

**4-Layer Event-Driven Architecture:**

```
Layer 1: INPUT      →  Telethon Client + TV Webhook + Telegram Bot
Layer 2: PROCESSING →  Signal Parser + Validator + Risk Manager + SQLite DB
Layer 3: EXECUTION  →  MT5 Python API (Path A) / TV Webhook (Path B)
Layer 4: REPORTING  →  Journal Engine (APScheduler) + Telegram Reporter
```

## Prerequisites

- Python 3.11+
- MetaTrader 5 installed (for Path A) - Windows only
- Telegram API credentials from [my.telegram.org](https://my.telegram.org)
- Telegram Bot Token from @BotFather (optional, for manual commands)
- TradingView account (for Path B webhook)

## Dependencies

```
# Core
telethon==1.37.0
python-telegram-bot==20.8
MetaTrader5==5.0.45     # Windows only

# Webhook (Path B)
flask==3.0.0

# Processing
pandas==2.2.0
numpy==1.26.0
pyyaml==6.0

# Scheduler
apscheduler==3.10.4

# Reporting
matplotlib==3.8.0

# OCR (optional)
pytesseract==0.3.10
opencv-python==4.9.0
```

## Implementation Steps

### Phase 1: Project Setup & Configuration
1. Create project structure (src/, data/, logs/, configs)
2. `config.py` - YAML config loader with .env override
3. `config.yaml` - All settings (Telegram, MT5, TV, Risk, Reporting)
4. `database.py` - SQLite schema + CRUD for signals, orders, journal

### Phase 2: Input Layer
1. `telethon_listener.py` - Async client listening to channels, NewMessage + MessageEdited events
2. `tv_webhook.py` - Flask POST endpoint for TradingView alerts
3. `telegram_bot.py` - python-telegram-bot for manual /trade commands
4. `ocr_reader.py` - pytesseract OCR for signal screenshots

### Phase 3: Processing Layer
1. `signal_parser.py` - Regex engine to extract: Symbol, Action, Entry, SL, TP list
   - Support BUY/SELL/BUYLIMIT/SELLLIMIT/BUYSTOP/SELLSTOP
   - Support TP¹ through TPⁿ
   - Handle multiple signal formats (plain text, formatted, emoji)
2. `signal_validator.py` - Duplicate check by message_id, symbol whitelist, price range filter
3. `risk_manager.py` - Position size: Lot = (Balance × Risk%) / (SL_Pips × PipValue)

### Phase 4: Execution Layer — Path A (MT5)
1. `mt5_gateway.py` - Connect to MT5 terminal, initialize, check connection health
2. `mt5_executor.py` - Order operations:
   - `open_market_order()` - Instant BUY/SELL
   - `open_pending_order()` - LIMIT/STOP orders
   - `modify_order()` - Move SL/TP
   - `close_position()` - Close by ticket or partial
   - `close_all_positions()` - Emergency close
3. `order_manager.py` - Unified interface that routes to MT5 or TV

### Phase 5: Execution Layer — Path B (TradingView Webhook)
1. `tv_executor.py` - Receive structured JSON from PineScript alerts
2. PineScript template for TradingView indicators sending webhook alerts

### Phase 6: Reporting Layer (Journal)
1. `journal_engine.py` - APScheduler tasks:
   - Daily at config time: query DB for today's trades, calculate PnL/win rate/drawdown
   - Weekly on Sunday: aggregate weekly stats, generate equity curve
2. `telegram_reporter.py` - Format and send reports to owner Telegram
3. `report_templates.py` - HTML table, CSV export, matplotlib chart generation

## Key Code Patterns

### Signal Parser Example (GOLD BUY/SELL format)
```python
import re

def parse_signal(text: str) -> dict | None:
    """Parse Telegram signal text into structured trade data.
    
    Supports multiple formats:
    Format 1: GOLD BUY 4444 / TP¹ 4447 / SL_4433
    Format 2: BUY XAUUSD 2000.50 / TP: 2010.50 / SL: 1995.00
    Format 3: SELL EURUSD @ 1.0850 / TP1 1.0830 / SL 1.0870
    """
    result = {}
    lines = text.strip().split('\n')
    
    # Line 1: Action + Symbol + Optional Entry
    first_line = lines[0].upper()
    
    # Detect action
    for action in ['BUY LIMIT', 'SELL LIMIT', 'BUY STOP', 'SELL STOP', 'BUY', 'SELL']:
        if action in first_line:
            result['action'] = action
            break
    
    # Extract symbol
    symbol_map = {
        'GOLD': 'XAUUSD', 'SILVER': 'XAGUSD', 'OIL': 'USOIL',
        'EURUSD': 'EURUSD', 'GBPUSD': 'GBPUSD', ...
    }
    
    # Extract prices with regex
    price_pattern = r'(\d+\.?\d*)'
    ...
    
    return result
```

### MT5 Order Execution
```python
import MetaTrader5 as mt5

def open_trade(symbol: str, action: str, volume: float, 
               price: float, sl: float, tp: float) -> dict:
    """Open a trade on MT5."""
    if not mt5.initialize():
        return {"error": "MT5 init failed"}
    
    order_type = {
        'BUY': mt5.ORDER_TYPE_BUY,
        'SELL': mt5.ORDER_TYPE_SELL,
        'BUY_LIMIT': mt5.ORDER_TYPE_BUY_LIMIT,
        'SELL_LIMIT': mt5.ORDER_TYPE_SELL_LIMIT,
        'BUY_STOP': mt5.ORDER_TYPE_BUY_STOP,
        'SELL_STOP': mt5.ORDER_TYPE_SELL_STOP,
    }[action.replace(' ', '_')]
    
    request = {
        "action": mt5.TRADE_ACTION_DEAL if order_type in [...] else mt5.TRADE_ACTION_PENDING,
        "symbol": symbol,
        "volume": volume,
        "type": order_type,
        "price": price,
        "sl": sl,
        "tp": tp,
        "deviation": 20,
        "magic": 234000,
        "comment": "Signal Copier",
        "type_time": mt5.ORDER_TIME_GTC,
        "type_filling": mt5.ORDER_FILLING_IOC,
    }
    
    result = mt5.order_send(request)
    return {"ticket": result.order, "error": mt5.last_error() if result.retcode != 10009 else None}
```

### Journal Report Generation
```python
from datetime import datetime, timedelta
import pandas as pd

def generate_daily_report(db_path: str, date: str = None) -> dict:
    """Generate daily trading journal report."""
    date = date or datetime.now().strftime('%Y-%m-%d')
    
    df = pd.read_sql(f"""
        SELECT * FROM orders 
        WHERE DATE(open_time) = '{date}'
    """, f'sqlite:///{db_path}')
    
    report = {
        'date': date,
        'total_trades': len(df),
        'wins': len(df[df['profit'] > 0]),
        'losses': len(df[df['profit'] <= 0]),
        'net_pnl': df['profit'].sum(),
        'win_rate': len(df[df['profit'] > 0]) / len(df) * 100 if len(df) > 0 else 0,
        'max_drawdown': df['drawdown'].max() if 'drawdown' in df else 0,
        'trades': df.to_dict('records')
    }
    return report
```

## Signal Format Support

The parser must handle these signal formats from the user's channels:

```
Format 1 (User's Example - GOLD):
  GOLD BUY 4444       → Symbol: XAUUSD, Action: BUY, Entry: 4444
  MORE BUY 4442
  TP¹ 4447            → TP1: 4447
  TP² 4450            → TP2: 4450
  SL_4433             → SL: 4433

Format 2 (Standard):
  BUY EURUSD 1.0850
  TP: 1.0870
  SL: 1.0830

Format 3 (Detailed):
  🔴 SELL XAUUSD @ 2030.50
  ✅ Entry: 2030.50
  🎯 TP1: 2025.00
  🎯 TP2: 2020.00
  🛑 SL: 2035.50
  💰 Lots: 0.10
```

## Database Schema

```sql
-- Raw signals from Telegram
CREATE TABLE signals (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    message_id INTEGER UNIQUE,
    channel_id TEXT,
    channel_name TEXT,
    raw_text TEXT,
    parsed_json TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Executed orders
CREATE TABLE orders (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    signal_id INTEGER REFERENCES signals(id),
    symbol TEXT,
    action TEXT,
    entry_price REAL,
    stop_loss REAL,
    take_profits TEXT,  -- JSON array
    volume REAL,
    ticket INTEGER,
    open_time TIMESTAMP,
    close_time TIMESTAMP,
    profit REAL,
    drawdown REAL,
    status TEXT DEFAULT 'open',  -- open, closed, cancelled
    execution_path TEXT,  -- 'mt5' or 'tradingview'
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Journal reports
CREATE TABLE journal_reports (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    report_type TEXT,  -- 'daily' or 'weekly'
    report_date TEXT,
    total_trades INTEGER,
    wins INTEGER,
    losses INTEGER,
    net_pnl REAL,
    win_rate REAL,
    profit_factor REAL,
    max_drawdown REAL,
    report_json TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Configuration
CREATE TABLE config (
    key TEXT PRIMARY KEY,
    value TEXT
);
```

## Common Pitfalls

1. **MT5 Terminal Required** - MT5 must be open with AutoTrading enabled for Python API to work
2. **Telegram Rate Limits** - Telethon auto-handles but watch for flood-waits
3. **Signal Format Variation** - Different channels use different formats; parser must be flexible
4. **Duplicate Signals** - Always check message_id to avoid re-executing edited/duplicate signals
5. **Risk Management First** - Always validate lot size before sending order
6. **Windows Only for MT5** - MetaTrader5 Python package only works on Windows
7. **TradingView Webhook Auth** - Use secret_token to verify incoming webhooks
8. **Timezones** - MT5 uses UTC time; convert for daily report cutoffs

## Verification

1. Run `pytest tests/` - All unit tests pass
2. Manual test: Send signal in Telegram → Check DB for parsed signal
3. Manual test: Run with MT5 demo account → Verify order appears in MT5 terminal
4. Manual test: Daily report triggers at configured time → Check Telegram message
5. `python -m src.main` starts all services without errors
