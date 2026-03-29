# PT1000 Temperaturmessung mit ESP8266 und MAX31865

Dieses Projekt beschreibt eine einfache und kostengünstige Lösung zur Messung von **vier Temperaturen mit PT1000-Sensoren**.

Die Sensoren werden über **MAX31865 RTD-Interface-Module** ausgelesen und von einem **ESP8266 (Wemos D1 mini)** verarbeitet.  
Die Messwerte werden anschließend in **Home Assistant** integriert und können optional per **MQTT** weitergegeben werden.

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
- Integration in Home Assistant
- optionalem MQTT-Export

---

# ⚠️ Wichtiger Hinweis zur Verdrahtung

Jeder PT1000 muss **mit beiden Leitungen ausschließlich an "sein" MAX31865-Modul angeschlossen werden**.

👉 Eine gemeinsame Rückleitung für mehrere Sensoren ist **nicht zulässig**.

Grund:
- führt zu Fehlmessungen
- erzeugt sporadische Faults
- kann komplette Messketten stören

👉 Jeder Sensor benötigt eine **separate Zweidrahtleitung direkt zum jeweiligen MAX-Modul**

---

# Hardware

Verwendete Komponenten:

- ESP8266 **Wemos D1 mini**
- 4 × **MAX31865 RTD Interface**
- 4 × **PT1000 Temperaturfühler**

---

## Referenzwiderstand

Viele MAX31865-Boards sind standardmäßig für **PT100** ausgelegt  
(**Rref = 430 Ω**).

Für PT1000 muss der Referenzwiderstand auf etwa **4.3 kΩ** geändert werden.

Hier wurde der Widerstand ersetzt durch:
4.7 kΩ || 47 kΩ ≈ 4.27 kΩ


Der originale SMD-Widerstand wurde entfernt und durch zwei parallel geschaltete Drahtwiderstände ersetzt.

Wichtig:  
Nach dem Austausch sollten die Referenzwiderstände **gemessen** werden.  
Diese Werte werden anschließend **in ESPHome eingetragen**.

### Gemessene Referenzwerte

| Sensor | Rref |
|--------|------|
| Sensor 1 | 4247 Ω |
| Sensor 2 | 4265 Ω |
| Sensor 3 | 4267 Ω |
| Sensor 4 | 4260 Ω |

---

# Verdrahtung

Alle MAX31865-Module teilen sich den gleichen **SPI-Bus**.  
Jedes Modul besitzt jedoch eine eigene **Chip-Select Leitung**.

## SPI Bus

| ESP8266 Pin | Funktion |
|--------------|----------|
| D5 (GPIO14) | SCK |
| D6 (GPIO12) | MISO |
| D7 (GPIO13) | MOSI |

---

## Chip Select

| Sensor | ESP8266 Pin |
|-------|-------------|
| MAX31865 #1 | D1 (GPIO5) |
| MAX31865 #2 | D3 (GPIO0) |
| MAX31865 #3 | D2 (GPIO4) |
| MAX31865 #4 | D0 (GPIO16) |

---

## Versorgung

| Signal | Verbindung |
|------|-------------|
| 5.0V | MAX31865 VCC |
| GND  | MAX31865 GND |

---

# Firmware (ESPHome)

Die Firmware basiert auf **ESPHome**.

## Konfiguration

Jeder MAX31865 wird als eigener Sensor betrieben:

- Sensor: **PT1000**
- Anschluss: **2-Wire**
- individueller Referenzwiderstand
- gemeinsamer SPI-Bus
- eigener CS-Pin je Modul

---

## Signalaufbereitung (Version 1)

Die aktuelle Konfiguration umfasst:

- Messintervall: **5 Sekunden**
- Filterung ungültiger Werte:
  - Kollektoren: **-20 °C bis +200 °C**
  - Solarleitungen: **0 °C bis +100 °C**
- Mittelwertbildung über **3 Messungen**
- Rundung auf **1 Nachkommastelle**

---

# Beispiel YAML (gekürzt)

```yaml
update_interval: 5000ms

filters:
  - lambda: |-
      if (isnan(x)) return {};
      if (x <= -20.0 || x >= 200.0) return {};
      return x;
  - sliding_window_moving_average:
      window_size: 3
      send_every: 3
  - round: 1
