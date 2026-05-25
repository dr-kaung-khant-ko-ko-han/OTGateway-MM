# src/ အဓိက Source ဖိုင်များ - အသေးစိတ်ရှင်းလင်းချက်

---

## 1. `main.cpp`
**လမ်းကြောင်း:** `OTGateway-master/src/main.cpp`

**ရှင်းလင်းချက်:** Firmware ၏ အဓိက entry point (Arduino `setup()` နှင့် `loop()`) ဖြစ်သည်။ ESP8266/ESP32 Scheduler library ကို အသုံးပြု၍ multitasking စနစ်ဖြင့် လုပ်ဆောင်သည်။

**လုပ်ဆောင်ချက်အဆင့်ဆင့် (Setup):**
1. **CrashRecorder** - Crash ဖြစ်ပါက မှတ်တမ်းတင်ရန် စတင်သတ်မှတ်ခြင်း
2. **Sensors** - Max sensors count (20) သတ်မှတ်ခြင်း
3. **LittleFS** - File system စတင်ခြင်း
4. **Log (TinyLogger)** - Logging စနစ် သတ်မှတ်ခြင်း (Serial သို့ output)
5. **Network Settings** - `/network.conf` ဖိုင်မှ network ဆက်တင်များ ဖတ်ခြင်း။ hostname မရှိပါက chip ID မှ auto-generate လုပ်ခြင်း
6. **Settings** - `/settings.conf` ဖိုင်မှ ဆက်တင်များ ဖတ်ခြင်း။ validation တန်ဖိုး မမှန်ကန်ပါက default သို့ ပြန်ပြောင်းခြင်း
7. **Sensors Settings** - `/sensors.conf` ဖိုင်မှ အာရုံခံဆက်တင်များ ဖတ်ခြင်း

**Multitasking Scheduler Setup:**
| Task | Enabled | Interval | လုပ်ဆောင်ချက် |
|------|---------|----------|--------------|
| `MqttTask` | No | 500ms | MQTT ဆက်သွယ်မှု၊ data publish/subscribe |
| `OpenThermTask` | Yes | 750ms | ဘွိုင်လာနှင့် OpenTherm ဆက်သွယ်မှု |
| `SensorsTask` | Yes | 1000ms | Dallas/NTC/BLE အာရုံခံများ ဖတ်ခြင်း |
| `RegulatorTask` | Yes | 10000ms | PID/Equitherm အပူချိန်ထိန်းညှိခြင်း |
| `PortalTask` | Yes | 0ms | Web server (event-driven) |
| `MainTask` | Yes | 100ms | ပင်မစောင့်ကြည့်မှု၊ Network loop၊ LED status |

**မှတ်ချက်:** ESP32 တွင် `loop()` က `vTaskDelete(NULL)` ဖြင့် Arduino loop task ကို ဖျက်ပစ်သည် (Scheduler ကို သာ အားကိုးရန်)။

---

## 2. `defines.h`
**လမ်းကြောင်း:** `OTGateway-master/src/defines.h`

**ရှင်းလင်းချက်:** ပရောဂျက်တစ်ခုလုံးတွင် အသုံးပြုသော constants နှင့် macro definitions များ သတ်မှတ်ထားသော header ဖိုင်။

**အရေးကြီး constant များ:**
- `PROJECT_NAME` = "OpenTherm Gateway"
- `MQTT_RECONNECT_INTERVAL` = 15000ms (MQTT ပြန်ဆက်သွယ်ရန် စောင့်ချိန်)
- `SETTINGS_VALID_VALUE` = "stvalid" (ဆက်တင်များ တရားဝင်ကြောင်း စစ်ဆေးရန် 8-character key)
- `GPIO_IS_NOT_CONFIGURED` = 0xFF (GPIO pin မသတ်မှတ်ထားကြောင်း ပြသည့် တန်ဖိုး)

**Default အပူချိန်များ:**
- Heating target: 40°C, min: 20°C, max: 90°C
- DHW target: 40°C, min: 30°C, max: 60°C
- Indoor thermostat: default 20°C, min 5°C, max 40°C

**NTC Sensor defaults:** Nominal resistance 10kΩ, Beta factor 3950, VREF 3300mV

**Build flags default များ** (platformio.ini သို့မဟုတ် command line မှ override လုပ်နိုင်):
- `DEFAULT_SERIAL_ENABLED`, `DEFAULT_SERIAL_BAUD`
- `DEFAULT_TELNET_ENABLED`, `DEFAULT_TELNET_PORT`
- `DEFAULT_HOSTNAME`, `DEFAULT_AP_SSID`, `DEFAULT_AP_PASSWORD`
- `DEFAULT_STA_SSID`, `DEFAULT_STA_PASSWORD`
- `DEFAULT_MQTT_ENABLED`, `DEFAULT_MQTT_SERVER`, `DEFAULT_MQTT_PORT`
- `DEFAULT_OT_IN_GPIO`, `DEFAULT_OT_OUT_GPIO`
- `SENSORS_AMOUNT` = 20

**UnitSystem enum:** METRIC (°C, bar, L/min) နှင့် IMPERIAL (°F, psi, gal/min)

---

## 3. `Settings.h`
**လမ်းကြောင်း:** `OTGateway-master/src/Settings.h`

**ရှင်းလင်းချက်:** ပရောဂျက်၏ ဆက်တင်အားလုံးကို struct များဖြင့် သတ်မှတ်ထားသော အဓိက header ဖိုင်။ နှစ်မျိုးခွဲထားသည်:

### (က) NetworkSettings struct
- **hostname** - Device hostname (25 chars)
- **useDhcp** - DHCP အသုံးပြုမှု
- **staticConfig** - Static IP, gateway, subnet, DNS
- **ap** - Access Point SSID, password, channel
- **sta** - Station (WiFi client) SSID, password, channel

### (ခ) Settings struct - ပင်မဆက်တင်များ
1. **system** - Log level, Serial/Telnet settings, NTP server/timezone, unit system, status LED pin
2. **portal** - Web portal auth, login, password, mDNS
3. **opentherm** - GPIO pins (in/out/rxLED), member ID, flags, power min/max, **အောက်ပါ options များ:**
   - `dhwSupport` - DHW support ရှိမရှိ
   - `coolingSupport` - Cooling support
   - `summerWinterMode` - နွေ/ဆောင်း မုဒ်
   - `ch2AlwaysEnabled` - Channel 2 အမြဲဖွင့်ထားရန်
   - `heatingToCh2` - Heating ကို CH2 သို့ ပို့ရန်
   - `dhwToCh2` - DHW ကို CH2 သို့ ပို့ရန်
   - `dhwBlocking` - DHW ပြုလုပ်နေစဉ် heating ရပ်ရန်
   - `maxTempSyncWithTargetTemp` - Max temp နှင့် target temp ကို တူညီစေရန်
   - `getMinMaxTemp` - ဘွိုင်လာမှ min/max temp များ ရယူရန်
   - `ignoreDiagState` - Diagnostic state လျစ်လျူရှုရန်
   - `autoFaultReset` - Fault အလိုအလျောက် reset လုပ်ရန်
   - `autoDiagReset` - Diagnostic အလိုအလျောက် reset လုပ်ရန်
   - `setDateAndTime` - ဘွိုင်လာသို့ ရက်စွဲ/အချိန် ပို့ရန်
   - `nativeHeatingControl` - ဘွိုင်လာ၏ native heating control ကို အသုံးပြုရန်
   - `immergasFix` - Immergas ဘွိုင်လာအတွက် အထူးပြင်ဆင်မှု

4. **mqtt** - Server, port, user, password, prefix, interval, HA discovery
5. **emergency** - Emergency mode တွင် အသုံးပြုမည့် target temp, threshold time
6. **heating** - Target temp, hysteresis, turbo, min/max temp, modulation, overheat protection, freeze protection
7. **dhw** - Target temp, min/max temp, modulation, overheat protection
8. **pid** - P/I/D factors, dt, min/max temp, deadband settings
9. **equitherm** - n_factor, k_factor, t_factor (အပူချိန်မျဉ်းကွေး သင်္ချာ constants)
10. **externalPump** - GPIO, post-circulation time, anti-stuck interval
11. **cascadeControl** - Input/output GPIO, threshold time, trigger conditions

### (ဂ) Sensors Settings Array
ကနဦး အာရုံခံ ၁၁ ခုကို အောက်ပါအတိုင်း သတ်မှတ်ထားသည်:
| Sensor | Purpose | Type | Enabled |
|--------|---------|------|---------|
| Outdoor temp | OUTDOOR_TEMP | OT_OUTDOOR_TEMP | No |
| Indoor temp | INDOOR_TEMP | DALLAS_TEMP | No |
| Heating temp | HEATING_TEMP | OT_HEATING_TEMP | Yes |
| Heating return temp | HEATING_RETURN_TEMP | OT_HEATING_RETURN_TEMP | Yes |
| Heating setpoint temp | TEMPERATURE | HEATING_SETPOINT_TEMP | Yes |
| DHW temp | DHW_TEMP | OT_DHW_TEMP | Yes |
| DHW flow rate | DHW_FLOW_RATE | OT_DHW_FLOW_RATE | Yes |
| Exhaust temp | EXHAUST_TEMP | OT_EXHAUST_TEMP | Yes |
| Pressure | PRESSURE | OT_PRESSURE | Yes |
| Modulation level | MODULATION_LEVEL | OT_MODULATION_LEVEL | Yes |
| Power | POWER | OT_CURRENT_POWER | Yes |

### (ဃ) Variables struct
`vars` အမည်ဖြင့် global variable တစ်ခုရှိပြီး ၎င်းတွင်:
- **master/slave** - OpenTherm master (Gateway) နှင့် slave (Boiler) အချက်အလက်များ
- **heating/dhw/ch2** - Heating, DHW, CH2 လက်ရှိအခြေအနေ
- **fault/diag** - Fault နှင့် diagnostic အခြေအနေ
- **modulation/power** - Modulation level နှင့် power
- **exhaust/fanSpeed/solar** - အိတ်ဇော၊ ပန်ကာ၊ ဆိုလာ အချက်အလက်များ
- **stats** - Burner starts, pump hours စသည့် စာရင်းဇယားများ
- **actions** - Restart, reset fault, reset diagnostic အမိန့်များ
- **states** - Restarting, upgrading လက်ရှိအခြေအနေ
- **network/mqtt/emergency/externalPump/cascadeControl** အခြေအနေများ

---

## 4. `strings.h`
**လမ်းကြောင်း:** `OTGateway-master/src/strings.h`

**ရှင်းလင်းချက်:** PROGMEM တွင် သိမ်းထားသော log labels နှင့် JSON key strings များ ပါဝင်သည်။ ၎င်းတို့သည် Flash memory တွင် သိမ်းထားပြီး RAM ကို ချွေတာသည်။

**အဓိက string အမျိုးအစားများ:**
- `L_` prefix - Log labels (e.g., `L_SETTINGS`, `L_MAIN`, `L_MQTT`, `L_OT`)
- `S_` prefix - JSON key strings (e.g., `S_ENABLED`, `S_TARGET`, `S_HEATING`)

စုစုပေါင်း strings ၂၀၀ ကျော် ပါဝင်သည်။

---

## 5. `utils.h`
**လမ်းကြောင်း:** `OTGateway-master/src/utils.h`

**ရှင်းလင်းချက်:** ပရောဂျက်တစ်ခုလုံးတွင် အသုံးပြုသော utility functions များ ပါဝင်သည်။

**အဓိက function များ:**

| Function | ရှင်းလင်းချက် |
|----------|-------------|
| `getChipId()` | ESP8266/ESP32 chip ID ကို hex string အဖြစ် ထုတ်ပေးသည် |
| `isLeapYear()` | ရက်ထပ်နှစ် စစ်ဆေးခြင်း |
| `mkgmtime()` | UTC tm struct မှ Unix epoch time သို့ ပြောင်းခြင်း |
| `isDigit()` | String က ဂဏန်းသီးသန့် ဟုတ်မဟုတ် စစ်ဆေးခြင်း |
| `liter2gallon()` / `gallon2liter()` | ထုထည်ယူနစ် ပြောင်းခြင်း |
| `bar2psi()` / `psi2bar()` | ဖိအားယူနစ် ပြောင်းခြင်း |
| `c2f()` / `f2c()` | အပူချိန်ယူနစ် ပြောင်းခြင်း (Celsius <-> Fahrenheit) |
| `convertTemp()` / `convertVolume()` / `convertPressure()` | UnitSystem အလိုက် ယူနစ်ပြောင်းခြင်း |
| `roundf()` | Float ကို decimal အလိုက် rounding လုပ်ခြင်း |
| `getTotalHeap()` / `getFreeHeap()` | Memory heap information ရယူခြင်း |
| `getMaxFreeBlockHeap()` / `getHeapFrag()` | Memory fragmentation စစ်ဆေးခြင်း |
| `getResetReason()` | ESP32/ESP8266 မှ restart ဖြစ်ရသည့် အကြောင်းရင်း ရယူခြင်း |
| `arr2str()` | Address array ကို string သို့ ပြောင်းခြင်း |
| `networkSettingsToJson()` | NetworkSettings struct ကို JSON သို့ ပြောင်းခြင်း |
| `jsonToNetworkSettings()` | JSON မှ NetworkSettings struct သို့ ပြောင်းခြင်း |
| `settingsToJson()` | Settings struct ကို JSON သို့ ပြောင်းခြင်း (ဆက်တင်အားလုံး) |
| `jsonToSettings()` | JSON မှ Settings struct သို့ ပြောင်းခြင်း (validation များပါဝင်) |
| `safeSettingsToJson()` | Safe mode ဖြင့် Settings ကို JSON သို့ ပြောင်းခြင်း (လျှို့ဝှက်အချက်များ မပါ) |

**မှတ်ချက်:** jsonToSettings() နှင့် jsonToNetworkSettings() တွင် valid value range စစ်ဆေးမှုများ၊ automatic unit conversion များ၊ interdependent field validation များ အသေးစိတ် ပါဝင်သည်။

---

## 6. `Sensors.h`
**လမ်းကြောင်း:** `OTGateway-master/src/Sensors.h`

**ရှင်းလင်းချက်:** အာရုံခံကိရိယာအားလုံးအတွက် static class တစ်ခုဖြစ်သည်။ အာရုံခံများ၏ data များကို စီမံခန့်ခွဲသည်။

**Sensor Types (အာရုံခံအမျိုးအစားများ):**
- **OpenTherm မှ တိုက်ရိုက်ရသော:** OT_OUTDOOR_TEMP, OT_HEATING_TEMP, OT_HEATING_RETURN_TEMP, OT_DHW_TEMP, OT_DHW_FLOW_RATE, OT_EXHAUST_TEMP, OT_PRESSURE, OT_MODULATION_LEVEL, OT_CURRENT_POWER, OT_SOLAR_STORAGE_TEMP, OT_SOLAR_COLLECTOR_TEMP, OT_FAN_SPEED, OT_BURNER_STARTS, OT_PUMP_HOURS (စုစုပေါင်း ၂၇ မျိုး)
- **Physical Sensors:** NTC_10K_TEMP, DALLAS_TEMP, BLUETOOTH
- **Virtual:** HEATING_SETPOINT_TEMP (PID/Equitherm output), MANUAL (MQTT/API မှ သတ်မှတ်သော)

**အဓိက static methods:**
- `setValueById()` / `setValueByType()` - အာရုံခံတန်ဖိုး သတ်မှတ်ခြင်း (offset/factor/filtering ပါဝင်)
- `getMeanValueByPurpose()` - ရည်ရွယ်ချက်တူ အာရုံခံများ၏ ပျမ်းမျှတန်ဖိုး ရယူခြင်း
- `existsConnectedSensorsByPurpose()` - သတ်မှတ်ရည်ရွယ်ချက်အတွက် ချိတ်ဆက်ထားသော အာရုံခံရှိမရှိ စစ်ခြင်း
- `cleanName()` / `makeObjectId()` - အာရုံခံအမည်များကို MQTT/HA-friendly format သို့ ပြောင်းခြင်း
- `bluetoothRssiToQuality()` - BLE RSSI (-110 to -50 dBm) ကို signal quality (0-100%) သို့ ပြောင်းခြင်း
- `setConnectionStatusById()` / `setConnectionStatusByType()` - အာရုံခံချိတ်ဆက်မှု အခြေအနေ သတ်မှတ်ခြင်း

**Filtering System:** အာရုံခံတန်ဖိုးများကို filtering factor ဖြင့် exponential moving average စစ်ထုတ်နိုင်သည်:
`new_value = old_value + (raw - old_value) * filtering_factor`

---

## 7. `CrashRecorder.h`
**လမ်းကြောင်း:** `OTGateway-master/src/CrashRecorder.h`

**ရှင်းလင်းချက်:** စက်ပျက်သွားသောအခါ (crash/panic) ၎င်း၏ backtrace, EPC (Exception Program Counter), heap size, core, uptime တို့ကို NOINIT memory တွင် မှတ်တမ်းတင်ထားသော crash handler ဖြစ်သည်။ Next boot တွင် `/api/debug` endpoint မှတစ်ဆင့် crash information များကို ပြန်လည်ကြည့်ရှုနိုင်သည်။

**ESP32:** `panicHandler()` function ကို `set_arduino_panic_handler()` ဖြင့် register လုပ်သည်
**ESP8266:** `custom_crash_callback()` function ကို crash callback အဖြစ် အသုံးပြုသည်

---

## 8. `idf_component.yml`
**လမ်းကြောင်း:** `OTGateway-master/src/idf_component.yml`

**ရှင်းလင်းချက်:** ESP-IDF component manager အတွက် dependency file ။
- IDF version: >= 5.3.2
- `h2zero/esp-nimble-cpp`: >= 2.2.1 (NimBLE Arduino library for ESP32)

---
