# Telegram-Signal-Copier---MM

# Telegram Signal Copier — မြန်မာလို ရှင်းလင်းချက်

## ဒီ Project က ဘာလဲ?

ဒီ Project က **Telegram Channel/Group** တွေထဲမှာ တစ်စုံတစ်ယောက်က Trading Signal ပို့လိုက်ရင်၊ အဲဒီ Signal ကို အလိုအလျောက်ဖမ်းယူပြီး သင့် **MT5** (MetaTrader 5) အကောင့်ထဲကို ဒါမှမဟုတ် **TradingView** ကို **ကိုယ်တိုင်မလုပ်ပဲ အလိုအလျောက်** Order တင်ပေးတဲ့ စနစ်တစ်ခုပါ။

**ဥပမာ** — တစ်ယောက်ယောက်က Telegram မှာ "GOLD BUY 4444 / SL 4433 / TP 4447" လို့ Signal ပို့လိုက်ရင်၊ ဒီစနစ်က အလိုအလျောက်:
1. အဲဒီ Signal ကို ဖတ်တယ်
2. ဘယ် Symbol (GOLD = XAUUSD), ဘယ် Action (BUY), ဘယ်စျေး (4444), SL, TP တွေကို ခွဲထုတ်တယ်
3. သင့် MT5 အကောင့်ထဲကို Order အလိုအလျောက်တင်ပေးတယ်
4. နေ့တိုင်း ကုန်သွယ်မှုအစီရင်ခံစာ (Journal Report) ကို Telegram ကနေ ပြန်ပို့ပေးတယ်

---

## System Architecture (စနစ်ဖွဲ့စည်းပုံ) — 4 အလွှာ

```
အလွှာ 1: INPUT     → Telegram ကနေ Signal တွေဖမ်းယူခြင်း
အလွှာ 2: PROCESS   → Signal တွေကို ခွဲခြမ်းစိတ်ဖြာခြင်း + အန္တရာယ်စီမံခန့်ခွဲခြင်း
အလွှာ 3: EXECUTE   → MT5 (သို့) TradingView ကို Order တင်ခြင်း
အလွှာ 4: REPORT    → နေ့စဉ်/အပတ်စဉ် Journal Report ပို့ခြင်း
```

### အလွှာ ၁ — Input Layer (Signal ဖမ်းယူခြင်း)

Signal ရတဲ့ နည်းလမ်း ၃ မျိုး:

1. **Telethon Client** — Telegram Channel/Group တွေကို Real-time စောင့်ကြည့်တယ်။ Signal အသစ်တစ်ခုပေါ်လာတာနဲ့ ချက်ချင်းဖမ်းယူတယ်
2. **TradingView Webhook** — TradingView က PineScript Alert တွေကို Flask Server ကနေ လက်ခံတယ်
3. **Telegram Bot** — ကိုယ်တိုင် `/trade BUY XAUUSD 2000` လို့ ရိုက်ထည့်လို့ရတယ်

### အလွှာ ၂ — Processing Layer (ခွဲခြမ်းစိတ်ဖြာခြင်း)

Signal တစ်ခုရောက်လာရင်:

1. **Signal Parser** — Regex သုံးပြီး Signal ထဲက Symbol (XAUUSD, EURUSD...), Action (BUY/SELL), Entry Price, Stop Loss, Take Profit တွေကို ခွဲထုတ်တယ်
2. **Validator** — Signal ကို စစ်ဆေးတယ် (Duplicate မဖြစ်အောင်, စျေးနှုန်းမှန်မမှန်)
3. **Risk Manager** — သင့် Account Balance ပေါ်မူတည်ပြီး ဘယ်လောက် Lot Size နဲ့ ကုန်သွယ်ရမလဲ တွက်ပေးတယ်
4. **Database (SQLite)** — Signal နဲ့ Order အားလုံးကိ် DB မှာ သိမ်းတယ်

### အလွှာ ၃ — Execution Layer (Order တင်ခြင်း)

နည်းလမ်း ၂ မျိုး:

- **Path A: MT5 Python API** — MT5 Terminal ကို တိုက်ရိုက် Order တင်တယ် (Windows ပေါ်မှာပဲ အလုပ်လုပ်)
- **Path B: TradingView Webhook** — TradingView ကို Webhook ကနေ Signal ပို့တယ် (ပြီးရင် ကိုယ်တိုင် Manual Trade လုပ်)

### အလွှာ ၄ — Reporting Layer (Journal Report)

**ဒါက ထူးခြားချက်ပါ** — တခြား Reference တွေမှာ မပါဘူး။

- **နေ့စဉ် (Daily)** — ည ၁၁:၃၀ မှာ အလိုအလျောက် Report ပို့တယ်
- **အပတ်စဉ် (Weekly)** — တနင်္ဂနွေည ၉ နာရီမှာ Report ပို့တယ်
- **Report ထဲမှာပါတဲ့ အချက်အလက်များ:**
  - စုစုပေါင်း Trades အရေအတွက်
  - မြတ်တဲ့အရေအတွက် (Win Rate %)
  - အမြတ်/အရှုံးပမာဏ (PnL)
  - Profit Factor, Drawdown
  - Trade တစ်ခုချင်းစီရဲ့ အသေးစိတ်

---

## Project Structure (ဖိုင်တွေစီစဉ်ပုံ)

```
telegram-signal-copier/
├── src/
│   ├── main.py                     # အဓိက run ရန်
│   ├── config.py                   # Config ဖတ်ရန်
│   ├── input/                      # Signal ဖမ်းယူခြင်း
│   │   ├── telethon_listener.py    # Telegram နားထောင်ခြင်း
│   │   ├── tv_webhook.py           # TradingView Webhook
│   │   └── telegram_bot.py         # Bot Command များ
│   ├── processing/                 # Signal ခွဲခြမ်းစိတ်ဖြာခြင်း
│   │   ├── signal_parser.py        # Regex Parser
│   │   ├── signal_validator.py     # စစ်ဆေးခြင်း
│   │   ├── risk_manager.py         # Lot Size တွက်ခြင်း
│   │   └── database.py             # SQLite DB
│   ├── execution/                  # Order တင်ခြင်း
│   │   ├── mt5_gateway.py          # MT5 ချိတ်ဆက်ခြင်း
│   │   ├── mt5_executor.py         # MT5 Order များ
│   │   └── order_manager.py        # စီမံခန့်ခွဲခြင်း
│   └── reporting/                  # Journal Report
│       ├── journal_engine.py       # Report ထုတ်ခြင်း
│       ├── telegram_reporter.py    # Telegram ပို့ခြင်း
│       └── report_templates.py     # Report ပုံစံ
├── config.yaml                     # Settings အားလုံး
├── data/signals.db                 # SQLite Database
├── PLAN.md                         # အသေးစိတ် Plan
├── SKILL.md                        # Skill မှတ်တမ်း
└── architecture-telegram-signal-copier.html  # ပုံကားချပ်
```

---

## Reference တွေနဲ့ နှိုင်းယှဉ်ချက်

| Feature | EchoTrade | FX Bot | **ငါတို့ Project** |
|---------|-----------|--------|-------------------|
| MT5 ချိတ်ဆက်မှု | EA File ကနေ | MetaAPI Cloud | **တိုက်ရိုက် Python API** |
| TradingView | ❌ မပါ | ❌ မပါ | **✅ ပါတယ် (Webhook)** |
| Risk Management | ❌ မပါ | အခြေခံပါတယ် | **✅ အသေးစိတ်ပါတယ်** |
| Journal Report | ❌ မပါ | ❌ မပါ | **✅ နေ့စဉ်/အပတ်စဉ်** |
| Database | ❌ မပါ | ❌ မပါ | **✅ SQLite** |
| OCR (ပုံကနေဖတ်) | ✅ ပါတယ် | ❌ မပါ | **✅ ထည့်လို့ရ** |

---

## Journal Report System အကြောင်း

**မေးခွန်း:** MT5 နဲ့ TradingView မှာ Journal ရပြီးသားဆိုရင် ထည့်စရာမလိုဘူးလား?

**အဖြေ:** ထည့်ရမယ်။ အကြောင်းကတော့...

- **MT5** မှာ Trading Journal ရှိပေမယ့် Telegram ကို **အလိုအလျောက် Report ပြန်မပို့ပေးဘူး**
- **TradingView** မှာ လုံးဝ Journal System မရှိဘူး
- ဒါကြောင့် **ကိုယ်ပိုင် Journal System** ထည့်တာက သင့် Telegram ကို တစ်နေ့တာအကုန်မှာ အလိုအလျောက် Report ရောက်လာမှာပါ

**Daily Report ပုံစံ:**
```
📊 Daily Trading Journal — 2026-06-07
━━━━━━━━━━━━━━━━━━━━━━━━━━
📈 Total Trades: 5
✅ Won: 3 (60.0%)
❌ Lost: 2 (40.0%)
💰 PnL: +$124.50
📉 Max Drawdown: $45.00
⚡ Profit Factor: 1.85
📋 BUY XAUUSD @4444 TP1✅ TP2✅ TP3❌
```

---

## နောက်ထပ် Kimi Agent ကထည့်ထားတဲ့ အကြံကောင်းများ

Kimi Agent က `Kimi_Agent_Telegram/` ဖိုဒါထဲမှာ နောက်ထပ် Design တွေထည့်ထားတယ်။ အကောင်းဆုံးတွေကို `docs/UNIFIED_DESIGN.md` မှာ ပေါင်းထားတယ်။

**Kimi ဆီက ယူထားတဲ့ အကြံကောင်းများ:**
- **ZeroMQ** — MT5 နဲ့ ပိုပြီးခိုင်မာတဲ့ ချိတ်ဆက်မှု
- **Redis + Celery** — အလုပ်တွေကို နောက်ခံမှာ queue လုပ်စီးတယ်
- **PostgreSQL** — Production အတွက် DB အဆင့်မြင့်
- **PDF Report** — matplotlib နဲ့ ဂရပ်ဖစ်တွေပါတဲ့ Report
- **R-Multiple** — ကုန်သွယ်မှုအရည်အသွေးကို တိုင်းတာတဲ့ နည်းလမ်း
- **Docker** — တစ်နေရာထဲမှာ အဆင်သင့် run လို့ရ

**ဒါပေမယ့် အခုစပြီး ရေးဖို့ဆိုရင် `PLAN.md` ထဲက Code တွေက ချက်ချင်းသုံးလို့ရတယ်။** ပြီးမှ လိုအပ်ရင် ZeroMQ, Redis, PostgreSQL, Docker တွေကို ထည့်လို့ရတယ်။

---

## ဘယ်ကစမလဲ?

1. `PLAN.md` ကို ဖွင့်ကြည့်ပါ — Task တစ်ခုချင်းစီမှာ Code အပြည့်အစုံပါတယ်
2. **Phase 1** ကနေ စတင်ပါ — Project Structure ဆောက်ခြင်း, Config, Database
3. **Phase 2** — Telegram Listener နဲ့ Signal Parser
4. **Phase 3** — MT5 ချိတ်ဆက်ခြင်း
5. နောက်ဆုံး **Phase 6** — Journal Report System

အခုစပြီး Code တွေရေးဖို့ အဆင်သင့်ဖြစ်ပြီလား? ကျွန်တော် Phase 1 ကနေ စရေးပေးပါရစေ။
