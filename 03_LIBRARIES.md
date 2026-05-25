# lib/ စာကြည့်တိုက်များ - အသေးစိတ်ရှင်းလင်းချက်

`lib/` directory တွင် ပရောဂျက်အတွက် အထူးရေးသားထားသော custom library များ ပါဝင်သည်။ ၎င်းတို့သည် PlatformIO ၏ dependency management အပြင်ဘက်မှ ဖြစ်သည်။

---

## 1. `CustomOpenTherm/CustomOpenTherm.h`
**လမ်းကြောင်း:** `OTGateway-master/lib/CustomOpenTherm/CustomOpenTherm.h`

**ရှင်းလင်းချက်:** OpenTherm protocol ကို စိတ်ကြိုက်ပြင်ဆင်ထားသော wrapper library ။ မူလ OpenTherm Library ကို အခြေခံပြီး ESP32 timer issues များကို ဖြေရှင်းထားသည်။

**ထပ်တိုးလုပ်ဆောင်ချက်များ:**
- **ESP32 timer fix** - ESP32 ၏ hardware timer ပြဿနာများကို ဖြေရှင်းထားသော fork (`Laxilef/opentherm_library#esp32_timer`)
- **After Send Request Callback** - Request ပို့ပြီးတိုင်း log မှတ်သားနိုင်သည့် callback
- **Delay Callback** - Scheduler-safe delay function
- **RX LED Support** - Data လက်ခံရရှိတိုင်း RX LED blink ပြုလုပ်ခြင်း
- OpenTherm message ID interpretation, response type checking, status checking

---

## 2. `MqttWriter/MqttWriter.h` & `MqttWiFiClient.h`
**လမ်းကြောင်း:** `OTGateway-master/lib/MqttWriter/`

**ရှင်းလင်းချက်:** Buffered MQTT publishing စနစ်။

### `MqttWriter.h`
- **Buffered Publishing** - 256-byte buffer ဖြင့် MQTT publish လုပ်ခြင်း
- **Large Data Support** - Buffer ထက်ကြီးသော data များကို အပိုင်းလိုက်ပို့နိုင်ခြင်း
- **Yield Callback** - ESP32 တွင် တစ်ပိုင်းပို့ပြီးတိုင်း yield လုပ်ရန် callback
- **Publish/Flush Events** - Publish ပြီးတိုင်း နှင့် flush တိုင်း callback ခေါ်ခြင်း

ဤသည်မှာ ESP8266 ၏ memory limitation ကြောင့် ကြီးမားသော JSON data များကို တစ်ခါတည်းမပို့နိုင်သော ပြဿနာကို ဖြေရှင်းရန် ဖြစ်သည်။

### `MqttWiFiClient.h`
- WiFiClient အတွက် MQTT-specialized wrapper
- ESP8266 တွင် sync mode နှင့် noDelay mode သုံးနိုင်ခြင်း

---

## 3. `NetworkUtils/NetworkConnection.h`, `NetworkConnection.cpp`, `NetworkMgr.h`
**လမ်းကြောင်း:** `OTGateway-master/lib/NetworkUtils/`

**ရှင်းလင်းချက်:** WiFi ဆက်သွယ်ရေးကို စီမံသည့် library ဖြစ်သည်။

### `NetworkMgr.h`
WiFi connection management အတွက် အဓိက class။

**လုပ်ဆောင်ချက်များ:**
- **Dual Mode** - Station (STA) နှင့် Access Point (AP) mode နှစ်မျိုးလုံး တစ်ပြိုင်တည်းလုပ်ဆောင်နိုင်ခြင်း
- **Auto-connect** - STA mode တွင် WiFi credentials ရှိပါက အလိုအလျောက် ချိတ်ဆက်ခြင်း
- **Fallback to AP** - STA မရပါက AP mode သို့ အလိုအလျောက်ပြောင်းခြင်း
- **Static IP Support** - DHCP သို့မဟုတ် Static IP သုံးနိုင်ခြင်း
- **RSSI to Signal Quality** - WiFi RSSI ကို 0-100% signal quality သို့ ပြောင်းခြင်း
- **Reconnect Logic** - ဆက်သွယ်မှုပြတ်တောက်ပါက ပြန်လည်ချိတ်ဆက်ခြင်း
- **Hostname Setting** - Network hostname သတ်မှတ်ခြင်း

### `NetworkConnection.h` / `NetworkConnection.cpp`
တစ်ခုချင်းစီသော WiFi connection (STA သို့မဟုတ် AP) ကို စီမံသည်။ Connection attempt, timeout, reconnect logic များ ပါဝင်သည်။

---

## 4. `WebServerHandlers/DynamicPage.h`, `StaticPage.h`, `UpgradeHandler.h`
**လမ်းကြောင်း:** `OTGateway-master/lib/WebServerHandlers/`

### `StaticPage.h`
Static file serving အတွက် web handler ။ File system မှ file ကို ဖတ်ပြီး cache headers (ETag) ဖြင့် serve လုပ်သည်။ Gzip compressed files များကို အလိုအလျောက် သိရှိနိုင်သည်။

### `DynamicPage.h`
Template-based dynamic page handler ။ HTML template ထဲမှ `%VAR%` placeholder များကို callback ဖြင့် အစားထိုးပေးသည်။ (လက်ရှိတွင် index page အတွက် disabled ထားသည်)

### `UpgradeHandler.h`
OTA firmware upgrade အတွက် multipart form upload handler ။
- **Firmware upgrade** - `.bin` file upload
- **File system upgrade** - LittleFS image upload
- **Progress tracking** - Upload progress callback
- **Error handling** - Upgrade မအောင်မြင်ပါက error အသေးစိတ်ပြခြင်း

---

## 5. `BufferedWebServer/BufferedWebServer.h`
**လမ်းကြောင်း:** `OTGateway-master/lib/BufferedWebServer/`

**ရှင်းလင်းချက်:** ESP32/ESP8266 WebServer အတွက် buffered response wrapper ။

**လုပ်ဆောင်ချက်များ:**
- **Chunked Transfer** - Response data ကို 32-byte chunks ဖြင့် ပို့ခြင်း (ESP8266 memory ချွေတာရန်)
- **JSON Response** - JsonDocument ကို တိုက်ရိုက် serialize လုပ်ပြီး ပို့နိုင်ခြင်း
- **TCP Buffer Issue Fix** - ESP32 တွင် ERR_CONNECTION_RESET (Chrome browser) ပြဿနာကို ဖြေရှင်းရန် `setNoDelay(true)`

---

## 6. `Equitherm/Equitherm.h`
**လမ်းကြောင်း:** `OTGateway-master/lib/Equitherm/`

**ရှင်းလင်းချက်:** Equithermic heating curve တွက်ချက်ရန် library ။

အပြင်ဘက်အပူချိန် (outdoor temp) နှင့် အတွင်းဘက်အပူချိန် (indoor temp) ပေါ်မူတည်၍ heating water temperature ကို တွက်ချက်သည်။

**သင်္ချာပုံစံ:**
```
HeatingTemp = Kn × (TargetTemp - OutdoorTemp) + Kk
              + Kt × (TargetTemp - IndoorTemp)
```
- **Kn (n_factor)** - Outdoor temperature curve factor
- **Kk (k_factor)** - Base offset
- **Kt (t_factor)** - Indoor temperature compensation factor

**Limits:** အနိမ့်ဆုံးနှင့် အမြင့်ဆုံး heating temp ကန့်သတ်ချက်များ သတ်မှတ်နိုင်သည်။

---

## 7. `HomeAssistantHelper/HomeAssistantHelper.h` & `strings.h`
**လမ်းကြောင်း:** `OTGateway-master/lib/HomeAssistantHelper/`

### `HomeAssistantHelper.h`
Home Assistant MQTT Auto-Discovery အတွက် base class ။

**အဓိက လုပ်ဆောင်ချက်များ:**
- **Device Info** - Device name, model, version, manufacturer, config URL
- **Topic Management** - Device prefix ဖြင့် MQTT topic များ ဖန်တီးခြင်း
- **Config Topic Generation** - `homeassistant/{type}/{device}/{object_id}/config` format
- **Cached Topics** - Status, state, settings topics များကို cache လုပ်ခြင်း
- **Availability** - Online/Offline status အတွက် availability topic

### `strings.h` (HomeAssistantHelper)
HA configuration တွင် အသုံးပြုသည့် constant strings များ (HA_DEVICE_CLASS, HA_UNIT_OF_MEASUREMENT, etc.)

---
