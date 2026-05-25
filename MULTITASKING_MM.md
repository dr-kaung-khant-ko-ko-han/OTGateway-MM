# Multitasking Firmware - သဘောတရားနှင့် ရေးသားနည်း

## ၁။ Multitasking Firmware ဆိုတာဘာလဲ

**Multitasking Firmware** ဆိုသည်မှာ Microcontroller တစ်ခုတည်းပေါ်တွင် **အလုပ်များစွာကို တစ်ပြိုင်နက်တည်း (သို့) တစ်ပြိုင်နက်သဖွယ်** လုပ်ဆောင်နိုင်သော Firmware အမျိုးအစားဖြစ်သည်။

**သာမန် Firmware (Single-task)**:
```
setup() → loop() တစ်ခုတည်း တစ်လှည့်စီ run
┌────────┐    ┌────────┐    ┌────────┐
│ReadTemp│───►│Control │───►│ SendMQTT│───► loop
└────────┘    └────────┘    └────────┘
```
ပြဿနာ - MQTT send လုပ်နေစဉ် ၅ စက္ကန့်ကြာလျှင် temp reading လည်း ၅ စက္ကန့် နောက်ကျသွားမည်။

**Multitasking Firmware**:
```
┌──────────┐  ┌──────────┐  ┌──────────┐
│ReadTemp  │  │ Control  │  │ SendMQTT │  တစ်ပြိုင်နက်သဖွယ် run
│ (100ms)  │  │ (1 sec)  │  │ (5 sec)  │
└──────────┘  └──────────┘  └──────────┘
       ↕            ↕            ↕
┌─────────────────────────────────────┐
│          Scheduler                  │
│  (မည်သည့် Task ကို ဘယ်အချိန် run ရမလဲ ဆုံးဖြတ်)  │
└─────────────────────────────────────┘
```

---

## ၂။ Multitasking အမျိုးအစားများ

### (က) Cooperative Multitasking (ပူးပေါင်းဆောင်ရွက်သော)
OTGateway တွင် အသုံးပြုသော နည်းလမ်းဖြစ်သည်။

**သဘောတရား:** Task တစ်ခုက အလုပ်ပြီးမှ နောက် Task ကို run ခွင့်ပေးသည်။

```cpp
// Cooperative Multitasking ဥပမာ
void setup() {
  Scheduler.start(new Task1());  // LED Blink - 500ms
  Scheduler.start(new Task2());  // Read Sensor - 2000ms
  Scheduler.start(new Task3());  // Send MQTT - 5000ms
  Scheduler.begin();
}

// Task 1 ရေးနည်း
class Task1 : public Task {
protected:
  void loop() {
    digitalWrite(LED_PIN, !digitalRead(LED_PIN));
    delay(500);  // ဤ delay အတွင်း အခြား Task များ run နိုင်
  }
};
```

**အားသာချက်:** ရိုးရှင်း၊ race condition နည်း၊ debugging လွယ်
**အားနည်းချက်:** Task တစ်ခုက `delay()` မသုံးဘဲ အကြာကြီး run နေပါက အခြား Task များ ရပ်သွားနိုင်

### (ခ) Preemptive Multitasking (ကြားဖြတ်ဝင်သော) - FreeRTOS
ESP32 တွင် native support ဖြစ်သော နည်းလမ်း။

**သဘောတရား:** Scheduler က Task တစ်ခုကို အချိန်ပိုင်းအလိုက် အတင်း interrupt လုပ်၍ အခြား Task သို့ ပြောင်းပေးသည်။

```cpp
// FreeRTOS Preemptive Multitasking ဥပမာ
void Task1(void* pvParameters) {
  for (;;) {
    digitalWrite(LED_PIN, !digitalRead(LED_PIN));
    vTaskDelay(pdMS_TO_TICKS(500));  // 500ms block
  }
}

void Task2(void* pvParameters) {
  for (;;) {
    float temp = readSensor();
    vTaskDelay(pdMS_TO_TICKS(2000));
  }
}

void setup() {
  xTaskCreate(Task1, "LED", 2048, NULL, 1, NULL);
  xTaskCreate(Task2, "Sensor", 4096, NULL, 1, NULL);
  vTaskStartScheduler();
}
```

**အားသာချက်:** Task တစ်ခု stuck ဖြစ်လည်း အခြား Task များ run နိုင်
**အားနည်းချက်:** Race condition ဖြစ်နိုင်၊ synchronization လိုအပ်၊ debugging ခက်

---

## ၃။ OTGateway ၏ Task ရေးသားပုံ

### Task Class Structure
```cpp
// Task ရေးရန် Class Template
class MyTask : public Task {       // သို့မဟုတ် LeanTask
public:
  MyTask(bool _enabled = false, unsigned long _interval = 0) 
    : Task(_enabled, _interval) {
    // Constructor - variables အစပြုရန်
  }

  ~MyTask() {
    // Destructor - memory cleanup
  }

protected:
  #if defined(ARDUINO_ARCH_ESP32)
  const char* getTaskName() override {
    return "MyTask";  // Debugging တွင် ပြသရန် အမည်
  }
  
  BaseType_t getTaskCore() override {
    return 0;  // ESP32 Core 0 or 1
  }

  int getTaskPriority() override {
    return 3;  // 1=lowest, 5=highest
  }
  #endif

  void setup() {
    // Task start တွင် တစ်ခါသာ run မည့် code
  }

  void loop() {
    // Interval အလိုက် ထပ်တလဲလဲ run မည့် code
    readSensor();
    processData();
    
    // delay() အစား yield() သုံးပါက အခြား task run ခွင့်ရ
    this->yield();
  }
};
```

### OTGateway တွင် Task ဖန်တီးနည်း (main.cpp)
```cpp
// main.cpp ထဲတွင်
MqttTask* tMqtt;
OpenThermTask* tOt;
SensorsTask* tSensors;
RegulatorTask* tRegulator;
PortalTask* tPortal;
MainTask* tMain;

void setup() {
  // ...
  
  // Task များဖန်တီးပြီး စတင်
  tMqtt = new MqttTask(false, 500);     // start disabled, 500ms
  Scheduler.start(tMqtt);

  tOt = new OpenThermTask(true, 750);   // start enabled, 750ms  
  Scheduler.start(tOt);

  tSensors = new SensorsTask(true, 1000);
  Scheduler.start(tSensors);

  tRegulator = new RegulatorTask(true, 10000);
  Scheduler.start(tRegulator);

  tPortal = new PortalTask(true, 0);    // interval=0 = event-driven
  Scheduler.start(tPortal);

  tMain = new MainTask(true, 100);
  Scheduler.start(tMain);

  // Task များအားလုံး စတင် run မည့် အမိန့်
  Scheduler.begin();
}

void loop() {
  // ESP32 တွင် Arduino loop ကို delete
  vTaskDelete(NULL);
}
```

---

## ၄။ အရေးကြီး မှတ်သားချက်များ

### (က) Shared Data ကို ကာကွယ်ရန်
```cpp
// မကောင်းသော ဥပမာ (Race Condition)
// Task1 ကလည်း settings.heating.target ကိုရေး
// Task2 ကလည်း settings.heating.target ကိုဖတ်
// → တစ်ပြိုင်နက် ဖြစ်သွားပါက data corruption

// OTGateway ၏ ဖြေရှင်းနည်း:
// 1. Task တစ်ခုက ရေး၊ ကျန်တာတွေက ဖတ်
// 2. MainTask (100ms) ကသာ settings update လုပ်
// 3. RegulatorTask (10s) ကသာ setpoint တွက်ချက်
// → Task တစ်ခုချင်းစီ၏ တာဝန် သီးသန့်ခွဲထား
```

### (ခ) Interval သတ်မှတ်နည်း
```cpp
// High Priority, Fast Interval → Time-critical tasks
OpenThermTask:  interval=750ms,  priority=5  (ဘွိုင်လာဆက်သွယ်ရေး)

// Medium Priority → Data processing
MainTask:       interval=100ms,  priority=3  (စနစ်ကြီးကြပ်မှု)
SensorsTask:    interval=1000ms, priority=4  (အာရုံခံဖတ်ခြင်း)
MqttTask:       interval=500ms,  priority=2  (MQTT ဆက်သွယ်ရေး)

// Low Priority, Slow Interval → Heavy computation
RegulatorTask:  interval=10000ms,priority=4  (PID တွက်ချက်ခြင်း)

// Event-driven → Request ရှိမှ run
PortalTask:     interval=0,      priority=1  (Web server)
```

### (ဂ) Task များကြား ဆက်သွယ်နည်း
```cpp
// 1. Global Variables ဖြင့်
extern Variables vars;  // Task အားလုံး ဖတ်နိုင်
vars.master.heating.setpointTemp = 45.0f;  // RegulatorTask က ရေး
float temp = vars.slave.heating.currentTemp;  // OpenThermTask က ဖတ်

// 2. Static Class Methods ဖြင့်
Sensors::setValueById(sensorId, value, Sensors::ValueType::TEMPERATURE);
float avgTemp = Sensors::getMeanValueByPurpose(Sensors::Purpose::INDOOR_TEMP);

// 3. Scheduler Functions ဖြင့်
tMqtt->enable();   // MainTask က MqttTask ကို ဖွင့်/ပိတ်
tMqtt->disable();
tMqtt->isConnected();  // MainTask က MqttTask ရဲ့ status စစ်
```

---

## ၅။ OTGateway Task Flow Diagram

```
Time ──────────────────────────────────────────────────────────►

MainTask:    ■■■         ■■■         ■■■         ■■■
(100ms)     setup      loop        loop        loop

MqttTask:         ■■■■                  ■■■■
(500ms)          loop                  loop

OpenTherm:     ■■■■■■■            ■■■■■■■
(750ms)       setup+loop          loop

Sensors:                ■■■■■■■■■■■■■
(1000ms)                  loop

Regulator:                                      ■■■■■■
(10000ms)                                         loop

Portal:     ▼       ▼     ▼  ▼         ▼   ▼ ▼     ▼
(Event)   request events ရှိမှသာ handle
```

---

## ၆။ အခြား Microcontroller Multitasking နည်းလမ်းများ

| နည်းလမ်း | Platform | ရှင်းလင်းချက် |
|--------|----------|-------------|
| **ESP8266Scheduler** | ESP8266 | Cooperative, OTGateway တွင် ESP8266 အတွက်သုံး |
| **ESP32Scheduler** | ESP32 | Cooperative, OTGateway တွင် ESP32 အတွက်သုံး |
| **FreeRTOS** | ESP32, STM32 | Preemptive, ESP32 တွင် native support |
| **TaskScheduler** | Arduino | Cooperative, library သုံးရန် |
| **Protothreads** | Arduino | Lightweight, stackless threading |
| **Interrupt-based** | AVR, ARM | Timer interrupts ဖြင့် အချိန်ပိုင်း ခွဲခြင်း |

---

## ၇။ နှစ်မျိုးနှိုင်းယှဉ်ချက်

| | Cooperative (OTGateway) | Preemptive (FreeRTOS) |
|--|------------------------|----------------------|
| **Scheduler** | Task က စေတနာအလျောက် လွှဲပေး | Scheduler က အတင်းလွှဲယူ |
| **စောင့်ရသည့်အချိန်** | Task ပြီးတဲ့အထိ | Time slice ကုန်တဲ့အထိ |
| **Timing တိကျမှု** | အနည်းငယ်နိမ့် | ပိုမိုတိကျ |
| **Memory** | နည်း (shared stack) | များ (per-task stack) |
| **စေ့စပ်မှု** | ရိုးရှင်း | ရှုပ်ထွေး |
| **Bug ရှာရလွယ်မှု** | လွယ် | ခက် |
| **Synchronization** | မလိုသလောက် | Mutex, Semaphore လို |
| **သင့်တော်မှု** | ESP8266, AVR | ESP32, STM32, ARM |

**အချုပ်:** OTGateway သည် ESP8266 ၏ memory limitation ကြောင့် Cooperative Multitasking ကို ရွေးချယ်ထားသည်။ ESP8266 တွင် RAM 80KB သာရှိသောကြောင့် Task တစ်ခုချင်းစီအတွက် သီးသန့် stack ခွဲပေးရသော Preemptive (FreeRTOS) က အဆင်မပြေပေ။
