# Root ဖိုင်များ - အသေးစိတ်ရှင်းလင်းချက်

---

## 1. `.gitignore`
**လမ်းကြောင်း:** `OTGateway-master/.gitignore`

**ရှင်းလင်းချက်:** Git version control က မည်သည့်ဖိုင်များကို လျစ်လျူရှုရမည်ကို သတ်မှတ်ပေးသောဖိုင်။

ဤဖိုင်တွင် အောက်ပါတို့ကို Git တွင် မသိမ်းရန် သတ်မှတ်ထားသည်:
- `.pio` - PlatformIO build output ဖိုင်များ
- `.vscode` - VS Code editor ဆက်တင်ဖိုင်များ
- `build/*` - Build output ဖိုင်များ
- `data/*` - Web UI data ဖိုင်များ
- `node_modules/*` - Node.js modules များ
- `secrets.ini` - လျှို့ဝှက်အချက်အလက်များ (WiFi password, MQTT password စသည်)
- `sdkconfig.*` - ESP32 SDK config ဖိုင်များ (`sdkconfig.defaults` မှလွဲ၍)

---

## 2. `platformio.ini`
**လမ်းကြောင်း:** `OTGateway-master/platformio.ini`

**ရှင်းလင်းချက်:** PlatformIO IDE အတွက် အဓိက configuration ဖိုင်။ ဤဖိုင်တွင် Firmware ကို မည်သို့ build လုပ်မည်၊ မည်သည့် board များအတွက် compile လုပ်မည်၊ မည်သည့် library များကို အသုံးပြုမည်ဆိုသည်တို့ကို သတ်မှတ်ထားသည်။

**အဓိက အပိုင်းများ:**
- `[platformio]` - PlatformIO ဆက်တင်များ (secrets.ini ကို အသုံးပြုရန်)
- `[env]` - ဘုံသုံး environment ဆက်တင်များ
  - Version: **1.5.5**
  - Framework: **Arduino**
  - အဓိက Library များ:
    - `ArduinoJson` - JSON parsing အတွက်
    - `OpenTherm Library` - OpenTherm protocol အတွက်
    - `ArduinoMqttClient` - MQTT ဆက်သွယ်မှုအတွက်
    - `ESP Telnet` - Telnet debugging အတွက်
    - `GyverPID` - PID controller အတွက်
    - `OneWireNg` (Dallas temp sensors) - Dallas အပူချိန်အာရုံခံများအတွက်
    - `NimBLE-Arduino` - Bluetooth Low Energy အတွက်
  - build_flags များတွင် WiFi SSID/Password, MQTT ဆက်တင်များ စသည့် default တန်ဖိုးများ ပါဝင်သည်

- **Board များ (စုစုပေါင်း 10 board):**
  - `[env:d1_mini]` - Wemos D1 Mini (ESP8266)
  - `[env:d1_mini_lite]` - Wemos D1 Mini Lite (ESP8266)
  - `[env:d1_mini_pro]` - Wemos D1 Mini Pro (ESP8266)
  - `[env:nodemcu_8266]` - NodeMCU (ESP8266)
  - `[env:s2_mini]` - Lolin S2 Mini (ESP32-S2)
  - `[env:s3_mini]` - Lolin S3 Mini (ESP32-S3) - BLE ပါဝင်
  - `[env:c3_mini]` - Lolin C3 Mini (ESP32-C3) - BLE ပါဝင်
  - `[env:nodemcu_32]` - NodeMCU-32S (ESP32) - BLE ပါဝင်
  - `[env:d1_mini32]` - Wemos D1 Mini32 (ESP32) - BLE ပါဝင်
  - `[env:esp32_c6]` - ESP32-C6 - BLE ပါဝင်
  - `[env:otthing]` - OTThing device အတွက် အထူး board

- Board တစ်ခုချင်းစီအတွက် GPIO Pin သတ်မှတ်ချက်များ:
  - `OT_IN_GPIO` - OpenTherm input (ဘွိုင်လာမှ data လက်ခံမည့် pin)
  - `OT_OUT_GPIO` - OpenTherm output (ဘွိုင်လာသို့ data ပို့မည့် pin)
  - `SENSOR_OUTDOOR_GPIO` - အပြင်ဘက် အပူချိန်အာရုံခံ pin
  - `SENSOR_INDOOR_GPIO` - အတွင်းဘက် အပူချိန်အာရုံခံ pin
  - `STATUS_LED_GPIO` - Status LED pin
  - `OT_RX_LED_GPIO` - OpenTherm Data LED pin

---

## 3. `package.json`
**လမ်းကြောင်း:** `OTGateway-master/package.json`

**ရှင်းလင်းချက်:** Node.js project configuration ဖိုင်။ Web UI (HTML, CSS, JS) ဖိုင်များကို minify လုပ်ရန် နှင့် gzip လုပ်ရန် Gulp.js tools များကို သတ်မှတ်ထားသည်။

**devDependencies:**
- `gulp` - Task runner
- `cssnano` - CSS minify လုပ်ရန်
- `gulp-html-minifier-terser` - HTML minify လုပ်ရန်
- `gulp-terser` - JavaScript minify လုပ်ရန်
- `gulp-gzip` - Gzip compression လုပ်ရန်
- `gulp-jsonminify` - JSON minify လုပ်ရန်
- `gulp-postcss` - PostCSS processing အတွက်

---

## 4. `gulpfile.js`
**လမ်းကြောင်း:** `OTGateway-master/gulpfile.js`

**ရှင်းလင်းချက်:** Gulp task runner အတွက် build script ။ Web UI ဖိုင်များ (`src_data/` မှ) ကို minify လုပ်ပြီး `data/` directory အောက်သို့ ထုတ်ပေးသည်။

**လုပ်ဆောင်ချက်များ:**
1. **styles** - CSS ဖိုင်များ (`pico.min.css`, `iconly.css`, `app.css`) ကို ပေါင်းစည်းပြီး minify လုပ်ကာ gzip ချုံ့သည် -> `data/static/app.css.gz`
2. **scripts** - JS ဖိုင်များ (`i18n.min.js`, `lang.js`, `utils.js`) ကို ပေါင်းစည်းပြီး minify လုပ်ကာ gzip ချုံ့သည် -> `data/static/app.js.gz`
3. **jsonFiles** - ဘာသာစကား JSON ဖိုင်များ (`locales/*.json`) ကို minify လုပ်ပြီး gzip ချုံ့သည်
4. **staticFiles** - Font နှင့် Image ဖိုင်များကို gzip ချုံ့သည်
5. **pages** - HTML ဖိုင်များကို minify + gzip ချုံ့သည်

ထူးခြားချက်: `{BUILD_TIME}` placeholder ကို current UNIX timestamp ဖြင့် အစားထိုးပေးသည်။ ၎င်းသည် browser cache ပြဿနာကို ကာကွယ်ပေးသည်။

---

## 5. `esp32_partitions.csv`
**လမ်းကြောင်း:** `OTGateway-master/esp32_partitions.csv`

**ရှင်းလင်းချက်:** ESP32 ၏ Flash Memory ကို မည်သို့ အပိုင်းခွဲမည်ကို သတ်မှတ်သောဖိုင်။

| Partition | Size | ရှင်းလင်းချက် |
|-----------|------|-------------|
| `nvs` (0x5000) | 20KB | WiFi နှင့် system settings များ သိမ်းရန် |
| `otadata` (0x2000) | 8KB | OTA update data |
| `app0` (0x1B0000) | 1.7MB | Firmware version 0 (ပထမ) |
| `app1` (0x1B0000) | 1.7MB | Firmware version 1 (ဒုတိယ - OTA အတွက်) |
| `spiffs` (0x80000) | 512KB | LittleFS file system (Web UI files) |
| `coredump` (0x10000) | 64KB | Crash dump data |

**မှတ်ချက်:** app0 နှင့် app1 သည် OTA update အတွက် dual-slot စနစ်ဖြစ်သည်။ တစ်ခု run နေစဉ် အခြားတစ်ခုသို့ update တင်နိုင်သည်။

---

## 6. `sdkconfig.defaults`
**လမ်းကြောင်း:** `OTGateway-master/sdkconfig.defaults`

**ရှင်းလင်းချက်:** ESP32 SDK (ESP-IDF) အတွက် default configuration ဖိုင်။

**အရေးကြီး ဆက်တင်များ:**
- `CONFIG_FREERTOS_HZ=1000` - FreeRTOS scheduler clock 1000Hz (တုံ့ပြန်မှုမြန်စေရန်)
- Flash Size: **4MB**
- Compiler Optimization: **Size** (မှတ်ဉာဏ်ချွေတာရန်)
- **Bluetooth LE ဆက်တင်များ:**
  - `CONFIG_BT_ENABLED=y` - Bluetooth ကို ဖွင့်ထား
  - `CONFIG_BT_NIMBLE_ENABLED=y` - NimBLE stack ကို အသုံးပြု
  - `CONFIG_BTDM_CTRL_MODE_BLE_ONLY=y` - BLE only mode
- Arduino Selective Compilation ဖြင့် မလိုအပ်သော modules များ (Zigbee, Matter, RainMaker, etc.) ကို ပိတ်ထား

---

## 7. `secrets.default.ini`
**လမ်းကြောင်း:** `OTGateway-master/secrets.default.ini`

**ရှင်းလင်းချက်:** Default လျှို့ဝှက်ဆက်တင်များ ဖိုင်။ အသုံးပြုသူက `secrets.ini` အဖြစ် ကူးယူ၍ ကိုယ်ပိုင်ဆက်တင်များ ထည့်ရသည်။

**ဆက်တင်များ:**
- `build_type = release` - Release mode တွင် build လုပ်ရန်
- `hostname = opentherm` - Network hostname
- `ap_ssid = OpenTherm Gateway` - AP mode SSID
- `ap_password = otgateway123456` - AP mode စကားဝှက်
- `portal_login = admin` / `portal_password = admin` - Web Portal login
- `mqtt_enabled = false` - MQTT မဖွင့်သေးပါ
- `log_level = 5` (VERBOSE) - Log အသေးစိတ် မှတ်တမ်းအဆင့်

---

## 8. `README.md`
**လမ်းကြောင်း:** `OTGateway-master/README.md`

**ရှင်းလင်းချက်:** GitHub repository ၏ အဓိက ဖော်ပြချက်ဖိုင်။ ပရောဂျက်၏ အင်္ဂါရပ်များ၊ Wiki လင့်ခ်များ၊ ကျေးဇူးတင်လွှာများ ပါဝင်သည်။

---

## 9. `LICENSE`
**လမ်းကြောင်း:** `OTGateway-master/LICENSE`

**ရှင်းလင်းချက်:** GNU General Public License v3.0 ဖိုင်ဖြစ်သည်။ ဤပရောဂျက်သည် open source ဖြစ်ပြီး GPL-3.0 license အောက်တွင် ဖြန့်ဝေသည်။

---
