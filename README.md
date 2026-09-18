# Modulare Farbsortieranlage für Tischtennisbälle

**NWT-Jahresarbeit (Naturwissenschaft und Technik)**  
Entwicklung einer automatisierten Sortieranlage mit endlos umlaufendem Gliederband, ESP32-Steuerung und digitalem Daten-Tracking.

---

## Über das Projekt

Im Rahmen unserer diesjährigen NWT-Jahresarbeit entwickeln wir eine vollautomatische Sortieranlage für Standard-Tischtennisbälle (40 mm Durchmesser). Die Anlage erkennt Bälle anhand ihrer Oberflächenfarbe und wirft sie zielsicher an der passenden Station in Auffangbehälter ab.

Das Herzstück der Mechanik ist ein **endlos umlaufendes Förderband**, das vollständig aus **selbst konstruierten, 3D-gedruckten Kettengliedern** besteht. Das Band fährt nicht vor und zurück, sondern dreht sich wie ein industrielles Förderband kontinuierlich in eine Richtung im Takt vorwärts. 

Bereits sortierte oder leere Fächer laufen an der Unterseite der Anlage einfach wieder zurück zum Start, wo über ein Magazin neue Bälle nachrutschen können.

---

## Die Kernmechanik: 3D-gedruckte Kettenglieder mit Klicksystem

Statt ein vorgefertigtes Gummiband zu verwenden, auf dem Bälle wegrollen könnten, haben wir ein eigenes modulares Kettensystem in OpenSCAD konstruiert:

* **Snap-Fit Klicksystem:** Jedes Kettenglied besitzt an der Vorderseite einen robusten Gelenkbolzen und an der Rückseite federnde Schnappklauen. Die Glieder lassen sich ohne zusätzliches Werkzeug zu einer beliebig langen Endloskette ineinanderklicken und bei Bedarf über integrierte Hebelkerben wieder lösen.
* **Formschlüssige Kugelmulde:** Jedes Glied hat eine exakt berechnete Mulde mit 1,5 mm Spielraum, in der der Ball während der Taktbewegung erschütterungsfrei ruht.
* **Durchgehender Auswurfkanal:** Eine seitliche Aussparung im Glied erlaubt es dem servo-betriebenen Stößel, den Ball im 90-Grad-Winkel sauber und ohne Klemmen vom Band in die Sortierrutsche zu schieben.
* **Schwenkfreiraum:** An den Kanten der Glieder sind Freiwinkel eingelassen, damit die Kette sauber um die Antriebs- und Umlenkrollen an den Bandenden kurven kann.

---

## Die Software-Logik: Digitales Tracking im Umlauf

Anstatt an jeder Station teure Sensoren zu montieren, arbeitet das System mit einer zentralen Messstelle und einem virtuellen Schieberegister im ESP32:

1. **Ein zentraler Farbsensor (TCS34725):** Ganz am Anfang der oberen Förderstrecke (Station 0) wird die Farbe jedes ankommenden Balls genau einmal präzise erfasst.
2. **Daten-Shift im Speicher:** Der ESP32 speichert den Zustand aller sichtbaren Fächer in einem Array. Mit jedem Vorwärtsschritt des Bands rücken alle Datenwerte im Speicher exakt einen Index weiter.
3. **Selektiver Auswurf:** Jede Station prüft nur ihren eigenen Index:
   * Station 1 prüft Index 1: Liegt hier z. B. Rot? Wenn ja, drückt Servo 1 den Ball aus dem Fach.
   * Station 2 prüft Index 2: Liegt hier z. B. Weiß? Wenn ja, drückt Servo 2 den Ball aus dem Fach.
4. **Endloser Rücklauf:** Das nun leere Kettenglied wird als LEER markiert, läuft über die Umlenkrolle an der Unterseite zurück zum Magazin und wird dort neu beladen.

Dieses Konzept ist modular: Sollen später 4 oder 5 Farben sortiert werden, wird einfach die Kette verlängert und in der Software ein weiterer Auswurf-Index zugewiesen.

---

## Hardware- und Elektronik-Aufbau

* **Mikrocontroller:** ESP32 NodeMCU (zuständig für Taktschritte, Farbanalyse und Koordination der Servos).
* **Antrieb (Band):** NEMA 17 Schrittmotor über TMC2209/A4988 Treiber. Der Schrittmotor garantiert, dass das Band pro Takt exakt den Abstand von Fach zu Fach (75 mm) vorfährt und beim Stillstand aktiv gehalten wird.
* **Auswurf-Aktorik:** Servomotoren (z. B. MG90S mit Metallgetriebe), die über ein PCA9685 16-Kanal I2C-Servoboard angesteuert werden. Dadurch bleiben fast alle Pins des ESP32 frei.
* **Spannungsversorgung:** Zweikreis-System mit getrennter 12V/5V-Versorgung für Motor und Servos sowie 3,3V für Logik und Sensoren (verbunden über einen gemeinsamen Massebezug / Common Ground).

---

## Der kontinuierliche Taktzyklus

Im laufenden Betrieb wiederholt das System in einer festen Schleife folgende Schritte:

```text
[ SCHRITT 1: VORWÄRTS-TAKT ]
Schrittmotor dreht das Endlosband exakt 75 mm vor und blockiert die Position.

[ SCHRITT 2: DATEN-VERSCHIEBUNG ]
Das Array im ESP32 rückt alle Ball-Zustände um einen Platz weiter.

[ SCHRITT 3: MESSUNG ]
Der Farbsensor an Station 0 scannt den neu eingetroffenen Ball und schreibt das Ergebnis an Index 0.

[ SCHRITT 4: AUSWURF ]
Die Stationen prüfen zeitgleich ihre Indizes. Übereinstimmende Bälle werden parallel über Servos ausgeworfen.

[ SCHRITT 5: RÜCKSTELLUNG ]
Die Servos fahren zurück. Leere Kettenglieder laufen im Rücklaufkanal unten zurück zum Start.
