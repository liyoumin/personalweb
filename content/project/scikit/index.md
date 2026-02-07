---
title: Impact of Drought on U.S. Alfalfa Production and Export
date: 2025-10-26
# Link directly to your PDF on Overleaf
url_pdf: "/personalweb/project/scikit/Drought_impacts_on_alfalfa.pdf" 
url_slides: "/personalweb/project/scikit/Presentation_alfalfa.pdf" 
url_poster: "/personalweb/project/scikit/Motivation.png" 
url_source: "/personalweb/project/scikit/AAEADrought_alfalfa.pdf" 
url_video: "/personalweb/project/scikit/SimpleSDI.MOV" 
url_code: "" 
url_dataset: "#" 
url_project: ""

tags:
  - Smart Agriculture
  - Geo-computation
  - Drought
  - C++ (Arduino)
  - IoT
  - Irrigation
---

“Impact of Drought on U.S. Alfalfa Production and Export".

[📄 Download full paper](Drought_impacts_on_alfalfa.pdf)

## Introduction and background:
Alfalfa production in Western semi-arid states often faces high evaporative demand and tight irrigation constraints. Smart Drip Irrigation (SDI) systems can improve water efficiency by coupling real-time sensing with automatic control. This mini project demonstrates the core SDI control loop—sense → decide → actuate → display/log—using low-cost hardware. In this simplified version (based on available parts), the system uses temperature and relative humidity (DHT11) as a “hot & dry” trigger to simulate irrigation scheduling and proves actuation through a relay signal, a status LED, and a LCD message. An ESP-32S receives telemetry for future IoT extension (dashboard/cloud), while the Mega2560 remains the reliable control layer.

## Counter measure:
Build a simple Smart Agriculture IoT controller with Arduino Mega2560 + DHT11 + LCD1602A + Relay, and stream telemetry to ESP-32S.

## Video

This project builds a Smart Drip Irrigation (SDI) controller for alfalfa production in Western semi-arid states. The system reads soil moisture and air temperature/humidity to decide when to turn a water pump (pressure irrigation) on/off. The goal is to maintain soil moisture within a target range (avoid under-watering and over-watering) while adapting irrigation timing to hot/dry conditions that increase evapotranspiration. The controller logs sensor data and pump status for monitoring and debugging.

{{< youtube ciD3ILxgXzU >}}


**Video file**

<!-- Videos in `assets/media/` media library or in [page's folder](https://gohugo.io/content-management/page-bundles/), then embedding them with the _video_ shortcode: -->

    {{</* video src="SimpleSDI.mp4" controls="yes" */>}}


## Core functions

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
Optional upgrades (good for Step 2)
LCD display (1602 I2C or parallel): show Soil%, Temp, RH, Pump state
Data logging to SD card module (CSV file)
RTC clock module for time-stamped logs and “no watering at midday” rule
ESP-32D: send data to Wi-Fi dashboard / phone (IoT extension)

{{< spoiler text="👉 Click to view the solution" >}} Smart SDI 🎉 {{< /spoiler >}}

## Hardware

          Soil Moisture Sensor (Analog) -----> A0 (Mega)
          DHT Temp/RH (Digital)  -----------> D2 (Mega)
          Mega D8  -----------------------> Relay IN  -----> Pump power circuit
          Mega GND ------------------------> Relay GND
          Mega 5V  ------------------------> Relay VCC (if 5V relay board)
          Pump + Irrigation:
          Water tank -> Pump -> Filter -> Drip line -> Alfalfa plot
Optional:
LCD (I2C)  SDA/SCL -> Mega SDA/SCL
SD module  CS/MOSI/MISO/SCK -> Mega SPI pins
ESP-32D    WiFi upload (separate controller or co-processor)


## Code (C++)

```C++
/*
 Smart SDI for Alfalfa (ELEGOO Mega R3)
 Sensors: Soil moisture (A0), DHT (D2)
 Actuator: Relay->Pump (D8)
 Goals:
  - Maintain soil moisture between LOW/HIGH thresholds
  - Adjust thresholds if hot/dry
  - Non-blocking timing with millis()
  - Safety: max runtime + min off-time
*/

#include <DHT.h>

// Pins
const int PIN_SOIL  = A0;
const int PIN_DHT   = 2;
const int PIN_RELAY = 8;

// DHT setup
#define DHTTYPE DHT11  // or DHT22
DHT dht(PIN_DHT, DHTTYPE);

// Timing
unsigned long tRead = 0;
const unsigned long READ_MS = 2000;

// Calibration (Step 2: measure real values)
int dryADC = 800;
int wetADC = 350;

// Thresholds
float baseLOW  = 25.0;   // % turn ON
float baseHIGH = 35.0;   // % turn OFF (hysteresis)
float hotC = 30.0, dryRH = 30.0, bias = 3.0;

// Safety
bool pumpOn = false;
unsigned long pumpStart = 0;
unsigned long lastOff = 0;
const unsigned long MAX_ON  = 10UL * 60UL * 1000UL;
const unsigned long MIN_OFF = 2UL  * 60UL * 1000UL;

// Convert soil ADC to %
float soilPct(int adc){
  adc = constrain(adc, min(wetADC, dryADC), max(wetADC, dryADC));
  float pct = 100.0 * (float)(dryADC - adc) / (float)(dryADC - wetADC);
  return constrain(pct, 0.0, 100.0);
}

// Compute dynamic thresholds
void computeThresh(float T, float RH, float &LOW, float &HIGH){
  LOW = baseLOW; HIGH = baseHIGH;
  if (!isnan(T) && !isnan(RH) && T >= hotC && RH <= dryRH){
    LOW += bias; HIGH += bias;
  }
}

// Set pump with safety checks
void setPump(bool on){
  if (on == pumpOn) return;

  if (on){
    if (millis() - lastOff < MIN_OFF) return;
    pumpOn = true;
    pumpStart = millis();
    digitalWrite(PIN_RELAY, HIGH); // relay logic may be LOW=ON depending module
  } else {
    pumpOn = false;
    lastOff = millis();
    digitalWrite(PIN_RELAY, LOW);
  }
}

void setup(){
  Serial.begin(9600);
  pinMode(PIN_RELAY, OUTPUT);
  digitalWrite(PIN_RELAY, LOW);
  dht.begin();
}

void loop(){
  if (millis() - tRead >= READ_MS){
    tRead = millis();

    // 1) Read sensors
    float T = dht.readTemperature();
    float RH = dht.readHumidity();
    int adc = analogRead(PIN_SOIL);
    float soil = soilPct(adc);

    // 2) Fail-safe if sensor invalid
    if (isnan(soil)){
      setPump(false);
      return;
    }

    // 3) Compute thresholds
    float LOW, HIGH;
    computeThresh(T, RH, LOW, HIGH);

    // 4) Safety max on-time
    if (pumpOn && millis() - pumpStart > MAX_ON) setPump(false);

    // 5) Hysteresis logic
    if (!pumpOn && soil < LOW) setPump(true);
    if (pumpOn && soil > HIGH) setPump(false);

    // 6) Print status
    Serial.print("Soil%="); Serial.print(soil,1);
    Serial.print(" T="); Serial.print(T,1);
    Serial.print(" RH="); Serial.print(RH,1);
    Serial.print(" LOW/HIGH="); Serial.print(LOW,1); Serial.print("/");
    Serial.print(HIGH,1);
    Serial.print(" Pump="); Serial.println(pumpOn ? "ON" : "OFF");
  }
}
```

## Inline Images
Arduino
{{< icon name="c++" >}} C++

image:
  caption: 'Hardware: Mega2560 + DHT11 + LCD1602A + Relay + ESP-32S (telemetry)'
  
<!--more-->
