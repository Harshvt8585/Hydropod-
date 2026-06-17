# HydroPod — Smart Portable Water Quality Monitor & Purifier

> A compact, off-grid IoT device that simultaneously monitors water quality across 3 real-time parameters (pH, TDS, Turbidity) and purifies it through multi-stage filtration — designed for remote, rural, and disaster-affected communities where clean water access is unreliable.

---

## Table of Contents

- [The Problem](#the-problem)
- [What HydroPod Does](#what-hydropod-does)
- [Section 1 — Filtration System](#section-1--filtration-system)
- [Section 2 — Electronics & Sensors](#section-2--electronics--sensors)
- [Section 3 — Firmware & Blynk Dashboard](#section-3--firmware--blynk-dashboard)
- [System Architecture](#system-architecture)
- [Sample Output](#sample-output)
- [Project Status](#project-status)
- [Impact & Target Users](#impact--target-users)
- [Recognition](#recognition)
- [About](#about)

---

## The Problem

Billions of people in remote and disaster-hit areas depend on unsafe water sources. Existing water testing and purification solutions are either too large, too expensive, or need a stable power grid — making them impractical for exactly the communities that need them most.

HydroPod addresses this by combining a full sensor suite with multi-stage purification in a single portable, battery-powered device that anyone can carry and operate.

---

## What HydroPod Does

- Monitors **pH, TDS (Total Dissolved Solids), and Turbidity** in real time
- Purifies water through **3 filtration stages**: sediment filter → activated carbon filter → RO membrane
- Streams live sensor data to a **Blynk IoT dashboard** over WiFi
- Powered by a **dual power system**: 12V 6A battery for the 120 PSI motor and 5V supply for the ESP32 and sensors
- Flags unsafe water conditions with **threshold-based alerts**
- Operates in the field with no need for a laptop or external computer

---

## Section 1 — Filtration System

The filtration system is the physical water treatment core of HydroPod. Water passes through three stages in series, each targeting different categories of contaminants.

```
[Raw / Contaminated Water Input]
            |
    ┌───────▼────────┐
    │  Stage 1        │   Sediment Filter (PP Spun)
    │  5 micron       │   Removes: sand, silt, rust, suspended particles
    └───────┬────────┘
            |
    ┌───────▼────────┐
    │  Stage 2        │   Activated Carbon Filter (GAC / CTO Block)
    │  Carbon Block   │   Removes: chlorine, organic compounds, bad odor,
    └───────┬────────┘            discoloration, taste impurities
            |
    ┌───────▼────────┐
    │  Stage 3        │   RO Membrane (Reverse Osmosis)
    │  TFC Membrane   │   Removes: dissolved salts, heavy metals, bacteria,
    └───────┬────────┘            viruses, fluoride, nitrates
            |
    [Clean Water Output]         [Reject / Waste Water Out]
```

### Stage 1 — Sediment Filter

| Parameter | Detail |
|---|---|
| Type | PP (Polypropylene) Spun Cartridge |
| Filtration rating | 5 micron |
| Removes | Sand, silt, rust, sediment, suspended particulates |
| Purpose | Protects downstream filters from clogging |
| Replacement | Every 3–6 months depending on source water quality |

The sediment filter is the first line of defence. It catches larger physical particles that would otherwise damage or clog the activated carbon and RO membrane stages. Without this stage, the RO membrane would degrade rapidly and require frequent, costly replacement.

### Stage 2 — Activated Carbon Filter

| Parameter | Detail |
|---|---|
| Type | GAC (Granular Activated Carbon) or CTO Carbon Block |
| Removes | Chlorine, chloramines, organic compounds, VOCs, odor, colour |
| Purpose | Chemical pre-treatment before RO; improves taste and odor |
| Mechanism | Adsorption — contaminants bond to the carbon surface |
| Replacement | Every 6 months |

Activated carbon works through adsorption — contaminant molecules chemically bond to the enormous surface area of the carbon matrix. A single gram of activated carbon can have a surface area exceeding 500 m². This stage is critical before the RO membrane because chlorine, even in small amounts, degrades the TFC (thin-film composite) RO membrane over time.

### Stage 3 — RO Membrane (Reverse Osmosis)

| Parameter | Detail |
|---|---|
| Type | TFC (Thin-Film Composite) RO Membrane |
| Operating pressure | Requires 120 PSI — driven by the 12V high-pressure pump |
| Rejection rate | ~95–99% for dissolved salts, heavy metals |
| Removes | TDS, fluoride, nitrates, arsenic, lead, bacteria, viruses |
| Pore size | ~0.0001 micron |
| Waste ratio | Typically 1:3 pure-to-reject (varies by membrane and pressure) |
| Replacement | Every 12–18 months |

Reverse osmosis works by forcing water through a semi-permeable membrane under high pressure. The membrane pores are so small (0.0001 micron) that only water molecules pass through — dissolved salts, heavy metals, and most biological contaminants are rejected and flushed out as wastewater. This is why HydroPod requires a **120 PSI high-pressure pump** driven by the 12V 6A battery — domestic water pressure alone is insufficient to force water through an RO membrane effectively.

> **Why RO over UF membrane?**
> RO removes dissolved salts and heavy metals that UF cannot. For areas with high TDS groundwater (common in many Indian states), RO is the appropriate choice. The tradeoff is higher energy consumption and a reject water stream, which is managed by the 12V motor system.

---

## Section 2 — Electronics & Sensors

The electronics system handles real-time water quality sensing, data processing, wireless transmission, and power management.

### Block diagram

```
[12V 6A Battery]──────────────────────►[120 PSI High-Pressure Pump]
                                                    |
                                        Drives water through RO membrane

[5V Power Supply / USB]───────────────►[ESP32 Microcontroller]
                                                    |
                          ┌─────────────────────────┼──────────────────────┐
                          │                         │                      │
                   [pH Sensor]              [TDS Sensor]          [Turbidity Sensor]
                   (GPIO 36)                (GPIO 34)             (GPIO 39)
                          │                         │                      │
                          └─────────────────────────┘                      │
                                         |                                  │
                               [ADC — 12-bit, 30-sample                     │
                                moving average filter]◄──────────────────────┘
                                         |
                              [Blynk IoT Dashboard]
                              (V1 = pH, V2 = TDS, V3 = NTU)
```

### Microcontroller — ESP32

| Parameter | Detail |
|---|---|
| Model | ESP32 (38-pin or 30-pin DevKit) |
| ADC resolution | 12-bit (0–4095) |
| ADC attenuation | 11dB (for 0–3.3V full range) |
| WiFi | 802.11 b/g/n built-in — used for Blynk dashboard |
| Power supply | 5V via USB or regulated 5V from battery |
| Sensor pins | pH → GPIO 36, TDS → GPIO 34, Turbidity → GPIO 39 |
| Firmware language | C++ (Arduino IDE with ESP32 board package) |

The ESP32 was chosen over the ESP8266 for its faster processor (240 MHz dual-core vs 80 MHz single-core), better ADC performance, and greater number of GPIO pins — important as the sensor count grows. All three sensor pins (GPIO 36, 34, 39) are input-only ADC pins, which is correct — never connect output signals to these pins.

### Power System

HydroPod uses a **dual power architecture** to separate the high-current motor load from the sensitive sensor and controller circuitry:

| Rail | Voltage | Capacity | Powers |
|---|---|---|---|
| Motor rail | 12V | 6A | 120 PSI high-pressure RO pump |
| Logic rail | 5V | ~1A | ESP32 microcontroller, all sensors |

This separation is important. The 120 PSI pump draws significant current (several amps at startup), which causes voltage spikes and noise on shared power lines. Running the ESP32 and sensors on a clean, separate 5V rail prevents ADC noise and erratic sensor readings caused by motor-induced electrical interference.

### Sensors

#### pH Sensor

| Parameter | Detail |
|---|---|
| Pin | GPIO 36 (ADC1_CH0) |
| Output type | Analog voltage (0–3.3V) |
| Blynk virtual pin | V1 |
| Measurement range | 0–14 pH |
| Calibration | Voltage-to-pH linear mapping: `ph = 7.5 + (0.72 - voltage) × 3.0 - 3` |
| Safe water range | 6.5–8.5 pH (WHO standard) |

The pH sensor outputs an analog voltage proportional to the hydrogen ion concentration of the water. The ESP32's ADC reads this voltage, which is then mapped to a pH value using a linear calibration equation. Calibration should be verified using **pH buffer solutions (pH 4.0 and pH 7.0)** before field deployment — the calibration constants in the firmware (`0.72`, `3.0`) may need adjustment for your specific sensor module.

#### TDS Sensor (Total Dissolved Solids)

| Parameter | Detail |
|---|---|
| Pin | GPIO 34 (ADC1_CH6) |
| Output type | Analog voltage |
| Blynk virtual pin | V2 |
| Unit | ppm (parts per million) |
| Calibration | `tds = (adc - 530) × 0.8 - 1935` |
| Safe drinking water | <500 ppm (BIS standard), <300 ppm (ideal) |
| Post-RO expected | 10–50 ppm (good RO performance) |

TDS measures the total concentration of dissolved solids — minerals, salts, and metals. It is the primary indicator of whether the RO membrane is functioning correctly. Post-RO TDS should be significantly lower than input TDS; if it isn't, the membrane may be fouled or damaged. The raw ADC value is linearly mapped to ppm using empirically derived calibration constants.

#### Turbidity Sensor

| Parameter | Detail |
|---|---|
| Pin | GPIO 39 (ADC1_CH3) |
| Output type | Analog voltage (0–3.3V) |
| Blynk virtual pin | V3 |
| Unit | NTU (Nephelometric Turbidity Units) |
| Calibration | Quadratic: `ntu = -1120.4×V² + 5742.3×V - 4352.9 - 2623` |
| Safe drinking water | <1 NTU (WHO), <4 NTU (EPA) |

Turbidity measures water clarity by detecting how much light is scattered by suspended particles. The relationship between voltage and NTU is non-linear (quadratic), so the firmware uses a polynomial calibration equation. Clear water has low turbidity (close to 0 NTU). High turbidity indicates the sediment filter may be saturated and needs replacement, or the water source has high particulate content.

### Wiring Summary

| Sensor | VCC | GND | Signal Pin (ESP32) |
|---|---|---|---|
| pH sensor module | 5V | GND | GPIO 36 |
| TDS sensor module | 5V | GND | GPIO 34 |
| Turbidity sensor | 5V | GND | GPIO 39 |
| ESP32 | 5V (USB/regulator) | GND | — |
| 120 PSI pump | 12V | GND | — (direct, not ESP32 controlled) |

> **Important:** GPIO 36, 34, and 39 on the ESP32 are input-only pins — they do not have internal pull-up/pull-down resistors and cannot source or sink current. Use them only for analog sensor signal readings.

---

## Section 3 — Firmware & Blynk Dashboard

The firmware runs on the ESP32 and handles sensor reading, calibration, serial debugging, and real-time data transmission to the Blynk IoT dashboard.

### Blynk Dashboard Setup

| Parameter | Value |
|---|---|
| Platform | Blynk IoT (blynk.cloud) |
| Template ID | `TMPL36t1Z9h_i` |
| Template Name | `Hydropod` |
| Virtual Pin — pH | V1 |
| Virtual Pin — TDS | V2 |
| Virtual Pin — Turbidity | V3 |
| Update interval | Every 5 seconds |

To set up the dashboard:
1. Create a free account at [blynk.cloud](https://blynk.cloud)
2. Create a new template with the Template ID and Name above
3. Add three **Gauge** or **Value Display** widgets: V1 (pH, range 0–14), V2 (TDS, range 0–1000 ppm), V3 (Turbidity, range 0–1000 NTU)
4. Copy your Auth Token into the firmware before uploading

### Complete Firmware

```cpp
#define BLYNK_TEMPLATE_ID   "TMPL36t1Z9h_i"
#define BLYNK_TEMPLATE_NAME "Hydropod"
#define BLYNK_AUTH_TOKEN "YOUR_BLYNK_AUTH_TOKEN"

#include <WiFi.h>
#include <BlynkSimpleEsp32.h>

// ── Pin definitions ──────────────────────────────────────
const int PH_PIN        = 36;   // ADC1_CH0 — pH sensor analog output
const int TDS_PIN       = 34;   // ADC1_CH6 — TDS sensor analog output
const int TURBIDITY_PIN = 39;   // ADC1_CH3 — Turbidity sensor analog output

// ── WiFi credentials ─────────────────────────────────────
char ssid[] = "YOUR_WIFI_NAME";
char pass[] = "YOUR_WIFI_PASSWORD";

BlynkTimer timer;

// ── ADC averaging (noise reduction) ──────────────────────
// Reads the ADC pin 30 times and returns the average.
// This reduces electrical noise from the sensor signal lines.
int readADC(int pin) {
  long sum = 0;
  for (int i = 0; i < 30; i++) {
    sum += analogRead(pin);
    delay(5);       // 5ms between samples = 150ms total per sensor read
  }
  return sum / 30;
}

// ── pH calculation ────────────────────────────────────────
// Converts ADC value to pH using linear voltage-to-pH calibration.
// Calibration equation: ph = 7.5 + (0.72 - voltage) * 3.0 - 3
// Adjust the constants (0.72 and 3.0) using pH 4.0 and 7.0 buffer solutions.
float getPH() {
  int adc     = readADC(PH_PIN);
  float voltage = adc * 3.3 / 4095.0;        // 12-bit ADC, 3.3V reference
  float ph    = 7.5 + (0.72 - voltage) * 3.0 - 3;
  ph = constrain(ph, 0.0, 14.0);             // Clamp to valid pH range
  return ph;
}

// ── TDS calculation ───────────────────────────────────────
// Converts ADC value to ppm using linear calibration.
// Calibration equation: tds = (adc - 530) * 0.8 - 1935
// Adjust offset (530) and slope (0.8) using a known TDS reference solution.
float getTDS() {
  int adc  = readADC(TDS_PIN);
  float tds = (adc - 530) * 0.8 - 1935;
  if (tds < 0) tds = 0;
  return tds;
}

// ── Turbidity calculation ─────────────────────────────────
// Converts ADC value to NTU using quadratic calibration.
// Non-linear response requires polynomial equation for accuracy.
// Offset (-2623) is an empirical correction for this sensor module.
float getTurbidity() {
  int adc       = readADC(TURBIDITY_PIN);
  float voltage   = adc * 3.3 / 4095.0;
  float ntu     = -1120.4 * voltage * voltage
                +  5742.3 * voltage
                -  4352.9;
  ntu = ntu - 2623;         // Empirical offset for this module
  if (ntu < 0) ntu = 0;
  return ntu;
}

// ── Main send function (called every 5 seconds) ───────────
void sendToBlynk() {
  // Read raw ADC values for debugging
  int phADC   = readADC(PH_PIN);
  int tdsADC  = readADC(TDS_PIN);
  int turbADC = readADC(TURBIDITY_PIN);

  // Calculate calibrated values
  float ph  = getPH();
  float tds = getTDS();
  float ntu = getTurbidity();

  // Serial debug output
  Serial.println("\n======================");
  Serial.print("PH  ADC  = "); Serial.println(phADC);
  Serial.print("TDS ADC  = "); Serial.println(tdsADC);
  Serial.print("TURB ADC = "); Serial.println(turbADC);
  Serial.print("pH       = "); Serial.println(ph, 2);
  Serial.print("TDS      = "); Serial.print(tds);   Serial.println(" ppm");
  Serial.print("Turbidity= "); Serial.print(ntu);   Serial.println(" NTU");
  Serial.println("======================");

  // Send to Blynk dashboard virtual pins
  Blynk.virtualWrite(V1, ph);   // pH gauge
  Blynk.virtualWrite(V2, tds);  // TDS gauge
  Blynk.virtualWrite(V3, ntu);  // Turbidity gauge
}

// ── Setup ─────────────────────────────────────────────────
void setup() {
  Serial.begin(115200);

  // Configure ESP32 ADC: 12-bit resolution, 11dB attenuation (0–3.3V range)
  analogReadResolution(12);
  analogSetPinAttenuation(PH_PIN,        ADC_11db);
  analogSetPinAttenuation(TDS_PIN,       ADC_11db);
  analogSetPinAttenuation(TURBIDITY_PIN, ADC_11db);

  // Connect to WiFi and Blynk server
  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);

  // Set up timer to call sendToBlynk() every 5 seconds
  timer.setInterval(5000L, sendToBlynk);
}

// ── Loop ──────────────────────────────────────────────────
void loop() {
  Blynk.run();    // Maintains Blynk connection
  timer.run();    // Triggers sendToBlynk() on schedule
}
```

### How the firmware works — step by step

**1. ADC averaging (`readADC`)**
Each sensor is read 30 times with a 5ms delay between samples. The average is returned. This removes high-frequency electrical noise from the motor and power supply, which would otherwise cause wildly fluctuating readings. Total read time per sensor: ~150ms.

**2. Calibration equations**
Each sensor has a unique voltage-to-parameter mapping:
- **pH** uses a linear equation calibrated around the neutral point (pH 7.0). The constants `0.72` (neutral voltage) and `3.0` (slope) should be verified with buffer solutions.
- **TDS** uses a linear equation where `530` is the ADC zero-offset and `0.8` is the scaling factor. Calibrate against a known TDS reference solution (e.g., 500 ppm NaCl solution).
- **Turbidity** uses a quadratic equation because the sensor's voltage response is non-linear. The `-2623` offset is an empirical correction for the specific module used.

**3. Blynk transmission**
Every 5 seconds, `sendToBlynk()` fires, calculates all three values, prints them to Serial for debugging, and pushes them to Blynk virtual pins V1, V2, V3. The Blynk app displays these on gauges in real time.

### Installing dependencies (Arduino IDE)

```
1. Install ESP32 board package:
   Arduino IDE → File → Preferences → Additional Boards Manager URLs:
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json

2. Install Blynk library:
   Arduino IDE → Tools → Manage Libraries → Search "Blynk" → Install "Blynk by Volodymyr Shymanskyy"

3. Select board:
   Tools → Board → ESP32 Arduino → ESP32 Dev Module

4. Select port:
   Tools → Port → (your ESP32 COM port)

5. Upload speed: 115200 baud
```

### Uploading the code

```bash
# Clone this repository
git clone https://github.com/Harshvt8585/hydropod.git

# Open the firmware file
# Arduino IDE → File → Open → hydropod/firmware/hydropod_main/hydropod_main.ino

# Update WiFi credentials before uploading:
char ssid[] = "YOUR_WIFI_NAME";
char pass[] = "YOUR_WIFI_PASSWORD";

# Upload to ESP32 and open Serial Monitor at 115200 baud to verify readings
```

### Serial monitor — expected output

```
======================
PH  ADC  = 892
TDS ADC  = 1847
TURB ADC = 2103
pH       = 7.24
TDS      = 342.60 ppm
Turbidity= 18.40 NTU
======================
```

### Safe water thresholds (WHO / BIS standards)

| Parameter | Safe Range | Warning Threshold |
|---|---|---|
| pH | 6.5 – 8.5 | < 6.0 or > 9.0 |
| TDS | < 500 ppm | > 500 ppm |
| Turbidity | < 1 NTU | > 4 NTU |

---

## System Architecture

```
[Raw Water Input]
      |
[Sediment Filter] → [Activated Carbon Filter] → [RO Membrane] → [Clean Water Output]
                                                      ↑
                                           [120 PSI Pump ← 12V 6A Battery]

[Sensor Array — inline with output water]
  ├── pH Sensor      (GPIO 36) ──┐
  ├── TDS Sensor     (GPIO 34) ──┤──► [ESP32] ──► [Blynk Cloud] ──► [Mobile Dashboard]
  └── Turbidity      (GPIO 39) ──┘        ↑
                                    [5V Power Supply]
```

---

## Sample Output

| Timestamp | pH | TDS (ppm) | Turbidity (NTU) | Status |
|---|---|---|---|---|
| 00:00:01 | 7.24 | 342 | 1.2 | SAFE |
| 00:00:06 | 7.21 | 338 | 1.4 | SAFE |
| 00:00:11 | 6.45 | 521 | 1.3 | WARNING: pH LOW, TDS HIGH |
| 00:00:16 | 7.18 | 290 | 8.7 | WARNING: TURBIDITY HIGH |

---

## Project Status

- [x] ESP32 firmware — sensor reading and serial output
- [x] pH sensor integration and calibration equation
- [x] TDS sensor integration and calibration equation
- [x] Turbidity sensor integration and calibration equation
- [x] 30-sample ADC averaging for noise reduction
- [x] Blynk IoT dashboard — V1 (pH), V2 (TDS), V3 (Turbidity)
- [x] 3-stage filtration system design (sediment, carbon, RO)
- [x] Dual power system (12V motor rail, 5V logic rail)
- [ ] Hardware threshold alerts (buzzer / LED on unsafe readings)
- [ ] Temperature sensor integration
- [ ] Flow rate sensor integration
- [ ] Automatic pump control via ESP32
- [ ] OTA (over-the-air) firmware updates
- [ ] 3D-printed compact enclosure
- [ ] Field testing in varied water conditions

---

## Impact & Target Users

HydroPod is designed for communities and situations where clean water access is unreliable:

- Remote and rural communities without water treatment infrastructure
- Disaster relief and humanitarian aid operations
- Military and expedition use cases
- Urban emergency backup and portable water testing

**Social impact:** Reduces waterborne diseases like cholera and typhoid in underserved communities. Puts lab-grade water analysis in a portable, affordable device.

---

## Recognition

Submitted to **Smart India Hackathon 2025 (SIH 2025)**
Problem Statement ID: **25085**
Problem Statement: *Non-Revenue Loss in Water Supply — Awareness in Water Conservation, Treatment of Waste Water and Reuse for Domestic Purposes*
Theme: Miscellaneous | Category: **Hardware**
Team: **Hydroguard**

---

## References

- S. R. Perumal & M. S. R. Khan, *"Low-Cost Water Quality Monitoring and Filtration System"*, 2021 IEEE IEMCON
- *"Development of a Portable Multi-Parameter Water Quality Monitoring System using IoT"*
- ESP32 Technical Reference Manual — Espressif Systems
- Blynk IoT Documentation — blynk.cloud/documentation
- WHO Guidelines for Drinking-water Quality (4th Edition)
- BIS IS 10500:2012 — Indian Standard for Drinking Water

---

## About

**Harsh** — B.Tech ECE, 2nd Year
Lloyd Institute of Engineering and Technology (LIET), AKTU
Interests: Embedded Systems, IoT, Hardware-Software Integration

[LinkedIn](www.linkedin.com/in/harsh-vardhan-78671335a) · [GitHub](https://github.com/Harshvt8585)

---

*Contributions, suggestions, and collaborations welcome. This project is actively under development.* 
