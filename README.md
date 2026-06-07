# Telegram Signal Copier from MT5 or TradingView PineScript

> **Project Directory:** `D:\Forex\AI Vibe Codeing for FX\Telegram Signal Copier from MT5 or TradingView PineScript\`
>
> **Architecture Diagram:** → [`architecture-telegram-signal-copier.html`](architecture-telegram-signal-copier.html) (open in browser)
>
> **Implementation Plan:** → [`PLAN.md`](PLAN.md)
>
> **SKILL.md:** → [`SKILL.md`](SKILL.md)

---

## 📋 Project Overview

Telegram ကနေ Trading Signals တွေကို အလိုအလျောက်ဖမ်းယူပြီး **MT5** (MetaTrader 5) နဲ့ **TradingView** (via Webhook) ကို ပို့ဆောင်ပေးတဲ့ **Telegram Signal Copier** စနစ်တစ်ခုကို တည်ဆောက်မည်။

### 🔑 Core Features
- **Telegram Ingestion** — Telethon API to monitor channels/groups in real-time
- **Signal Parsing** — Regex-based extraction of Symbol, Action (BUY/SELL/LIMIT/STOP), Entry, SL, TP¹⁻ⁿ
- **Dual Execution Path**:
  - **Path A**: Direct MT5 Python API (`MetaTrader5` library) — orders sent directly to broker terminal
  - **Path B**: TradingView Webhook — Flask server receives alerts from TradingView PineScript
- **Risk Manager** — Dynamic lot sizing based on account balance, risk %, and SL distance
- **Journal Reporting** — Daily/Weekly performance reports via Telegram (PnL, Win Rate, Drawdown)
- **SQLite Database** — Full trade history, signal log, and configuration storage

### 🔧 Tech Stack
| Component | Technology |
|-----------|-----------|
| Telegram Client | Telethon (async) |
| Telegram Bot | python-telegram-bot |
| MT5 Integration | MetaTrader5 Python package |
| TradingView Webhook | Flask REST API |
| Database | SQLite3 / PostgreSQL |
| Scheduler | APScheduler |
| Reporting | Pandas, Matplotlib |
| OCR (optional) | pytesseract + OpenCV |

---

## 🏗 System Architecture (4-Layer Design)

```
┌─────────────────────────────────────────────────────────────────┐
│  LAYER 1: INPUT LAYER (Signal Ingestion)                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐      │
│  │ Telethon     │  │ TV Webhook   │  │ Telegram Bot API  │      │
│  │ Client       │  │ Flask Server │  │ (Manual Entry)    │      │
│  └──────┬───────┘  └──────┬───────┘  └────────┬─────────┘      │
│         │                 │                    │                 │
├─────────┼─────────────────┼────────────────────┼────────────────┤
│  LAYER 2: PROCESSING LAYER (Parse & Validate)  │                 │
│         │                 │                    │                 │
│  ┌──────▼────────────────▼────────────────────▼──────┐          │
│  │              Signal Parser Engine                   │          │
│  │  • Symbol Detection  • Action Classification        │          │
│  │  • Entry/SL/TP Extraction  • Lot Size Detection     │          │
│  └─────────────────────┬───────────────────────────────┘          │
│                        │                                          │
│  ┌─────────────────────▼───────────────────────────────┐          │
│  │  Validation:  • Duplicate Check  • Price Filter     │          │
│  │              • Symbol Whitelist  • Session Filter    │          │
│  └─────────────────────┬───────────────────────────────┘          │
│                        │                                          │
│  ┌─────────────────────▼───────────────────────────────┐          │
│  │  Risk Manager:  Lot = (Balance × Risk%) / SL_Pips   │          │
│  └─────────────────────┬───────────────────────────────┘          │
├────────────────────────┼──────────────────────────────────────────┤
│  LAYER 3: EXECUTION LAYER (Dual Path)                  │          │
│                        │                                          │
│         ┌──────────────┴──────────────┐                           │
│         ▼                              ▼                           │
│  ┌──────────────┐          ┌──────────────────────┐                │
│  │  PATH A:     │          │  PATH B:             │                │
│  │  MT5 Direct  │          │  TradingView Webhook │                │
│  │  Python API  │          │  Flask -> PineScript  │               │
│  └──────┬───────┘          └──────────┬───────────┘                │
│         │                             │                            │
│  ┌──────▼───────┐          ┌──────────▼───────────┐                │
│  │ MT5 Broker   │          │ TradingView Alert     │                │
│  │ Terminal     │          │ > Manual Trade        │               │
│  └──────────────┘          └──────────────────────┘                │
├────────────────────────────────────────────────────────────────────┤
│  LAYER 4: REPORTING LAYER (Journal)                      │         │
│                                                                    │
│  ┌──────────────────────┐  ┌──────────────────────────────┐        │
│  │ Journal Engine       │  │ Telegram Reporter             │        │
│  │ (APScheduler+Pandas) │  │ (Auto-Send Daily/Weekly)      │        │
│  └──────────────────────┘  └──────────────────────────────┘        │
└────────────────────────────────────────────────────────────────────┘
```

---

## 📂 Project Structure

```
telegram-signal-copier/
├── src/
│   ├── __init__.py
│   ├── main.py                     # Entry point — starts all services
│   ├── config.py                   # YAML/ENV configuration loader
│   │
│   ├── input/                      # LAYER 1: Input
│   │   ├── __init__.py
│   │   ├── telethon_listener.py    # Telethon client — channel monitoring
│   │   ├── tv_webhook.py           # Flask server for TradingView alerts
│   │   ├── telegram_bot.py         # python-telegram-bot (manual commands)
│   │   └── ocr_reader.py           # OCR fallback for screenshots
│   │
│   ├── processing/                 # LAYER 2: Processing
│   │   ├── __init__.py
│   │   ├── signal_parser.py        # Regex parser — extract trade data
│   │   ├── signal_validator.py     # Validation: dupe, filter, whitelist
│   │   ├── risk_manager.py         # Dynamic lot sizing calculator
│   │   └── database.py             # SQLite CRUD operations
│   │
│   ├── execution/                  # LAYER 3: Execution
│   │   ├── __init__.py
│   │   ├── mt5_gateway.py          # Path A: MT5 Python API
│   │   ├── mt5_executor.py         # MT5 order lifecycle (open/modify/close)
│   │   ├── tv_executor.py          # Path B: TradingView webhook sender
│   │   └── order_manager.py        # Unified order interface
│   │
│   └── reporting/                  # LAYER 4: Reporting
│       ├── __init__.py
│       ├── journal_engine.py       # APScheduler — daily/weekly calculations
│       ├── telegram_reporter.py    # Send reports to Telegram owner
│       └── report_templates.py     # HTML/CSV formatting
│
├── data/
│   ├── signals.db                  # SQLite database
│   ├── symbols.txt                 # Whitelist symbols
│   └── keywords.json               # Parser keywords config
│
├── logs/                           # Application logs
│   └── .gitkeep
│
├── config.yaml                     # Main configuration file
├── .env.example                    # Environment variables template
├── requirements.txt                # Python dependencies
├── SKILL.md                        # Reusable skill doc
├── PLAN.md                         # Implementation plan
├── architecture-telegram-signal-copier.html  # Architecture diagram
└── README.md                       # This file
```

---

## ⚙️ Configuration (`config.yaml`)

```yaml
# === TELEGRAM ===
telegram:
  api_id: YOUR_API_ID          # From my.telegram.org
  api_hash: YOUR_API_HASH
  phone: +959XXXXXXXXX
  session_name: signal_copier
  channels:
    - https://t.me/signal_channel_1
    - https://t.me/signal_channel_2
  bot_token: YOUR_BOT_TOKEN    # From @BotFather (for manual commands)

# === EXECUTION PATH ===
execution:
  mode: "mt5"                  # "mt5" | "tradingview" | "both"

# === MT5 ===
mt5:
  server: IC Markets-Demo
  login: 12345678
  password: your_password
  enable: true

# === TRADINGVIEW WEBHOOK ===
tradingview:
  webhook_host: 0.0.0.0
  webhook_port: 5000
  secret_token: your_webhook_secret
  enable: true

# === RISK MANAGEMENT ===
risk:
  default_lot_size: 0.01
  risk_percent: 2.0           # 2% risk per trade
  max_daily_loss: 500.0       # USD
  max_consecutive_losses: 3
  max_spread: 50              # points
  session_start: "00:00"
  session_end: "23:59"

# === REPORTING (JOURNAL) ===
reporting:
  daily_report_time: "23:30"       # Daily report at 11:30 PM
  weekly_report_day: "sunday"      # Weekly on Sunday
  weekly_report_time: "21:00"      # Weekly report at 9 PM
  send_to_chat_id: YOUR_CHAT_ID    # Where to send reports
  enable_daily: true
  enable_weekly: true

# === DATABASE ===
database:
  path: data/signals.db
  enable: true
```

---

## 📊 Journal Report System

> **✅ Decision:** Custom journal system ထည့်မယ်။ MT5 Trading Journal ရှိပေမဲ့ Telegram ကို auto-forward လုပ်ပေးတဲ့ built-in feature မရှိလို့ပါ။ TradingView မှာလည်း automated journal reporting မရှိပါဘူး။

### Daily Report Structure
```
📊 Daily Trading Journal — 2026-06-07
━━━━━━━━━━━━━━━━━━━━━━━━━━
📈 Total Trades: 5
✅ Won: 3 (60.0%)
❌ Lost: 2 (40.0%)
💰 PnL: +$124.50
📉 Max Drawdown: $45.00
📊 Win Rate: 60.0%
⚡ Profit Factor: 1.85
💼 Current Balance: $10,124.50
📋 Trade Breakdown:
  BUY  XAUUSD  @4444  TP1✅ TP2✅ TP3❌
  SELL EURUSD  @1.085 TP1✅ SL❌
  ...
```

### Weekly Report Structure
```
📈 Weekly Trading Journal — Jun 1–7, 2026
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 Total Trades: 23  |  Won: 14  |  Lost: 9
💰 Net PnL: +$567.30
📉 Max Drawdown: $89.00
📊 Win Rate: 60.9%
⚡ Profit Factor: 2.1
📈 Avg Win: $85.50  |  Avg Loss: $42.30
🔄 Best Day: Jun 3 (+$210)
🔴 Worst Day: Jun 5 (-$67)
💼 Ending Balance: $10,567.30
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Equity Curve Chart Attached]
```

---

## 📋 Reference Repositories Analysis

| Feature | oluklef17 (EchoTrade) | oogunjob (FX Bot) | Our Design |
|---------|----------------------|-------------------|------------|
| Telegram Client | Telethon (PyQt5 GUI) | python-telegram-bot | Telethon + Bot API |
| Execution | MT4/MT5 EAs via files | MetaAPI Cloud | MT5 Python API + TV Webhook |
| Risk Manager | ❌ | ✅ (Risk Factor %) | ✅ (Advanced) |
| Journal | ❌ | ❌ | ✅ (Daily/Weekly) |
| OCR | ✅ (image_reader.py) | ❌ | ✅ (Optional) |
| TV Webhook | ❌ | ❌ | ✅ (Path B) |
| DB | ❌ | ❌ | ✅ (SQLite) |

---

## 🚀 Getting Started

```bash
# 1. Clone the project
cd /d/Forex/AI Vibe Codeing for FX/Telegram Signal Copier from MT5 or TradingView PineScript

# 2. Create virtual environment
python -m venv venv
source venv/Scripts/activate    # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. Configure
cp .env.example .env
# Edit .env with your Telegram API credentials & MT5 settings

# 5. Run
python src/main.py
```

---

## References

- [oluklef17/telegram-mt4-mt5-signal-copier](https://github.com/oluklef17/telegram-mt4-mt5-signal-copier.git)
- [oogunjob/FX-Signal-Copier-Telegram-Bot](https://github.com/oogunjob/FX-Signal-Copier-Telegram-Bot.git)
- [MetaTrader5 Python Package](https://pypi.org/project/MetaTrader5/)
- [Telethon Library](https://docs.telethon.dev/)
- [TradingView Webhooks](https://www.tradingview.com/support/solutions/43000529348/)
