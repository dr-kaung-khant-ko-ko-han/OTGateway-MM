# OTGateway Program - လုပ်ဆောင်ပုံ အသေးစိတ်ရှင်းလင်းချက်

## ၁။ စနစ် တည်ဆောက်ပုံ အကျဉ်းချုပ်

OTGateway သည် **ESP32/ESP8266** microcontroller ပေါ်တွင် အလုပ်လုပ်သော **Multitasking Firmware** ဖြစ်သည်။ ၎င်းသည် အိမ်သုံးဘွိုင်လာ (Boiler) နှင့် **OpenTherm Protocol** ဖြင့် ဆက်သွယ်ပြီး **Smart Heating Control** ကို ဆောင်ရွက်ပေးသည်။

```
┌─────────────────────────────────────────────────────────────┐
│                     OTGateway Firmware                      │
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │ MainTask │  │MqttTask  │  │OpenTherm │  │SensorsT. │  │
│  │  (100ms) │  │ (500ms)  │  │  (750ms) │  │ (1000ms) │  │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘  │
│       │             │             │             │          │
│  ┌────┴─────────────┴─────────────┴─────────────┴────┐     │
│  │              Scheduler (Cooperative)               │     │
│  └────────────────────────────────────────────────────┘     │
│                                                             │
│  ┌──────────┐  ┌──────────┐                                │
│  │Regulator │  │ Portal   │                                │
│  │ (10000ms)│  │ (Event)  │                                │
│  └──────────┘  └──────────┘                                │
└─────────────────────────────────────────────────────────────┘
```

---

## ၂။ Boot Process (စတင်ချိန် လုပ်ဆောင်ချက်များ)

### Step 1: CrashRecorder အစပြုခြင်း
ESP32/ESP8266 ၏ panic handler ကို register လုပ်သည်။ စက်ပျက်ပါက backtrace အချက်အလက်များကို NOINIT memory တွင် သိမ်းထားသည်။

### Step 2: File System (LittleFS) အစပြုခြင်း
```
Flash Memory Layout:
┌──────────┬──────────┬──────────┬──────────┬──────────┐
│   NVS    │ OTA Data │  App 0   │  App 1   │ LittleFS │
│  20 KB   │   8 KB   │ 1.7 MB   │ 1.7 MB   │  512 KB  │
└──────────┴──────────┴──────────┴──────────┴──────────┘
```

### Step 3: Configuration ဖိုင်များ ဖတ်ခြင်း
| ဖိုင် | ပါဝင်သော အချက်အလက်များ |
|------|----------------------|
| `/network.conf` | WiFi SSID, Password, Hostname, DHCP/Static IP |
| `/settings.conf` | Heating, DHW, PID, Equitherm, MQTT ဆက်တင်များ |
| `/sensors.conf` | အာရုံခံကိရိယာ တစ်ခုချင်းစီ၏ ဆက်တင်များ |

### Step 4: Network Manager အစပြုခြင်း
WiFi connection ကို STA (WiFi router သို့ချိတ်) သို့မဟုတ် AP (Access Point အဖြစ်) စတင်ခြင်း။

### Step 5: 6 Tasks များ စတင်ခြင်း
```
Priority Order (ESP32):
  5 = OpenThermTask   (Core 1)  ← အမြင့်ဆုံး
  4 = SensorsTask     (Core 0)  
  4 = RegulatorTask   (Core 0)
  3 = MainTask        (Core 0)
  2 = MqttTask        (Core 0)
  1 = PortalTask      (Core 0)  ← အနိမ့်ဆုံး
```

---

## ၃။ Data Flow (အချက်အလက်စီးဆင်းပုံ)

### (က) OpenTherm Communication Loop
```
    ┌──────────┐                          ┌──────────┐
    │ Gateway  │ ◄──── OpenTherm Bus ────► │  Boiler  │
    │ (Master) │     (2-wire, digital)    │ (Slave)  │
    └──────────┘                          └──────────┘
```

**750ms တိုင်း လုပ်ဆောင်သည်များ:**
```
1. setBoilerStatus()  →  Boiler သို့ heating/dhw/ch2 on/off command
2. setMaxModulation() →  Max modulation level သတ်မှတ်
3. updateModulation() →  Modulation level + Power ဖတ်
4. updateDhwTemp()    →  DHW ရေပူအပူချိန် ဖတ်
5. updateHeatingTemp()→  Heating အပူချိန် ဖတ်
6. updatePressure()   →  Pressure ဖတ်
7. ... (စုစုပေါင်း ဒေတာအမျိုးအစား ၂၀ ကျော်)
```

**တစ်မိနစ် (၆၀ စက္ကန့်) တိုင်း လုပ်ဆောင်သည်များ:**
```
- Date/Time သတ်မှတ်
- Min/Max temp များ ဖတ်
- Fault code / Diag code ဖတ်
- Burner/Pump start count များ ဖတ်
- Burner/Pump hours များ ဖတ်
```

### (ခ) Sensor Data Flow
```
                      ┌────────────────┐
    Dallas (1-Wire) ─►│                │
    NTC 10K         ─►│ SensorsTask    │──► vars.master.*
    BLE Sensor      ─►│ (1 sec poll)   │──► Sensors::results[]
    OpenTherm Data  ─►│                │
                      └────────────────┘
```

**အာရုံခံတန်ဖိုး Processing:**
```
Raw Value → Offset (ပေါင်း) → Factor (မြှောက်) → Filtering → Final Value
```
- Filtering: `new = old + (raw - old) × 0.15` (exponential moving average)
- Connection timeout: ၂ မိနစ် data မရရင် disconnected အဖြစ်သတ်မှတ်

### (ဂ) Temperature Control (Regulator) Flow
```
┌──────────────────────────────────────────────────────────────┐
│                      RegulatorTask (၁၀ စက္ကန့်တိုင်း)        │
│                                                              │
│  ┌──────────────┐    ┌──────────┐    ┌──────────────────┐   │
│  │ Outdoor Temp │    │ Heating  │    │ Emergency Mode?  │   │
│  │ Indoor Temp  │───►│ Target   │───►│ YES → Emergency  │   │
│  └──────────────┘    │   Temp   │    │      Target Temp │   │
│                      └──────────┘    └────────┬─────────┘   │
│                                               │ NO           │
│         ┌─────────────────────────────────────┘              │
│         ▼                                                    │
│  ┌──────────────────────────────────────────────┐           │
│  │         Control Mode Selection               │           │
│  │                                              │           │
│  │  Manual    Equitherm      PID      Native    │           │
│  │  ──────    ─────────    ─────     ──────     │           │
│  │  Direct    Outdoor       PID       Boiler    │           │
│  │  target    temp curve    control   handled   │           │
│  └────────────────┬─────────────────────────────┘           │
│                   ▼                                          │
│  ┌──────────────────────────────────────────────┐           │
│  │         SETPOINT TEMP CALCULATION            │           │
│  │                                              │           │
│  │  = Equitherm Result + PID Result + Turbo     │           │
│  │  (Each component can be ON/OFF individually) │           │
│  └────────────────┬─────────────────────────────┘           │
│                   ▼                                          │
│  ┌──────────────────────────────────────────────┐           │
│  │         SAFETY CHECKS                        │           │
│  │  - Hysteresis (blocking if near target)      │           │
│  │  - Overheat Protection (OFF if too hot)      │           │
│  │  - Freeze Protection (ON if too cold)        │           │
│  │  - Min/Max Temp Clamping                     │           │
│  └────────────────┬─────────────────────────────┘           │
│                   ▼                                          │
│         vars.master.heating.setpointTemp                     │
└──────────────────────────────────────────────────────────────┘
```

### (ဃ) MQTT Data Flow (Home Assistant နှင့် ဆက်သွယ်မှု)
```
┌──────────┐     MQTT Broker     ┌───────────────────┐
│ OTGateway│ ◄══════════════════► │  Home Assistant   │
│          │  publish/subscribe  │  (Auto-Discovery) │
└──────────┘                     └───────────────────┘

Publish Topics (Gateway → HA):
  {prefix}/status           → "online" / "offline"
  {prefix}/state            → JSON: လက်ရှိ state အားလုံး
  {prefix}/settings         → JSON: ဆက်တင်အားလုံး
  {prefix}/sensors/{name}   → JSON: အာရုံခံ data
  homeassistant/**/config   → Auto-Discovery configs

Subscribe Topics (HA → Gateway):
  {prefix}/settings/set     → ဆက်တင်ပြောင်းရန်
  {prefix}/state/set        → Variable ပြောင်းရန်
  {prefix}/sensors/{name}/set → Manual sensor set
```

---

## ၄။ Multitasking System အလုပ်လုပ်ပုံ

### Scheduling Model
OTGateway သည် **Cooperative Multitasking** ကို အသုံးပြုသည်။ Task တစ်ခုစီသည် ၎င်း၏ interval ပြည့်သည့်အခါ `loop()` function ကို run သည်။

```
Timeline:
Time:     0ms    100ms   200ms   300ms   400ms   500ms   600ms   700ms   750ms   800ms   ...
MainTask:  RUN    RUN     RUN     RUN     RUN     RUN     RUN     RUN     RUN     RUN    ...
MqttTask:                                 RUN                                     RUN    ...
OpenTherm:                RUN                                     RUN                     ...
Sensors:                                          RUN                                    ...
Regulator:                                                                               ...
Portal:   event-driven (request ရှိမှ run)
```

**Timeout Protection:** Task တစ်ခုက အချိန်ကြာနေပါက `yield()` သို့မဟုတ် `delay()` ဖြင့် အခြား task များ run နိုင်ရန် လုပ်ဆောင်ပေးသည်။

---

## ၅။ Emergency Mode (အရေးပေါ် မုဒ်)

```
Emergency ဖြစ်ပွားနိုင်သော အခြေအနေများ:
┌────────────────────────────────────────────────────────┐
│  Flag  │ Condition                                     │
│────────│───────────────────────────────────────────────│
│  0x01  │ Equitherm ON ဖြစ်ပြီး Outdoor အာရုံခံမရှိ   │
│  0x02  │ PID ON ဖြစ်ပြီး Indoor အာရုံခံမရှိ          │
│  0x04  │ Native Control ON ဖြစ်ပြီး Indoor အာရုံခံမရှိ│
└────────────────────────────────────────────────────────┘

Emergency Mode တွင်:
- Heating = ON (မဖြစ်မနေ)
- Setpoint Temp = settings.emergency.target (default 40°C)
- အိမ်အေးခဲခြင်းမှ ကာကွယ်ရန်
```

---

## ၆။ Safety Features (လုံခြုံရေး အင်္ဂါရပ်များ)

| အင်္ဂါရပ် | လုပ်ဆောင်ပုံ |
|---------|-----------|
| **Overheat Protection** | Heating/DHW temp > `highTemp` → OFF; temp < `lowTemp` → restart OK |
| **Freeze Protection** | Indoor/Heating temp < `lowTemp` (10°C) → Heating auto ON |
| **Crash Recovery** | Heap < 2048 bytes → Auto restart |
| **Watchdog** | FreeRTOS Task Watchdog (ESP32) |
| **Bus Reset** | OpenTherm ၂ မိနစ်ကျော် disconnected → bus reset |
| **Auto Fault Reset** | Fault ဖြစ်ပါက အလိုအလျောက် reset ကြိုးစားခြင်း |

---

## ၇။ စနစ်တစ်ခုလုံး အလုပ်လုပ်ပုံ အကျဉ်းချုပ်

```
1. BOOT
   ├─ CrashRecorder Init
   ├─ LittleFS Mount
   ├─ Config Files Read (3 files)
   └─ Network Manager Start

2. TASKS START
   └─ 6 Scheduler Tasks Created

3. MAIN LOOP (100ms)
   ├─ Network Loop (WiFi reconnect, status)
   ├─ Settings Files Watch (auto-save)
   ├─ NTP Time Sync (when connected)
   ├─ Telnet Start/Stop
   ├─ MQTT Enable/Disable
   ├─ LED Status Update
   ├─ Emergency Detection
   ├─ Freeze Protection
   ├─ External Pump Control
   ├─ Cascade Control (multi-boiler)
   └─ Memory Monitoring

4. OPENTHERM LOOP (750ms)
   ├─ Boiler Status Set (heating/dhw/ch2)
   ├─ Real-time Data Read (temp, pressure, modulation...)
   ├─ Temperature Set (heating/dhw/ch2 target)
   ├─ Safety Checks (overheat)
   └─ Fault/Diag Reset

5. SENSORS LOOP (1000ms)
   ├─ Dallas (1-Wire) Polling
   ├─ NTC 10K Voltage Read
   ├─ BLE Data Subscribe
   └─ Connection Status Check

6. REGULATOR LOOP (10000ms)
   ├─ Indoor/Outdoor Temp Update
   ├─ Equitherm Calculation
   ├─ PID Calculation
   ├─ Turbo Boost
   ├─ Hysteresis Check
   └─ Final Setpoint Temp Output

7. MQTT LOOP (500ms)
   ├─ Broker Connect/Reconnect
   ├─ Publish State (interval seconds)
   ├─ Publish Settings (interval × 10)
   ├─ Publish Sensor Data
   ├─ HA Discovery Publish
   └─ Incoming Message Handle

8. PORTAL LOOP (event-driven)
   ├─ Web Server (HTTP on port 80)
   ├─ DNS Server (Captive Portal)
   ├─ REST API Endpoints
   └─ OTA Firmware Upgrade
```

---

**Core Concept:** OTGateway သည် ဘွိုင်လာထံမှ အချက်အလက်များ ရယူခြင်း၊ အာရုံခံကိရိယာများမှ အပူချိန်များ ဖတ်ခြင်း၊ ၎င်းတို့ကို PID/Equitherm algorithm များဖြင့် တွက်ချက်ခြင်း၊ သင့်လျော်သော setpoint temperature ကို ဘွိုင်လာသို့ ပြန်ပို့ခြင်း၊ နှင့် အချက်အလက်အားလုံးကို MQTT/Web UI မှတစ်ဆင့် အသုံးပြုသူထံ ပြသခြင်း တို့ကို တစ်ပြိုင်နက်တည်း ဆောင်ရွက်ပေးသော Smart Heating Controller တစ်ခု ဖြစ်သည်။
