# System- und Architekturplanung: Modularer Tischtennisball-Farbsortierer

## 1. Systemübersicht & Zielsetzung
Ziel des Projekts ist die Entwicklung einer automatisierten, getakteten Sortieranlage für Tischtennisbälle (Standarddurchmesser: 40 mm) basierend auf deren Oberflächenfarbe. Das System arbeitet nach dem Prinzip eines diskreten Fördersystems mit zentraler Vorab-Erkennung und virtuellem Daten-Tracking.

### Kernanforderungen:
- **Taktgenauigkeit:** Exakter Transport der Bälle von Station zu Station ohne Schlupf.
- **Hohe Sortierpräzision:** Zuverlässige Farberkennung unabhängig von wechselndem Umgebungslicht.
- **Modularität:** Skalierbarkeit von aktuell 2 Auswurfstationen auf $N$ Stationen ohne tiefgreifende Änderungen an Hard- oder Software.
- **Fehlertoleranz:** Erkennung und Ausschluss von Fehlsortierungen und mechanischen Verklemmungen.

---

## 2. Mathematisch-Physikalische Dimensionierung

### 2.1. Raster- und Geometriedaten
- **Ball-Durchmesser ($D_{ball}$):** $40\,\text{mm}$
- **Fachbreite / Slotgröße:** $50\,\text{mm}$ (ermöglicht $5\,\text{mm}$ Spielraum pro Seite zur Vermeidung von Reibung)
- **Rastermaß / Takt-Distanz ($S_{step}$):** $75\,\text{mm}$ von Taschenmitte zu Taschenmitte.
- **Stationenabstand:** Alle Auswurfstationen sind in ganzzahligen Vielfachen des Rastermaßes angeordnet ($Distance = n \times S_{step}$).
  - Station 0 (Farbsensor): $0\,\text{mm}$ (Index 0)
  - Station 1 (Auswurf Farbe 1, z. B. Rot): $1 \times 75\,\text{mm} = 75\,\text{mm}$ (Index 1)
  - Station 2 (Auswurf Farbe 2, z. B. Weiß): $2 \times 75\,\text{mm} = 150\,\text{mm}$ (Index 2)
  - Station $N$ (Modular erweiterbar): $N \times 75\,\text{mm}$ (Index $N$)

### 2.2. Kinematik & Antriebsberechnung
- **Antrieb:** Schrittmotor (NEMA 17, 200 Schritte/Umdrehung, $1{,}8^\circ$ Schrittwinkel)
- **Treiberrad-Umfang ($U_{rad}$):** z. B. Zahnriemenrad GT2 (mit $T=40$ Zähnen $\rightarrow$ $U = 40 \times 2\,\text{mm} = 80\,\text{mm}$).
- **Schritte pro Takt:**
  $$\text{Schritte} = \frac{S_{step}}{U_{rad}} \times 200 \times \text{Microstepping-Faktor}$$
  *(Exakte Kalibrierung erfolgt spielfrei über Microstepping im Treiber).*

---

## 3. Systemarchitektur

### 3.1. Mechanische Teilsysteme
1. **Vereinzelung & Zuführung (Feeder):**
   - Schwerkraft-Magazin (Rutsche mit $42\,\text{mm}$ Innenkanal).
   - Mechanische Wippe / Sperrklinke, die synchronisiert mit dem Band immer exakt einen Ball freigibt, sobald ein leerer Slot unter der Zuführung steht.
2. **Getaktetes Förderband (Indexer Conveyor):**
   - Förderkette/Zahnriemen mit aufgesetzten 3D-Druck-Gabeln/Taschen.
   - Lineare Führung des Bandes in einer U-Profil-Schiene, um Durchhängen und Vibrationen zu verhindern.
3. **Auswerfereinheit (Ejector):**
   - Linearhebel, angetrieben durch Servomotoren.
   - Auswurf erfolgt im $90^\circ$-Winkel zur Förderrichtung in segmentierte Fallkanäle/Auffangbehälter.

### 3.2. Elektronische Architektur & Bus-Topologie
- **Zentraler Controller:** ESP32 NodeMCU (3.3V Logik).
- **Sensor-Bus (I2C):** Farbsensor TCS34725 läuft über SDA (GPIO 21) und SCL (GPIO 22).
- **Aktor-Bus (I2C):** PCA9685 16-Kanal PWM-Treiber am selben I2C-Bus. Servos werden nicht direkt über die ESP32-GPIOs angesteuert, was die Skalierbarkeit garantiert (bis zu 16 Stationen ohne zusätzliche Pins).
- **Leistungsentkopplung:**
  - Versorgungskreis A (Logik): 5V USB / 3.3V ESP32 ($\approx 200\,\text{mA}$).
  - Versorgungskreis B (Aktoren): 5V–6V High-Current (min. 3A für Servos).
  - Versorgungskreis C (Schrittmotor): 12V DC (min. 2A für NEMA 17).
  - **Gemeinsame Masse (Common GND) über alle Kreise hinweg zwingend erforderlich.**

---

## 4. Software-Architektur (Schieberegister-Prinzip)

Das System nutzt ein datenbasiertes Schieberegister (**FIFO-Array**), das den physikalischen Zustand des Förderbands im RAM des ESP32 1:1 abbildet.

### 4.1. Datenstruktur
Ein Array fester Länge $N+1$, wobei jeder Index einem physikalischen Slot auf dem Band entspricht:
```text
Index 0 = Position Farbsensor
Index 1 = Position Auswurfstation 1
Index 2 = Position Auswurfstation 2
...
Index N = Position Auswurfstation N
