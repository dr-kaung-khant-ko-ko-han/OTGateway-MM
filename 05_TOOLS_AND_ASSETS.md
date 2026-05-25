# tools/ Build Tools နှင့် အခြားဖိုင်များ - အသေးစိတ်ရှင်းလင်းချက်

---

## 1. `tools/build.py`
**လမ်းကြောင်း:** `OTGateway-master/tools/build.py` (2,176 bytes)

**ရှင်းလင်းချက်:** PlatformIO build process တွင် post-processing script အဖြစ် run သည်။ Firmware binary ကို build လုပ်ပြီးတိုင်း အလိုအလျောက်လုပ်ဆောင်သည်။

**လုပ်ဆောင်ချက်များ:**
1. **Build Info Header ဖန်တီးခြင်း** - `build_info.h` file ကိုဖန်တီးပြီး build date, version, environment name တို့ ထည့်သွင်းခြင်း
2. **Firmware File များ စုစည်းခြင်း** - Build output ဖိုင်များကို `build/` directory အောက်သို့ ကူးယူခြင်း
3. **Release Package** - `.bin` firmware file ကို release-ready အဖြစ် ပြင်ဆင်ခြင်း
4. **File System Image Processing** - LittleFS image ကို verify လုပ်ခြင်း

---

## 2. `tools/esp32.py`
**လမ်းကြောင်း:** `OTGateway-master/tools/esp32.py` (2,735 bytes)

**ရှင်းလင်းချက်:** ESP32-specific post-processing script ။ ESP32 firmware binary ကို build လုပ်ပြီးတိုင်း run သည်။

**လုပ်ဆောင်ချက်များ:**
1. **ESP32 Bootloader Merge** - Bootloader, partition table, firmware, filesystem image များကို single `.bin` file အဖြစ် ပေါင်းစည်းခြင်း
2. **Flash Offset Calculation** - Partition table အရ firmware နှင့် filesystem ကို မှန်ကန်သော offset တွင် ထည့်သွင်းခြင်း
3. **ESPTool Invocation** - `esptool.py` ကို အသုံးပြု၍ flash image ဖန်တီးခြင်း
4. **Factory Image ဖန်တီးခြင်း** - Factory reset အတွက် combined image

---

## 3. `.github/` Directory

### `.github/dependabot.yaml` (204 bytes)
**ရှင်းလင်းချက်:** GitHub Dependabot configuration ။ Project dependencies (GitHub Actions, pip packages) များကို အလိုအလျောက် update လုပ်ရန် သတ်မှတ်ချက်။

### `.github/workflows/pio-dependabot.yaml` (525 bytes)
**ရှင်းလင်းချက်:** GitHub Actions workflow - `pio_dependabot` အမည်ရှိသော platformio dependency updater ။ PlatformIO `platformio.ini` ဖိုင်ထဲမှ library dependencies များကို အလိုအလျောက် update PR ဖန်တီးပေးသည်။

### `.github/workflows/stale.yaml` (673 bytes)
**ရှင်းလင်းချက်:** GitHub Actions workflow - Stale issue/PR manager ။ ၆၀ ရက်ကျော် လှုပ်ရှားမှုမရှိသော issues/PRs များကို stale အဖြစ် အမှတ်အသားပြုပြီး ၇ ရက်အတွင်း ထပ်မံလှုပ်ရှားမှုမရှိပါက အလိုအလျောက် close လုပ်သည်။

---

## 4. `assets/` Directory

Hardware-related files - PCB, Schematic, Assembly files:

### PCB & Assembly Images
| ဖိုင် | Size | ရှင်းလင်းချက် |
|------|------|-------------|
| `2D_PCB_bottom.png` | 290 KB | PCB အောက်ဘက်မြင်ကွင်း |
| `2D_PCB_top.png` | 396 KB | PCB အပေါ်ဘက်မြင်ကွင်း |
| `3D_PCB.png` | 870 KB | PCB 3D မြင်ကွင်း |
| `after_assembly.png` | 2 MB | တပ်ဆင်ပြီး PCB ဓာတ်ပုံ |
| `connection.png` | 100 KB | ချိတ်ဆက်ပုံ ရုပ်ပြ |

### Design Files
| ဖိုင် | Size | ရှင်းလင်းချက် |
|------|------|-------------|
| `Schematic.pdf` | 275 KB | ဆားကစ်ပုံစံ (Circuit Schematic) |
| `BOM.xlsx` | 12 KB | Bill of Materials (ပစ္စည်းစာရင်း) |
| `CPL.csv` | 14 KB | Component Placement List (SMT နေရာချထားမှု) |
| `gerber.zip` | 262 KB | PCB ထုတ်လုပ်ရေး Gerber ဖိုင်များ |

### Marketing/Poster Images
| ဖိုင် | Size | ရှင်းလင်းချက် |
|------|------|-------------|
| `poster-1.png` | 1.1 MB | Dashboard မြင်ကွင်း ပိုစတာ |
| `poster-2.png` | 1.1 MB | Configuration မြင်ကွင်း ပိုစတာ |
| `poster-3.png` | 996 KB | Home Assistant ပေါင်းစည်းမှု ပိုစတာ |
| `logo.svg` | 803 B | Logo (vector) |

### Other Design Files
| ဖိုင် | Size | ရှင်းလင်းချက် |
|------|------|-------------|
| `blueprint_import.svg` | 3.3 KB | PCB blueprint import file |
| `adapter_schematic_o.png` | 15.5 KB | OpenTherm adapter schematic |
| `dhw_meter.png` | 38 KB | DHW meter connection diagram |
| `equitherm_calc.xlsx` | 25 KB | Equitherm curve calculator spreadsheet |

### Home Assistant Configuration (`assets/ha/`)
| ဖိုင် | Size | ရှင်းလင်းချက် |
|------|------|-------------|
| `dhw_meter.yaml` | 479 B | HA တွင် DHW meter entity သတ်မှတ်ရန် YAML |
| `report_temp_to_otgateway.yaml` | 1.4 KB | HA sensor temperature ကို OTGateway သို့ report လုပ်ရန် automation |
| `report_temp_to_otgateway_from_weather.yaml` | 1.4 KB | HA weather integration မှ outdoor temp ကို OTGateway သို့ report လုပ်ရန် automation |

---

## 5. Empty Placeholder Directories

### `build/.gitkeep`
Build output directory placeholder ။ Git က empty directory များကို track မလုပ်သောကြောင့် `.gitkeep` file ထည့်ထားသည်။

### `data/.gitkeep` & `data/static/.gitkeep`
Web UI data directory placeholder ။ `gulpfile.js` run ပြီးမှသာ ဖိုင်များ ထုတ်ပေးမည်။

---
