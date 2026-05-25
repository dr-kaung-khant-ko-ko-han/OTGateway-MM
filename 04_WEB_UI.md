# src_data/ Web UI ဖိုင်များ - အသေးစိတ်ရှင်းလင်းချက်

`src_data/` directory တွင် Gateway ၏ Web User Interface အတွက် HTML, CSS, JavaScript, font, image, နှင့် locale (ဘာသာစကား) ဖိုင်များ ပါဝင်သည်။ ၎င်းဖိုင်များကို `gulpfile.js` မှတစ်ဆင့် minify + gzip လုပ်ပြီး `data/` directory အောက်သို့ ထုတ်ပေးသည်။

---

## HTML Pages (`src_data/pages/`)

### 1. `index.html` (9,950 bytes)
**ရှင်းလင်းချက်:** Landing/welcome page ။ Gateway ၏ အခြေခံသတင်းအချက်အလက်များ ပြသသည်။
- Firmware version ပြသခြင်း
- WiFi network အခြေအနေ
- OpenTherm ဆက်သွယ်မှုအခြေအနေ

### 2. `dashboard.html` (30,239 bytes)
**ရှင်းလင်းချက်:** အဓိက Dashboard page ။ ဘွိုင်လာ၏ လက်ရှိအခြေအနေအားလုံးကို real-time ပြသသည်။
- **အပူချိန်များ:** Heating, DHW, Indoor, Outdoor, Exhaust, Return
- **အခြေအနေများ:** Heating on/off, DHW on/off, Flame, Fault, Emergency
- **Control များ:** Target temperature, mode switching
- **Charts/Graphs:** အပူချိန်ပြောင်းလဲမှု ဂရပ်များ
- **Alerts:** Fault code, diagnostic code ပြသခြင်း

### 3. `network.html` (8,330 bytes)
**ရှင်းလင်းချက်:** WiFi network configuration page ။
- WiFi networks scan လုပ်ပြီး စာရင်းပြသခြင်း
- STA mode settings (SSID, password) ထည့်သွင်းခြင်း
- AP mode settings
- Static IP/DHCP ရွေးချယ်ခြင်း
- Hostname ပြောင်းလဲခြင်း

### 4. `sensors.html` (14,930 bytes)
**ရှင်းလင်းချက်:** အာရုံခံကိရိယာများ စီမံခန့်ခွဲခြင်းစာမျက်နှာ။
- Sensor တစ်ခုချင်းစီ၏ ဆက်တင်များ (enable/disable, name, type, purpose, GPIO, address)
- Sensor data များ (temperature, humidity, battery, RSSI) ပြသခြင်း
- Sensor offset/factor/filtering ဆက်တင်များ
- Dallas sensor address configuration

### 5. `settings.html` (48,807 bytes)
**ရှင်းလင်းချက်:** အကြီးဆုံး HTML page - စနစ်ဆက်တင်အားလုံး ပါဝင်သည်။
- **System:** Log level, Serial/Telnet, NTP, Unit system, Status LED
- **Portal:** Authentication, mDNS
- **OpenTherm:** GPIO pins, Member ID, Options (၁၇ ခု)
- **MQTT:** Server, port, credentials, prefix, HA discovery
- **Heating:** Target, hysteresis, turbo, min/max temp, modulation, overheat/freeze protection
- **DHW:** Target, min/max temp, modulation, overheat protection
- **PID:** Enable, P/I/D factors, DT, min/max temp, deadband
- **Equitherm:** Enable, N/K/T factors
- **Emergency:** Target temp, threshold time
- **External Pump:** GPIO, timings
- **Cascade Control:** Input/output GPIO, trigger conditions

### 6. `upgrade.html` (4,041 bytes)
**ရှင်းလင်းချက်:** OTA firmware upgrade page ။
- Firmware (.bin) file upload
- Filesystem image upload
- Upgrade progress bar
- Result display (success/failure)

---

## Stylesheets (`src_data/styles/`)

### 1. `pico.min.css` (82,194 bytes)
**ရှင်းလင်းချက်:** Pico CSS framework - minimalist CSS framework ။ Button, form, table, grid စသည့် UI components များအတွက် styling ပေးသည်။ Light/Dark mode အလိုအလျောက် support ပါဝင်သည်။

### 2. `iconly.css` (1,549 bytes)
**ရှင်းလင်းချက်:** Iconly icon font အတွက် CSS ။ Font-face declaration နှင့် icon classes များ သတ်မှတ်ထားသည်။

### 3. `app.css` (6,413 bytes)
**ရှင်းလင်းချက်:** OTGateway အတွက် သီးသန့် custom CSS styles ။ Dashboard layout, sensor cards, control panels, responsive design adjustments များ ပါဝင်သည်။ **`{BUILD_TIME}`** placeholder ကို အသုံးပြု၍ cache busting လုပ်သည်။

---

## JavaScript (`src_data/scripts/`)

### 1. `i18n.min.js` (2,961 bytes)
**ရှင်းလင်းချက်:** Internationalization (i18n) library - Lite version ။ ဘာသာစကားမျိုးစုံ ပံ့ပိုးရန် translation strings များကို စီမံခန့်ခွဲသည်။

### 2. `lang.js` (3,361 bytes)
**ရှင်းလင်းချက်:** Language module ။ Locale JSON ဖိုင်များကို load လုပ်ပြီး UI တွင် ဘာသာစကားပြောင်းလဲနိုင်ရန် လုပ်ဆောင်သည်။

### 3. `utils.js` (20,722 bytes)
**ရှင်းလင်းချက်:** Web UI ၏ အဓိက JavaScript utility file ။ အောက်ပါတို့ ပါဝင်သည်:
- **API Calls** - `/api/*` endpoints များသို့ AJAX requests ပို့ခြင်း
- **Real-time Updates** - Dashboard data အလိုအလျောက် refresh လုပ်ခြင်း (polling)
- **Settings Management** - Settings form များ၏ data binding နှင့် validation
- **Sensor Configuration** - Sensor settings များကို UI တွင် edit လုပ်နိုင်ခြင်း
- **WiFi Scan** - Network scan result များကို ပြသခြင်း
- **Unit Conversion** - °C/°F, bar/psi conversion logic
- **Event Handling** - Button clicks, form submissions, toggle switches
- **Error Handling** - API error messages ပြသခြင်း

---

## Locales (`src_data/locales/`)

| ဖိုင် | Size | ရှင်းလင်းချက် |
|------|------|-------------|
| `en.json` | 18,814 bytes | English ဘာသာပြန်များ |
| `ru.json` | 27,424 bytes | Russian ဘာသာပြန်များ |
| `it.json` | 19,948 bytes | Italian ဘာသာပြန်များ |
| `cn.json` | 18,549 bytes | Chinese ဘာသာပြန်များ |

---

## Static Assets

### Fonts (`src_data/fonts/`)
Iconly icon font family - Web UI တွင်အသုံးပြုသော icon font:
| ဖိုင် | Size | Format |
|------|------|--------|
| `iconly.eot` | 4,656 B | Embedded OpenType |
| `iconly.svg` | 27,195 B | SVG font |
| `iconly.ttf` | 4,496 B | TrueType font |
| `iconly.woff` | 2,500 B | WOFF |
| `iconly.woff2` | 1,912 B | WOFF2 (အသေးဆုံး) |

### Images (`src_data/images/`)
- `favicon.ico` (15,086 bytes) - Browser tab icon

### Other (`src_data/`)
- `robots.txt` (25 bytes) - Search engine crawler များကို မဖတ်ရန် တားမြစ်ချက်
- `timezones.json` (16,391 bytes) - Timezone အမည်များ စာရင်း (UTC နှင့် country/city အမည်များ)

---

## Data Flow (Build Process)

```
src_data/pages/*.html     ─┐
src_data/styles/*.css     ─┤
src_data/scripts/*.js     ─┼─► gulpfile.js ─► data/ (minified + gzipped)
src_data/locales/*.json   ─┤
src_data/fonts/*.*        ─┤
src_data/images/*.*       ─┘
```

ESP32/ESP8266 ၏ LittleFS file system တွင် `data/` directory ကို flash လုပ်ပြီး Web server က serve လုပ်သည်။

---
