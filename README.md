# PT1000 Temperaturmessung mit ESP8266 und MAX31865

Dieses Projekt beschreibt eine einfache und kostengünstige Lösung zur Messung von **vier Temperaturen mit PT1000-Sensoren**.

Die Sensoren werden über **MAX31865 RTD-Interface-Module** ausgelesen und von einem **ESP8266 (Wemos D1 mini)** verarbeitet.  
Die Messwerte werden anschließend über **MQTT** bereitgestellt.

Besonderheit dieses Projekts ist ein **kompaktes 3D-druckbares Gehäuse**, das speziell für diese Kombination entworfen wurde und Platz für:

- 1 × ESP8266 D1 mini  
- 4 × MAX31865 Module  
- Anschlussleitungen für die Sensoren  

Das Gehäuse ist ebenfalls in diesem Repository als **STL-Datei** enthalten.

---

# Projektübersicht

Das System besteht aus:

- ESP8266 D1 mini
- 4 × MAX31865 RTD-Interface
- 4 × PT1000 Sensoren
- gemeinsamem SPI-Bus
- MQTT-Anbindung

---

# Hardware

Verwendete Komponenten:

- ESP8266 **Wemos D1 mini**
- 4 × **MAX31865 RTD Interface**
- 4 × **PT1000 Temperaturfühler**

## Referenzwiderstand

Viele MAX31865-Boards sind standardmäßig für **PT100** ausgelegt  
(**Rref = 430 Ω**).

Für PT1000 muss der Referenzwiderstand auf etwa **4.3 kΩ** geändert werden.

Hier wurde der Widerstand ersetzt durch:

4.7 kΩ || 47 kΩ ≈ 4.27 kΩ

Ich habe den SMD-Widerstand auf dem Bord "abgelötet" und durch einen die beiden parallel geschalteten "normalen" Widerstände ersetzt. Etwas "fummelig" aber wenn man die neuen Widerstände auf dem Bord noch mit etwas Heißkleber fixiert, passt alles.

Wichtig:  
Nach dem Austausch sollten die Referenzwiderstände **gemessen** werden.  
Der gemessene Wert kann anschließend **exakt in ESPEasy eingetragen werden**, um die Messgenauigkeit zu verbessern.

Beispiel der tatsächlich gemessenen Referenzwerte in diesem Aufbau:

Sensor 1: 4247 Ω
Sensor 2: 4265 Ω
Sensor 3: 4267 Ω
Sensor 4: 4267 Ω


Diese Werte werden jeweils im entsprechenden ESPEasy-Device als **Rref** eingetragen.

---

# Verdrahtung

Alle MAX31865-Module teilen sich den gleichen **SPI-Bus**.  
Jedes Modul besitzt jedoch eine eigene **Chip-Select Leitung**.

## SPI Bus

| ESP8266 Pin | Funktion |
|--------------|----------|
| D5 | SCK |
| D6 | MISO |
| D7 | MOSI |

Diese Leitungen werden mit allen vier MAX31865 verbunden.

---

## Chip Select

| Sensor | ESP8266 Pin |
|-------|-------------|
| MAX31865 #1 | D1 |
| MAX31865 #2 | D2 |
| MAX31865 #3 | D0 |
| MAX31865 #4 | D3 |

---

## Versorgung

| Signal | Verbindung |
|------|-------------|
| 3.3V | MAX31865 VCC |
| GND | MAX31865 GND |

Die Spannung (3.3V) kommt direkt vom ESP8266.

---

# Firmware

Die Firmware basiert auf **ESPEasy**. Bei mir Build: ESP_Easy_mega_20260121_normal_ESP8266_4M1M Jan 21 2026

Für jeden MAX31865 wird ein eigenes **Device** angelegt.

Typische Konfiguration:

- Device: **MAX31865**
- Sensor: **PT1000**
- Anschluss: **2-Wire**
- Referenzwiderstand: gemessener Wert (siehe oben)
- SPI Pins entsprechend der Verdrahtung

Die Messwerte werden anschließend über **MQTT** veröffentlicht.

---

# 3D-Druck Gehäuse

Das Repository enthält ein **3D-druckbares Gehäuse**, das speziell für diese Kombination entwickelt wurde.

Eigenschaften:

- Platz für **1× D1 mini**
- Platz für **4× MAX31865**
- Kabelausgänge für **4 PT1000 Sensoren**
- kompakte Bauform
- einfache Montage

