---
title: How Drought Shocks Alfalfa Production and Export? Evidence from U.S. Alfalfa Spatial Diagnostics and Panel Analysis
date: 2025-10-26
# Link directly to your PDF 
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
Adopte a Smart Agriculture IoT controller with Arduino Mega2560 + DHT11 + LCD1602A + Relay, and stream telemetry to ESP-32S.

## Video

This project builds a Smart Drip Irrigation (SDI) controller for alfalfa production in Western semi-arid states. The system reads soil moisture and air temperature/humidity to decide when to turn a water pump (pressure irrigation) on/off. ESP-32 can receive the Opean Weather live data in a larger scale and receive commands from phone. The goal is to maintain soil moisture within a target range (avoid under-watering and over-watering) while adapting irrigation timing to hot/dry conditions that increase evapotranspiration. The controller logs sensor data and pump status for monitoring and debugging.

{{< youtube 37SznnXZTIg >}}

Here is a tutorial example:
{{< youtube ciD3ILxgXzU >}}


**Video file**
<!-- Videos in `assets/media/` media library or in [page's folder]: -->
    {{</* video src="smartSDI.mp4" controls="yes" */>}}


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
LCD display (1602 I2C or parallel): show Soil%, Temp, RH, Pump state
Data logging to SD card module (CSV file)
RTC clock module for time-stamped logs and “no watering at midday” rule
ESP-32D: send live data to Wi-Fi dashboard / phone (IoT extension)
ESP-32: receive OpenWeather data and upload it with other live to Thingspeak cloud platform.
```

## Hardware
```
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
   ESP-32D:    WiFi upload data and commands send from phone (MEGA RX0-ESP TX; MEGA TX0-ESP RX)
   Pinpad instruction: A -> starts irrigation; B -> stops pump; C -> forces MANUAL Mod; 
   number + # -> set irrigation volum mL; D -> forces AUTO
```

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



{{< spoiler text="👉 Click to view the ESP data read" >}}
```c++
/*
  ESP32 Dev Module code for Smart Irrigation System
  -------------------------------------------------
  Mega2560 protocol. 192.168.4.1

  Functions:
  - Reads telemetry from Mega2560 over UART
  - Hosts a local web dashboard on Wi-Fi
  - Sends commands back to Mega:
      AUTO
      MAN
      START
      STOP
      VOL=xxx

  ESP32 board: "ESP32 Dev Module"
*/

#include <WiFi.h>
#include <WebServer.h>

// ===================== WIFI =====================
// Option 1: fill in your Wi-Fi credentials
const char* WIFI_SSID = "iphone Youmin li";
const char* WIFI_PASS = "12345678";

// Option 2: if WIFI_SSID is blank, ESP32 starts its own AP
const char* AP_SSID   = "SmartIrrigation-Youmin";
const char* AP_PASS   = "12345678";

// ===================== UART TO MEGA =====================
// ESP32 Serial2 pins (change if your wiring is different)
static const int MEGA_RX_PIN = 16;   // ESP32 RX2  <- Mega TX1 (through voltage divider)
static const int MEGA_TX_PIN = 17;   // ESP32 TX2  -> Mega RX1
static const uint32_t MEGA_BAUD = 9600;

HardwareSerial MegaSerial(2);
WebServer server(80);

// ===================== TELEMETRY STATE =====================
struct Telemetry {
  String mode = "NA";
  int soil_raw = 0;
  float soil_pct = NAN;
  int wl_raw = 0;
  float wl_pct = NAN;
  float temp_c = NAN;
  float humidity = NAN;
  String pump = "OFF";
  unsigned int vol_ml = 100;
  int manual = 0;
  unsigned long lastUpdateMs = 0;
} tel;

String rxBuffer;

// ===================== HELPERS =====================
bool telemetryFresh() {
  return (millis() - tel.lastUpdateMs) < 15000UL;
}

String fmtFloat(float x, int digits = 1) {
  if (isnan(x)) return "--";
  return String(x, digits);
}

String jsonFloat(float x, int digits = 1) {
  if (isnan(x)) return "null";
  return String(x, digits);
}

String currentIP() {
  wifi_mode_t mode = WiFi.getMode();
  if (mode == WIFI_AP || mode == WIFI_AP_STA) {
    return WiFi.softAPIP().toString();
  }
  return WiFi.localIP().toString();
}

void sendMegaCommand(String cmd) {
  cmd.trim();
  if (cmd.length() == 0) return;
  MegaSerial.println(cmd);

  Serial.print("TX -> MEGA: ");
  Serial.println(cmd);
}

void updateField(const String& key, const String& val) {
  if (key == "mode") {
    tel.mode = val;
  } else if (key == "soil_raw") {
    tel.soil_raw = val.toInt();
  } else if (key == "soil_pct") {
    tel.soil_pct = val.toFloat();
  } else if (key == "wl_raw") {
    tel.wl_raw = val.toInt();
  } else if (key == "wl_pct") {
    tel.wl_pct = val.toFloat();
  } else if (key == "temp_c") {
    tel.temp_c = val.toFloat();
  } else if (key == "humidity") {
    tel.humidity = val.toFloat();
  } else if (key == "pump") {
    tel.pump = val;
  } else if (key == "vol_ml") {
    tel.vol_ml = (unsigned int)val.toInt();
  } else if (key == "manual") {
    tel.manual = val.toInt();
  }
}

void parseTelemetryLine(String line) {
  line.trim();
  if (line.length() == 0) return;

  int start = 0;
  while (start < line.length()) {
    int comma = line.indexOf(',', start);
    String token;

    if (comma == -1) {
      token = line.substring(start);
      start = line.length();
    } else {
      token = line.substring(start, comma);
      start = comma + 1;
    }

    token.trim();
    int eq = token.indexOf('=');
    if (eq == -1) continue;

    String key = token.substring(0, eq);
    String val = token.substring(eq + 1);
    key.trim();
    val.trim();

    updateField(key, val);
  }

  tel.lastUpdateMs = millis();

  Serial.print("RX <- MEGA: ");
  Serial.println(line);
}

void readMegaSerial() {
  while (MegaSerial.available()) {
    char c = (char)MegaSerial.read();

    if (c == '\r') continue;

    if (c == '\n') {
      if (rxBuffer.length() > 0) {
        parseTelemetryLine(rxBuffer);
        rxBuffer = "";
      }
    } else {
      if (rxBuffer.length() < 300) {
        rxBuffer += c;
      } else {
        rxBuffer = "";
      }
    }
  }
}

// ===================== WIFI =====================
void startAccessPoint() {
  WiFi.mode(WIFI_AP);
  WiFi.softAP(AP_SSID, AP_PASS);

  Serial.println();
  Serial.println("ESP32 started in AP mode.");
  Serial.print("AP SSID: ");
  Serial.println(AP_SSID);
  Serial.print("AP IP: ");
  Serial.println(WiFi.softAPIP());
}

void connectWiFi() {
  if (strlen(WIFI_SSID) == 0) {
    startAccessPoint();
    return;
  }

  WiFi.mode(WIFI_STA);
  WiFi.begin(WIFI_SSID, WIFI_PASS);

  Serial.print("Connecting to Wi-Fi");
  unsigned long t0 = millis();

  while (WiFi.status() != WL_CONNECTED && millis() - t0 < 15000UL) {
    delay(500);
    Serial.print(".");
  }
  Serial.println();

  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("Wi-Fi connected.");
    Serial.print("ESP32 IP: ");
    Serial.println(WiFi.localIP());
  } else {
    Serial.println("Wi-Fi failed. Falling back to AP mode.");
    WiFi.disconnect(true, true);
    delay(500);
    startAccessPoint();
  }
}

// ===================== WEB =====================
String makeJson() {
  String s = "{";
  s += "\"online\":";
  s += (telemetryFresh() ? "true" : "false");
  s += ",\"mode\":\"" + tel.mode + "\"";
  s += ",\"soil_raw\":" + String(tel.soil_raw);
  s += ",\"soil_pct\":" + jsonFloat(tel.soil_pct, 1);
  s += ",\"wl_raw\":" + String(tel.wl_raw);
  s += ",\"wl_pct\":" + jsonFloat(tel.wl_pct, 1);
  s += ",\"temp_c\":" + jsonFloat(tel.temp_c, 1);
  s += ",\"humidity\":" + jsonFloat(tel.humidity, 1);
  s += ",\"pump\":\"" + tel.pump + "\"";
  s += ",\"vol_ml\":" + String(tel.vol_ml);
  s += ",\"manual\":" + String(tel.manual);
  s += "}";
  return s;
}

void handleJson() {
  server.send(200, "application/json", makeJson());
}

void handleCmd() {
  if (!server.hasArg("x")) {
    server.send(400, "text/plain", "Missing x");
    return;
  }

  String cmd = server.arg("x");
  cmd.trim();
  cmd.toUpperCase();

  bool valid = false;

  if (cmd == "AUTO" || cmd == "MAN" || cmd == "START" || cmd == "STOP") {
    valid = true;
  } else if (cmd.startsWith("VOL=")) {
    String n = cmd.substring(4);
    n.trim();
    int v = n.toInt();
    if (v > 0) valid = true;
  }

  if (!valid) {
    server.send(400, "text/plain", "Invalid command");
    return;
  }

  sendMegaCommand(cmd);
  server.send(200, "text/plain", "OK");
}

void handleRoot() {
  String html = R"rawliteral(
<!DOCTYPE html>
<html>
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Smart Irrigation ESP32</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 20px;
      background: #f4f7fb;
      color: #1f2937;
    }
    .card {
      background: white;
      padding: 16px;
      border-radius: 14px;
      box-shadow: 0 2px 10px rgba(0,0,0,.08);
      margin-bottom: 16px;
    }
    .grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
      gap: 12px;
    }
    .item {
      background: #f9fafb;
      border-radius: 10px;
      padding: 10px;
    }
    .label {
      font-size: 12px;
      color: #6b7280;
      margin-bottom: 4px;
    }
    .value {
      font-size: 20px;
      font-weight: 700;
    }
    button {
      padding: 12px 16px;
      border: none;
      border-radius: 10px;
      margin: 6px 6px 6px 0;
      font-size: 15px;
      cursor: pointer;
    }
    input[type=number] {
      padding: 10px;
      border-radius: 8px;
      border: 1px solid #cbd5e1;
      width: 130px;
      font-size: 16px;
    }
    .small {
      color: #6b7280;
      font-size: 13px;
    }
  </style>
</head>
<body>
  <div class="card">
    <h2>Smart Irrigation Dashboard</h2>
    <div class="small">ESP32 IP: <b>)rawliteral";
  html += currentIP();
  html += R"rawliteral(</b></div>
    <div class="small">Refresh every 2 seconds</div>
  </div>

  <div class="card">
    <div class="grid">
      <div class="item"><div class="label">Link</div><div class="value" id="online">--</div></div>
      <div class="item"><div class="label">Mode</div><div class="value" id="mode">--</div></div>
      <div class="item"><div class="label">Pump</div><div class="value" id="pump">--</div></div>
      <div class="item"><div class="label">Manual</div><div class="value" id="manual">--</div></div>
      <div class="item"><div class="label">Soil %</div><div class="value" id="soil_pct">--</div></div>
      <div class="item"><div class="label">Soil Raw</div><div class="value" id="soil_raw">--</div></div>
      <div class="item"><div class="label">Water %</div><div class="value" id="wl_pct">--</div></div>
      <div class="item"><div class="label">Water Raw</div><div class="value" id="wl_raw">--</div></div>
      <div class="item"><div class="label">Temp (C)</div><div class="value" id="temp_c">--</div></div>
      <div class="item"><div class="label">Humidity (%)</div><div class="value" id="humidity">--</div></div>
      <div class="item"><div class="label">Target Volume (mL)</div><div class="value" id="vol_ml">--</div></div>
    </div>
  </div>

  <div class="card">
    <h3>Commands</h3>
    <button onclick="sendCmd('AUTO')">AUTO</button>
    <button onclick="sendCmd('MAN')">MANUAL</button>
    <button onclick="sendCmd('START')">START</button>
    <button onclick="sendCmd('STOP')">STOP</button>
    <br><br>
    <input id="vol" type="number" min="1" value="100">
    <button onclick="setVolume()">Set Volume</button>
  </div>

  <script>
    function showValue(id, val) {
      document.getElementById(id).innerText = (val === null || val === undefined) ? '--' : val;
    }

    async function refreshData() {
      try {
        const res = await fetch('/json');
        const d = await res.json();

        showValue('online', d.online ? 'ONLINE' : 'OFFLINE');
        showValue('mode', d.mode);
        showValue('pump', d.pump);
        showValue('manual', d.manual === 1 ? 'YES' : 'NO');
        showValue('soil_pct', d.soil_pct);
        showValue('soil_raw', d.soil_raw);
        showValue('wl_pct', d.wl_pct);
        showValue('wl_raw', d.wl_raw);
        showValue('temp_c', d.temp_c);
        showValue('humidity', d.humidity);
        showValue('vol_ml', d.vol_ml);
      } catch (e) {
        showValue('online', 'NO DATA');
      }
    }

    async function sendCmd(cmd) {
      await fetch('/cmd?x=' + encodeURIComponent(cmd));
      setTimeout(refreshData, 250);
    }

    function setVolume() {
      const v = document.getElementById('vol').value.trim();
      if (v.length > 0) {
        sendCmd('VOL=' + v);
      }
    }

    refreshData();
    setInterval(refreshData, 2000);
  </script>
</body>
</html>
)rawliteral";

  server.send(200, "text/html", html);
}

void setupWeb() {
  server.on("/", handleRoot);
  server.on("/json", handleJson);
  server.on("/cmd", handleCmd);
  server.begin();

  Serial.println("Web server started.");
}

// ===================== SETUP / LOOP =====================
void setup() {
  Serial.begin(115200);
  delay(800);

  MegaSerial.begin(MEGA_BAUD, SERIAL_8N1, MEGA_RX_PIN, MEGA_TX_PIN);

  Serial.println();
  Serial.println("ESP32 Smart Irrigation Interface Starting...");

  connectWiFi();
  setupWeb();

  Serial.println("Ready.");
}

void loop() {
  readMegaSerial();
  server.handleClient();
}
```
{{< /spoiler >}}


{{< spoiler text="👉 Click to view the ESP Open-Weather read" >}}
```c++
#include <WiFi.h>
#include <HTTPClient.h>
const char* ssid = "iPhone Youmin li";
const char* password = "123456789";
// Use your OpenWeatherMap API key here
String apiKey = "1da5602c98819cc5e77582d0676746f8***";
void setup() {
  // put your setup code here, to run once:
  Serial.begin(115200);
  WiFi.begin(ssid, password);
    while(WiFi.status() != WL_CONNECTED){
     delay(100);
     Serial.println("connecting to WiFi...");
     }
  Serial.println("Connected to the WiFi network");
  Serial.println(WiFi.localIP());
 // Construct the API request URL
 String url = "http://api.openweathermap.org/data/2.5/weather?q=Tampa,FL,US&appid=" + apiKey;
 // Create an HTTPClient object
  HTTPClient http;
    // Begin the request
  http.begin(url);
    // Send the request
  int httpCode = http.GET();
    // Check the returning http code
  if (httpCode > 0) {
    String payload = http.getString();
    Serial.println(payload);
  } else {
    Serial.println("Error on HTTP request");
  }
  http.end(); //End HTTP connection
}
void loop() {
  // put your main code here, to run repeatedly:
}
```
{{< /spoiler >}}

{{< spoiler text="👉 Click to view the ESP upload Cloud Data" >}}
```c++
#define TS_ENABLE_SSL

#include <WiFi.h>
#include <WiFiClientSecure.h>
#include <DHT.h>
#include <ThingSpeak.h>

const char* ssid = "iPhone Youmin li";
const char* password = "12345678";

WiFiClientSecure client;

unsigned long myChannelID = 3347147;
const char * myWriteAPIKey = "V85NE5L7FEY0ZIUU";

// Timer variables
unsigned long lastTime = 0;
unsigned long timerDelay = 30000;

#define DHTPIN 23
#define DHTTYPE DHT11 
DHT dht(DHTPIN, DHTTYPE);

float temperature;

void setup() {
  // put your setup code here, to run once:
  Serial.begin(115200);  //Initialize serial
  WiFi.begin(ssid, password);
      while(WiFi.status() != WL_CONNECTED){
       delay(100);
       Serial.println("connecting to WiFi...");
       }
  Serial.println("Connected to the WiFi network");
  Serial.println(WiFi.localIP()); 
  ThingSpeak.begin(client);  // Initialize ThingSpeak
  dht.begin();
}


void getData(){
  delay(2000);// Wait a few seconds between measurements.
  // Reading temperature or humidity takes about 250 milliseconds!
  // Sensor readings may also be up to 2 seconds ‘old’ (its a very slow sensor)
    temperature = dht.readTemperature();// Read temperature as Celsius (the default)
    if (isnan(temperature) ){
    Serial.println("Failed to read from DHT sensor.");
    return;
     }
  Serial.print("Temperature (C): ");
  Serial.println(temperature);
}


void loop() {
  // put your main code here, to run repeatedly:
if ((millis() - lastTime) > timerDelay) {
  getData();
   int x = ThingSpeak.writeField(myChannelID, 1, temperature, myWriteAPIKey);
   if(x == 200){
      Serial.println("Channel update successful.");
    }
    else{
      Serial.println("Problem updating channel. HTTP error code " + String(x));
    }
    lastTime = millis();
  }
}
```
{{< /spoiler >}}


{{< spoiler text="👉 Click to view the code for IP address" >}}
```c++
#include <WiFi.h>

const char* ssid = "SmartIrrigation_Youmin";
const char* password = "12345678";

// Set your Static IP address for the Soft Access Point
IPAddress local_IP(192, 168, 125, 1);  // The common default IP for ESP32 SAP is 192.168.4.1
// Set your Gateway IP address (usually same as local_IP for SAP)
IPAddress gateway(192, 168, 125, 1);
// Set your Subnet mask
IPAddress subnet(255, 255, 255, 0);

WiFiServer server(80);
String html = "<!DOCTYPE html> \
<html> \
<body> \
<center><h1>ESP32 Soft access point</h1></center> \
<center><h2>Web Server Testing: Works good and can read live data!</h2></center> \
</body> \
</html>";

void setup() {
  Serial.begin(115200);
  Serial.print("Setting soft access point mode");

  if (!WiFi.softAPConfig(local_IP, gateway, subnet)) {
    Serial.println("SoftAPConfig has failed!");
  } else {
    Serial.println("SoftAPConfig has been successful!");
  }

  WiFi.softAP(ssid, password);
  IPAddress IP = WiFi.softAPIP();
  Serial.print("AP IP address: ");
  Serial.println(IP);
  server.begin();
}

void loop() {
  WiFiClient client = server.available();
  if (client) {
    client.print(html);
  }
}
```
{{< /spoiler >}}

## Softeware coding
Arduino - C++
{{< icon name="c++" >}};
ArcGIS - Arcpy {{< icon name="python" >}};
Econometric Statistic - R {{< icon name="R" >}}

code and data open access
url: https://github.com/liyoumin/Geospatial-AgEcon {{< icon name="github" >}}

Smart Irrigation (IoT)
url: https://github.com/liyoumin/Geospatial-AgEcon/tree/main/smart-irrigation {{< icon name="github" >}}
<!--more-->
