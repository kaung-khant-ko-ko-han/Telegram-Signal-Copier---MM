# Standard Operating Procedure (SOP)
## Telegram Signal Copier — MT5 & TradingView

**Version:** 1.0.0
**Last Updated:** 2026-06-07
**Author:** AI Development Team
**Classification:** Internal Use Only

---

## မာတိကာ (Table of Contents)

1. [စနစ်အကြောင်း နိဒါန်း](#1-စနစ်အကြောင်း-နိဒါန်း)
2. [လိုအပ်ချက်များ (Prerequisites)](#2-လိုအပ်ချက်များ-prerequisites)
3. [တပ်ဆင်ခြင်း (Installation)](#3-တပ်ဆင်ခြင်း-installation)
4. [ဖွဲ့စည်းမှု Configuration](#4-ဖွဲ့စည်းမှု-configuration)
5. [စနစ်စတင်ခြင်း (Startup)](#5-စနစ်စတင်ခြင်း-startup)
6. [နေ့စဉ်လည်ပတ်ခြင်း (Daily Operations)](#6-နေ့စဉ်လည်ပတ်ခြင်း-daily-operations)
7. [Signal ဖော်မတ်များ (Signal Formats)](#7-signal-ဖော်မတ်များ-signal-formats)
8. [Journal Report စနစ် (Reporting)](#8-journal-report-စနစ်-reporting)
9. [ပြဿနာဖြေရှင်းခြင်း (Troubleshooting)](#9-ပြဿနာဖြေရှင်းခြင်း-troubleshooting)
10. [ဘေးကင်းရေး စည်းမျဉ်းများ (Safety Rules)](#10-ဘေးကင်းရေး-စည်းမျဉ်းများ-safety-rules)
11. [အရေးပေါ် လုပ်ထုံးလုပ်နည်းများ (Emergency Procedures)](#11-အရေးပေါ်-လုပ်ထုံးလုပ်နည်းများ-emergency-procedures)
12. [ပြုပြင်ထိန်းသိမ်းခြင်း (Maintenance)](#12-ပြုပြင်ထိန်းသိမ်းခြင်း-maintenance)

---

## 1. စနစ်အကြောင်း နိဒါန်း

### 1.1 စနစ်၏ ရည်ရွယ်ချက်

Telegram Signal Copier သည် Telegram Channel/Group များမှ Trading Signal များကို အလိုအလျောက်ဖမ်းယူ၍ MT5 (MetaTrader 5) နှင့် TradingView သို့ Order တင်ပေးသည့် အလိုအလျောက်စနစ်ဖြစ်သည်။

### 1.2 စနစ်ဖွဲ့စည်းပုံ (4-Layer Architecture)

```
အလွှာ 1: INPUT     → Telegram (Telethon) + TradingView Webhook + Bot
အလွှာ 2: PROCESS   → Signal Parser (Regex) + Validator + Risk Manager + DB
အလွှာ 3: EXECUTE   → Path A: MT5 Python API  |  Path B: TradingView Webhook
အလွှာ 4: REPORT    → Journal Engine (APScheduler) → Telegram Reporter
```

### 1.3 အဓိကအင်္ဂါရပ်များ

| # | အင်္ဂါရပ် | အသေးစိတ် |
|---|-----------|-----------|
| 1 | Telegram Channel Monitoring | Telethon API ဖြင့် Real-time စောင့်ကြည့်ခြင်း |
| 2 | Signal Parsing | Regex ဖြင့် Symbol, Action, Entry, SL, TP ခွဲထုတ်ခြင်း |
| 3 | Dual Execution Path | MT5 (တိုက်ရိုက်) + TradingView (Webhook) |
| 4 | Risk Management | Balance ပေါ်မူတည်၍ Lot Size အလိုအလျောက်တွက်ခြင်း |
| 5 | Journal Reporting | နေ့စဉ်/အပတ်စဉ် Report များ Telegram သို့ပို့ခြင်း |
| 6 | SQLite Database | Signal နှင့် Trade History အားလုံးသိမ်းဆည်းခြင်း |

---

## 2. လိုအပ်ချက်များ (Prerequisites)

### 2.1 Hardware လိုအပ်ချက်များ

| အစိတ်အပိုင်း | အနိမ့်ဆုံး | အကြံပြုချက် |
|-------------|------------|-------------|
| CPU | Dual Core 2.0 GHz | Quad Core 2.5 GHz+ |
| RAM | 4 GB | 8 GB |
| Storage | 10 GB Free | 50 GB Free (SSD) |
| Internet | Stable Connection | Fiber/VPS (24/7) |

### 2.2 Software လိုအပ်ချက်များ

| Software | Version | မှတ်ချက် |
|----------|---------|-----------|
| Windows | 10/11 (64-bit) | MT5 အတွက် မဖြစ်မနေ |
| Python | 3.11+ | [Download](https://python.org) |
| MetaTrader 5 | Latest | Windows တွင်သာ အလုပ်လုပ် |
| Git | Latest | [Download](https://git-scm.com) |

### 2.3 အကောင့်လိုအပ်ချက်များ

| အကောင့် | ရယူရန် နေရာ | လိုအပ်သော Data |
|----------|--------------|-----------------|
| Telegram API | [my.telegram.org](https://my.telegram.org) | `api_id`, `api_hash` |
| Telegram Bot | [@BotFather](https://t.me/botfather) | `bot_token` (Optional) |
| MT5 Broker | သင့် Broker | `Server`, `Login`, `Password` |
| TradingView | [TradingView](https://tradingview.com) | Webhook URL (Optional) |

### 2.4 Python Packages လိုအပ်ချက်များ

```
# Core
telethon>=1.37
python-telegram-bot>=20.0
apscheduler>=3.10

# Config
pyyaml>=6.0
python-dotenv>=1.0

# Processing
pandas>=2.0
numpy>=1.24

# Reporting
matplotlib>=3.7

# Webhook
flask>=3.0

# MT5 (Windows Only)
MetaTrader5>=5.0
```

---

## 3. တပ်ဆင်ခြင်း (Installation)

### 3.1 Project ကို Clone လုပ်ခြင်း

```bash
# Project Folder သို့သွားရန်
cd /d/Forex/AI Vibe Codeing for FX/Telegram Signal Copier from MT5 or TradingView PineScript

# (သို့) အသစ်ဆောက်လျှင်
mkdir telegram-signal-copier
cd telegram-signal-copier
```

### 3.2 Virtual Environment ဆောက်ခြင်း

```bash
# Virtual Environment ဆောက်ရန်
python -m venv venv

# Activate လုပ်ရန် (Windows)
source venv/Scripts/activate

# Activate လုပ်ရန် (Git Bash)
. venv/Scripts/activate
```

### 3.3 Dependencies တပ်ဆင်ခြင်း

```bash
# requirements.txt ရှိရင်
pip install -r requirements.txt

# (သို့) တစ်ခုချင်း
pip install telethon python-telegram-bot apscheduler pyyaml \
            python-dotenv pandas numpy matplotlib flask
```

### 3.4 MT5 Python Package တပ်ဆင်ခြင်း

> ⚠️ **သတိပေးချက်:** MetaTrader5 package သည် Windows တွင်သာ အလုပ်လုပ်သည်။

```bash
# MT5 Terminal ပိတ်ထားရမည်
pip install MetaTrader5
```

---

## 4. ဖွဲ့စည်းမှု (Configuration)

### 4.1 Environment Variables (.env)

```bash
# .env.example ကို copy ကူးရန်
cp .env.example .env
```

**.env ဖိုင်ကို တည်းဖြတ်ရန်:**

```env
# === TELEGRAM (မဖြစ်မနေ) ===
TELEGRAM_API_ID=123456               # my.telegram.org မှရယူပါ
TELEGRAM_API_HASH=abc123def456       # my.telegram.org မှရယူပါ
TELEGRAM_PHONE=+959XXXXXXXXX         # သင့်ဖုန်းနံပါတ်

# === MT5 (Path A အတွက်) ===
MT5_SERVER=ICMarkets-Demo
MT5_LOGIN=12345678
MT5_PASSWORD=your_password

# === TRADINGVIEW WEBHOOK (Path B အတွက်) ===
TV_SECRET_TOKEN=your_webhook_secret

# === REPORTING ===
REPORT_CHAT_ID=123456789            # Telegram Chat ID

# === RISK ===
RISK_PERCENT=2.0                     # 2% risk per trade
MAX_DAILY_LOSS=500.0                 # USD
MAX_CONSECUTIVE_LOSSES=3
```

### 4.2 Config YAML ဖိုင်

**`config.yaml` ကို တည်းဖြတ်ရန်:**

```yaml
# === TELEGRAM ===
telegram:
  api_id: YOUR_API_ID
  api_hash: YOUR_API_HASH
  phone: "+959XXXXXXXXX"
  session_name: signal_copier
  channels:
    - https://t.me/your_signal_channel_1
    - https://t.me/your_signal_channel_2
  bot_token: YOUR_BOT_TOKEN

# === EXECUTION PATH ===
execution:
  mode: "mt5"        # "mt5" | "tradingview" | "both"

# === MT5 ===
mt5:
  server: "ICMarkets-Demo"
  login: 12345678
  password: "your_password"
  enable: true

# === TRADINGVIEW ===
tradingview:
  webhook_host: "0.0.0.0"
  webhook_port: 5000
  secret_token: "your_webhook_secret"
  enable: false

# === RISK MANAGEMENT ===
risk:
  default_lot_size: 0.01
  risk_percent: 2.0
  max_daily_loss: 500.0
  max_consecutive_losses: 3
  session_start: "00:00"
  session_end: "23:59"

# === REPORTING ===
reporting:
  daily_report_time: "23:30"
  weekly_report_day: "sunday"
  weekly_report_time: "21:00"
  send_to_chat_id: "YOUR_CHAT_ID"
  enable_daily: true
  enable_weekly: true
```

### 4.3 Signal Parser Keywords ပြင်ဆင်ခြင်း

**`data/keywords.json`** — သင့် Signal Channel ရဲ့ စာသားပုံစံနဲ့ ကိုက်ညီအောင် ပြင်ဆင်ပါ။

```json
{
  "ExceptionKeywords": "report,result,summary",
  "EntryPointKeywords": "at,price,entry,@,now,zone",
  "StopLossKeywords": "stop loss,stoploss,sl,sl@,s/l",
  "TakeProfitKeywords": "take profit,takeprofit,tp,tp@,t/p,target",
  "BuyKeywords": "buy,long",
  "SellKeywords": "sell,short",
  "BuyLimitKeywords": "buy limit,buy-limit,buylimit",
  "SellLimitKeywords": "sell limit,sell-limit,selllimit"
}
```

### 4.4 Symbol Whitelist ပြင်ဆင်ခြင်း

**`data/symbols.txt** — ကုန်သွယ်လိုသော Symbol များ တစ်ကြောင်းချင်းထည့်ပါ။

```
XAUUSD
XAGUSD
EURUSD
GBPUSD
USDJPY
AUDUSD
USDCAD
BTCUSD
ETHUSD
USOIL
```

---

## 5. စနစ်စတင်ခြင်း (Startup)

### 5.1 ပထမဆုံးအကြိမ် စတင်ခြင်း

```bash
# Virtual Environment ဖွင့်ရန်
cd /d/Forex/AI Vibe Codeing for FX/Telegram Signal Copier from MT5 or TradingView PineScript
source venv/Scripts/activate

# စနစ်စတင်ရန်
python src/main.py
```

**ပထမဆုံးအကြိမ် Run သောအခါ:**
1. Telethon က Telegram သို့ Login ဝင်ရန် ဖုန်းနံပါတ်တောင်းလိမ့်မည်
2. Telegram မှ Code ရိုက်ထည့်ရန် လိုအပ်လိမ့်မည်
3. 2FA ဖွင့်ထားလျှင် Password ထည့်ရန် လိုအပ်လိမ့်မည်
4. အောင်မြင်လျှင် Session File (`signal_copier.session`) သိမ်းဆည်းပေးမည်
   - နောက်တစ်ခါမှ Code ပြန်မထည့်တော့ပါ

### 5.2 ပုံမှန်စတင်ခြင်း

```bash
# Shortcut Script (Windows)
@echo off
cd /d D:\Forex\AI Vibe Codeing for FX\Telegram Signal Copier from MT5 or TradingView PineScript
call venv\Scripts\activate.bat
python src\main.py
pause
```

### 5.3 Startup Checklist

စတင်ပြီးနောက် အောက်ပါတို့ကို စစ်ဆေးပါ:

- [ ] `logs/app.log` မှာ `"Telethon listener started"` ဆိုတဲ့ Message ရှိလား
- [ ] MT5 ကို ချိတ်ဆက်နိုင်လား (MT5 Terminal ဖွင့်ထားရန်)
- [ ] TradingView Webhook Server စတင်နိုင်လား
- [ ] Telegram Bot Response ပြန်နိုင်လား
- [ ] Database File (`data/signals.db`) ပေါ်လာလား

---

## 6. နေ့စဉ်လည်ပတ်ခြင်း (Daily Operations)

### 6.1 နံနက်ခင်း စစ်ဆေးရန် (Morning Checklist)

| # | လုပ်ဆောင်ချက် | ပြီးဆုံး |
|---|---------------|---------|
| 1 | MT5 Terminal ဖွင့်ထားကြောင်း စစ်ဆေးပါ | ☐ |
| 2 | AutoTrading ဖွင့်ထားကြောင်း စစ်ဆေးပါ (MT5: Algo Trading Button) | ☐ |
| 3 | System Run နေကြောင်း Log စစ်ဆေးပါ | ☐ |
| 4 | Telegram Channel များ Signal ပို့နေဆဲလား စစ်ဆေးပါ | ☐ |
| 5 | Internet Connection ကောင်းကြောင်း စစ်ဆေးပါ | ☐ |

### 6.2 ညနေခင်း စစ်ဆေးရန် (Evening Checklist)

| # | လုပ်ဆောင်ချက် | ပြီးဆုံး |
|---|---------------|---------|
| 1 | ယနေ့ Signal များ မှန်ကန်စွာဖမ်းယူနိုင်ခဲ့လား Log စစ်ဆေးပါ | ☐ |
| 2 | Order များ မှန်ကန်စွာတင်နိုင်ခဲ့လား MT5 Positions Tab စစ်ဆေးပါ | ☐ |
| 3 | Daily Report ရောက်လာလား Telegram စစ်ဆေးပါ | ☐ |
| 4 | Error Message တစ်စုံတစ်ရာရှိလား Log စစ်ဆေးပါ | ☐ |

### 6.3 System Status စစ်ဆေးခြင်း

**Telegram Bot မှတစ်ဆင့်:**
```
/status
```
Bot မှ ပြန်ပို့မည့် အချက်အလက်များ:
```
🟢 System Status
━━━━━━━━━━━━━━━
Telethon Listener: ✅ Connected (3 channels)
MT5 Gateway: ✅ Connected (Balance: $10,124.50)
TV Webhook: ⏸️ Disabled
Journal Engine: ✅ Running
DB Size: 2.3 MB
Uptime: 12h 34m
```

**Log File မှတစ်ဆင့်:**
```bash
# နောက်ဆုံး Log 20 ကြောင်းကြည့်ရန်
tail -20 logs/app.log

# ယနေ့ Error များကြည့်ရန်
grep ERROR logs/app.log
```

---

## 7. Signal ဖော်မတ်များ (Signal Formats)

### 7.1 ပံ့ပိုးထားသော Signal ပုံစံများ

**ပုံစံ 1: GOLD BUY/SELL (သင်ပေးထားသည့်ပုံစံ)**
```
GOLD BUY 4444
TP¹ 4447
TP² 4450
TP³ 4453
SL_4433
```

**ပုံစံ 2: Standard Forex**
```
BUY EURUSD 1.0850
TP: 1.0870
SL: 1.0830
```

**ပုံစံ 3: Emoji Format**
```
🔴 SELL XAUUSD @ 2030.50
✅ Entry: 2030.50
🎯 TP1: 2025.00
🎯 TP2: 2020.00
🛑 SL: 2035.50
💰 Lots: 0.10
```

**ပုံစံ 4: Detailed**
```
NEW SIGNAL
Pair: XAUUSD
Direction: BUY
Entry: 2000.00
Stop Loss: 1990.00
Take Profit 1: 2010.00
Take Profit 2: 2020.00
```

### 7.2 ပံ့ပိုးထားသော Order အမျိုးအစားများ

| Order Type | Telegram မှာရေးပုံ | MT5 သို့ |
|-----------|-------------------|----------|
| Market Buy | `BUY XAUUSD 2000` | Market Buy |
| Market Sell | `SELL EURUSD 1.08` | Market Sell |
| Buy Limit | `BUY LIMIT XAUUSD 1950` | Pending Buy Limit |
| Sell Limit | `SELL LIMIT EURUSD 1.09` | Pending Sell Limit |
| Buy Stop | `BUY STOP XAUUSD 2050` | Pending Buy Stop |
| Sell Stop | `SELL STOP EURUSD 1.07` | Pending Sell Stop |

### 7.3 Signal ပုံစံအသစ်ထည့်ရန်

Signal ပုံစံအသစ်ထည့်လိုပါက `src/processing/signal_parser.py` အတွင်းရှိ အောက်ပါ Function များကို ပြင်ဆင်ပါ:

```python
# signal_parser.py ထဲမှာ

# 1. Symbol အသစ်ထည့်ရန် — SYMBOL_MAP ထဲထည့်ပါ
SYMBOL_MAP = {
    "GOLD": "XAUUSD",
    "SILVER": "XAGUSD",
    # ဒီနေရာမှာ Symbol အသစ်ထည့်ပါ
    "NAS100": "NAS100",
    "US30": "US30",
}

# 2. ပုံစံအသစ်အတွက် Regex Pattern ထည့်ရန်
def _extract_prices(text, lines):
    # ဒီနေရာမှာ Pattern အသစ်ထည့်ပါ
    new_format = re.search(r'ENTRY:\s*(\d+\.?\d*)', text, re.IGNORECASE)
    if new_format:
        prices["entry_price"] = float(new_format.group(1))
```

---

## 8. Journal Report စနစ် (Reporting)

### 8.1 Report အချိန်ဇယား

| Report အမျိုးအစား | အချိန် | ပို့မည့်နေရာ |
|------------------|--------|-------------|
| နေ့စဉ် (Daily) | ည ၁၁:၃၀ | Telegram Chat |
| အပတ်စဉ် (Weekly) | တနင်္ဂနွေည ၉:၀၀ | Telegram Chat |

### 8.2 Report ပါဝင်မည့် အချက်အလက်များ

**Daily Report Example:**
```
📊 Daily Trading Journal — 2026-06-07
━━━━━━━━━━━━━━━━━━━━━━━━━━
📈 Total Trades: 5
✅ Won: 3 (60.0%)
❌ Lost: 2 (40.0%)
💰 PnL: +$124.50
📉 Max Drawdown: $45.00
⚡ Profit Factor: 1.85
💼 Balance: $10,124.50

📋 Trade Breakdown:
  BUY  XAUUSD  @4444  TP1✅ TP2✅ TP3❌
  SELL EURUSD  @1.085 TP1✅ SL❌
```

**Weekly Report Example:**
```
📈 Weekly Trading Journal — Jun 1-7, 2026
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📊 Total Trades: 23  |  Won: 14  |  Lost: 9
💰 Net PnL: +$567.30
📈 Win Rate: 60.9%
⚡ Profit Factor: 2.1
📈 Avg Win: $85.50  |  Avg Loss: $42.30
🔄 Best Day: Jun 3 (+$210)
🔴 Worst Day: Jun 5 (-$67)
💼 Ending Balance: $10,567.30
```

### 8.3 Report အချိန်ပြောင်းလိုပါက

`config.yaml` ထဲမှာ ပြောင်းပါ:

```yaml
reporting:
  daily_report_time: "23:30"       # နေ့စဉ် ည ၁၁:၃၀
  weekly_report_day: "sunday"      # တနင်္ဂနွေ
  weekly_report_time: "21:00"      # ည ၉ နာရီ
```

### 8.4 Report ပိတ်လိုပါက

```yaml
reporting:
  enable_daily: false
  enable_weekly: false
```

---

## 9. ပြဿနာဖြေရှင်းခြင်း (Troubleshooting)

### 9.1 အဖြစ်များသော Error များ

| Error | အကြောင်းရင်း | ဖြေရှင်းနည်း |
|-------|-------------|-------------|
| `MT5 init failed` | MT5 Terminal မဖွင့်ထားဘူး | MT5 ကိုဖွင့်ပါ |
| `No symbol detected` | Signal ဖော်မတ် မကိုက်ဘူး | keywords.json ကို ပြင်ပါ |
| `Order failed` | MT5 AutoTrading ပိတ်နေတယ် | Algo Trading Button နှိပ်ပါ |
| `Telegram flood wait` | Telegram API ကို မြန်လွန်းအောင်ခေါ်တယ် | ခဏစောင့်ပါ (auto-retry) |
| `Database locked` | တစ်ချိန်တည်း DB ကိုသုံးနေတယ် | WAL mode ဖွင့်ထားပါ (auto) |
| `Session expired` | Telethon Session သက်တမ်းကုန် | `.session` ဖိုင်ဖျက်ပြီး ပြန် Login ဝင်ပါ |

### 9.2 MT5 ပြဿနာများ

**လက္ခဏာ:** MT5 သို့ Order တင်လို့မရဘူး

**စစ်ဆေးရန်:**
1. MT5 Terminal ဖွင့်ထားလား → Window ရှိမရှိစစ်ပါ
2. AutoTrading ဖွင့်ထားလား → MT5 ရဲ့ Algo Trading Button (ညာဘက်အပေါ်)
3. Network Connection ရှိလား → MT5 တွင် Symbols စျေးတက်နေလား
4. Login ဝင်ထားလား → MT5 File → Login to Trade Account

**ဖြေရှင်းနည်း:**
```python
# MT5 Connection Test Script
import MetaTrader5 as mt5
if not mt5.initialize():
    print(f"MT5 init failed: {mt5.last_error()}")
else:
    print(f"MT5 version: {mt5.version()}")
    print(f"Terminal info: {mt5.terminal_info()}")
    print(f"Account info: {mt5.account_info()}")
    mt5.shutdown()
```

### 9.3 Telegram ပြဿနာများ

**လက္ခဏာ:** Telegram Channel မှ Signal မဖမ်းနိုင်ဘူး

**စစ်ဆေးရန်:**
1. `config.yaml` ထဲက Channel Link မှန်လား
2. Channel က Private လား → သင့် Telegram အကောင့် Member ဖြစ်ရန် လိုအပ်
3. API ID/Hash မှန်လား → [my.telegram.org](https://my.telegram.org) စစ်ပါ

**ဖြေရှင်းနည်း:**
```bash
# Session ဖျက်ပြီး ပြန်ဝင်ရန်
rm signal_copier.session
python src/main.py  # Code အသစ်ထည့်ရန် ပြန်တောင်းလိမ့်မည်
```

### 9.4 Signal Parse မှားနေလျှင်

**လက္ခဏာ:** Signal ဖမ်းမိပေမယ့် မှားယွင်းစွာ Parse လုပ်နေတယ်

**စစ်ဆေးရန်:**
```bash
# Log ကြည့်ရန်
grep "Parsed signal" logs/app.log

# DB ထဲက Raw Signal စစ်ရန်
sqlite3 data/signals.db "SELECT raw_text, parsed_json FROM signals ORDER BY id DESC LIMIT 5;"
```

**ဖြေရှင်းနည်း:**
1. `data/keywords.json` ကို သင့် Signal ဖော်မတ်နှင့် ကိုက်ညီအောင် ပြင်ပါ
2. `src/processing/signal_parser.py` ထဲက Regex Pattern များကို ပြင်ပါ
3. လိုအပ်လျှင် `_detect_symbol()`, `_detect_action()`, `_extract_prices()` တို့ကို ပြင်ဆင်ပါ

---

## 10. ဘေးကင်းရေး စည်းမျဉ်းများ (Safety Rules)

### 10.1 မဖြစ်မနေလိုက်နာရန်

| # | စည်းမျဉ်း | အရေးကြီးမှု |
|---|-----------|------------|
| 1 | **Demo Account ဖြင့် စမ်းသပ်ပါ** — ပထမဆုံး Demo နဲ့ အသေအချာစမ်းပါ | 🔴 မဖြစ်မနေ |
| 2 | **Risk Parameter များကို သေချာထားပါ** — Risk Percent 1-2% ထက်မပိုပါနဲ့ | 🔴 မဖြစ်မနေ |
| 3 | **Daily Loss Limit ထားပါ** — တစ်နေ့တာအတွင်း ဆုံးရှုံးနိုင်တဲ့ ပမာဏ သတ်မှတ်ပါ | 🔴 မဖြစ်မနေ |
| 4 | **Signal ကို ကိုယ်တိုင်စစ်ပါ** — စနစ်ကိုသာ လုံးဝအားမကိုးပါနဲ့ | 🟡 အရေးကြီး |
| 5 | **MT5 AutoTrading ကို စစ်ဆေးပါ** — မတော်တဆပိတ်မနေစေရန် | 🟡 အရေးကြီး |
| 6 | **Log File ကို နေ့စဉ်စစ်ပါ** — Error ရှိရင် ချက်ချင်းသိရန် | 🟢 သတိပြု |

### 10.2 Risk Management Settings (အကြံပြုချက်များ)

| Risk Level | Risk % Per Trade | Max Daily Loss | Max Consecutive Losses |
|-----------|-----------------|----------------|----------------------|
| 🟢 Conservative | 0.5% - 1.0% | 3% of Balance | 2 |
| 🟡 Moderate | 1.0% - 2.0% | 5% of Balance | 3 |
| 🔴 Aggressive | 2.0% - 3.0% | 8% of Balance | 5 |

**Lot Size Formula:**
```
Lot Size = (Balance × Risk%) / (SL_Pips × Pip_Value × 10)
```

**ဥပမာ:**
- Balance: $10,000
- Risk: 2% = $200
- SL: 20 pips
- Pip Value (XAUUSD): $10/lot
- Lot Size = $200 / (20 × $10 × 10) = **0.10 lots**

### 10.3 ရှောင်ရှားရမည့် အလေ့အထများ

| ရှောင်ရန် | အကြောင်းရင်း |
|----------|-------------|
| High Risk % (5%+) | Account ကုန်နိုင်သည် |
| Signal ကိုလုံးဝအားကိုးခြင်း | Signal Provider လည်း မှားနိုင်သည် |
| Risk Parameter မပါပဲ run ခြင်း | Unexpected Loss ဖြစ်နိုင်သည် |
| Log ကိုလစ်လျူရှုခြင်း | Error ကို နောက်ကျမှသိနိုင်သည် |
| MT5 ကိုပိတ်ထားခြင်း | Order တင်လို့မရတော့ပါ |

---

## 11. အရေးပေါ် လုပ်ထုံးလုပ်နည်းများ (Emergency Procedures)

### 11.1 စနစ်ချက်ချင်းရပ်ရန် (Emergency Stop)

**နည်းလမ်း 1: Keyboard**
```
Terminal Window ထဲမှာ → Ctrl + C နှိပ်ပါ
```

**နည်းလမ်း 2: Process Kill (Windows)**
```bash
# Python Process အားလုံးကိုရပ်ရန်
taskkill /F /IM python.exe
```

### 11.2 MT5 ကိုချက်ချင်းရပ်ရန်

**နည်းလမ်း 1: AutoTrading ပိတ်ရန်**
```
MT5 Terminal → Algo Trading Button နှိပ်ပါ (အစိမ်းရောင်မှ အနီရောင်သို့)
```

**နည်းလမ်း 2: Position အားလုံးပိတ်ရန်**
```
MT5 → Tools → Options → Expert Advisors → Uncheck "Allow Automated Trading"
```

### 11.3 အရေးပေါ် Scenario များ

**Scenario 1: အလွန်အကျွံအရှုံးပေါ်နေလျှင်**
```
1. Ctrl + C → System ရပ်ရန်
2. MT5 AutoTrading ပိတ်ရန်
3. Open Positions များကို ကိုယ်တိုင်စစ်ဆေးပါ
4. Risk Parameters ကို ပြန်လည်သုံးသပ်ပါ
```

**Scenario 2: Signal Provider မှားယွင်းနေလျှင်**
```
1. System ရပ်ရန် (Ctrl + C)
2. config.yaml ထဲက Channel List မှ ဖယ်ရှားပါ
3. Open Orders များကို ကိုယ်တိုင်စီမံပါ
4. System ပြန်စတင်ပါ
```

**Scenario 3: MT5 Disconnect ဖြစ်နေလျှင်**
```
1. MT5 Terminal ပြန်ဖွင့်ပါ
2. File → Login to Trade Account → ပြန် Login ဝင်ပါ
3. AutoTrading ဖွင့်ပါ
4. System က auto-reconnect လုပ်ပါလိမ့်မည်
```

**Scenario 4: Internet ပြတ်နေလျှင်**
```
1. Signal များကို Queue ထဲမှာ စောင့်ဆိုင်းနေပါလိမ့်မည်
2. Internet ပြန်ရသည်နှင့် အလိုအလျောက်ဆက်လက်လုပ်ဆောင်ပါမည်
3. သို့သော် Signal ဟောင်းများဖြစ်နေနိုင်ပါသည် — ကိုယ်တိုင်စစ်ဆေးပါ
```

---

## 12. ပြုပြင်ထိန်းသိမ်းခြင်း (Maintenance)

### 12.1 နေ့စဉ်ပြုလုပ်ရန်

| လုပ်ဆောင်ချက် | အချိန် |
|---------------|--------|
| Log File စစ်ဆေးပါ | ၅ မိနစ် |
| MT5 Terminal ဖွင့်ထားကြောင်း စစ်ဆေးပါ | ၁ မိနစ် |
| Daily Report ရောက်လာလား စစ်ဆေးပါ | ၁ မိနစ် |

### 12.2 အပတ်စဉ်ပြုလုပ်ရန်

| လုပ်ဆောင်ချက် | အချိန် |
|---------------|--------|
| Weekly Report ကိုသုံးသပ်ပါ | ၁၀ မိနစ် |
| Signal Provider Performance စစ်ဆေးပါ | ၅ မိနစ် |
| Log File ကို Cleanup လုပ်ပါ (၇ ရက်ထက်ဟောင်းသော) | ၂ မိနစ် |
| DB Size စစ်ဆေးပါ | ၁ မိနစ် |

### 12.3 လစဉ်ပြုလုပ်ရန်

| လုပ်ဆောင်ချက် | အချိန် |
|---------------|--------|
| Risk Parameters ကို ပြန်သုံးသပ်ပါ | ၁၅ မိနစ် |
| Full System Backup လုပ်ပါ | ၁၀ မိနစ် |
| Python Packages Update လုပ်ပါ | ၁၀ မိနစ် |
| Keyword Patterns ကို Update လုပ်ပါ | ၅ မိနစ် |

### 12.4 Backup လုပ်နည်း

```bash
# 1. Database Backup
cp data/signals.db data/backup/signals_$(date +%Y%m%d).db

# 2. Config Backup
cp config.yaml data/backup/config_$(date +%Y%m%d).yaml
cp .env data/backup/env_$(date +%Y%m%d).txt

# 3. Log Backup
cp -r logs data/backup/logs_$(date +%Y%m%d)

# 4. Session Backup
cp signal_copier.session data/backup/
```

### 12.5 Log File စီမံခန့်ခွဲခြင်း

Log File များသည် `logs/` ဖိုဒါတွင် သိမ်းဆည်းပြီး အောက်ပါအတိုင်း စီမံပါ:

```bash
# Log File အဟောင်းများကို ဖျက်ရန် (၇ ရက်ထက်ဟောင်းသော)
find logs/ -name "*.log" -mtime +7 -delete

# Log File ကို Compress လုပ်ရန်
gzip logs/app.log
mv logs/app.log.gz logs/archive/
```

### 12.6 Database Size စစ်ဆေးခြင်း

```bash
# SQLite Database Size ကြည့်ရန်
ls -lh data/signals.db

# DB Size ကန့်သတ်ရန် (Auto-vacuum ဖွင့်ရန်)
sqlite3 data/signals.db "PRAGMA auto_vacuum = 1; VACUUM;"
```

---

## နောက်ဆက်တွဲများ (Appendices)

### Appendix A: Quick Reference Card

**Start System:**
```bash
cd /d/Forex/... && source venv/Scripts/activate && python src/main.py
```

**Check Status:**
```bash
tail -20 logs/app.log | grep -E "ERROR|Connected|started"
```

**Emergency Stop:**
```
Ctrl + C  →  MT5 Algo Button → OFF
```

**View DB:**
```bash
sqlite3 data/signals.db "SELECT * FROM signals ORDER BY id DESC LIMIT 5;"
```

### Appendix B: File Paths Reference

| File | Path |
|------|------|
| Main Entry | `src/main.py` |
| Config | `config.yaml` |
| Environment | `.env` |
| Database | `data/signals.db` |
| Log | `logs/app.log` |
| Keywords | `data/keywords.json` |
| Symbols | `data/symbols.txt` |
| Telethon Session | `signal_copier.session` |

### Appendix C: Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2026-06-07 | Initial Release |

---

**END OF DOCUMENT**

---

*ဤ SOP သည် Telegram Signal Copier စနစ်ကို ဘေးကင်းလုံခြုံစွာ အသုံးပြုနိုင်ရန် လမ်းညွှန်ချက်ဖြစ်သည်။*
*စနစ်အသုံးမပြုမီ ဤစာရွက်စာတမ်းကို အပြည့်အစုံဖတ်ရှုပါ။*
*မေးခွန်းများရှိပါက AI Development Team ကို ဆက်သွယ်ပါ။*
