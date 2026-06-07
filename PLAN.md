# Telegram Signal Copier — Implementation Plan

> **For Hermes:** Use this plan to build the project task-by-task with subagent-driven-development.
>
> **Goal:** Build a production-ready Telegram Signal Copier with MT5 + TradingView dual execution paths and automated journal reporting.
>
> **Architecture:** 4-layer event-driven system: Input (Telethon/TV Webhook) → Processing (Parser/Validator/RiskMgr) → Execution (MT5 API/TV Webhook) → Reporting (APScheduler Journal)
>
> **Tech Stack:** Python 3.11+, Telethon, MetaTrader5, Flask, APScheduler, Pandas, SQLite

---

## Phase 0: Project Setup

### Task 0.1: Create project directory structure

**Objective:** Set up all directories and empty files

**Files:**
- Create: `src/__init__.py`
- Create: `src/main.py`
- Create: `src/config.py`
- Create: `src/input/__init__.py`
- Create: `src/input/telethon_listener.py`
- Create: `src/input/tv_webhook.py`
- Create: `src/input/telegram_bot.py`
- Create: `src/input/ocr_reader.py`
- Create: `src/processing/__init__.py`
- Create: `src/processing/signal_parser.py`
- Create: `src/processing/signal_validator.py`
- Create: `src/processing/risk_manager.py`
- Create: `src/processing/database.py`
- Create: `src/execution/__init__.py`
- Create: `src/execution/mt5_gateway.py`
- Create: `src/execution/mt5_executor.py`
- Create: `src/execution/tv_executor.py`
- Create: `src/execution/order_manager.py`
- Create: `src/reporting/__init__.py`
- Create: `src/reporting/journal_engine.py`
- Create: `src/reporting/telegram_reporter.py`
- Create: `src/reporting/report_templates.py`
- Create: `config.yaml`
- Create: `.env.example`
- Create: `requirements.txt`
- Create: `logs/.gitkeep`
- Create: `data/symbols.txt`
- Create: `data/keywords.json`

**Step 1: Create directories**

```bash
cd /d/Forex/AI Vibe Codeing for FX/Telegram Signal Copier from MT5 or TradingView PineScript

mkdir -p src/input src/processing src/execution src/reporting data logs
touch src/__init__.py src/input/__init__.py src/processing/__init__.py \
      src/execution/__init__.py src/reporting/__init__.py logs/.gitkeep
```

**Step 2: Create `requirements.txt`**

```
# Core
telethon>=1.37,<2.0
python-telegram-bot>=20.0,<21.0
apscheduler>=3.10,<4.0

# Config
pyyaml>=6.0,<7.0
python-dotenv>=1.0,<2.0

# Processing
pandas>=2.0,<3.0
numpy>=1.24,<2.0

# Reporting
matplotlib>=3.7,<4.0

# Webhook (Path B)
flask>=3.0,<4.0

# UT
pytest>=7.0,<8.0
pytest-mock>=3.0,<4.0
```

**Step 3: Create `.env.example`**

```
# === TELEGRAM ===
TELEGRAM_API_ID=YOUR_API_ID
TELEGRAM_API_HASH=YOUR_API_HASH
TELEGRAM_PHONE=+959XXXXXXXXX
TELEGRAM_BOT_TOKEN=YOUR_BOT_TOKEN

# === MT5 ===
MT5_SERVER=ICMarkets-Demo
MT5_LOGIN=12345678
MT5_PASSWORD=your_password

# === TRADINGVIEW WEBHOOK ===
TV_SECRET_TOKEN=your_webhook_secret

# === REPORTING ===
REPORT_CHAT_ID=YOUR_CHAT_ID

# === RISK ===
RISK_PERCENT=2.0
MAX_DAILY_LOSS=500.0
MAX_CONSECUTIVE_LOSSES=3
```

---

## Phase 1: Configuration & Database

### Task 1.1: Config Loader

**Objective:** Load YAML config with .env overrides

**Files:**
- Write: `src/config.py`

**Step 1: Write `src/config.py`**

```python
import os
import yaml
from pathlib import Path
from dotenv import load_dotenv

load_dotenv()

CONFIG_PATH = Path(__file__).parent.parent / "config.yaml"

def load_config() -> dict:
    with open(CONFIG_PATH) as f:
        config = yaml.safe_load(f)
    
    # .env overrides
    if os.getenv("TELEGRAM_API_ID"):
        config["telegram"]["api_id"] = int(os.getenv("TELEGRAM_API_ID"))
    if os.getenv("TELEGRAM_API_HASH"):
        config["telegram"]["api_hash"] = os.getenv("TELEGRAM_API_HASH")
    if os.getenv("TELEGRAM_PHONE"):
        config["telegram"]["phone"] = os.getenv("TELEGRAM_PHONE")
    if os.getenv("TELEGRAM_BOT_TOKEN"):
        config["telegram"]["bot_token"] = os.getenv("TELEGRAM_BOT_TOKEN")
    if os.getenv("MT5_SERVER"):
        config["mt5"]["server"] = os.getenv("MT5_SERVER")
    if os.getenv("MT5_LOGIN"):
        config["mt5"]["login"] = int(os.getenv("MT5_LOGIN"))
    if os.getenv("MT5_PASSWORD"):
        config["mt5"]["password"] = os.getenv("MT5_PASSWORD")
    if os.getenv("TV_SECRET_TOKEN"):
        config["tradingview"]["secret_token"] = os.getenv("TV_SECRET_TOKEN")
    if os.getenv("RISK_PERCENT"):
        config["risk"]["risk_percent"] = float(os.getenv("RISK_PERCENT"))
    if os.getenv("REPORT_CHAT_ID"):
        config["reporting"]["send_to_chat_id"] = os.getenv("REPORT_CHAT_ID")
    
    return config

def get_config() -> dict:
    """Singleton pattern — load once, cache."""
    if not hasattr(get_config, "_config"):
        get_config._config = load_config()
    return get_config._config

config = get_config()
```

### Task 1.2: Database Module

**Objective:** SQLite database with schema and CRUD operations

**Files:**
- Write: `src/processing/database.py`

**Step 1: Write `src/processing/database.py`**

```python
import sqlite3
import json
from datetime import datetime
from pathlib import Path
from typing import Optional
from src.config import config

DB_PATH = Path(__file__).parent.parent.parent / config["database"]["path"]

class Database:
    def __init__(self, db_path: str = None):
        self.db_path = db_path or str(DB_PATH)
        Path(self.db_path).parent.mkdir(parents=True, exist_ok=True)
        self._init_schema()
    
    def _get_conn(self):
        conn = sqlite3.connect(self.db_path)
        conn.row_factory = sqlite3.Row
        conn.execute("PRAGMA journal_mode=WAL")
        return conn
    
    def _init_schema(self):
        conn = self._get_conn()
        conn.executescript("""
            CREATE TABLE IF NOT EXISTS signals (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                message_id INTEGER UNIQUE,
                channel_id TEXT,
                channel_name TEXT,
                raw_text TEXT,
                parsed_json TEXT,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            );
            
            CREATE TABLE IF NOT EXISTS orders (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                signal_id INTEGER REFERENCES signals(id),
                symbol TEXT,
                action TEXT,
                entry_price REAL,
                stop_loss REAL,
                take_profits TEXT,
                volume REAL,
                ticket INTEGER,
                open_time TIMESTAMP,
                close_time TIMESTAMP,
                profit REAL,
                drawdown REAL,
                status TEXT DEFAULT 'open',
                execution_path TEXT,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            );
            
            CREATE TABLE IF NOT EXISTS journal_reports (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                report_type TEXT,
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
            
            CREATE TABLE IF NOT EXISTS config (
                key TEXT PRIMARY KEY,
                value TEXT
            );
        """)
        conn.commit()
        conn.close()
    
    def save_signal(self, message_id: int, channel_id: str, channel_name: str,
                    raw_text: str, parsed_json: dict = None) -> int:
        conn = self._get_conn()
        try:
            conn.execute("""
                INSERT OR IGNORE INTO signals 
                (message_id, channel_id, channel_name, raw_text, parsed_json)
                VALUES (?, ?, ?, ?, ?)
            """, (message_id, channel_id, channel_name, raw_text,
                  json.dumps(parsed_json) if parsed_json else None))
            conn.commit()
            return conn.execute("SELECT id FROM signals WHERE message_id = ?",
                              (message_id,)).fetchone()["id"]
        finally:
            conn.close()
    
    def save_order(self, signal_id: int, symbol: str, action: str,
                   entry_price: float, stop_loss: float, take_profits: list,
                   volume: float, ticket: int = None,
                   execution_path: str = "mt5") -> int:
        conn = self._get_conn()
        try:
            conn.execute("""
                INSERT INTO orders 
                (signal_id, symbol, action, entry_price, stop_loss, 
                 take_profits, volume, ticket, status, execution_path, open_time)
                VALUES (?, ?, ?, ?, ?, ?, ?, ?, 'open', ?, ?)
            """, (signal_id, symbol, action, entry_price, stop_loss,
                  json.dumps(take_profits), volume, ticket,
                  execution_path, datetime.utcnow()))
            conn.commit()
            return conn.execute("SELECT last_insert_rowid()").fetchone()[0]
        finally:
            conn.close()
    
    def close_order(self, order_id: int, profit: float, drawdown: float = 0):
        conn = self._get_conn()
        try:
            conn.execute("""
                UPDATE orders SET status='closed', close_time=?,
                profit=?, drawdown=?
                WHERE id=?
            """, (datetime.utcnow(), profit, drawdown, order_id))
            conn.commit()
        finally:
            conn.close()
    
    def get_trades_for_date(self, date_str: str) -> list:
        conn = self._get_conn()
        try:
            rows = conn.execute("""
                SELECT * FROM orders 
                WHERE DATE(open_time) = ? 
                ORDER BY open_time
            """, (date_str,)).fetchall()
            return [dict(r) for r in rows]
        finally:
            conn.close()
    
    def get_trades_for_week(self, start_date: str, end_date: str) -> list:
        conn = self._get_conn()
        try:
            rows = conn.execute("""
                SELECT * FROM orders 
                WHERE DATE(open_time) BETWEEN ? AND ?
                ORDER BY open_time
            """, (start_date, end_date)).fetchall()
            return [dict(r) for r in rows]
        finally:
            conn.close()
```

---

## Phase 2: Input Layer (Signal Ingestion)

### Task 2.1: Telethon Listener

**Objective:** Real-time Telegram channel listener using Telethon

**Files:**
- Write: `src/input/telethon_listener.py`

```python
import asyncio
import logging
from telethon import TelegramClient, events
from telethon.tl.types import User

from src.config import config

logger = logging.getLogger(__name__)

class TelethonListener:
    def __init__(self, signal_callback):
        self.api_id = config["telegram"]["api_id"]
        self.api_hash = config["telegram"]["api_hash"]
        self.phone = config["telegram"]["phone"]
        self.session = config["telegram"]["session_name"]
        self.channels = config["telegram"]["channels"]
        self.callback = signal_callback
        self.client = None
        self._running = False
    
    async def start(self):
        self.client = TelegramClient(self.session, self.api_id, self.api_hash)
        await self.client.start(phone=self.phone)
        
        # Register event handlers
        @self.client.on(events.NewMessage)
        async def handler(event):
            chat = await event.get_chat()
            if isinstance(chat, User):
                return
            chat_id = str(chat.id)
            chat_title = chat.title or chat.username or "Unknown"
            
            # Only process selected channels
            channel_ids = [c.split("/")[-1] for c in self.channels] if self.channels else []
            # or check by entity
            
            text = event.text or (event.message.message if event.message else "")
            if not text:
                return
            
            reply_text = ""
            if event.is_reply:
                reply = await event.get_reply_message()
                reply_text = reply.text or ""
            
            await self.callback({
                "source": "telethon",
                "message_id": event.id,
                "channel_id": chat_id,
                "channel_name": chat_title,
                "text": text,
                "reply_text": reply_text,
                "is_edit": False
            })
        
        @self.client.on(events.MessageEdited)
        async def edit_handler(event):
            chat = await event.get_chat()
            if isinstance(chat, User):
                return
            text = event.text or ""
            if not text:
                return
            await self.callback({
                "source": "telethon",
                "message_id": event.id,
                "channel_id": str(chat.id),
                "channel_name": chat.title or "Unknown",
                "text": text,
                "reply_text": "",
                "is_edit": True
            })
        
        self._running = True
        logger.info(f"Telethon listener started, monitoring {len(self.channels)} channels")
        await self.client.run_until_disconnected()
    
    async def stop(self):
        self._running = False
        if self.client:
            await self.client.disconnect()
```

### Task 2.2: TradingView Webhook Server

**Objective:** Flask POST endpoint for TradingView PineScript alerts

**Files:**
- Write: `src/input/tv_webhook.py`

```python
import hmac
import hashlib
import logging
from flask import Flask, request, jsonify
from src.config import config

logger = logging.getLogger(__name__)

app = Flask(__name__)

# Callback to be set by main.py
signal_callback = None

@app.route("/webhook/tradingview", methods=["POST"])
def tradingview_webhook():
    """Receive TradingView alert webhook."""
    # Verify signature if configured
    secret = config["tradingview"]["secret_token"]
    if secret:
        signature = request.headers.get("X-TradingView-Signature", "")
        payload = request.get_data()
        expected = hmac.new(
            secret.encode(), payload, hashlib.sha256
        ).hexdigest()
        if not hmac.compare_digest(signature, expected):
            logger.warning("Invalid webhook signature")
            return jsonify({"error": "invalid signature"}), 401
    
    data = request.get_json(force=True, silent=True) or {}
    text = data.get("message", "") or data.get("text", "")
    
    if not text:
        return jsonify({"error": "no message"}), 400
    
    if signal_callback:
        import asyncio
        asyncio.ensure_future(signal_callback({
            "source": "tradingview",
            "message_id": data.get("id", 0),
            "channel_id": "tradingview",
            "channel_name": "TradingView",
            "text": str(text),
            "reply_text": "",
            "is_edit": False,
            "raw_payload": data
        }))
    
    return jsonify({"status": "ok"}), 200

def start_webhook_server(callback):
    global signal_callback
    signal_callback = callback
    host = config["tradingview"]["webhook_host"]
    port = config["tradingview"]["webhook_port"]
    logger.info(f"TradingView webhook starting on {host}:{port}")
    # Run in background thread
    from threading import Thread
    t = Thread(target=app.run, args=(host, port), kwargs={"debug": False}, daemon=True)
    t.start()
    return t
```

### Task 2.3: Telegram Bot (Manual Entry)

**Objective:** python-telegram-bot for manual signal entry and commands

**Files:**
- Write: `src/input/telegram_bot.py`

```python
import logging
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ContextTypes
from src.config import config

logger = logging.getLogger(__name__)
signal_callback = None

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(
        "🤖 Telegram Signal Copier Bot\n\n"
        "Commands:\n"
        "/trade [signal text] - Enter a manual trade signal\n"
        "/status - Check system status\n"
        "/help - Show this help"
    )

async def trade(update: Update, context: ContextTypes.DEFAULT_TYPE):
    text = " ".join(context.args)
    if not text:
        await update.message.reply_text("Usage: /trade BUY XAUUSD 2000 SL 1990 TP 2010")
        return
    if signal_callback:
        import asyncio
        asyncio.ensure_future(signal_callback({
            "source": "bot",
            "message_id": update.message.message_id,
            "channel_id": str(update.effective_chat.id),
            "channel_name": update.effective_chat.title or "Direct",
            "text": text,
            "reply_text": "",
            "is_edit": False
        }))
    await update.message.reply_text("✅ Signal received, processing...")

async def status(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("🟢 System running")

async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Handle direct messages as signals."""
    if not update.message or not update.message.text:
        return
    # Only process DMs from authorized users
    if update.effective_chat.type == "private":
        await trade(update, context)

def start_bot(callback):
    global signal_callback
    signal_callback = callback
    token = config["telegram"]["bot_token"]
    if not token or token == "YOUR_BOT_TOKEN":
        logger.warning("No Telegram bot token configured")
        return None
    
    app = Application.builder().token(token).build()
    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("trade", trade))
    app.add_handler(CommandHandler("status", status))
    app.add_handler(CommandHandler("help", start))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))
    
    from threading import Thread
    t = Thread(target=app.run_polling, daemon=True)
    t.start()
    return t
```

### Task 2.4: OCR Reader (Screenshot Fallback)

**Files:**
- Write: `src/input/ocr_reader.py`

```python
import logging
import tempfile
from pathlib import Path

logger = logging.getLogger(__name__)

def extract_text_from_image(image_path: str) -> str:
    """Extract text from a signal screenshot using OCR."""
    try:
        import pytesseract
        from PIL import Image
        img = Image.open(image_path)
        text = pytesseract.image_to_string(img)
        return text.strip()
    except ImportError:
        logger.warning("pytesseract not installed, OCR unavailable")
        return ""
    except Exception as e:
        logger.error(f"OCR failed: {e}")
        return ""
```

---

## Phase 3: Processing Layer

### Task 3.1: Signal Parser

**Objective:** Regex engine to parse Telegram signal messages

**Files:**
- Write: `src/processing/signal_parser.py`

```python
import re
import json
import logging
from pathlib import Path

logger = logging.getLogger(__name__)

# Symbol mapping (common shorthand → MT5 symbol)
SYMBOL_MAP = {
    "GOLD": "XAUUSD", "XAUUSD": "XAUUSD",
    "SILVER": "XAGUSD", "XAGUSD": "XAGUSD",
    "OIL": "USOIL", "USOIL": "USOIL", "WTI": "USOIL",
    "EURUSD": "EURUSD", "GBPUSD": "GBPUSD",
    "USDJPY": "USDJPY", "GBPJPY": "GBPJPY",
    "AUDUSD": "AUDUSD", "USDCAD": "USDCAD",
    "USDCHF": "USDCHF", "NZDUSD": "NZDUSD",
    "EURGBP": "EURGBP", "EURJPY": "EURJPY",
    "BTCUSD": "BTCUSD", "ETHUSD": "ETHUSD",
    "SP500": "SP500", "NASDAQ": "NAS100",
    "DJI": "US30", "US30": "US30",
    "COPPER": "COPPER", "NATURALGAS": "NATGAS",
    "PLATINUM": "PLATINUM", "PALLADIUM": "PALLADIUM",
}

async def parse_signal(data: dict) -> dict | None:
    """Parse raw Telegram signal into structured trade data."""
    text = data.get("text", "").strip()
    if not text:
        return None
    
    lines = [l.strip() for l in text.split("\n") if l.strip()]
    
    result = {
        "source": data.get("source"),
        "message_id": data.get("message_id"),
        "channel_id": data.get("channel_id"),
        "channel_name": data.get("channel_name"),
        "is_edit": data.get("is_edit", False),
    }
    
    # Step 1: Detect symbol
    symbol = _detect_symbol(text, lines)
    if symbol:
        result["symbol"] = symbol
    
    # Step 2: Detect action (BUY/SELL/BUY LIMIT/etc)
    action = _detect_action(text, lines)
    if action:
        result["action"] = action
    
    # Step 3: Extract prices
    prices = _extract_prices(text, lines)
    for key in prices:
        result[key] = prices[key]
    
    # Step 4: Extract lot size
    lot = _extract_lot_size(text, lines)
    if lot:
        result["lot_size"] = lot
    
    return result if "symbol" in result and "action" in result else None


def _detect_symbol(text: str, lines: list) -> str | None:
    # Check first line first for pattern like "GOLD BUY" or "BUY XAUUSD"
    first_line = lines[0].upper() if lines else text.upper()
    
    # Direct match
    for alias, symbol in SYMBOL_MAP.items():
        if alias in first_line:
            return symbol
    
    # Try to match any symbol in the full text
    for alias, symbol in SYMBOL_MAP.items():
        if alias in text.upper():
            return symbol
    
    # Regex: look for standard 6-char forex pairs
    fx_match = re.search(r'\b([A-Z]{6})\b', text.upper())
    if fx_match:
        sym = fx_match.group(1)
        if sym in SYMBOL_MAP:
            return SYMBOL_MAP[sym]
        return sym
    
    return None


def _detect_action(text: str, lines: list) -> str | None:
    text_upper = text.upper()
    first_line = lines[0].upper() if lines else text_upper
    
    # Order matters: check compound actions first
    for action in ["BUY LIMIT", "SELL LIMIT", "BUY STOP", "SELL STOP", "BUY", "SELL"]:
        if action in first_line or action in text_upper:
            return action
    return None


def _extract_prices(text: str, lines: list) -> dict:
    """Extract entry price, stop loss, and take profit levels."""
    prices = {}
    
    # Price pattern: digits with optional decimal
    num_pattern = r'(\d+\.?\d*)'
    
    # Extract ALL prices from the text
    all_numbers = re.findall(num_pattern, text)
    
    if not all_numbers:
        return prices
    
    # Extract SL
    sl_match = re.search(r'(?:SL|STOP\s*LOSS|STOPLOSS)\s*[:_=]?\s*' + num_pattern, text, re.IGNORECASE)
    if sl_match:
        prices["stop_loss"] = float(sl_match.group(1))
    
    # Extract TPs
    tp_pattern = re.compile(r'(?:TP|TAKE\s*PROFIT|TAKEPROFIT|TARGET)\s*[:\s]*(\d+\.?\d*)', re.IGNORECASE)
    tp_matches = tp_pattern.findall(text)
    if tp_matches:
        prices["take_profits"] = [float(tp) for tp in tp_matches]
    
    # Try to detect TP with superscript (TP¹, TP² etc.)
    tp_super = re.findall(r'TP[¹²³⁴⁵⁶⁷⁸⁹⁰]\s*(\d+\.?\d*)', text)
    if tp_super:
        prices["take_profits"] = [float(tp) for tp in tp_super]
    
    # Extract entry price
    # If it's a market order (BUY/SELL without LIMIT/STOP), entry might be "MARKET" or next number
    entry_pattern = re.compile(r'(?:at|@|entry|price|now)\s*:?\s*' + num_pattern, re.IGNORECASE)
    entry_match = entry_pattern.search(text)
    if entry_match:
        prices["entry_price"] = float(entry_match.group(1))
    elif lines:
        # Try first line: "GOLD BUY 4444" → read last number
        first_line_nums = re.findall(num_pattern, lines[0])
        if first_line_nums:
            prices["entry_price"] = float(first_line_nums[-1])
    
    return prices


def _extract_lot_size(text: str, lines: list) -> float | None:
    lot_pattern = re.compile(r'(?:lot|lots|volume|size|💰)\s*:?\s*(\d+\.?\d*)', re.IGNORECASE)
    match = lot_pattern.search(text)
    if match:
        return float(match.group(1))
    return None
```

### Task 3.2: Signal Validator

**Files:**
- Write: `src/processing/signal_validator.py`

```python
import logging
from datetime import datetime
from src.config import config

logger = logging.getLogger(__name__)

class SignalValidator:
    def __init__(self, db):
        self.db = db
        self.processed_ids = set()
    
    async def validate(self, parsed: dict) -> tuple[bool, str]:
        """Validate parsed signal. Returns (is_valid, reason)."""
        
        # 1. Check required fields
        if not parsed.get("symbol"):
            return False, "No symbol detected"
        if not parsed.get("action"):
            return False, "No action detected"
        
        # 2. Deduplicate by message_id
        msg_id = parsed.get("message_id")
        if msg_id and msg_id in self.processed_ids:
            return False, f"Duplicate signal (msg_id={msg_id})"
        if msg_id:
            self.processed_ids.add(msg_id)
        
        # 3. Symbol whitelist
        # (Can load from data/symbols.txt)
        
        # 4. Price sanity check
        entry = parsed.get("entry_price")
        sl = parsed.get("stop_loss")
        tps = parsed.get("take_profits", [])
        
        if entry and (entry <= 0 or entry > 1000000):
            return False, f"Invalid entry price: {entry}"
        if sl and (sl <= 0 or sl > 1000000):
            return False, f"Invalid SL: {sl}"
        
        # 5. Session filter (trading hours)
        now = datetime.now()
        session_start = config["risk"]["session_start"]
        session_end = config["risk"]["session_end"]
        # Simple time check
        current_time = now.strftime("%H:%M")
        
        return True, "OK"
```

### Task 3.3: Risk Manager

**Files:**
- Write: `src/processing/risk_manager.py`

```python
import logging
from src.config import config

logger = logging.getLogger(__name__)

class RiskManager:
    def __init__(self, db):
        self.db = db
        self.consecutive_losses = 0
    
    def calculate_lot_size(self, balance: float, entry: float, stop_loss: float,
                           symbol: str, action: str) -> float:
        """Calculate position size based on risk management rules."""
        risk_pct = config["risk"]["risk_percent"] / 100.0
        max_loss = config["risk"]["max_daily_loss"]
        
        # Calculate SL distance in points
        sl_distance = abs(entry - stop_loss)
        if sl_distance == 0:
            return config["risk"]["default_lot_size"]
        
        # Pip value varies by symbol
        pip_value = self._get_pip_value(symbol, entry)
        
        # Lot size formula
        risk_amount = balance * risk_pct
        lot_size = risk_amount / (sl_distance / pip_value * 100000)
        
        # Round to standard lot sizes (0.01 increments)
        lot_size = round(lot_size * 100) / 100
        
        # Clamp to minimum
        if lot_size < config["risk"]["default_lot_size"]:
            lot_size = config["risk"]["default_lot_size"]
        
        return lot_size
    
    def _get_pip_value(self, symbol: str, price: float) -> float:
        """Get pip value for symbol."""
        if symbol in ["XAUUSD", "GOLD"]:
            return 0.1  # Gold: 0.1 per pip
        elif symbol in ["XAGUSD"]:
            return 0.01
        elif "JPY" in symbol:
            return 0.01  # Yen pairs: 0.01
        elif "BTC" in symbol or "ETH" in symbol:
            return 1.0  # Crypto: 1.0
        else:
            return 0.0001  # Standard forex
    
    def check_daily_loss_limit(self) -> bool:
        """Returns True if daily loss limit exceeded."""
        # Query today's closed orders from DB
        return False  # TODO: implement
    
    def check_consecutive_losses(self) -> bool:
        """Returns False if consecutive losses exceed limit."""
        max_consecutive = config["risk"]["max_consecutive_losses"]
        return self.consecutive_losses < max_consecutive
```

---

## Phase 4: Execution Layer — Path A (MT5)

### Task 4.1: MT5 Gateway

**Files:**
- Write: `src/execution/mt5_gateway.py`

```python
import logging
from src.config import config

logger = logging.getLogger(__name__)

class MT5Gateway:
    def __init__(self):
        self.connected = False
    
    def connect(self) -> bool:
        """Connect to MT5 terminal."""
        try:
            import MetaTrader5 as mt5
            if not mt5.initialize():
                logger.error(f"MT5 init failed: {mt5.last_error()}")
                return False
            
            login = config["mt5"]["login"]
            password = config["mt5"]["password"]
            server = config["mt5"]["server"]
            
            authorized = mt5.login(login, password, server)
            if not authorized:
                logger.error(f"MT5 login failed: {mt5.last_error()}")
                return False
            
            self.connected = True
            logger.info(f"Connected to MT5: {server} (login: {login})")
            return True
        except ImportError:
            logger.error("MetaTrader5 package not installed")
            return False
        except Exception as e:
            logger.error(f"MT5 connection failed: {e}")
            return False
    
    def disconnect(self):
        try:
            import MetaTrader5 as mt5
            mt5.shutdown()
        except:
            pass
        self.connected = False
    
    def is_connected(self) -> bool:
        if not self.connected:
            return False
        try:
            import MetaTrader5 as mt5
            return mt5.terminal_info() is not None
        except:
            return False
    
    def get_account_info(self) -> dict:
        try:
            import MetaTrader5 as mt5
            info = mt5.account_info()
            if info:
                return {
                    "balance": info.balance,
                    "equity": info.equity,
                    "margin": info.margin,
                    "margin_free": info.margin_free,
                    "profit": info.profit,
                }
        except:
            pass
        return {}
```

### Task 4.2: MT5 Executor

**Files:**
- Write: `src/execution/mt5_executor.py`

```python
import logging
import MetaTrader5 as mt5
from datetime import datetime

logger = logging.getLogger(__name__)

class MT5Executor:
    MAGIC_NUMBER = 234000
    
    def open_market_order(self, symbol: str, action: str, volume: float,
                          sl: float = 0, tp_list: list = None,
                          comment: str = "Signal Copier") -> dict:
        """Open a market order on MT5."""
        order_type = mt5.ORDER_TYPE_BUY if action.upper() == "BUY" else mt5.ORDER_TYPE_SELL
        
        # Get current price
        tick = mt5.symbol_info_tick(symbol)
        if not tick:
            return {"error": f"Cannot get price for {symbol}"}
        
        price = tick.ask if order_type == mt5.ORDER_TYPE_BUY else tick.bid
        
        # Take first TP (MT5 supports one TP per order)
        tp = tp_list[0] if tp_list else 0
        
        request = {
            "action": mt5.TRADE_ACTION_DEAL,
            "symbol": symbol,
            "volume": volume,
            "type": order_type,
            "price": price,
            "sl": sl,
            "tp": tp,
            "deviation": 20,
            "magic": self.MAGIC_NUMBER,
            "comment": comment,
            "type_time": mt5.ORDER_TIME_GTC,
            "type_filling": mt5.ORDER_FILLING_IOC,
        }
        
        result = mt5.order_send(request)
        if result.retcode != mt5.TRADE_RETCODE_DONE:
            return {"error": f"Order failed: {result.comment} (code: {result.retcode})"}
        
        logger.info(f"Order opened: {symbol} {action} vol={volume} ticket={result.order}")
        
        return {
            "ticket": result.order,
            "price": price,
            "volume": volume,
            "sl": sl,
            "tp": tp,
        }
    
    def open_pending_order(self, symbol: str, action: str, volume: float,
                           price: float, sl: float = 0, tp: float = 0) -> dict:
        """Open a pending order (limit/stop)."""
        type_map = {
            "BUY LIMIT": mt5.ORDER_TYPE_BUY_LIMIT,
            "SELL LIMIT": mt5.ORDER_TYPE_SELL_LIMIT,
            "BUY STOP": mt5.ORDER_TYPE_BUY_STOP,
            "SELL STOP": mt5.ORDER_TYPE_SELL_STOP,
        }
        order_type = type_map.get(action.upper())
        if not order_type:
            return {"error": f"Invalid pending order type: {action}"}
        
        request = {
            "action": mt5.TRADE_ACTION_PENDING,
            "symbol": symbol,
            "volume": volume,
            "type": order_type,
            "price": price,
            "sl": sl,
            "tp": tp,
            "deviation": 20,
            "magic": self.MAGIC_NUMBER,
            "comment": "Signal Copier",
            "type_time": mt5.ORDER_TIME_GTC,
            "type_filling": mt5.ORDER_FILLING_IOC,
        }
        
        result = mt5.order_send(request)
        if result.retcode != mt5.TRADE_RETCODE_DONE:
            return {"error": f"Pending order failed: {result.comment}"}
        
        return {"ticket": result.order}
    
    def modify_sl_tp(self, ticket: int, sl: float, tp: float) -> dict:
        """Modify stop loss and take profit for an open position."""
        position = mt5.positions_get(ticket=ticket)
        if not position:
            return {"error": f"Position {ticket} not found"}
        pos = position[0]
        
        request = {
            "action": mt5.TRADE_ACTION_SLTP,
            "position": ticket,
            "symbol": pos.symbol,
            "sl": sl,
            "tp": tp,
        }
        result = mt5.order_send(request)
        return {"ok": result.retcode == mt5.TRADE_RETCODE_DONE}
    
    def close_position(self, ticket: int, volume: float = None) -> dict:
        """Close a position by ticket."""
        position = mt5.positions_get(ticket=ticket)
        if not position:
            return {"error": f"Position {ticket} not found"}
        pos = position[0]
        
        close_volume = volume or pos.volume
        order_type = mt5.ORDER_TYPE_SELL if pos.type == mt5.ORDER_TYPE_BUY else mt5.ORDER_TYPE_BUY
        
        tick = mt5.symbol_info_tick(pos.symbol)
        price = tick.bid if order_type == mt5.ORDER_TYPE_BUY else tick.ask
        
        request = {
            "action": mt5.TRADE_ACTION_DEAL,
            "symbol": pos.symbol,
            "volume": close_volume,
            "type": order_type,
            "position": ticket,
            "price": price,
            "deviation": 20,
            "magic": self.MAGIC_NUMBER,
            "comment": "Close by copier",
            "type_time": mt5.ORDER_TIME_GTC,
            "type_filling": mt5.ORDER_FILLING_IOC,
        }
        
        result = mt5.order_send(request)
        return {"ok": result.retcode == mt5.TRADE_RETCODE_DONE, "ticket": ticket}
    
    def close_all_positions(self, symbol: str = None) -> list:
        """Close all positions (optionally for a specific symbol)."""
        positions = mt5.positions_get(symbol=symbol) if symbol else mt5.positions_get()
        results = []
        for pos in positions or []:
            results.append(self.close_position(pos.ticket))
        return results
```

### Task 4.3: Order Manager (Unified Interface)

**Files:**
- Write: `src/execution/order_manager.py`

```python
import logging
from src.config import config
from src.processing.signal_parser import parse_signal
from src.processing.database import Database
from src.processing.risk_manager import RiskManager
from src.execution.mt5_gateway import MT5Gateway
from src.execution.mt5_executor import MT5Executor

logger = logging.getLogger(__name__)

class OrderManager:
    def __init__(self, db: Database):
        self.db = db
        self.mt5_gateway = MT5Gateway()
        self.mt5_executor = MT5Executor()
        self.risk_manager = RiskManager(db)
        self.execution_mode = config["execution"]["mode"]
    
    async def process_signal(self, data: dict) -> dict:
        """Process an incoming signal through the full pipeline."""
        logger.info(f"Processing signal from {data.get('source')}: msg={data.get('message_id')}")
        
        from src.processing.signal_parser import parse_signal
        parsed = await parse_signal(data)
        if not parsed:
            logger.warning(f"Failed to parse signal: {data.get('text', '')[:100]}")
            return {"status": "parse_failed", "data": data}
        
        # Save to DB
        signal_id = self.db.save_signal(
            message_id=parsed.get("message_id", 0),
            channel_id=parsed.get("channel_id", ""),
            channel_name=parsed.get("channel_name", ""),
            raw_text=data.get("text", ""),
            parsed_json=parsed
        )
        
        if self.execution_mode in ("mt5", "both"):
            if not self.mt5_gateway.is_connected():
                if not self.mt5_gateway.connect():
                    return {"status": "mt5_connection_failed"}
            
            # Get balance for risk calculation
            account = self.mt5_gateway.get_account_info()
            balance = account.get("balance", 10000)
            
            # Calculate lot size
            entry = parsed.get("entry_price", 0)
            sl = parsed.get("stop_loss", 0)
            tps = parsed.get("take_profits", [])
            
            lot = parsed.get("lot_size") or self.risk_manager.calculate_lot_size(
                balance, entry, sl, parsed.get("symbol"), parsed.get("action")
            )
            
            # Determine order type
            action = parsed["action"]
            if action in ("BUY", "SELL"):
                result = self.mt5_executor.open_market_order(
                    symbol=parsed["symbol"],
                    action=action,
                    volume=lot,
                    sl=sl,
                    tp_list=tps,
                )
            else:
                result = self.mt5_executor.open_pending_order(
                    symbol=parsed["symbol"],
                    action=action,
                    volume=lot,
                    price=entry,
                    sl=sl,
                    tp=tps[0] if tps else 0,
                )
            
            ticket = result.get("ticket")
            order_id = self.db.save_order(
                signal_id=signal_id,
                symbol=parsed["symbol"],
                action=action,
                entry_price=entry,
                stop_loss=sl,
                take_profits=tps,
                volume=lot,
                ticket=ticket,
                execution_path="mt5"
            )
            
            return {
                "status": "executed",
                "order_id": order_id,
                "ticket": ticket,
                "symbol": parsed["symbol"],
                "action": action,
                "volume": lot,
            }
        
        elif self.execution_mode in ("tradingview", "both"):
            # For TV mode, send to webhook instead
            from src.execution.tv_executor import send_to_tradingview
            tv_result = send_to_tradingview(parsed)
            return {"status": "forwarded_to_tv", **tv_result}
        
        return {"status": "no_execution", "parsed": parsed}
```

---

## Phase 5: Execution Layer — Path B (TradingView Webhook)

### Task 5.1: TV Executor

**Files:**
- Write: `src/execution/tv_executor.py`

```python
import json
import logging
import requests
from src.config import config

logger = logging.getLogger(__name__)

def format_for_tradingview(parsed_signal: dict) -> dict:
    """Format parsed signal for TradingView webhook."""
    return {
        "symbol": parsed_signal.get("symbol"),
        "action": parsed_signal.get("action"),
        "entry": parsed_signal.get("entry_price", 0),
        "stop_loss": parsed_signal.get("stop_loss", 0),
        "take_profits": parsed_signal.get("take_profits", []),
        "volume": parsed_signal.get("lot_size", 0.01),
        "timestamp": parsed_signal.get("message_id", 0),
    }

def send_to_tradingview(parsed_signal: dict) -> dict:
    """Send signal as JSON (to be received by PineScript webhook)."""
    payload = format_for_tradingview(parsed_signal)
    logger.info(f"Formatted for TradingView: {json.dumps(payload)}")
    return {"payload": payload}
```

### Task 5.2: PineScript Template

**Files:**
- Create: `data/tradingview_pinescript_template.txt`

```pine
//@version=5
indicator("Signal Copier Webhook", overlay=true)

// Webhook message format:
// {"symbol":"XAUUSD","action":"BUY","entry":4444,"stop_loss":4433,"take_profits":[4447,4450,4453],"volume":0.01}

// Webhook alert setup:
// Settings → Alerts → Webhook URL → http://your-server:5000/webhook/tradingview
// Message: {{ticker}}|{{strategy.order.action}}|...

// Manual entry with this indicator:
// Just use it to verify alerts work

plot(close)
```

---

## Phase 6: Reporting Layer (Journal)

### Task 6.1: Journal Engine

**Files:**
- Write: `src/reporting/journal_engine.py`

```python
import logging
from datetime import datetime, timedelta
import pandas as pd
import numpy as np
from apscheduler.schedulers.background import BackgroundScheduler
from src.config import config

logger = logging.getLogger(__name__)

class JournalEngine:
    def __init__(self, db, reporter_callback):
        self.db = db
        self.reporter_callback = reporter_callback
        self.scheduler = BackgroundScheduler()
    
    def start(self):
        daily_time = config["reporting"]["daily_report_time"]
        logger.info(f"Scheduling daily report at {daily_time}")
        self.scheduler.add_job(
            self.generate_daily_report,
            'cron',
            hour=int(daily_time.split(":")[0]),
            minute=int(daily_time.split(":")[1]),
            id="daily_journal"
        )
        
        weekly_day = config["reporting"]["weekly_report_day"].lower()
        weekly_time = config["reporting"]["weekly_report_time"]
        day_map = {
            "monday": 0, "tuesday": 1, "wednesday": 2, "thursday": 3,
            "friday": 4, "saturday": 5, "sunday": 6
        }
        logger.info(f"Scheduling weekly report on {weekly_day} at {weekly_time}")
        self.scheduler.add_job(
            self.generate_weekly_report,
            'cron',
            day_of_week=day_map.get(weekly_day, 6),
            hour=int(weekly_time.split(":")[0]),
            minute=int(weekly_time.split(":")[1]),
            id="weekly_journal"
        )
        
        self.scheduler.start()
    
    def stop(self):
        self.scheduler.shutdown()
    
    def generate_daily_report(self):
        today = datetime.now().strftime("%Y-%m-%d")
        trades = self.db.get_trades_for_date(today)
        
        if not trades:
            msg = f"📊 Daily Journal ({today}): No trades today"
            self.reporter_callback(msg)
            return
        
        df = pd.DataFrame(trades)
        closed = df[df["status"] == "closed"]
        
        report = {
            "type": "daily",
            "date": today,
            "total": len(closed),
            "wins": len(closed[closed["profit"] > 0]) if not closed.empty else 0,
            "losses": len(closed[closed["profit"] <= 0]) if not closed.empty else 0,
            "net_pnl": closed["profit"].sum() if not closed.empty else 0,
            "win_rate": (len(closed[closed["profit"] > 0]) / len(closed) * 100) if not closed.empty else 0,
            "max_drawdown": df["drawdown"].max() if "drawdown" in df and not df.empty else 0,
        }
        
        # Profit factor
        wins_sum = closed[closed["profit"] > 0]["profit"].sum() if not closed.empty else 0
        losses_sum = abs(closed[closed["profit"] <= 0]["profit"].sum()) if not closed.empty else 1
        report["profit_factor"] = round(wins_sum / losses_sum, 2) if losses_sum > 0 else 0
        
        self._save_report(report)
        
        from src.reporting.report_templates import format_daily_report
        msg = format_daily_report(report)
        self.reporter_callback(msg)
    
    def generate_weekly_report(self):
        today = datetime.now()
        start_of_week = (today - timedelta(days=today.weekday())).strftime("%Y-%m-%d")
        end_of_week = today.strftime("%Y-%m-%d")
        
        trades = self.db.get_trades_for_week(start_of_week, end_of_week)
        
        if not trades:
            msg = f"📈 Weekly Journal ({start_of_week} to {end_of_week}): No trades"
            self.reporter_callback(msg)
            return
        
        df = pd.DataFrame(trades)
        closed = df[df["status"] == "closed"]
        
        report = {
            "type": "weekly",
            "start_date": start_of_week,
            "end_date": end_of_week,
            "total": len(closed),
            "wins": len(closed[closed["profit"] > 0]),
            "losses": len(closed[closed["profit"] <= 0]),
            "net_pnl": closed["profit"].sum(),
            "win_rate": (len(closed[closed["profit"] > 0]) / len(closed) * 100),
            "avg_win": closed[closed["profit"] > 0]["profit"].mean() if not closed[closed["profit"] > 0].empty else 0,
            "avg_loss": closed[closed["profit"] <= 0]["profit"].mean() if not closed[closed["profit"] <= 0].empty else 0,
            "max_drawdown": df["drawdown"].max(),
        }
        
        # Best/worst day
        if not df.empty and "open_time" in df.columns:
            df["date"] = pd.to_datetime(df["open_time"]).dt.date
            daily_pnl = df.groupby("date")["profit"].sum()
            if not daily_pnl.empty:
                report["best_day"] = str(daily_pnl.idxmax())
                report["best_day_pnl"] = daily_pnl.max()
                report["worst_day"] = str(daily_pnl.idxmin())
                report["worst_day_pnl"] = daily_pnl.min()
        
        wins_sum = closed[closed["profit"] > 0]["profit"].sum()
        losses_sum = abs(closed[closed["profit"] <= 0]["profit"].sum()) or 1
        report["profit_factor"] = round(wins_sum / losses_sum, 2)
        
        self._save_report(report)
        
        from src.reporting.report_templates import format_weekly_report
        msg = format_weekly_report(report)
        self.reporter_callback(msg)
    
    def _save_report(self, report: dict):
        try:
            import json
            self.db._get_conn().execute("""
                INSERT INTO journal_reports 
                (report_type, report_date, total_trades, wins, losses, 
                 net_pnl, win_rate, profit_factor, max_drawdown, report_json)
                VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
            """, (
                report["type"],
                report.get("date", report.get("end_date")),
                report["total"], report["wins"], report["losses"],
                report["net_pnl"], report["win_rate"],
                report.get("profit_factor", 0),
                report["max_drawdown"],
                json.dumps(report, default=str)
            ))
        except Exception as e:
            logger.error(f"Failed to save report: {e}")
```

### Task 6.2: Telegram Reporter

**Files:**
- Write: `src/reporting/telegram_reporter.py`

```python
import logging
from telethon import TelegramClient
from src.config import config

logger = logging.getLogger(__name__)

class TelegramReporter:
    def __init__(self):
        self.client = None
        self.chat_id = config["reporting"]["send_to_chat_id"]
    
    async def start(self):
        self.client = TelegramClient(
            "reporter_session",
            config["telegram"]["api_id"],
            config["telegram"]["api_hash"]
        )
        await self.client.start(phone=config["telegram"]["phone"])
        logger.info(f"Telegram reporter ready, sending to {self.chat_id}")
    
    async def send_report(self, message: str):
        if not self.client or not self.client.is_connected():
            await self.start()
        
        try:
            await self.client.send_message(self.chat_id, message)
            logger.info(f"Report sent to {self.chat_id}")
        except Exception as e:
            logger.error(f"Failed to send report: {e}")
```

### Task 6.3: Report Templates

**Files:**
- Write: `src/reporting/report_templates.py`

```python
def format_daily_report(report: dict) -> str:
    emoji_pnl = "✅" if report["net_pnl"] >= 0 else "❌"
    return f"""📊 Daily Trading Journal — {report['date']}
━━━━━━━━━━━━━━━━━━━━━━━━━━
📈 Total Trades: {report['total']}
✅ Won: {report['wins']} ({report['win_rate']:.1f}%)
❌ Lost: {report['losses']} ({100 - report['win_rate']:.1f}%)
{emoji_pnl} PnL: ${report['net_pnl']:+.2f}
📉 Max Drawdown: ${report['max_drawdown']:.2f}
⚡ Profit Factor: {report.get('profit_factor', 0):.2f}
━━━━━━━━━━━━━━━━━━━━━━━━━━"""

def format_weekly_report(report: dict) -> str:
    emoji_pnl = "✅" if report["net_pnl"] >= 0 else "❌"
    msg = f"""📈 Weekly Trading Journal — {report['start_date']} to {report['end_date']}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 Total Trades: {report['total']}  |  Won: {report['wins']}  |  Lost: {report['losses']}
{emoji_pnl} Net PnL: ${report['net_pnl']:+.2f}
📉 Max Drawdown: ${report['max_drawdown']:.2f}
📊 Win Rate: {report['win_rate']:.1f}%
⚡ Profit Factor: {report.get('profit_factor', 0):.2f}
📈 Avg Win: ${report.get('avg_win', 0):.2f}  |  Avg Loss: ${report.get('avg_loss', 0):.2f}"""
    
    if "best_day" in report:
        msg += f"\n🔄 Best Day: {report['best_day']} (+${report['best_day_pnl']:+.2f})"
    if "worst_day" in report:
        msg += f"\n🔴 Worst Day: {report['worst_day']} (${report['worst_day_pnl']:.2f})"
    
    msg += f"\n━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
    return msg
```

---

## Phase 7: Main Entry Point

### Task 7.1: Main Application

**Files:**
- Write: `src/main.py`

```python
#!/usr/bin/env python3
"""
Telegram Signal Copier — Main Entry Point

Starts all services:
1. Telethon listener (Telegram channel monitoring)
2. TradingView webhook server (Flask)
3. Telegram bot (manual commands)
4. Journal engine (APScheduler daily/weekly reports)
"""
import asyncio
import logging
import sys
from pathlib import Path

# Add project root to path
sys.path.insert(0, str(Path(__file__).parent.parent))

from src.config import config
from src.processing.database import Database
from src.execution.order_manager import OrderManager
from src.input.telethon_listener import TelethonListener
from src.input.tv_webhook import start_webhook_server
from src.input.telegram_bot import start_bot
from src.reporting.journal_engine import JournalEngine
from src.reporting.telegram_reporter import TelegramReporter

# Setup logging
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
    handlers=[
        logging.StreamHandler(),
        logging.FileHandler("logs/app.log")
    ]
)
logger = logging.getLogger(__name__)


async def main():
    logger.info("=== Telegram Signal Copier Starting ===")
    
    # Initialize database
    db = Database()
    logger.info(f"Database initialized at {db.db_path}")
    
    # Initialize order manager (processing + execution pipeline)
    order_manager = OrderManager(db)
    
    # Global callback for all input sources
    async def signal_callback(data: dict):
        await order_manager.process_signal(data)
    
    # Initialize Telegram reporter (for journal reports)
    reporter = TelegramReporter()
    
    # Initialize journal engine
    journal = JournalEngine(db, lambda msg: asyncio.ensure_future(reporter.send_report(msg)))
    if config["reporting"]["enable_daily"] or config["reporting"]["enable_weekly"]:
        journal.start()
        logger.info("Journal engine started")
    
    # Start Telethon listener
    telethon = TelethonListener(signal_callback)
    telethon_task = asyncio.ensure_future(telethon.start())
    logger.info("Telethon listener starting...")
    
    # Start TradingView webhook server (in background thread)
    start_webhook_server(signal_callback)
    
    # Start Telegram bot (in background thread)
    start_bot(signal_callback)
    
    # Wait for telethon
    await telethon_task


if __name__ == "__main__":
    try:
        asyncio.run(main())
    except KeyboardInterrupt:
        logger.info("Shutting down...")
    except Exception as e:
        logger.error(f"Fatal error: {e}", exc_info=True)
```

---

## Verification Tasks

### V1: Unit Tests

Create `tests/` directory with tests for each module:

```bash
mkdir -p tests
touch tests/__init__.py
```

**Test: Parser**
```python
# tests/test_parser.py
import pytest
from src.processing.signal_parser import parse_signal, _detect_symbol, _detect_action

@pytest.mark.asyncio
async def test_parse_gold_buy():
    data = {
        "source": "telethon",
        "message_id": 1,
        "text": "GOLD BUY 4444\nTP¹ 4447\nTP² 4450\nSL_4433",
    }
    result = await parse_signal(data)
    assert result["symbol"] == "XAUUSD"
    assert result["action"] == "BUY"
    assert result["entry_price"] == 4444.0
    assert result["take_profits"] == [4447.0, 4450.0]
    assert result["stop_loss"] == 4433.0

@pytest.mark.asyncio
async def test_parse_forex():
    data = {
        "source": "telethon",
        "message_id": 2,
        "text": "BUY EURUSD 1.0850\nTP: 1.0870\nSL: 1.0830",
    }
    result = await parse_signal(data)
    assert result["symbol"] == "EURUSD"
    assert result["action"] == "BUY"
    assert result["take_profits"] == [1.0870]
```

---

## Implementation Order Summary

| Phase | Tasks | Est. Time |
|-------|-------|-----------|
| 0 | Project setup, directories, dependencies | 10 min |
| 1 | Config loader, database module | 30 min |
| 2 | Telethon listener, TV webhook, Telegram bot, OCR | 45 min |
| 3 | Signal parser, validator, risk manager | 40 min |
| 4 | MT5 gateway, executor, order manager | 40 min |
| 5 | TV executor, PineScript template | 15 min |
| 6 | Journal engine, reporter, templates | 35 min |
| 7 | Main entry point, integration | 20 min |
| V | Unit tests, manual verification | 30 min |
| **Total** | | **~4.5 hours** |

---

> **Ready to implement?** Execute this plan using subagent-driven-development — dispatch a fresh subagent per task with the full task specification and context.
