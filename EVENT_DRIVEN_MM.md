# Event-Driven Programming - သဘောတရားနှင့် ရေးသားနည်း

## ၁။ Event-Driven ဆိုတာဘာလဲ

**Event-Driven** ဆိုသည်မှာ Program က အလုပ်တစ်ခုကို **သတ်မှတ်ထားသော အချိန်ကာလ (interval) အလိုက်** run ခြင်းမဟုတ်ဘဲ **Event (ဖြစ်ရပ်) တစ်ခု ဖြစ်ပေါ်လာမှသာ** run သော ပရိုဂရမ်ရေးနည်းဖြစ်သည်။

### Interval-Driven vs Event-Driven

```
Interval-Driven (ဥပမာ - SensorTask, interval=1000ms)
    1 sec    1 sec    1 sec    1 sec
    ┌────┐  ┌────┐  ┌────┐  ┌────┐
    │READ│  │READ│  │READ│  │READ│   ← Event မရှိလည်း run
    └────┘  └────┘  └────┘  └────┘
    ဒေတာမပြောင်းလည်း ပုံမှန်ဖတ်

Event-Driven (ဥပမာ - PortalTask, interval=0)
    REQ     REQ              REQ
     ▼       ▼                ▼
    ┌──┐   ┌──┐            ┌──┐
    │  │   │  │            │  │      ← Event ရှိမှသာ run
    └──┘   └──┘            └──┘
    Web browser က request ပို့မှသာ response ပြန်
```

**OTGateway တွင်:** `PortalTask(interval=0)` ဟု သတ်မှတ်ထားသည်။ `interval=0` ဆိုသည်မှာ အချိန်ကာလပိုင်း run ရန်မဟုတ်ဘဲ **Event ရှိမှ run** ရန်ဖြစ်သည်။

---

## ၂။ Interval-Driven Task ဘာကြောင့် Web Server အတွက် မသင့်လဲ

```cpp
// မသင့်တော်သော နည်း (Interval-Driven)
class BadWebServer : public Task {
  void loop() {
    webServer.handleClient();  // 100ms တိုင်း check
    // Client 99.9% idle ချိန်တွင် CPU အလကားသုံး
  }
};
// → CPU ကို အချိန်ပြည့် အလုပ်ပေးထား (100% busy)

// သင့်တော်သော နည်း (Event-Driven)
class GoodWebServer : public Task {
  void loop() {
    webServer.handleClient();  // Client ရှိမှ process
    // Client မရှိပါက sleep/idle
    this->delay(250);  // CPU ကို အနားပေး
  }
};
// → Client ရှိမှသာ CPU သုံး (idle ချိန် 99.9%)
```

---

## ၃။ OTGateway PortalTask တွင် Event-Driven ရေးထားပုံ

```cpp
// PortalTask.h မှ (အတိုချုံ့)
class PortalTask : public LeanTask {
public:
  PortalTask(bool _enabled = false, unsigned long _interval = 0) 
    : LeanTask(_enabled, _interval) {
    // interval = 0 → Event-Driven Mode
  }

protected:
  void setup() {
    // HTTP Route များ သတ်မှတ် (Event Handlers)
    
    // GET /api/vars → vars data ပြန်ရန်
    this->webServer->on("/api/vars", HTTP_GET, [this]() {
      JsonDocument doc;
      varsToJson(vars, doc);
      this->bufferedWebServer->send(200, "application/json", doc);
    });

    // POST /api/settings → settings update လုပ်ရန်
    this->webServer->on("/api/settings", HTTP_POST, [this]() {
      bool changed = jsonToSettings(parseJson(), settings);
      fsSettings.update();
      this->webServer->send(changed ? 201 : 200);
    });

    // Static file serve (/dashboard.html)
    this->webServer->addHandler(
      new StaticPage("/dashboard.html", &LittleFS, "/pages/dashboard.html")
    );
  }

  void loop() {
    // DNS Server (Captive Portal)
    if (this->stateDnsServer()) {
      this->dnsServer->processNextRequest();
      yield();
    }

    // Web Server - ဤ တစ်ကြောင်းတည်းက Event-Driven
    if (this->stateWebServer()) {
      this->webServer->handleClient();  // → Request ရှိမှ process
    }

    // Web/DNS server မရှိပါက sleep
    if (!this->stateDnsServer() && !this->stateWebServer()) {
      this->delay(250);  // CPU idle
    }
  }
};
```

**လုပ်ဆောင်ပုံ:**
1. Loop ထဲတွင် `webServer.handleClient()` ကိုသာ ခေါ်သည်
2. `handleClient()` က HTTP request ရှိမရှိ check လုပ်
3. Request ရှိပါက → သက်ဆိုင်ရာ handler callback function ကို run
4. Request မရှိပါက → ချက်ချင်း return → CPU idle

---

## ၄။ Event-Driven ရေးနည်း အခြေခံ Patterns

### Pattern 1: Callback Functions
```cpp
// Event Handler သတ်မှတ်ခြင်း
webServer.on("/api/temp", HTTP_GET, []() {
  float temperature = readSensor();
  String json = "{\"temp\":" + String(temperature) + "}";
  webServer.send(200, "application/json", json);
});
// → Browser က /api/temp ကို GET request ပို့မှသာ run မည်
```

### Pattern 2: အခြေအနေပြောင်းလဲမှုကို Flag ဖြင့် စစ်ဆေးခြင်း
```cpp
// main.cpp နှင့် MainTask.h တွင် တွေ့ရသော နည်း

// Global variable
bool restartSignalReceived = false;

// Event producer (PortalTask က request လက်ခံ)
webServer.on("/restart.html", HTTP_GET, [this]() {
  vars.actions.restart = true;  // Flag set
  webServer.send(302);
});

// Event consumer (MainTask က စစ်ဆေး)
void MainTask::loop() {
  if (vars.actions.restart) {
    this->restartSignalReceived = true;
    vars.actions.restart = false;
    // Restart sequence စတင်
  }
}
```

### Pattern 3: MQTT Message Callback
```cpp
// MqttTask.h မှ
this->client->onMessage([this](void*, size_t length) {
  // Topic နှင့် payload ဖတ်
  const String& topic = this->client->messageTopic();
  uint8_t payload[length];
  for (size_t i = 0; i < length; i++) {
    payload[i] = this->client->read();
  }
  
  // JSON parse လုပ်ပြီး settings/state update
  this->onMessage(topic, payload, length);
});
// → MQTT broker မှ message ရောက်မှသာ run မည်
```

### Pattern 4: ပြင်ပ Pin အပြောင်းအလဲ
```cpp
// Cascade Control Input (MainTask.h)
if (settings.cascadeControl.input.enabled) {
  bool value = digitalRead(configuredInputGpio) == HIGH;
  
  if (value != inputTempValue) {    // အခြေအနေ ပြောင်းလဲမှသာ
    inputTempValue = value;
    inputChangedTs = millis();
    
  } else if (millis() - inputChangedTs >= thresholdTime * 1000) {
    vars.cascadeControl.input = value;  // Debounce လုပ်ပြီးမှ event fire
  }
}
```

---

## ၅။ OTGateway တွင် Event-Driven ဖြစ်သော အပိုင်းများ

| Component | Event Source | Trigger |
|-----------|-------------|---------|
| **Web Server** | HTTP Request | Browser က page/api ခေါ်မှ |
| **DNS Server** | DNS Query | Device က captive portal detect လုပ်မှ |
| **MQTT Subscribe** | MQTT Message | HA/Broker က setting/command ပို့မှ |
| **BLE Notification** | BLE Notify | BLE sensor က data ပို့မှ |
| **Restart Signal** | Flag Change | Web/API က restart command ပို့မှ |
| **Emergency Flip** | Flag Change | Sensor disconnect detect ဖြစ်မှ |
| **Cascade Control** | GPIO Change | External switch အခြေအနေပြောင်းမှ |
| **Telnet** | TCP Connection | Telnet client က connect လုပ်မှ |

---

## ၆။ Hybrid ရေးနည်း (Interval + Event)

OTGateway တွင် Task အများစုသည် **Hybrid** နည်းဖြင့် ရေးထားသည်။
- Interval ဖြင့် ပုံမှန် check လုပ်
- Event ရှိမှသာ လိုအပ်သော process လုပ်

```cpp
// Hybrid ဥပမာ - MqttTask
void MqttTask::loop() {
  // Part 1: Interval-based (၅၀၀ms တိုင်း check)
  if (!this->connected) {
    this->tryReconnect();        // ပုံမှန် reconnect ကြိုးစား
  }

  // Part 2: Event-based (အခြေအနေ ပြောင်းမှသာ publish)
  if (this->newConnection || millis() - prevPubVarsTime > interval) {
    this->publishVariables();    // Connect ဖြစ်မှ OR interval ပြည့်မှ
  }

  // Part 3: Event-based (data update ရှိမှ publish)
  for (sensor...) {
    if (needUpdate) {            // Sensor data အသစ်ရှိမှ
      this->publishSensor(id);
    }
  }
}
```

---

## ၇။ Event-Driven ရေးနည်း၏ အားသာချက်/အားနည်းချက်

| အားသာချက် | အားနည်းချက် |
|---------|---------|
| CPU အချိန်ချွေတာ (idle ချိန်များ) | Event လက်လွတ်သွားနိုင် |
| Response မြန် (ချက်ချင်းတုံ့ပြန်) | Code အဖွဲ့အစည်း ရှုပ်ထွေးနိုင် |
| Memory ချွေတာ | Debugging ခက်နိုင် |
| Concurrent event များကို ကောင်းစွာ handle | Callback hell ဖြစ်နိုင် |
| IoT/Limited-resource device များအတွက် သင့်တော် | ရှည်လျားသော process များတွင် မသင့် |

---

## ၈။ အကျဉ်းချုပ်

```
                 Event-Driven Task
                 ═════════════════
                        │
              ┌─────────┼─────────┐
              ▼         ▼         ▼
          Web Server  MQTT      GPIO Change
          (HTTP req)  (message) (pin state)
              │         │         │
              └─────────┼─────────┘
                        ▼
              ┌─────────────────┐
              │  Event Handler  │
              │  (Callback Fn)  │
              └────────┬────────┘
                       ▼
              ┌─────────────────┐
              │  Process +      │
              │  Response       │
              └─────────────────┘

interval=0 သည် "ငါ run ချင်တဲ့အချိန် run မယ်" မဟုတ်ဘဲ
"Event ရှိမှ run မယ်" ဟု Scheduler ကို ပြောခြင်းဖြစ်သည်။
```
