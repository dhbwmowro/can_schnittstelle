# MowRo-Schnittstelle
In diesem Repository wird die Schnittstelle zwischen dem CAN-Bus und der Fahrlogik entwickelt. Dafür werden die CAN-Befehle für die Ansteuerung der Motoren in eine eine einfache Schnittstelle Integriert. Diese Schnittstelle übeprüft, ob alle Befehle richtig ausgeführt wurden und gibt dies als Rückgabewert wieder.
Die Überprüfung der Fahrbefehle soll in mehreren Stufen erfolgen.
  1. Sensorüberprüfung bei Bootup
  2. Aktoren überprüfen (noch undefiniert wie genau)
  3. Befehle überprüfen (über Sensoren, Aktoren und unterschiedliche Berechnungen)
 
Die genaue Konzeptieonierung der einzelnen Funktionen sieht wie folgt aus:
--

**Funktion: Sensorüberprüfung**
Berechnung der Drehsensoren über abfahren definierter Strecke mit einem Motor -> Verhältnisberechnung -> Plausiblisierung, ob Motoren funktionieren.


**Funktion: LeftTurnForward/Backward**
  Eingabe Parameter: speed, mode, angle, radius

  Überprüfungen: 
  - Drehzahlen der Motoren -> Geschwindigkeit
  - Verhältnis der Drehzahlen -> Radius/Winkel
  - bei zugroßem Offset-Korrektur
  Rückgaben:
  - Geforderter Winkel & realer Winkel
  - Geforderte Geschwindigkeit & reale Geschwindigkeit (pro Motor)
  - Fehlermeldung/Korrektur bei zu großem Offset bei Winkel oder Geschwindigkeit
  
**Funktion: RightTurnForward/Backward**
  Eingabe Parameter: speed, mode, angle, radius Kompass(Gyroskop)?
  - Lenkwinkel in %(0-100) -> 100% bedeutet TurnMiddle 
  - Speed in km/h da die Einheit am besten vertraut ist.
  - Detektion von Gegenständen im nahen Umfeld -> maximale Geschwindigkeit reduzieren.
  - Aus Rückführung der Räderrotation die tatsächliche Drehung bestimmen.
  
  Überprüfungen: 
  - Drehzahlen der Motoren -> Geschwindigkeit
  - Verhältnis der Drehzahlen -> Radius/Winkel
  - bei zugroßem Offset-Korrektur
  
  Rückgaben:
  - Geforderter Winkel & realer Winkel
  - Geforderte Geschwindigkeit & reale Geschwindigkeit (pro Motor)
  - Fehlermeldung/Korrektur bei zu großem Offset bei Winkel oder Geschwindigkeit

**Funktion: Forwards**
  Eingabe Parameter: speed, direction, winkel = 0, Kompass(Gyroskop)?

  Überprüfungen: 
  - Drehzahlen der Motoren -> Geschwindigkeit
  - Verhältnis der Drehzahlen -> Radius/Winkel
  - Kompasswerte
   
  Rückgaben:
  - Realer Winkel
  - Geforderte Geschwindigkeit & reale Geschwindigkeit (pro Motor)
  - Fehlermeldung/Korrektur bei zu großem Offset oder Geschwindigkeit
  - Mäher fährt geradeaus oder sind Drehzahlen einfach gleich?
  
**Funktion: Backwards**
  Eingabe Parameter: speed, direction, winkel = 0, Kompass(Gyroskop)?

  Überprüfungen: 
  - Drehzahlen der Motoren -> Geschwindigkeit
  - Verhältnis der Drehzahlen -> Radius/Winkel
  - Kompasswerte
   
  Rückgaben:
  - Realer Winkel
  - Geforderte Geschwindigkeit & reale Geschwindigkeit (pro Motor)
  - Fehlermeldung/Korrektur bei zu großem Offset oder Geschwindigkeit
  - Mäher fährt rückwärts oder sind Drehzahlen einfach gleich?

**Funktion: turnMiddle**
  Eingabe Parameter: speed, direction

  Überprüfungen: 
  - Drehzahlen der Motoren -> Geschwindigkeit. Einer der beiden Motoren muss Drehzahl 0 haben. 
  - Richtung überprüfen - welcher Motor hat Drehzahl 0?
   
  Rückgaben:
  - Geforderte Geschwindigkeit & reale Geschwindigkeit (pro Motor)
  - Drehrichtung
  - Fehlermeldung/Korrektur bei zu großem Offset oder Geschwindigkeit


**Funktion: Stops**
  Eingabe Parameter: speed, GPS, Mähwerk, Bremsen

  Überprüfungen:
  - Drehzahl der Motoren 
  - Status des Mähwerks
  - Elektromotoren Aktiv bremsen
  
  Rückgaben:
  - Reale Geschwindigkeit aus Rotation der Räder bzw- GPS Daten
  - Signalleuchte -> Farbe definieren



Rückgabewerte
--
1. Ausführung des Befehls ?
2. Wurde der Befehl vollständig und exakt ausgeführt ?

Einbindung eines Gyroskops / GPS

- 0000 0000 -> Alles IO
- 0000 0001 -> Befehl wurde nicht ausgeführt. Unveränderter Zustand
- 0000 0010 -> Zustand des Mähwerks stimmt nicht mit Befehl überein
- 0000 0010 -> Fahrtrichtung (Mode) stimmt nicht mit Befehl überein
- 0000 0011 -> Geschwindigkeit stimmt nicht mit Befehl überein
- 0000 0100 -> Radius stimmt nicht mit Befehl überein
- 0000 0101 -> Winkel stimmt nicht mit Befehl überein
- 0000 0110 -> Drehzahlsensor rechts in Fahrtrichtung
- 0000 0111 -> Drehzahlsensor links in Fahrtrichtung
- 0000 1000 -> Niederspannung 12V Batterie
- 0000 1001 -> Drehzahl des Generators zu gering
- 0000 1010 -> Drehzahl des Generators zu hoch
- 0000 1011 -> Drehzahl des Verbrennungsmotors zu gering
- 0000 1100 -> Drehzahl des Verbrennungsmotors zu hoch
- 0000 1101 -> Kollision im Anbaumodul
- 0000 1110 -> Kollision Rückwärtige Stoßstange
- 0000 1111 -> Feuchtigkeitssensor defekt
- 0001 0000 -> Temperatursensor defekt

Berechnung der Geschwindigkeit von km/h in cm/s
--
- Eingabewert zwischen **0 km/h** und **9,14 km/h** 
- **Minimale Geschwindigkeit** in jede Richtung ist **0,54 km/h**
