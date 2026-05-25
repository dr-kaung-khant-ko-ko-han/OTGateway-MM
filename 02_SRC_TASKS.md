# src/ Task ဖိုင်များ (Multitasking System) - အသေးစိတ်ရှင်းလင်းချက်

OTGateway သည် **ESP8266Scheduler/ESP32Scheduler** library ကို အသုံးပြု၍ cooperative multitasking စနစ်ဖြင့် လုပ်ဆောင်သည်။ Task တစ်ခုချင်းစီသည် သီးခြား class အဖြစ် ရေးသားထားပြီး `Task` သို့မဟုတ် `LeanTask` class မှ အမွေဆက်ခံထားသည်။

---

## 1. `MainTask.h`
**လမ်းကြောင်း:** `OTGateway-master/src/MainTask.h`

**ရှင်းလင်းချက်:** 100ms interval ဖြင့် run သော ပင်မ task။ ၎င်းသည် စနစ်တစ်ခုလုံး၏ "ဦးနှောက်" နှင့်တူသည်။

**အဓိက တာဝန်များ:**

| လုပ်ဆောင်ချက် | ရှင်းလင်းချက် |
|--------------|-------------|
| **network loop** | NetworkMgr ကို 100ms တိုင်း run စေခြင်း |
| **ဆက်တင်ဖိုင်များ စောင့်ကြည့်ခြင်း** | `fsNetworkSettings.tick()`, `fsSettings.tick()`, `fsSensorsSettings.tick()` တို့ဖြင့် ဆက်တင်ဖိုင်များ update ဖြစ်မဖြစ် စစ်ဆေးခြင်း |
| **Restart စီမံခြင်း** | Restart signal ရရှိပါက ၁၅ စက္ကန့်အတွင်း ဆက်တင်များ save လုပ်ပြီး restart ချခြင်း |
| **NTP (Network Time)** | Network ချိတ်ဆက်မိပါက NTP server မှ အချိန်ရယူခြင်း |
| **Telnet** | WiFi ချိတ်ဆက်မိပါက Telnet server စတင်ခြင်း |
| **MQTT Enable/Disable** | Network အခြေအနေနှင့် ဆက်တင်အရ MQTT task ကို ဖွင့်/ပိတ် ပြုလုပ်ခြင်း |
| **Misc (၁ စက္ကန့်တိုင်း):** | |
| **heating()** | Freeze protection - အပူချိန် 10°C အောက် ရောက်ပါက heating အလိုအလျောက်ဖွင့်ခြင်း |
| **emergency()** | Emergency mode ထိန်းချုပ်ခြင်း - အာရုံခံများ ပျက်သွားပါက emergency mode သို့ ပြောင်းခြင်း |
| **cascadeControl()** | Cascade input/output GPIO ထိန်းချုပ်ခြင်း (ဘွိုင်လာများစွာ ဆက်တွဲချိတ်ဆက်ရန်) |
| **externalPump()** | External pump control - heating on/off နှင့် anti-stuck logic |
| **ledStatus()** | Status LED မှတစ်ဆင့် စနစ်အခြေအနေ ပြသခြင်း (blink code စနစ်ဖြင့်) |
| **heap()** | Memory usage စောင့်ကြည့်ခြင်း၊ heap နည်းလွန်းပါက restart ချခြင်း |

**LED Blink Code:**
- 2 blinks = Network မရှိ
- 3 blinks = OpenTherm ဆက်သွယ်မှု ပြတ်တောက်
- 4 blinks = Boiler fault
- 5 blinks = Emergency mode

---

## 2. `MqttTask.h`
**လမ်းကြောင်း:** `OTGateway-master/src/MqttTask.h`

**ရှင်းလင်းချက်:** 500ms interval ဖြင့် run သော MQTT ဆက်သွယ်ရေး task ။ MQTT broker သို့ ချိတ်ဆက်ခြင်း၊ data publish/subscribe လုပ်ခြင်းတို့ကို တာဝန်ယူသည်။

**အဓိက လုပ်ဆောင်ချက်များ:**
| လုပ်ဆောင်ချက် | ရှင်းလင်းချက် |
|--------------|-------------|
| **MQTT Connect** | Broker သို့ ၁၅ စက္ကန့်တိုင်း reconnect ကြိုးစားခြင်း |
| **LWT (Last Will)** | "offline" message ဖြင့် LWT သတ်မှတ်ခြင်း |
| **Settings Subscribe** | `{prefix}/settings/set` နှင့် `{prefix}/state/set` topics များကို subscribe |
| **Sensor Subscribe** | Manual type အာရုံခံများအတွက် `sensors/{name}/set` topics များကို subscribe |
| **Publish Variables** | `settings.mqtt.interval` တိုင်း state data များ publish |
| **Publish Settings** | interval × 10 တိုင်း ဆက်တင်များ publish |
| **Publish Sensors** | အာရုံခံတစ်ခုချင်းစီ၏ data ကို activity အလိုက် publish |
| **Home Assistant Discovery** | MQTT Auto-Discovery ဖြင့် HA entities များ အလိုအလျောက် ဖန်တီးခြင်း |
| **Message Handler** | Incoming MQTT messages များကို JSON parsing လုပ်ပြီး settings/state/sensors များ update လုပ်ခြင်း |

**အသုံးပြုသော Helper များ:**
- `MqttWriter` - Buffered MQTT publishing (ကြီးမားသော data များကို အပိုင်းလိုက်ပို့ရန်)
- `HaHelper` - Home Assistant MQTT Discovery entities များ publish ရန်
- `MqttWiFiClient` - WiFi မှတစ်ဆင့် MQTT ဆက်သွယ်ရန်

**Publish Interval Logic:**
- Variables: interval စက္ကန့်တိုင်း (default 5s)
- Settings: interval × 10 စက္ကန့်တိုင်း (default 50s)
- Sensors: expire_after မတိုင်မီ update

---

## 3. `OpenThermTask.h`
**လမ်းကြောင်း:** `OTGateway-master/src/OpenThermTask.h`

**ရှင်းလင်းချက်:** 750ms interval ဖြင့် run သော OpenTherm ဆက်သွယ်ရေး task ။ ဘွိုင်လာနှင့် OpenTherm protocol အသုံးပြု၍ ဆက်သွယ်ခြင်း၊ data ဖတ်ခြင်း၊ command ပို့ခြင်းတို့ကို တာဝန်ယူသည်။

**အဓိက လုပ်ငန်းစဉ်များ:**

### (က) Boiler Status (setBoilerStatus)
- Heating enable/disable
- DHW enable/disable
- Cooling mode
- CH2 (Channel 2) control
- Summer/winter mode
- DHW blocking
- Immergas fix (အထူး status byte 0xCA)

### (ခ) Initialization
Boiler နှင့် ချိတ်ဆက်မိပါက:
1. Slave version ဖတ်ခြင်း
2. Master version သတ်မှတ်ခြင်း
3. Slave OT version ဖတ်ခြင်း
4. Master OT version (2.2f) သတ်မှတ်ခြင်း
5. Slave config (member ID, flags) ဖတ်ခြင်း
6. Master config သတ်မှတ်ခြင်း

### (ဂ) Data ဖတ်ခြင်း (တစ်မိနစ်တိုင်း)
- **Date/Time** သတ်မှတ်ခြင်း (နှစ်၊ လ/ရက်၊ အချိန်)
- **Min modulation level & max power** ဖတ်ခြင်း
- **DHW min/max temp** ဖတ်ခြင်း နှင့် settings auto-correction
- **Heating min/max temp** ဖတ်ခြင်း နှင့် settings auto-correction
- **Fault code** ဖတ်ခြင်း
- **Diagnostic code** ဖတ်ခြင်း
- **Burner starts / Pump starts** ဖတ်ခြင်း (heating & DHW)
- **Burner hours / Pump hours** ဖတ်ခြင်း (heating & DHW)
- **Auto fault/diag reset** (ဆက်တင်ရှိပါက)

### (ဃ) Real-time Data (loop တိုင်း)
- **Max modulation level** သတ်မှတ်ခြင်း
- **Modulation level & Power** ဖတ်ခြင်း
- **DHW temperature & flow rate** ဖတ်ခြင်း
- **Heating temperature & return temperature** ဖတ်ခြင်း
- **CH2 temperature** ဖတ်ခြင်း
- **Exhaust temperature, CO2, fan speed** ဖတ်ခြင်း
- **Heat exchanger temperature** ဖတ်ခြင်း
- **Outdoor temperature** ဖတ်ခြင်း
- **Solar storage/collector temperature** ဖတ်ခြင်း
- **Pressure** ဖတ်ခြင်း
- **Fan speed (setpoint & current)** ဖတ်ခြင်း

### (င) Temperature Control
- **DHW temp** သတ်မှတ်ခြင်း (ဆက်တင် target အရ)
- **Native Heating Control** - Indoor temp နှင့် room setpoint ကို boiler သို့တိုက်ရိုက်ပို့ခြင်း
- **Normal Heating Control** - Max heating temp နှင့် target temp သတ်မှတ်ခြင်း
- **CH2 temp** သတ်မှတ်ခြင်း

### (စ) Safety
- **Heating Overheat Protection** - အပူချိန် သတ်မှတ်ထားသည်ထက် ကျော်ပါက heating ရပ်ခြင်း
- **DHW Overheat Protection** - DHW အပူချိန် သတ်မှတ်ထားသည်ထက် ကျော်ပါက DHW ရပ်ခြင်း
- **Fault/Diag Reset** - Boiler fault နှင့် diagnostic reset command ပို့ခြင်း

**မှတ်ချက်:** OpenThermTask သည် **Core 1** တွင် run ပြီး **Priority 5** ဖြင့် အမြင့်ဆုံး priority ရှိသည် (အချိန်မှန် communication လိုအပ်သောကြောင့်)။

---

## 4. `PortalTask.h`
**လမ်းကြောင်း:** `OTGateway-master/src/PortalTask.h`

**ရှင်းလင်းချက်:** Event-driven web server task (LeanTask, interval=0) ။ Web UI ကို serve လုပ်ရန် HTTP server နှင့် captive portal DNS server တို့ကို စီမံသည်။

**Web Server Endpoints:**

### Static Pages
| Endpoint | ရှင်းလင်းချက် |
|----------|-------------|
| `/` | Index page (welcome/landing) |
| `/dashboard.html` | Dashboard (ဘွိုင်လာအခြေအနေ) |
| `/network.html` | Network ဆက်တင်များ |
| `/settings.html` | အပူချိန်ဆက်တင်များ |
| `/sensors.html` | အာရုံခံကိရိယာ ဆက်တင်များ |
| `/upgrade.html` | Firmware upgrade page |

### API Endpoints
| Endpoint | Method | ရှင်းလင်းချက် |
|----------|--------|-------------|
| `/api/network/settings` | GET/POST | Network ဆက်တင်များ ဖတ်/ရေးခြင်း |
| `/api/network/scan` | GET | WiFi networks scan လုပ်ခြင်း |
| `/api/settings` | GET/POST | စနစ်ဆက်တင်များ ဖတ်/ရေးခြင်း |
| `/api/sensors` | GET | အာရုံခံများ၏ လက်ရှိ data |
| `/api/sensor` | GET/POST | အာရုံခံတစ်ခုချင်း ဖတ်/ရေးခြင်း |
| `/api/vars` | GET/POST | Variables များ ဖတ်/ရေးခြင်း |
| `/api/info` | GET | စနစ်သတင်းအချက်အလက် (version, network, heap, chip) |
| `/api/debug` | GET | Debug info + crash info download |
| `/api/upgrade` | POST | OTA firmware/filesystem upgrade |
| `/api/backup/save` | GET | Settings backup (JSON download) |
| `/api/backup/restore` | POST | Settings restore (JSON upload) |
| `/restart.html` | GET | Device restart trigger |

### Captive Portal
WiFi AP mode တွင် ချိတ်ဆက်ထားသော စက်ပစ္စည်းများကို Gateway ၏ Web UI သို့ အလိုအလျောက် redirect လုပ်ပေးသည်။

### mDNS Support
`settings.portal.mdns` ဖွင့်ထားပါက `{hostname}.local` ဖြင့် ချိတ်ဆက်နိုင်သည်။

### Dynamic State Management
- **Web Server:** AP mode သို့မဟုတ် STA connected အခြေအနေတွင် အလိုအလျောက်ဖွင့်
- **DNS Server:** AP mode ဖြင့် STA not connected အခြေအနေတွင် အလိုအလျောက်ဖွင့်
- **Authentication:** STA mode နှင့် portal auth ဖွင့်ထားပါက Basic Auth စစ်ဆေး

---

## 5. `RegulatorTask.h`
**လမ်းကြောင်း:** `OTGateway-master/src/RegulatorTask.h`

**ရှင်းလင်းချက်:** ၁၀ စက္ကန့်တိုင်း run သော အပူချိန်ထိန်းညှိ task ။ PID controller နှင့် Equithermic curves များကို အသုံးပြု၍ heating setpoint temperature ကို တွက်ချက်ပေးသည်။

**အဓိက အပူချိန်ထိန်းမုဒ်များ:**

| မုဒ် | ရှင်းလင်းချက် |
|------|-------------|
| **Manual** | `settings.heating.target` ကို တိုက်ရိုက်အသုံးပြု |
| **Equitherm** | Outdoor temperature နှင့် indoor temperature ပေါ်မူတည်၍ heating curve တွက်ချက် |
| **PID** | Indoor temperature နှင့် target ကွာခြားချက်ပေါ် မူတည်၍ P/I/D control |
| **Native** | Boiler ၏ native room temperature control ကို အသုံးပြု |
| **Emergency** | Emergency mode တွင် `settings.emergency.target` ကို အသုံးပြု |

**အထူးအင်္ဂါရပ်များ:**
- **Turbo Mode** - Target နှင့် လက်ရှိ indoor temp ကွာခြားချက်ကို `turboFactor` ဖြင့် မြှင့်တင်ခြင်း
- **Hysteresis** - အပူချိန် သတ်မှတ်တန်ဖိုးသို့ ရောက်ပါက heating ခဏရပ်ခြင်း (အပူချိန် တည်ငြိမ်စေရန်)
- **Deadband (PID)** - Target နှင့် နီးကပ်နေချိန် PID factors များကို လျှော့ချခြင်း
- **Indoor/Outdoor အာရုံခံမရှိပါက** အလိုအလျောက် Emergency mode သို့ ပြောင်းခြင်း

**Temperature Calculation Flow:**
```
Final Setpoint Temp = Equitherm Result + PID Result + Turbo Boost
(Each component can be independently enabled/disabled)
```

---

## 6. `SensorsTask.h`
**လမ်းကြောင်း:** `OTGateway-master/src/SensorsTask.h`

**ရှင်းလင်းချက်:** ၁ စက္ကန့်တိုင်း run သော အာရုံခံဖတ်ခြင်း task (သို့သော် ၁၅ စက္ကန့်တိုင်းမှသာ polling လုပ်) ။ Dallas (1-wire), NTC 10k, Bluetooth BLE အာရုံခံများမှ data ဖတ်ခြင်းကို တာဝန်ယူသည်။

**အာရုံခံအမျိုးအစားအလိုက် လုပ်ဆောင်ချက်များ:**

### Dallas (1-Wire) Sensors
- GPIO တစ်ခုချင်းစီအတွက် OneWire နှင့် DallasTemperature instance များ ဖန်တီးခြင်း
- Bus ပေါ်ရှိ devices များကို ၆၀ စက္ကန့်တိုင်း search လုပ်ခြင်း
- Address မရှိသေးသော sensors များအတွက် address auto-assign လုပ်ခြင်း
- 12-bit resolution ဖြင့် temperature ဖတ်ခြင်း (၁၀ စက္ကန့်တိုင်း)
- Chinese clone DS18B20 များအတွက် အထူး conversion time calculation

### NTC 10K Sensors
- ESP32: `analogReadMilliVolts()` ဖြင့် တိကျသော voltage ဖတ်ခြင်း
- ESP8266: `analogRead()` ဖြင့် voltage တွက်ချက်ခြင်း
- Voltage threshold စစ်ဆေးခြင်း (sensor short/open သိရှိနိုင်ရန်)
- Resistance တွက်ချက်ပြီး Beta equation ဖြင့် temperature သို့ ပြောင်းလဲခြင်း

### Bluetooth (BLE) Sensors
- NimBLE Device initialization (power level 9)
- ENV Service (0x181A) မှ temperature notification subscribe:
  - 0x2A6E: Temperature x0.01°C (pvvx format)
  - 0x2A1F: Temperature x0.1°C (atc1441/pvvx format)
  - 0x2A6F: Humidity x0.01% (pvvx format)
- Battery Service (0x180F) မှ battery level notification subscribe:
  - 0x2A19: Battery 0-99%
- BLE sensor များသို့ date/time အခါအားလျော်စွာ set ပေးခြင်း
- RSSI ကို အခြေအနေ update တိုင်း ဖတ်ခြင်း

### Connection Status Management
- Sensor မှ data မရတော့ပါက ၂ မိနစ် (120000ms) ကြာလျှင် disconnected အဖြစ် သတ်မှတ်
- Disabled/not configured sensors များကို disconnected အဖြစ် အလိုအလျောက် သတ်မှတ်

---

## 7. `HaHelper.h`
**လမ်းကြောင်း:** `OTGateway-master/src/HaHelper.h`

**ရှင်းလင်းချက်:** Home Assistant MQTT Auto-Discovery အတွက် helper class ။ `HomeAssistantHelper` class မှ အမွေဆက်ခံပြီး OTGateway ၏ entities အားလုံးကို HA တွင် အလိုအလျောက် ပေါ်လာစေရန် publish လုပ်ပေးသည်။

**အဓိက Entity အမျိုးအစားများ:**

| Entity | HA Type | ရှင်းလင်းချက် |
|--------|---------|-------------|
| **Status** | Binary Sensor | Gateway Online/Offline |
| **Emergency** | Binary Sensor | Emergency mode active |
| **OT Status** | Binary Sensor | OpenTherm connected |
| **Heating** | Binary Sensor | Heating active |
| **DHW** | Binary Sensor | DHW active |
| **Flame** | Binary Sensor | Burner flame on/off |
| **Fault** | Binary Sensor | Boiler fault |
| **Diag** | Binary Sensor | Boiler diagnostic |
| **External Pump** | Binary Sensor | External pump running |
| **Fault Code** | Sensor | Fault error code |
| **Diagnostic Code** | Sensor | Diagnostic error code |
| **RSSI** | Sensor | WiFi signal strength |

**Config Entities (Number):**
- `heating_turbo` - Switch for turbo
- `heating_hysteresis` - Hysteresis value
- `heating_turbo_factor` - Turbo boost factor
- `heating_min_temp` / `heating_max_temp` - Temp limits
- `dhw_min_temp` / `dhw_max_temp` - DHW temp limits
- `pid` - Switch for PID
- `pid_p_factor`, `pid_i_factor`, `pid_d_factor` - PID constants
- `pid_dt` - PID calculation interval
- `pid_min_temp` / `pid_max_temp` - PID temp limits
- `equitherm` - Switch for equitherm
- `equitherm_n_factor`, `equitherm_k_factor`, `equitherm_t_factor` - Equitherm constants

**Dynamic Sensor Entities:**
- အာရုံခံတစ်ခုချင်းစီအတွက် HA sensor entity အလိုအလျောက် ဖန်တီးပေးခြင်း
- Device class (temperature, humidity, pressure, power, etc.) အလိုအလျောက် သတ်မှတ်ခြင်း
- Unit of measurement (°C/°F, bar/psi, L/min/gal/min) ဆက်တင်အရ သတ်မှတ်ခြင်း
- BLE sensors အတွက် multi-value entities (temperature, humidity, battery, RSSI, connection status, signal quality)
- Manual sensors အတွက် Number entities (HA မှတစ်ဆင့် set နိုင်ရန်)

**Climate Entity:**
- `climate_heating` - Heating climate control (target temp, current temp, min/max)
- `climate_dhw` - DHW climate control (DHW support ရှိမှသာ)

---
