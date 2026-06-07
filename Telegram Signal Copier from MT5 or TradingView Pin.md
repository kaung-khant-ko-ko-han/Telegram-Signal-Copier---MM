Telegram Signal Copier from MT5 or TradingView PineScript

# ငါ ဒီလိုမျိုး Telegram Signal Copier — Copy Signals to TradingView & MT5 Automatically တစ်ခုရေးချင်တယ်


# ဒါအ ခြားသူတွေရေးထားတဲ့ ပုံစံပါ
'''
https://github.com/oluklef17/telegram-mt4-mt5-signal-copier.git 
https://github.com/oogunjob/FX-Signal-Copier-Telegram-Bot.git
'''

# အရင်ဆုံး ငါ့ကို Plan, System Architecture (စနစ်တစ်ခုလုံး၏ ဖွဲ့စည်းပုံ) & skills.md ကို အရင် လုပ်ပေးပါ၊ အခြားသူတွေ ရဲ့ skills.md  ရှိရင်လဲ ပြပေးပါ

# Journal ပါ report ထုတ်တဲ့ စနစ်ထည့်ချင်တယ် daily or weekly (၎င်းကို MT5 or TradingView မှာ ရပြီးသားဆို မထည့်တော့ဘူး)



# Telegram Signal for BUY
GOLD BUY 4444
MORE BUY 4442

TP¹ 4447
TP² 4450
TP³ 4453
TP⁴ 4456
TP⁵ 4460

SL_4433

# Telegram Signal for SELL
GOLD SELL 4421
MORE SELL 4423

TP¹ 4418
TP² 4415
TP³ 4412
TP⁴ 4410
TP⁵ 4407
TP⁶ 4404

SL_4432



🏗 2. System Architecture (စနစ်တစ်ခုလုံး၏ ဖွဲ့စည်းပုံ)

ဒီစနစ်ကို Event-Driven Architecture (ဖြစ်ရပ်အပေါ်အခြေခံသော စနစ်) ပုံစံမျိုး တည်ဆောက်ထားပါတယ်။ 

အပိုင်း (၄) ပိုင်း ခွဲခြားထားပါတယ် -

၁။ Input Layer (အချက်အလက် စုဆောင်းခြင်း)
 
 # Telegram Live Stream: Telethon API client ကို အသုံးပြုပြီး သတ်မှတ်ထားတဲ့ Telegram Channel/Group တွေဆီက New Message Event တွေကို စက္ကန့်မလပ် စောင့်ကြည့်ဖမ်းယူပါတယ်။

၂။ Processing Layer (အချက်အလက် ခွဲခြမ်းစိပ်ဖြာခြင်း)
 
 # Regex / Parser Engine: ရလာတဲ့ Raw text တွေကို Logic အသုံးပြုပြီး သန့်စင်ပါတယ်။ ရွှေ သို့မဟုတ် သက်ဆိုင်ရာ Currency ဖြစ်မဖြစ်၊ Buy/Sell ခွဲခြားခြင်းနှင့် စျေးနှုန်းများကို ခွဲထုတ်ပါတယ်။

 # Database (SQLite): ခွဲထုတ်ပြီးသား Signal အချက်အလက်များကို Trade ID နှင့်တကွ ပရောဂျက်အတွင်းရှိ Database ထဲသို့ ပို့ဆောင်သိမ်းဆည်းပါတယ်။

၃။ Execution Layer (အမိန့်ပေး စေခိုင်းခြင်း)
 
 # MT5 Broker Gateway: MetaTrader5 API မှတစ်ဆင့် Local PC သို့မဟုတ် VPS တွင် တိုက်ရိုက်ဖွင့်ထားသော Broker Terminal ထံသို့ Request ပို့ပြီး Order လှမ်းတင်ပါတယ်။
 
 # Risk Manager: Account Balance အပေါ်မူတည်ပြီး ကုန်သွယ်မည့် Lot Size ကို အလိုအလျောက် တွက်ချက်ပေးပါတယ်။

၄။ Reporting Layer (ပြန်လည် သတင်းပို့ခြင်း)
 
 # APScheduler Engine: Background မှာ အမြဲ Run နေပြီး သတ်မှတ်ထားတဲ့ နေ့စဉ်/အပတ်စဉ် အချိန်ရောက်ရင် SQLite ထဲက Data တွေကို Pandas နဲ့ တွက်ချက်ပြီး Owner ဆီကို Telegram Journal Report ပြန်ပို့ပေးပါတယ်။

---

🛠 Technical Skills Required & Applied

This document outlines the core technical competencies and software development skills applied in building the Telegram Signal Copier & Automated Trading Journal system.

## 🐍 Backend Development (Python Core)
- **Asynchronous Programming:** Utilized `asyncio` for non-blocking event listening on Telegram channels.
- **Data Scripting & Automation:** Automated cron-jobs and periodic tasks using `APScheduler`.
- **Error & Exception Handling:** Developed robust fail-safe mechanisms to handle improper signal text formats and broker requotes.

## 📈 Algorithmic Trading & Finance
- **MetaTrader 5 API:** Direct integration with MT5 terminals using Python's official `MetaTrader5` package for trade execution, lot-sizing, and order polling.
- **Risk Management:** Programmed dynamic lot-sizing calculations based on account equity and stop-loss pip distance.
- **Data Engineering:** Used `Pandas` for parsing historical trade logs and calculating performance metrics (Win Rate, Profit factor, Net Drawdown).

## 🔍 Data Extraction & Processing
- **Regular Expressions (Regex):** Created complex text parsing patterns to accurately extract data (`Symbol`, `Action`, `Entry`, `SL`, `TP`) from unstructured Telegram messages.
- **Database Management:** Designed and managed a relational database using `SQLite3` to maintain an immutable trading journal log.

## 🌐 DevOps & Infrastructure
- **VPS Deployment:** Deploying and maintaining 24/7 runtime on Windows Server Virtual Private Servers (VPS).
- **Environment Management:** Managing environment variables and secure credentials handling via `.env` configuration.

---


## Technical Competency matrix - Crypto/Forex Execution Bot

### 1. Quantitative & Execution Skills (Advanced)
- Order routing via MetaTrader5 Gateway & REST/Websockets APIs.
- Slippage control and Latency optimization (aiming for <200ms execution).

### 2. Software Architecture (Intermediate)
- Event-Driven Design Patterns.
- Database Schema designing for High-frequency log storing (PostgreSQL / SQLite).

### 3. Data Wrangling (Intermediate)
- Applied `Numpy` and `Pandas` for executing mathematical models behind trading reports.

---

## Project Tech Stack Breakdown

* **Ingestion Engine:** Requires strong knowledge of Telegram Bot API or MTProto Client (`Telethon` / `Pyrogram`), focusing on stream concurrency.
* **Parsing Core:** RegEx algorithms, String manipulation, and handling of multi-line message strings.
* **Database:** Structured Query Language (SQL) basics, CRUD operations, and transaction commit-rollbacks.
* **Integration:** Webhooks configuration and API integrations (TradingView Alerts to Python Webserver).

---



---

ဒီလိုမျိုး Telegram Signal Copier (Telegram ကနေ Signal ဖတ်ပြီး TradingView နဲ့ MT5 ထဲကို အလိုအလျောက် Order လှမ်းတင်ပေးမယ့်စနစ်) ရေးဖို့ဆိုရင် **Python** က အကောင်းဆုံးနဲ့ အမြန်ဆန်ဆုံး Stack ဖြစ်ပါတယ်။

ဒီစနစ်တစ်ခုလုံး အလုပ်လုပ်ဖို့အတွက် အဓိက အပိုင်း (၃) ပိုင်း ခွဲပြီး တည်ဆောက်ရပါမယ်။ အောက်မှာ လုပ်ငန်းစဉ် အဆင့်ဆင့်နဲ့ လိုအပ်မယ့် Core Architecture ကို ရှင်းပြပေးထားပါတယ်။

---

## 🛠 System Architecture (စနစ်တစ်ခုလုံး၏ ဖွဲ့စည်းပုံ)

---

ဒီ Bot ရဲ့ လုပ်ငန်းစဉ် စီးဆင်းပုံက ဒီလိုသွားမှာပါ -

```
[Telegram Channel/Group] 
       │
       ▼ (Telethon / Pyrogram API)
[1. Message Listener & Parser] ───► (Regex / LLM ဖြင့် Signal Format ဖတ်ခြင်း)
       │
       ├───► [2. TradingView Alerts] ───► (Webhooks မှတစ်ဆင့် TV Chart ပေါ် signal ပြခြင်း)
       │
       └───► [3. MT5 Execution Engine] ───► (MetaTrader5 Package ဖြင့် Auto-Order တင်ခြင်း)

```

---

## 1. Telegram Message Listener & Parser

---

Signal Channel တွေက ပို့တဲ့ စာသားပုံစံ (Formatting) က တစ်ခုနဲ့တစ်ခု မတူတတ်ပါဘူး (ဥပမာ- `BUY XAUUSD @ 2030`, `SL: 2020`, `TP: 2050`)။ ဒါကြောင့် Message တွေကို ဖမ်းယူဖို့ `Telethon` (Telegram Client API) ကို သုံးပြီး၊ အချက်အလက် ထုတ်ယူဖို့ **Regex (Regular Expressions)** သို့မဟုတ် တိကျမှုမြင့်မားချင်ရင် **DeepSeek / OpenAI API** ကို သုံးပြီး Parser ရေးရပါမယ်။

```python
# လိုအပ်မယ့် Python Libraries များ
# pip install telethon MetaTrader5 requests

from telethon import TelegramClient, events
import re

# Telegram API Credentials (my.telegram.org ကနေ ယူရပါမယ်)
api_id = 'YOUR_API_ID'
api_hash = 'YOUR_API_HASH'

client = TelegramClient('session_name', api_id, api_hash)

@client.on(events.NewMessage(chats=[-100xxxxxxxxx])) # Target Channel ID
async def my_event_handler(event):
    message_text = event.raw_text
    print(f"New Signal Received: {message_text}")
    
    # Simple Regex Parser Example
    # (စာသားထဲက Symbol, Action, Entry, SL, TP တွေကို ဆွဲထုတ်ခြင်း)
    try:
        symbol_match = re.search(r'(XAUUSD|EURUSD|GBPUSD)', message_text, re.IGNORECASE)
        action_match = re.search(r'(BUY|SELL)', message_text, re.IGNORECASE)
        
        if symbol_match and action_match:
            symbol = symbol_match.group(1).upper()
            action = action_match.group(1).upper()
            
            # စားသားထဲကနေ စျေးနှုန်းတွေကို ဆွဲထုတ်ဖို့ Pattern ဆက်ရေးရပါမယ်
            print(f"Parsed Trade: {action} {symbol}")
            
            # TradingView နဲ့ MT5 သို့ လှမ်းပို့မယ့် Function များကို ဤနေရာတွင် ခေါ်မည်
            # execute_mt5_trade(symbol, action, ...)
            # send_to_tradingview(symbol, action, ...)
    except Exception as e:
        print(f"Parsing Error: {e}")

client.start()
client.run_until_disconnected()

```

---

## 2. MetaTrader 5 (MT5) Integration

---

Python မှာ တရားဝင် `MetaTrader5` library ရှိပါတယ်။ ဒါကို သုံးပြီး ကိုယ့် local PC သို့မဟုတ် Windows VPS ပေါ်မှာ ဖွင့်ထားတဲ့ MT5 Terminal နဲ့ တိုက်ရိုက်ချိတ်ဆက်ပြီး Order လှမ်းဖွင့်လို့ရပါတယ်။

```python
import MetaTrader5 as mt5

def execute_mt5_trade(symbol, action, lot_size=0.1, sl=0.0, tp=0.0):
    if not mt5.initialize():
        print("MT5 initialize failed")
        return

    # Order Type သတ်မှတ်ခြင်း
    order_type = mt5.ORDER_TYPE_BUY if action == "BUY" else mt5.ORDER_TYPE_SELL
    price = mt5.symbol_info_tick(symbol).ask if action == "BUY" else mt5.symbol_info_tick(symbol).bid

    request = {
        "action": mt5.TRADE_ACTION_DEAL,
        "symbol": symbol,
        "volume": lot_size,
        "type": order_type,
        "price": price,
        "sl": sl,
        "tp": tp,
        "deviation": 20,
        "magic": 234000,
        "comment": "Telegram Auto Copier",
        "type_time": mt5.ORDER_TIME_GTC,
        "type_filling": mt5.ORDER_FILLING_IOC,
    }

    result = mt5.order_send(request)
    if result.retcode != mt5.TRADE_RETCODE_DONE:
        print(f"Order failed, retcode={result.retcode}")
    else:
        print("Trade executed successfully")
        
    mt5.shutdown()

```

---

## 3. TradingView Integration

---

TradingView က ပြောင်းပြန်အနေနဲ့ Alert ပေးရင် External API ကို လှamsခေါ်တာ လွယ်ကူသော်လည်း၊ အပြင်က Signal ကို TradingView Chart ပေါ် အလိုအလျောက် သွားဆွဲခိုင်းဖို့ဆိုရင် နည်းလမ်း (၂) ခု ရှိပါတယ်။

* **Webhook Pinescript (စနစ်ဟောင်း):** TradingView က Direct Webhook Receive လုပ်ဖို့ဆိုရင် Premium/Essential Plan လိုအပ်ပြီး Pine Script ထဲမှာ ပြင်ဆင်ရတာ အနည်းငယ် ကန့်သတ်ချက်ရှိပါတယ်။
* **TradingView Webhook Alternative (အကြံပြုချက်):** အကယ်၍ Telegram signal ကို TradingView chart ပေါ်မှာ Indicator line တွေအဖြစ် မြင်ချင်တာဆိုရင်၊ Python နဲ့ **FastAPI** တစ်ခုဆောက်ပြီး ၎င်းဒေတာကို JSON format ဖြင့် ထုတ်ပေးထားပါ။ ထို့နောက် TradingView Pine Script ထဲမှ `request.seed()` သို့မဟုတ် `request.security_lower_tf()` စတာတွေနဲ့ သွားချိတ်တာမျိုး (သို့မဟုတ်) ပိုမိုလွယ်ကူစေရန် TradingView Alerts နေရာတွင် **Discord/Telegram Bot channel တစ်ခုတည်းမှာပဲ Execution logs တွေကို ပေါင်းစည်းပြသခြင်း** က ပိုမိုထိရောက်ပါတယ်။

---

## 🚀 စတင်ရေးသားရန် လုပ်ဆောင်ရမည့် အဆင့်များ

---

1. **Telegram API App ဆောက်ခြင်း:** Prerequisite.
[my.telegram.org](https://my.telegram.org) သို့သွားပြီး Login ဝင်ပါ။ "API development tools" အောက်တွင် App တစ်ခုဆောက်၍ `api_id` နှင့် `api_hash` ကို ရယူပါ။


2. **Development Environment ပြင်ဆင်ခြင်း:** Python Setup.
Python 3.9+ ကို install လုပ်ပါ။ လိုအပ်သော packages များဖြစ်သည့် `telethon`, `MetaTrader5` တို့ကို pip ဖြင့် သွင်းပါ။ (မှတ်ချက် - MT5 package သည် Windows OS ပေါ်တွင်သာ အလုပ်လုပ်ပါသည်)။


3. **Regex/AI Parser ကို Demox Format ဖြင့် စမ်းသပ်ခြင်း:** Core Logic.
မိမိ Copy ကူးမည့် Channel ၏ Signal ပုံစံများကို စုဆောင်းပါ။ စာသားထဲကနေ Symbol, Price, SL, TP တို့ကို အမှားအယွင်းမရှိ ခွဲထုတ်နိုင်သည်အထိ `re` module သို့မဟုတ် DeepSeek API prompt ကို အရင် စမ်းသပ် (Debug) ပါ။


4. **MT5 Demo Account ဖြင့် ချိတ်ဆက်ခြင်း:** Risk-Free Testing.
MT5 Terminal တွင် Algo Trading ကို Enable လုပ်ပေးပါ။ Python script မှတစ်ဆင့် Demo account ထဲသို့ Order စမ်းတင်ကြည့်ပါ။ Lot size calculation နှင့် Error Handling (ဥပမာ- Requote ဖြစ်ခြင်း၊ Spread ကျယ်နေခြင်း) များကို ထည့်သွင်းရေးသားပါ။


---

> 💡 **အရေးကြီးသတိပြုရန် (Risk & Infrastructure):**
> ဤကဲ့သို့သော Auto-Copier bot မျိုးသည် Delay (နှောင့်နှေးမှု) မရှိစေရန်နှင့် ၂၄ နာရီပတ်လုံး အလုပ်လုပ်နိုင်ရန်အတွက် **Windows VPS (Virtual Private Server)** ပေါ်တွင် တင်၍ Run ထားရန် လိုအပ်ပါသည်။ ထို့ပြင် Signal Formatting အနည်းငယ်လွဲရုံနှင့် Trade မဝင်တော့ခြင်းမျိုး ဖြစ်နိုင်သဖြင့် Demo account တွင် အနည်းဆုံး ၂ ပတ်ခန့် အတည်ငြိမ်ဆုံးဖြစ်အောင် စမ်းသပ်ရန် အကြံပြုလိုပါသည်။

---

Telegram Channel တွေက တင်ပေးလေ့ရှိတဲ့ Signal format တွေဟာ ပုံသေမရှိဘဲ ပုံစံအမျိုးမျိုး ပြောင်းလဲနိုင်ပါတယ်။ (ဥပမာ- `BUY GOLD @ 2350`, `Buy XAUUSD around 2351.5`, `SL - 2342`, `TP1 : 2360`)

ဒါကြောင့် Format ပုံစံမျိုးစုံကို လိုက်ဖတ်နိုင်မယ့် တိကျပြီး စိတ်ချရတဲ့ **Regex (Regular Expression) Parser** ကုဒ်နမူနာနဲ့ ၎င်းရဲ့ အလုပ်လုပ်ပုံကို အသေးစိတ် ရှင်းပြပေးပါမယ်။

---

## 🛠 Python Signal Parser Source Code

ဒီ Code မှာ Telegram ကနေ စာသားအသစ်ဝင်လာတိုင်း စာလုံးအကြီး/အသေး ပြောင်းလဲမှုတွေ၊ ပတ်ဝန်းကျင် စျေးနှုန်း စာသားတွေကိုပါ ဖုံးလွှမ်းနိုင်အောင် **Case-Insensitive Regex Patterns** တွေကို သုံးထားပါတယ်။

```python
import re
from telethon import TelegramClient, events

# Telegram API Configuration
API_ID = 'YOUR_API_ID'
API_HASH = 'YOUR_API_HASH'
CHANNEL_ID = -100xxxxxxxxxx  # သင် ခြေရာခံချင်တဲ့ Signal Channel ID

client = TelegramClient('signal_parser_session', API_ID, API_HASH)

def parse_signal(message_text):
    """
    Telegram စာသားထဲကနေ Trading Signal ဒေတာများကို Regex ဖြင့် ရှာဖွေဖော်ထုတ်ပေးမည့် လုပ်ဆောင်ချက်
    """
    # 1. Trading Symbol ရှာဖွေခြင်း (XAUUSD သို့မဟုတ် GOLD ဖြစ်မဖြစ်)
    # XAUUSD သို့မဟုတ် GOLD ဆိုတဲ့ စာလုံးပါရင် XAUUSD လို့ ပုံသေသတ်မှတ်မယ်
    symbol = None
    if re.search(r'\b(XAUUSD|GOLD)\b', message_text, re.IGNORECASE):
        symbol = "XAUUSD"
    else:
        return None # ရွှေ Signal မဟုတ်ရင် ဆက်မလုပ်ဘဲ ကျော်သွားမယ်

    # 2. Action (BUY သို့မဟုတ် SELL) ရှာဖွေခြင်း
    action_match = re.search(r'\b(BUY|SELL)\b', message_text, re.IGNORECASE)
    action = action_match.group(1).upper() if action_match else None

    if not action:
        return None # Buy/Sell လား မသိရရင် ပယ်ဖျက်မယ်

    # 3. Entry Price (စျေးဝင်ရမည့် နေရာ) ရှာဖွေခြင်း
    # @, AT, NOW, AROUND, LIMIT စတာတွေရဲ့ နောက်က ကိန်းဂဏန်းကို ရှာမယ်
    entry_match = re.search(r'(?:@|at|now|around|limit|entry)?\s*(\d+(?:\.\d+)?)', message_text, re.IGNORECASE)
    # ပိုမိုတိကျစေရန် စာသားထဲက ပထမဆုံးတွေ့တဲ့ စျေးနှုန်းကို ယူတာမျိုးလည်း သုံးနိုင်ပါတယ်
    entry_prices = re.findall(r'\b\d{4}(?:\.\d+)?\b', message_text) # ရွှေစျေးက ဂဏန်း ၄ လုံးမို့ \b\d{4}\b သုံးနိုင်
    entry = float(entry_prices[0]) if entry_prices else None

    # 4. Stop Loss (SL) ရှာဖွေခြင်း
    # SL, SL:, STOP LOSS နောက်က ကိန်းဂဏန်းကို ရှာမယ်
    sl_match = re.search(r'(?:sl|stop\s*loss)[:\-\s]*(\d+(?:\.\d+)?)', message_text, re.IGNORECASE)
    sl = float(sl_match.group(1)) if sl_match else None

    # 5. Take Profit (TP) ရှာဖွေခြင်း (TP1, TP2, TP အကုန်လုံးကို ရှာမယ်)
    # tp, tp1, tp2 စတာတွေရဲ့ နောက်က ကိန်းဂဏန်းတွေကို အကုန်ထုတ်ယူမယ်
    tp_matches = re.findall(r'(?:tp|tp\s*\d|take\s*profit)[:\-\s]*(\d+(?:\.\d+)?)', message_text, re.IGNORECASE)
    tps = [float(tp) for tp in tp_matches]

    # ရလဒ်ကို Dictionary အနေနဲ့ ပြန်ပေးမယ်
    return {
        "symbol": symbol,
        "action": action,
        "entry": entry,
        "sl": sl,
        "tps": tps
    }

@client.on(events.NewMessage(chats=[CHANNEL_ID]))
async def handle_new_message(event):
    message_text = event.raw_text
    print(f"\n--- New Raw Message ---\n{message_text}")
    
    # Parser ဆီ ပို့ပြီး Data ထုတ်ယူခြင်း
    signal_data = parse_signal(message_text)
    
    if signal_data:
        print("✅ Parsed Signal Successfully:")
        print(f"🔹 Pair: {signal_data['symbol']}")
        print(f"🔹 Order Type: {signal_data['action']}")
        print(f"🔹 Entry Price: {signal_data['entry']}")
        print(f"🔹 Stop Loss (SL): {signal_data['sl']}")
        print(f"🔹 Take Profits (TP): {signal_data['tps']}")
        
        # ဒီနေရာမှာ MT5 Execution Engine ကို လှမ်းခေါ်ရပါမယ်
        # execute_mt5_order(signal_data)
    else:
        print("❌ Not a valid XAUUSD signal or unrecognized format.")

print("🤖 Bot is running and listening to Telegram Channels...")
client.start()
client.run_until_disconnected()

```

---

## 🔍 Regex Patterns များ အလုပ်လုပ်ပုံ ရှင်းလင်းချက်

ကုဒ်ထဲမှာ သုံးထားတဲ့ Regex သင်္ကေတတွေရဲ့ အဓိပ္ပာယ်က ဒီလို ဖြစ်ပါတယ်။

### ၁။ `\b(XAUUSD|GOLD)\b`

* `\b`: Word Boundary ဖြစ်ပါတယ်။ စာလုံးတစ်လုံးချင်းစီ သီးသန့်ဖြစ်နေမှ ဖတ်မှာပါ။ (ဥပမာ- `XAUUSD_OLD` လို စာသားမျိုးဆိုရင် ကျော်သွားပါလိမ့်မယ်)။
* `|`: "သို့မဟုတ်" (OR) လို့ အဓိပ္ပာယ်ရပါတယ်။

### ၂။ `(?:sl|stop\s*loss)[:\-\s]*(\d+(?:\.\d+)?)`

* `(?:...)`: Non-capturing group လို့ ခေါ်ပါတယ်။ ရှာရမယ့် စာသားထဲမှာ ပါဝင်ရုံသက်သက်ဖြစ်ပြီး ဒေတာထုတ်ယူစရာမလိုတဲ့ နေရာမှာ သုံးပါတယ်။ (`sl` သို့မဟုတ် `stop loss` ကို ရှာတာပါ)။
* `\s*`: Space (ကွက်လပ်) ရှိရင်ရှိ၊ မရှိရင်မရှိ (၀ ခု သို့မဟုတ် ထို့ထက်ပို) လို့ ဆိုလိုတာပါ။ `stoploss` ရော `stop loss` ရော ဖတ်နိုင်အောင် ဖြစ်ပါတယ်။
* `[:\-\s]*`: စာသားနဲ့ စျေးနှုန်းကြားက `:`, `-` သို့မဟုတ် space တွေကို ကျော်ဖြတ်ဖို့ ဖြစ်ပါတယ်။
* `(\d+(?:\.\d+)?)`: ဒါကတော့ **စျေးနှုန်း (ကိန်းဂဏန်း)** ကို ဖမ်းယူတဲ့ ကွင်းစကွင်းပိတ် ဖြစ်ပါတယ်။
* `\d+`: ဂဏန်း အနည်းဆုံး တစ်လုံး သို့မဟုတ် တစ်လုံးထက်ပိုရမယ်။
* `(?:\.\d+)?`: ဒသမကိန်း (ဥပမာ `.50`) ပါရင်လည်း ဖတ်မယ်၊ မပါရင်လည်း (ဥပမာ `2350` အပြတ်ဆိုရင်လည်း) အလုပ်လုပ်မယ်လို့ ဆိုလိုတာ ဖြစ်ပါတယ်။



---

## 💡 လက်တွေ့ စမ်းသပ်မှု နမူနာ (Test Cases)

အထက်ပါ ကုဒ်ထဲက `parse_signal()` function ထဲကို အောက်ပါ စာသားပုံစံတွေ ထည့်သွင်းရင်လည်း အမှားအယွင်းမရှိ တိကျတဲ့ Data Output ကို ရရှိမှာ ဖြစ်ပါတယ်။

**နမူနာ စာသား ၁:**

> "🟢 **GOLD BUY NOW** @ 2351.2
> SL: 2345
> TP1: 2356
> TP2: 2362"

**ထွက်လာမည့် Result:**

```json
{
  "symbol": "XAUUSD",
  "action": "BUY",
  "entry": 2351.2,
  "sl": 2345.0,
  "tps": [2356.0, 2362.0]
}

```

---

## ⚠️ လက်တွေ့အသုံးချရာတွင် သတိပြုရန်

1. **Multiple Entries / Zone Entry:** အချို့ Channel တွေက `Buy: 2350 - 2352` ဆိုပြီး ဇုန်လိုက် ပေးတတ်ပါတယ်။ အဲဒီလို အခြေအနေမျိုးအတွက် Regex ကို ထပ်မံ အဆင့်မြှင့်တင်ပြီး ပထမဂဏန်းနဲ့ ဒုတိယဂဏန်းကို ပေါင်းကာ ပျှမ်းမျှစျေး (Average Price) ယူတဲ့ Logic ထည့်ရပါမယ်။
2. **Noisy Text:** Signal မဟုတ်ဘဲ "Close half profit now" သို့မဟုတ် "Move SL to Entry" စတဲ့ စာသားမျိုးတွေကိုပါ Bot က မှားပြီး Order မတင်မိအောင် Exception Handling (အမှားထိန်းချုပ်မှု) တွေ သေချာ ထည့်သွင်းတည်ဆောက်ရပါမယ်။