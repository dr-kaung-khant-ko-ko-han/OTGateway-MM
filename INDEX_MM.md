# OpenTherm Gateway (OTGateway) - မြန်မာဘာသာ ရှင်းလင်းချက် မာတိကာ

## စီမံကိန်း အကျဉ်းချုပ်

**OTGateway** သည် ESP32/ESP8266 မိုက်ခရိုကွန်ထရိုလာပေါ်တွင် အခြေခံထားသော **OpenTherm ဂိတ်ဝေး** တစ်ခုဖြစ်သည်။ ၎င်းသည် သင့်အိမ်ရှိ ဘွိုင်လာ (Boiler) ကို အဝေးမှ ထိန်းချုပ်နိုင်ရန်၊ အပူချိန်များကို စောင့်ကြည့်နိုင်ရန်နှင့် Home Assistant ကဲ့သို့သော Smart Home စနစ်များနှင့် ချိတ်ဆက်နိုင်ရန် ပြုလုပ်ထားသော Open Source ပရောဂျက်ဖြစ်သည်။

### အဓိက အင်္ဂါရပ်များ
- DHW (ရေပူ) အပူချိန် ထိန်းချုပ်မှု
- အခန်းအပူပေးမှု (Heating) အပူချိန် ထိန်းချုပ်မှု
- PID နှင့် Equithermic curves အပူချိန်ထိန်းမုဒ်များ
- Dallas (1-wire), NTC 10k, Bluetooth (BLE) အာရုံခံကိရိယာများ ထည့်သွင်းနိုင်မှု
- Emergency mode (အန္တရာယ်ရှိသည့်အခါ ဘွိုင်လာကို ကာကွယ်မှု)
- Home Assistant နှင့် MQTT မှတစ်ဆင့် ချိတ်ဆက်နိုင်မှု
- Web Portal မှတစ်ဆင့် ထိန်းချုပ်နိုင်မှု
- OTA Firmware အဆင့်မြှင့်တင်နိုင်မှု

### ရှင်းလင်းချက်ဖိုင်များ

| အကြောင်းအရာ | ဖိုင်အမည် |
|------------|---------|
| **Root ဖိုင်များ** (config, build setup) | [00_ROOT_FILES.md](00_ROOT_FILES.md) |
| **src/ အဓိက Source ဖိုင်များ** | [01_SRC_MAIN.md](01_SRC_MAIN.md) |
| **src/ Task ဖိုင်များ** (Multitasking) | [02_SRC_TASKS.md](02_SRC_TASKS.md) |
| **lib/ စာကြည့်တိုက်များ** | [03_LIBRARIES.md](03_LIBRARIES.md) |
| **src_data/ Web UI ဖိုင်များ** | [04_WEB_UI.md](04_WEB_UI.md) |
| **tools/ Build Tools** | [05_TOOLS.md](05_TOOLS.md) |
| **assets/ Hardware ဖိုင်များ** | [06_ASSETS.md](06_ASSETS.md) |

---

**Version:** 1.5.5 | **Repository:** https://github.com/Laxilef/OTGateway
