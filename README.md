# jahresarbeit_2026_jacob_glathe
# Farbsortieranlage für Tischtennisbälle

**NWT-Jahresarbeit (Naturwissenschaft und Technik)**  
Entwicklung eines automatisierten, getakteten Sortiersystems auf Basis eines ESP32-Mikrocontrollers.

---

## Über das Projekt

In unserer diesjährigen NWT-Jahresarbeit bauen wir eine vollautomatische Sortieranlage für Tischtennisbälle. Die Anlage soll Bälle anhand ihrer Farbe erkennen und präzise in separate Auffangbehälter sortieren.

Beim Entwurf war uns wichtig, nicht einfach eine simple Bastellösung zu bauen, sondern ein System zu entwickeln, das echten industriellen Standards folgt. Die größte Herausforderung dabei: Das System muss modular sein. Wir starten aktuell mit zwei Farben, aber die gesamte Mechanik, Elektronik und Software ist so ausgelegt, dass man später problemlos weitere Farben und Stationen hinzufügen kann, ohne das Projekt von Grund auf neu zu planen.

---

## Die Kernidee: Digitales Tracking statt Sensor-Chaos

Ein typischer Anfängerfehler bei solchen Projekten ist es, an jeder einzelnen Auswurfstation einen eigenen Farbsensor zu montieren. Das ist teuer, fehleranfällig und lässt sich schlecht erweitern.

Wir haben uns für einen smarteren Weg entschieden:
1. **Ein zentraler Farbsensor** ganz am Anfang der Strecke misst die Farbe des Balls genau einmal.
2. **Ein virtuelles Schieberegister im ESP32:** Der Mikrocontroller speichert das gesamte Förderband als eine Kette von Positionen im Speicher. Wenn das Band einen Schritt vorfährt, rücken auch die Farbwerte im Programm exakt eine Position weiter.
3. **Gezielter Auswurf:** Jede Station prüft nur, ob an ihrer Position im Speicher gerade die passende Farbe liegt. Wenn ja, drückt ein Servomotor den Ball vom Band.

Durch diesen Ansatz benötigt das System immer nur einen Sensor, ganz egal ob wir am Ende 2, 4 oder 10 Farben sortieren.

---

## Wie das System aufgebaut ist

### Mechanik und Antrieb
* **Getaktetes Förderband:** Statt eines glatten, rutschigen Bands nutzen wir ein Band mit festen Fächern im Abstand von 75 mm. So hat jeder Ball seinen festen Platz.
* **Schrittmotor (NEMA 17):** Ein normaler Gleichstrommotor läuft nach dem Abschalten immer ein Stück nach. Wir setzen stattdessen auf einen Schrittmotor, dem wir auf den Millimeter genau sagen können: *„Fahre exakt 75 mm vor und bleibe stehen.“*
* **Auswurfmechanik:** Kleine Servomotoren bewegen Stößel, die den Ball im richtigen Moment seitlich vom Band in eine Rutsche befördern.

### Elektronik und Schaltung
* **Gehirn:** Ein ESP32 NodeMCU. Er ist schnell, hat ausreichend Speicher und bringt alle nötigen Schnittstellen mit.
* **Sensorik:** Ein TCS34725 Farbsensor. Er verfügt über eine integrierte weiße LED und wird in einem lichtdichten Gehäuse verbaut, damit Tageslicht oder Schatten im Raum die Messergebnisse nicht verfälschen.
* **Aktorik-Erweiterung (I2C):** Alle Servomotoren werden über ein PCA9685 PWM-Treiberboard gesteuert. Das Geniale daran: Wir steuern bis zu 16 Servos über dieselben zwei Datenleitungen wie den Farbsensor. Dem ESP32 gehen also nie die Pins aus.
* **Saubere Stromversorgung:** Motoren und Servos ziehen bei Bewegung viel Strom und erzeugen Störspitzen. Deshalb sind die Stromkreise für die Motoren (12V/5V) und die Steuerung (3,3V Logik) getrennt aufgebaut, teilen sich aber eine gemeinsame Masse (Common Ground).

---

## Der Ablauf im laufenden Betrieb

Sobald das System eingeschaltet ist, wiederholt es kontinuierlich fünf Schritte:

1. **Vorfahren:** Der Schrittmotor bewegt das Band um genau ein Fach weiter und hält die Position aktiv fest.
2. **Daten weiterrücken:** Im internen Speicher rücken alle bisherigen Ball-Positionen um einen Index weiter.
3. **Messen:** Der Farbsensor liest den neu eingefahrenen Ball an Station 0 ein und speichert dessen Farbwert am Index 0 ab.
4. **Auswerfen:** Der ESP32 prüft alle Stationen gleichzeitig. Liegt an Station 1 ein roter Ball, schlägt Servo 1 aus. Liegt an Station 2 ein weißer Ball, schlägt Servo 2 aus.
5. **Rückstellung:** Die Servos fahren in die Ausgangsposition zurück und das Band ist bereit für den nächsten Schritt.

---

## Aufbau dieses Repositories

Die gesamte Planung und Entwicklung ist in diesem Repository dokumentiert und in mehrere Bereiche gegliedert:

* **docs/SYSTEM_PLANUNG.md**  
  Die ausführliche technische Planung mit allen mathematischen Berechnungen, Geometriedaten und der Software-Architektur.
* **docs/HARDWARE_BOM.md**  
  Die vollständige Stückliste (Bill of Materials) inklusive aller Bauteile, Spezifikationen und der Pin-Belegung für den ESP32.
* **docs/AUSFUEHRUNGSPLAN.md**  
  Unser Schritt-für-Schritt-Fahrplan von den ersten Versuchen auf dem Breadboard über die CAD-Konstruktion bis hin zur finalen Testmatrix und Fehlerbehandlung.
* **src/**  
  Der Quellcode für den ESP32 (folgt in Phase 3/4).
* **cad/**  
  Die 3D-Druck-Dateien für Bandglieder, Sensorgehäuse und Auswurfmechanismen.

---

## Projektstatus

Wir befinden uns aktuell in der Umsetzungsphase gemäß unserem Ausführungsplan. Die Sensor-Kalibrierung und die Elektroniktests laufen parallel zum 3D-Druck der ersten Bandelemente.
