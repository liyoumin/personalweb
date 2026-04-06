---
title: How Drought Shocks Alfalfa Production and Export? Evidence from U.S. Alfalfa Spatial Diagnostics and Panel Analysis
date: 2025-10-26
# Link directly to your PDF on Overleaf
url_pdf: "/personalweb/project/scikit/Drought_impacts_on_alfalfa.pdf" 
url_slides: "/personalweb/project/scikit/Presentation_alfalfa.pdf" 
url_poster: "/personalweb/project/scikit/poster.pdf" 
url_source: "/personalweb/project/scikit/AAEADrought_alfalfa.pdf" 
url_video: "/personalweb/project/scikit/SimpleSDI.MP4" 
url_code: "" 
url_dataset: "/personalweb/project/scikit/Irrigation output.png" 
url_project: "/personalweb/project/scikit/Motivation.png"

tags:
  - Smart Agriculture
  - Geo-computation
  - Drought
  - C++ (Arduino)
  - IoT
  - Irrigation
---


## Introduction and background:
Alfalfa production in Western semi-arid states often faces high evaporative demand and tight irrigation constraints. Smart Drip Irrigation (SDI) systems can improve water efficiency by coupling real-time sensing with automatic control. This mini project demonstrates the core SDI control loop—sense → decide → actuate → display/log—using low-cost hardware. In this simplified version (based on available parts), the system uses temperature and relative humidity (DHT11) as a “hot & dry” trigger to simulate irrigation scheduling and proves actuation through a relay signal, a status LED, and a LCD message. An ESP-32S receives telemetry for future IoT extension (dashboard/cloud), while the Mega2560 remains the reliable control layer.

## Counter measure:
Build a simple Smart Agriculture IoT controller with Arduino Mega2560 + DHT11 + LCD1602A + Relay, and stream telemetry to ESP-32S.

## Video

This example project builds a Smart Drip Irrigation (SDI) controller for alfalfa production in Western semi-arid states. The system reads soil moisture and air temperature/humidity to decide when to turn a water pump (pressure irrigation) on/off. The goal is to maintain soil moisture within a target range (avoid under-watering and over-watering) while adapting irrigation timing to hot/dry conditions that increase evapotranspiration. The controller logs sensor data and pump status for monitoring and debugging.

{{< youtube ciD3ILxgXzU >}}


**Video file**

<!-- Videos in `assets/media/` media library or in [page's folder](https://gohugo.io/content-management/page-bundles/), then embedding them with the _video_ shortcode: -->

    {{</* video src="SimpleSDI.mp4" controls="yes" */>}}


## Core functions
```
Soil moisture monitoring (Sensor #1)
Read analog soil moisture level and convert to a % using calibration.
Microclimate monitoring (Sensor #2)
Read temperature and relative humidity (DHT11/DHT22).
Automatic irrigation control
Pump ON/OFF using a relay module with hysteresis thresholds:
ON when moisture < LOW
OFF when moisture > HIGH
Semi-arid “smart” logic
If hot + dry, irrigation triggers slightly earlier (threshold bias).
If cool + humid, irrigation triggers later.
Safety + reliability
Maximum pump run time per event (prevents flooding).
Minimum off time between events (prevents relay “chattering”).
Sensor fault → fail-safe pump OFF.

Optional upgrades
LCD display (1602 I2C or parallel): show Soil%, Temp, RH, Pump state
Data logging to SD card module (CSV file)
RTC clock module for time-stamped logs and “no watering at midday” rule
ESP-32D: send data to Wi-Fi dashboard / phone (IoT extension)
```

## Hardware

   Soil Moisture Sensor (Analog) -----> A0 (Mega)
   Water level --------------> A1
   DHT Temp/RH (Digital)  -----------> D2 (Mega)
   Mega D8  -----------------------> Relay IN  -----> Pump power circuit
   Mega GND ------------------------> Breadboard GND
   Mega 5V  ------------------------> Breadboard positive 
   Pump + Irrigation:
   Water tank -> Pump -> Filter -> Drip line -> Alfalfa plot
   LCD (I2C)  SDA/SCL -> Mega SDA/SCL; V0 -> potentionmeter
   SD module  CS/MOSI/MISO/SCK -> Mega SPI pins
   ESP-32D:    WiFi upload data and commands send from phone (MEGA RX1-ESP TX; MEGA TX1-Voltage divide-ESP RX)
   Pinpad instruction: A -> starts irrigation; B -> stops pump; C -> forces MANUAL Mod; 
   number + # -> set irrigation volum mL; D -> forces AUTO

## Code (C++)
```cpp
#include <DHT.h>
#include <LiquidCrystal.h>
#include <Keypad.h>

// ===================== PINS =====================
const int PIN_SOIL      = A0;
const int PIN_WL        = A1;
const int PIN_DHT       = 2;
const int PIN_RELAY     = 8;
const int PIN_MODE_BTN  = 3;

// LCD1602 4-bit: RS, E, D4, D5, D6, D7
LiquidCrystal lcd(22, 23, 24, 25, 26, 27);

// ===================== DHT ======================
#define DHTTYPE DHT11
DHT dht(PIN_DHT, DHTTYPE);

// ===================== KEYPAD ===================
const byte ROWS = 4;
const byte COLS = 4;

char keys[ROWS][COLS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};

byte rowPins[ROWS] = {30, 31, 32, 33};
byte colPins[COLS] = {34, 35, 36, 37};

Keypad keypad = Keypad(makeKeymap(keys), rowPins, colPins, ROWS, COLS);
```

{{< spoiler text="👉 Click to view the full code" >}}
```cpp
// ===================== TIMING ===================
unsigned long lastReadMs      = 0;
unsigned long lastSendMs      = 0;
unsigned long modeBtnLastMs   = 0;
unsigned long manualStartMs   = 0;
unsigned long pumpStartMs     = 0;
unsigned long lastPumpOffMs   = 0;

const unsigned long READ_INTERVAL_MS = 3000;
const unsigned long SEND_INTERVAL_MS = 5000;
const unsigned long DEBOUNCE_MS      = 250;
const unsigned long MAX_ON_MS        = 10UL * 60UL * 1000UL; // 10 min
const unsigned long MIN_OFF_MS       = 10UL * 1000UL;        // 10 sec protect relay

// ===================== CALIBRATION ==============
// Soil sensor calibration
int soilDryADC = 850;   // update after calibration
int soilWetADC = 350;   // update after calibration

// Water level calibration
int wlLowADC  = 180;    // low/empty water level
int wlHighADC = 650;    // high/full water level
int wlSafeMinADC = 220; // below this, pump will not run

// Auto irrigation thresholds
float baseLOW  = 30.0;  // auto ON below this
float baseHIGH = 40.0;  // auto OFF above this

// Semi-arid environmental adjustment
float hotC  = 30.0;
float dryRH = 30.0;
float bias  = 3.0;

// Pump flow estimate for manual / remote volume control
// Example: if pump fills 250 mL in 10 seconds, then flow = 25 mL/s
const float PUMP_FLOW_ML_PER_SEC = 25.0;

// Relay polarity
const bool RELAY_ACTIVE_LOW = false;

// ===================== STATE ====================
bool pumpOn = false;
bool autoMode = true;
bool manualRunning = false;

float soilPct = NAN;
float T = NAN;
float RH = NAN;
float LOW_T = NAN;
float HIGH_T = NAN;

int soilADC = 0;
int wlADC   = 0;
float wlPct = NAN;

// manual / remote target irrigation volume
unsigned int targetVolumeML = 100;   // default
String keypadBuffer = "";
unsigned long manualRunDurationMs = 0;

// ===================== HELPERS ==================
void relayWrite(bool on) {
  if (!RELAY_ACTIVE_LOW) digitalWrite(PIN_RELAY, on ? HIGH : LOW);
  else                   digitalWrite(PIN_RELAY, on ? LOW : HIGH);
}

float mapToPercent(int adc, int dryVal, int wetVal) {
  int lo = min(wetVal, dryVal);
  int hi = max(wetVal, dryVal);
  adc = constrain(adc, lo, hi);

  float pct = 100.0 * (float)(dryVal - adc) / (float)(dryVal - wetVal);
  return constrain(pct, 0.0, 100.0);
}

float soilAdcToPercent(int adc) {
  return mapToPercent(adc, soilDryADC, soilWetADC);
}

float wlAdcToPercent(int adc) {
  int lo = min(wlLowADC, wlHighADC);
  int hi = max(wlLowADC, wlHighADC);
  adc = constrain(adc, lo, hi);

  float pct = 100.0 * (float)(adc - wlLowADC) / (float)(wlHighADC - wlLowADC);
  return constrain(pct, 0.0, 100.0);
}

void computeThresholds(float tC, float rh, float &lowT, float &highT) {
  lowT = baseLOW;
  highT = baseHIGH;

  if (!isnan(tC) && !isnan(rh) && tC >= hotC && rh <= dryRH) {
    lowT  += bias;
    highT += bias;
  }
}

bool waterLevelSafe() {
  return (wlADC >= wlSafeMinADC);
}

void setPump(bool on) {
  if (on == pumpOn) return;

  if (on) {
    // safety: low water level prevents pump start
    if (!waterLevelSafe()) return;

    // protect relay/pump from rapid cycling
    if (millis() - lastPumpOffMs < MIN_OFF_MS) return;

    pumpOn = true;
    pumpStartMs = millis();
    relayWrite(true);
  } else {
    pumpOn = false;
    manualRunning = false;
    lastPumpOffMs = millis();
    relayWrite(false);
  }
}

void readSensors() {
  soilADC = analogRead(PIN_SOIL);
  wlADC   = analogRead(PIN_WL);

  soilPct = soilAdcToPercent(soilADC);
  wlPct   = wlAdcToPercent(wlADC);

  float newRH = dht.readHumidity();
  float newT  = dht.readTemperature();

  // keep last good DHT values
  if (!isnan(newRH) && !isnan(newT)) {
    RH = newRH;
    T = newT;
  }

  computeThresholds(T, RH, LOW_T, HIGH_T);
}

void autoControlLogic() {
  // hard safety: stop if low water level
  if (!waterLevelSafe()) {
    setPump(false);
    return;
  }

  // max runtime safety
  if (pumpOn && (millis() - pumpStartMs > MAX_ON_MS)) {
    setPump(false);
    return;
  }

  if (isnan(soilPct)) {
    setPump(false);
    return;
  }

  // hysteresis control
  if (!pumpOn && soilPct < LOW_T) setPump(true);
  else if (pumpOn && soilPct > HIGH_T) setPump(false);
}

void startManualIrrigation(unsigned int volML) {
  if (volML == 0) return;
  if (!waterLevelSafe()) return;

  manualRunDurationMs = (unsigned long)((volML / PUMP_FLOW_ML_PER_SEC) * 1000.0);
  manualStartMs = millis();
  manualRunning = true;
  setPump(true);
}

void handleManualIrrigation() {
  // stop if tank too low during run
  if (!waterLevelSafe()) {
    setPump(false);
    return;
  }

  if (manualRunning) {
    if (millis() - manualStartMs >= manualRunDurationMs) {
      setPump(false);
    }
  }

  if (pumpOn && (millis() - pumpStartMs > MAX_ON_MS)) {
    setPump(false);
  }
}

void toggleMode() {
  autoMode = !autoMode;
  manualRunning = false;
  setPump(false);
}

void handleModeButton() {
  if (digitalRead(PIN_MODE_BTN) == LOW) {
    if (millis() - modeBtnLastMs > DEBOUNCE_MS) {
      modeBtnLastMs = millis();
      toggleMode();
    }
  }
}

void handleKeypad() {
  char key = keypad.getKey();
  if (!key) return;

  Serial.print("Key=");
  Serial.println(key);

  if (key >= '0' && key <= '9') {
    if (keypadBuffer.length() < 4) keypadBuffer += key;
    return;
  }

  switch (key) {
    case '*':   // clear entry
      keypadBuffer = "";
      break;

    case '#':   // save entered volume
      if (keypadBuffer.length() > 0) {
        targetVolumeML = keypadBuffer.toInt();
        keypadBuffer = "";
      }
      break;

    case 'A':   // start manual irrigation
      if (!autoMode) {
        startManualIrrigation(targetVolumeML);
      }
      break;

    case 'B':   // stop pump
      setPump(false);
      break;

    case 'D':   // force AUTO mode
      autoMode = true;
      manualRunning = false;
      setPump(false);
      break;

    case 'C':   // force MANUAL mode
      autoMode = false;
      setPump(false);
      break;
  }
}

// ===================== ESP32 SERIAL COMMANDS ==================
void handleRemoteCommand(String cmd) {
  cmd.trim();
  cmd.toUpperCase();

  if (cmd == "AUTO") {
    autoMode = true;
    manualRunning = false;
    setPump(false);
  }
  else if (cmd == "MAN") {
    autoMode = false;
    manualRunning = false;
    setPump(false);
  }
  else if (cmd == "STOP") {
    manualRunning = false;
    setPump(false);
  }
  else if (cmd.startsWith("VOL=")) {
    String valueStr = cmd.substring(4);
    unsigned int vol = valueStr.toInt();
    if (vol > 0) {
      targetVolumeML = vol;
    }
  }
  else if (cmd == "START") {
    if (!autoMode) {
      startManualIrrigation(targetVolumeML);
    }
  }
}

void readESP32Commands() {
  while (Serial1.available()) {
    String cmd = Serial1.readStringUntil('\n');
    Serial.print("ESP32 CMD: ");
    Serial.println(cmd);
    handleRemoteCommand(cmd);
  }
}

// ===================== LCD ======================
void lcdStartup() {
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("Smart Irrig.");
  lcd.setCursor(0,1);
  lcd.print("Mega2560 Ready");
  delay(1500);
  lcd.clear();
}

void updateLCD() {
  lcd.clear();

  lcd.setCursor(0,0);
  lcd.print("IRR:");
  lcd.print(pumpOn ? "ON " : "OFF");
  lcd.print(" S:");
  lcd.print((int)soilPct);
  lcd.print("%");

  lcd.setCursor(0,1);
  lcd.print("WL:");
  lcd.print(wlADC);
  lcd.print(" ");
  lcd.print(autoMode ? "A" : "M");
}

// ===================== TELEMETRY =================
void telemetry() {
  // PC serial monitor
  Serial.print("soil_raw=");  Serial.print(soilADC);
  Serial.print(",soil_pct="); Serial.print(soilPct,1);
  Serial.print(",wl_raw=");   Serial.print(wlADC);
  Serial.print(",wl_pct=");   Serial.print(wlPct,1);
  Serial.print(",temp_c=");   Serial.print(T,1);
  Serial.print(",humidity="); Serial.print(RH,1);
  Serial.print(",pump=");     Serial.print(pumpOn ? "ON" : "OFF");
  Serial.print(",mode=");     Serial.print(autoMode ? "AUTO" : "MAN");
  Serial.print(",low=");      Serial.print(LOW_T,1);
  Serial.print(",high=");     Serial.print(HIGH_T,1);
  Serial.print(",vol_ml=");   Serial.print(targetVolumeML);
  Serial.print(",manual=");   Serial.println(manualRunning ? 1 : 0);

  // ESP32 serial
  Serial1.print("mode=");
  Serial1.print(autoMode ? "AUTO" : "MAN");
  Serial1.print(",soil_raw=");
  Serial1.print(soilADC);
  Serial1.print(",soil_pct=");
  Serial1.print(soilPct,1);
  Serial1.print(",wl_raw=");
  Serial1.print(wlADC);
  Serial1.print(",wl_pct=");
  Serial1.print(wlPct,1);
  Serial1.print(",temp_c=");
  Serial1.print(T,1);
  Serial1.print(",humidity=");
  Serial1.print(RH,1);
  Serial1.print(",pump=");
  Serial1.print(pumpOn ? "ON" : "OFF");
  Serial1.print(",vol_ml=");
  Serial1.print(targetVolumeML);
  Serial1.print(",manual=");
  Serial1.println(manualRunning ? 1 : 0);
}

// ===================== SETUP ====================
void setup() {
  Serial.begin(9600);
  Serial1.begin(9600);   // to/from ESP32

  pinMode(PIN_RELAY, OUTPUT);
  relayWrite(false);

  pinMode(PIN_MODE_BTN, INPUT_PULLUP);

  dht.begin();
  delay(1000);

  lcd.begin(16, 2);
  lcdStartup();

  readSensors();
  updateLCD();
}

// ===================== LOOP =====================
void loop() {
  unsigned long now = millis();

  // remote commands from ESP32
  readESP32Commands();

  // local controls
  handleModeButton();
  handleKeypad();

  if (now - lastReadMs >= READ_INTERVAL_MS) {
    lastReadMs = now;
    readSensors();

    if (autoMode) {
      autoControlLogic();
    } else {
      handleManualIrrigation();
    }
  }

  if (!autoMode) {
    handleManualIrrigation();
  }

  updateLCD();

  if (now - lastSendMs >= SEND_INTERVAL_MS) {
    lastSendMs = now;
    telemetry();
  }
}
```
{{< /spoiler >}}

## Inline Images
Arduino
{{< icon name="c++" >}} C++
  
<!--more-->
