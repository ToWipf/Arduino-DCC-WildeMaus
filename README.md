# Arduino DCC Eisenbahn WildeMaus Steuerung

Arduino sketch für _FALLER 140425 - Jubiläumsmodell „WildeMaus“_

|F Taste|Funktion|Pin|
|:-:|:-:|:-:|
|F1|Motor für Kettenzug An/Aus|A0|
|F2|WildeMaus Anzeige An/Aus|A1|
|F3|Birnchen Teil 1|A2|
|F4|Birnchen Teil 2|A3|
|F5|WildeMaus Anzeige Blinken|A1|
|F6|Laufliche Mode 1|D3,D4,D5|
|F7|Laufliche Mode 2|D3,D4,D5|
|F8|Laufliche Mode 3|D3,D4,D5|
|F9|Laufliche Mode 4|D3,D4,D5|

## Schaltplan

Das IC der Lauflichtssteuerung muss herrausgenommen werden!

Unter einer DCC Adresse, kann mit diesen Arduino Programm über die Tasten F1 bis F9 die WildeMaus gesteuert werden.

![Bild](https://raw.githubusercontent.com/ToWipf/DCC-WildeMaus-Arduino/master/WildeMaus_Plan.PNG)

## Aufbau der Wilden Maus

![Bild](https://raw.githubusercontent.com/ToWipf/DCC-WildeMaus-Arduino/master/WildeMaus.jpg)
![Bild](https://raw.githubusercontent.com/ToWipf/DCC-WildeMaus-Arduino/master/WildeMaus%20(2).jpg)
![Bild](https://raw.githubusercontent.com/ToWipf/DCC-WildeMaus-Arduino/master/WildeMaus%20(3).jpg)

Link zur [DCC_Decoder_libraries](https://github.com/MynaBay/DCC_Decoder) Datei

Der Schaltplan wurde von [JoFri](http://www.7fun.de/jofri/) erstellt.
