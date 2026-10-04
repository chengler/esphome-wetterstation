# README esphome-wetterstation

Außensensor für Temperatur, Luftfeuchte und Luftdruck auf Basis ESPHome.

> Ziel: Heizgradtage, Vorhersage der Vereisung von Luft-Wasser-Wärmepumpen und
> Wetterdaten in Amateurstations-Qualität per MQTT.

## Hardware

|Bauteil|Aufgabe|
|-|-|
|QT Py ESP32-S2 (Adafruit)|Mikrocontroller, WLAN, MQTT|
|TMP117 (TI)|Lufttemperatur – Referenzwert|
|SHT45 (Sensirion)|Luftfeuchte (mit Heizer)|
|DPS310 (Infineon)|Luftdruck|
|TFA Dostmann 98.1114.02|Schutzhülle|

Alle Sensoren per STEMMA QT (I2C) verkettet, keine Lötarbeiten.
Details und Datenblätter: siehe `Hardware.md`.

## Warum drei Sensoren?

**Feuchte – SHT45 mit Heizer:**
Bei sehr hoher Luftfeuchte (Nebel, Tau) kann sich Wasser auf dem Sensor
niederschlagen; der Sensor reagiert dann nicht mehr auf Feuchteänderungen.
Außerdem driftet die Messung bei dauerhaft hoher Feuchte ("Creep").
Der eingebaute Heizer trocknet den Sensor kurzzeitig ab (Details: Abschnitt
Heiz-Automatik).

Nachteil: Während und kurz nach dem Heizen sind Temperatur *und* Feuchte des
SHT45 verfälscht. Zusätzlich kann thermisch bedingter mechanischer Stress die
Temperaturmessung versetzen (Datenblatt SHT4x 4.9).

**Temperatur – TMP117 als Referenz:**
Deshalb kommt die Lufttemperatur von einem separaten Sensor ohne Heizer.
Der TMP117 hat eine garantierte Genauigkeit von ±0,1 °C (−20…50 °C). Der SHT45 schafft ±0,1 °C nur typisch und erst ab ca. +5 °C, garantiert sind ±0,3 °C (Datenblatt SHT4x 2.2, Temperaturgenauigkeit).

Zusätzlich dient der TMP117 als Vergleichswert: Daran lässt sich prüfen, ob und wenn wie
stark und wie lange die Heizpulse die SHT45-Temperatur bei hoher Luftfeuchte
verfälschen.

**Luftdruck – DPS310:**
Der DPS310 misst zusätzlich auch die Temperatur (±0,5 °C). Diese dient vor allem der
internen Druckkompensation und ist für die Lufttemperatur zu ungenau.
Druckgenauigkeit: relativ ±0,06 hPa (entspricht ca. ±0,5 m Höhe), absolut ±1 hPa.

**Alternative Ein-Chip-Lösung (BME280):**
Misst Feuchte, Temperatur und Druck in einem Chip – einfacher, aber deutlich
ungenauer und ohne Heizer:

|Messgröße|BME280|hier verwendet|
|-|-|-|
|Feuchte|±3 %RH|±1 %RH (SHT45)|
|Temperatur|±1 °C|±0,1 °C (TMP117)|
|Druck relativ|±0,12 hPa|±0,06 hPa (DPS310)|

## Aufbau

- Sensoren in der Schutzhülle, Nordseite, beschattet, mit Abstand zur Hauswand.
- I2C ist für kurze Leitungen ausgelegt: STEMMA-QT-Kabel kurz halten.
- TMP117 möglichst unterhalb des SHT45 – warme Luft (Heizer, Eigenerwärmung)
  steigt nach oben.
- ESP32 und DPS310 können in ein separates, geschütztes Gehäuse; dieses darf
  nicht luftdicht sein (Druckausgleich).
- Stromversorgung per USB (5 V). Mit AWG-20-Adern reicht das für gut 10 m.

## Heiz-Automatik (SHT45)

ESPHome kann den Heizer nicht zur Laufzeit schalten, daher eigene Lösung
(`firmware.yaml`, Abschnitt 1a). Nach jeder Feuchtemessung wird geprüft, ob
geheizt wird – zweistufig:

|Stufe|Zweck|Schwelle|Mindestabstand|Heizpuls|
|-|-|-|-|-|
|niedrig|Creep vorbeugen|≥ 80 %RH (`heat_rh_low`)|15 min (`heat_pause_low_ms`)|110 mW, ca. 1 s (`heat_cmd_low` = `0x2F`)|
|hoch|Kondenswasser entfernen|≥ 95 %RH (`heat_rh_high`)|5 min (`heat_pause_high_ms`)|200 mW, ca. 1 s (`heat_cmd_high` = `0x39`)|

* 80 %RH ist die Obergrenze des empfohlenen Bereichs (Datenblatt SHT4x 2.3).
  Alle übrigen Werte sind Startwerte (nicht aus dem Datenblatt) und werden
  anhand des TMP117-Vergleichs noch angepasst.
* Heizen direkt nach der Messung → bis zur nächsten Messung (60 s) Abkühlzeit.
* Duty Cycle max. ca. 0,33 % (Stufe hoch) bzw. 0,11 % (Stufe niedrig) – die
  Obergrenze laut Datenblatt liegt bei 10 %.
* Jeder Heizpuls erscheint im Log (`sht45_heater`) mit Befehl und Feuchte.

Verfügbare Heizstufen (Datenblatt SHT4x Tabelle 7), einstellbar in den
`substitutions`:

|Leistung|1 s|0,1 s|
|-|-|-|
|200 mW|`0x39` (Stufe hoch)|`0x32`|
|110 mW|`0x2F` (Stufe niedrig)|`0x24`|
|20 mW|`0x1E`|`0x15`|

200 mW bringt die meiste Wärme pro Puls – Kondenswasser wird am zuverlässigsten
entfernt. Nachteil: stärkere Verfälschung danach, mehr thermischer Stress,
Stromspitze bis ca. 75 mA. Deshalb nur bei sehr hoher Feuchte; gegen Creep
sollte reicht die schonendere Stufe reichen.

## MQTT-Topics

Broker: `venus.internal`, Präfix: `wetter/nord`

| Topic                                         | Sensor | Einheit | Bemerkung           |
|-----------------------------------------------|--------|---------|---------------------|
| `wetter/nord/sensor/temperature/state`        | TMP117 | °C      | Referenz            |
| `wetter/nord/sensor/humidity/state`           | SHT45  | %RH     |                     |
| `wetter/nord/sensor/pressure/state`           | DPS310 | hPa     | Stationsdruck       |
| `wetter/nord/sensor/sht45_temperature/state`  | SHT45  | °C      | Plausibilisierung   |
| `wetter/nord/sensor/dps310_temperature/state` | DPS310 | °C      | intern/Kompensation |
| `wetter/nord/status`                          | –      | –       | online / offline    |

## Inbetriebnahme

1. `secrets-todo.yaml` kopieren nach `secrets.yaml`, Werte eintragen.
2. In `firmware.yaml` anpassen:
   - `mqtt: broker:` – eigener MQTT-Broker
   - `wifi: use_address:` – Hostname oder IP des ESP für OTA-Updates.\
     mDNS ist deaktiviert, der Name muss daher im eigenen DNS eingetragen sein.\
     Alternativ feste IP verwenden oder mDNS aktivieren (`mdns: disabled: false`,
     `use_address` entfällt, Gerät dann unter `<esphome: name:>.local`, z.B. `wetter-nord.local`).
   - bei Bedarf:
     - `mqtt: topic_prefix:` – anderer Standort
     - `esphome: name:` / `friendly_name:` – bei weiterem Gerät eindeutig wählen
     - `i2c: sda:` / `scl:` – bei anderem Board (Pins siehe Kommentar im YAML)
3. Prüfen: `esphome config firmware.yaml`
4. Erstes Flashen per USB: BOOT halten, RESET drücken, BOOT loslassen, dann
   `esphome run firmware.yaml`
5. Weitere Updates per OTA über die Adresse aus `wifi: use_address:`.
6. Web-UI und Log im Browser öffnen:
   - mit DNS/fester IP: Adresse aus `wifi: use_address:`, z.B. `http://wetter-nord.internal`
   - mit mDNS: Wert aus `esphome: name:` plus `.local`, z.B. `http://wetter-nord.local`

   Im Log muss der I2C-Scan beim Start 0x44, 0x48 und 0x77 zeigen.

## Status

Firmware kompiliert, Hardware bestellt.