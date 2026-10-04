# Außensensor – Datenblätter

Board: QT Py ESP32-S2, Sensoren per STEMMA QT (I2C) verkettet.

## QT Py ESP32-S2 – Mikrocontroller (Adafruit)

* Anleitung/Pinout: <https://learn.adafruit.com/adafruit-qt-py-esp32-s2>

|Parameter|Wert|
|-|-|
|Prozessor|ESP32-S2, Single-Core 240 MHz|
|Speicher|4 MB Flash, 2 MB PSRAM|
|Funk|WLAN 2,4 GHz, kein Bluetooth|
|USB|USB-C, natives USB|
|I2C|2 Busse: Lötpads und STEMMA-QT-Buchse|
|STEMMA QT|SDA1 = GPIO41, SCL1 = GPIO40|
|Besonderheiten|RGB-NeoPixel, Reset-Taste, BOOT-Taste an GPIO0|
|ESPHome|`board: esp32-s2-saola-1`, `variant: esp32s2`|

Erstes Flashen: BOOT halten, RESET drücken, BOOT loslassen → Bootloader-Modus.

> Alternativen in gleicher Bauform (mit STEMMA QT):
> - QT Py ESP32-S3: Dual-Core, natives USB, zusätzlich Bluetooth LE
> - QT Py ESP32 Pico: klassischer ESP32 (Dual-Core), WLAN + Bluetooth, USB über Seriell-Wandler
>
> Für dieses Projekt nicht nötig – Bluetooth wird nicht genutzt, die Leistung des S2 reicht.
> Bei Wechsel in ESPHome `board`/`variant` und die I2C-Pins der STEMMA-QT-Buchse
> anpassen (laut Pinout des jeweiligen Boards).

## SHT45 – Temperatur/Feuchte (Sensirion)

* Produktseite: <https://sensirion.com/products/catalog/SHT45>
* Datenblatt: <https://sensirion.com/media/documents/33FD6951/6A7C10A0/HT_DS_Datasheet_SHT4x_V7.3.pdf>

|Parameter|Wert|
|-|-|
|Genauigkeit Feuchte|±1,0 %RH|
|Genauigkeit Temperatur|±0,1 °C|
|Betriebsbereich|0…100 %RH, −40…125 °C|
|Versorgung|1,08…3,6 V|
|I2C-Adresse (SHT45-AD1B)|0x44|
|Besonderheiten|voll funktionsfähig bei Kondensation, Heizer schaltbar, PTFE-Membran|

## TMP117 – Temperatur (Texas Instruments)

* Produktseite: <https://www.ti.com/product/TMP117>
* Datenblatt: <https://ti.com/document-viewer/TMP117/datasheet/GUID-905B29F5-5C04-4A8E-9551-EC48393EAE26>

|Parameter|Wert|
|-|-|
|Genauigkeit −20…50 °C|±0,1 °C|
|Genauigkeit −40…70 °C|±0,15 °C|
|Genauigkeit −40…100 °C|±0,2 °C|
|Auflösung|0,0078 °C (16 Bit)|
|Betriebsbereich|−55…150 °C|
|Versorgung|1,7…5,5 V|
|Stromaufnahme|3,5 µA bei 1 Hz|
|I2C-Adressen|4 wählbar|
|Besonderheiten|NIST-rückführbar, Mittelwertbildung wählbar, Offset-Korrektur|

## DPS310 – Luftdruck (Infineon)

* Produktseite Nachfolger DPS368: <https://www.infineon.com/dps368>
* Datenblatt DPS310: <https://community.infineon.com/gfawx74859/attachments/gfawx74859/KnowledgeBaseArticles/10104/3/Infineon-DPS310-DataSheet.pdf>

|Parameter|Wert|
|-|-|
|Druckbereich|300…1200 hPa|
|Temperaturbereich|−40…85 °C|
|Relative Genauigkeit|±0,06 hPa|
|Absolute Genauigkeit|±1 hPa|
|Präzision (High Precision Mode)|±0,002 hPa|
|Temperaturgenauigkeit|±0,5 °C|
|Schnittstelle|I2C / SPI|
|Besonderheiten|32-Werte-FIFO, individuell kalibriert|

> Hinweis: Der DPS310 wird nicht mehr gefertigt – nur für Ersatzbeschaffung relevant.
> Nachfolger DPS368 ist laut ESPHome-Doku Drop-in-Ersatz: gleiche Register, I2C-Adressen
> und Kalibrierung, keine Änderung an der Konfiguration nötig (`platform: dps310` bleibt).
> Mit DPS368 selbst noch nicht getestet.

## Schutzhülle: TFA Dostmann 98.1114.02

Innenmaß Ø 60 × 160 mm – günstig, schützt vor Niederschlag und hält direkte Sonne vom
Sensor fern.  →  an geschützter, schattiger Stelle gut geeignet (z.B. Nordseite mit genügend Abstand zur Hauswand).

Besser bei freier Aufstellung: Lamellen-Strahlungsschutz (Multi-Plate, z.B. Davis 7714);
noch besser: belüftet (mit Lüfter, "aspirated").

>Hintergrund: Die TFA-Hülle ist ein geschlossenes Gehäuse mit Lüftungsöffnungen. In der Sonne heizt sie sich stärker auf als ein Lamellenschirm, der ringsum umströmt wird. An der beschatteten Nordwand ist der Unterschied gering.