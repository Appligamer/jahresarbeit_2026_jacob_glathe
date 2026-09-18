---

# DATEI 3: `AUSFUEHRUNGSPLAN.md`

```markdown
# Phasen- und Ausführungsplan: Tischtennisball-Sortieranlage

Dieser Plan führt schrittweise durch Entwicklung, Bau, Inbetriebnahme und Qualitätssicherung. Jede Phase enthält verbindliche Meilensteine (Definition of Done), bevor die nächste Phase begonnen wird.

---

## Phase 1: Elektronik-Prototyping & Sensor-Kalibrierung (Breadboard)
**Ziel:** Nachweis der funktionierenden Messkette und Aktor-Ansteuerung ohne Mechanik.

- [ ] **Schritt 1.1:** Aufbau der Stromversorgung auf dem Breadboard (12V für Motor, 5V Step-Down für Servos, Common Ground sicherstellen).
- [ ] **Schritt 1.2:** I2C-Bus einrichten und I2C-Scanner ausführen $\rightarrow$ Bestätigung der Adressen `0x29` (Sensor) und `0x40` (PCA9685).
- [ ] **Schritt 1.3:** Kalibrierversuche Farbsensor TCS34725:
  - Bau einer provisorischen, lichtdichten Messkammer (Abstand Ball zu Sensor: exakt $10\,\text{mm}$ bis $15\,\text{mm}$).
  - Erfassung der Rohdaten (Red, Green, Blue, Clear) für:
    - Ball Rot
    - Ball Weiß
    - Leerzustand (kein Ball in Messkammer)
  - Definition der Schwellenwert-Logik im Farbraum (RGB zu HSV-Umrechnung empfohlen zur Helligkeitsunabhängigkeit).
- [ ] **Schritt 1.4:** Testlauf des Schrittmotortreibers (Microstepping-Jumper auf 1/16-Schritt für vibrationsfreien Lauf).
- **Meilenstein 1:** Alle Aktoren und Sensoren lassen sich isoliert fehlerfrei ansteuern; Farberkennung unterscheidet Rot, Weiß und Leer mit 100% Wiederholgenauigkeit.

---

## Phase 2: CAD-Konstruktion & Mechanischer Aufbau
**Ziel:** Fertigstellung des physischen Förder- und Sortiersystems.

- [ ] **Schritt 2.1:** CAD-Design der funktionskritischen Teile:
  - Bandkettenglieder mit Kugelmulden ($D = 40\,\text{mm}$, Raster $75\,\text{mm}$).
  - Auswurfhebel (Radius passend zur Ballkrümmung, um Verkeilen zu verhindern).
  - Sensor-Messbrücke mit integrierter LED-Abschirmung.
  - Magazin-Rutsche zur automatischen Schwerkraft-Zuführung.
- [ ] **Schritt 2.2:** 3D-Druck aller Komponenten mit hoher Maßhaltigkeit (Infill $\ge 25\%$).
- [ ] **Schritt 2.3:** Montage des Grundrahmens aus 2020-Aluprofilen.
- [ ] **Schritt 2.4:** Einbau von Band, Schrittmotor, Linearführung und Riemenspannern.
- [ ] **Schritt 2.5:** Montage der Servos an Position $x_1 = 75\,\text{mm}$ (Station 1) und $x_2 = 150\,\text{mm}$ (Station 2).
- **Meilenstein 2:** Die Mechanik lässt sich händisch ohne Ruckeln durchdrehen; der Schrittmotor bewegt das Band spielfrei.

---

## Phase 3: Hardware-Software-Integration & Kalibrierung
**Ziel:** Verknüpfung von Steuerungslogik mit der physischen Anlage.

- [ ] **Schritt 3.1:** Ermittlung der exakten Schrittzahl pro Takt ($S_{step} = 75\,\text{mm}$).
  - Implementierung einer Beschleunigungs- und Bremsrampe (Trapezprofil), damit die Bälle beim Anfahren/Stoppen nicht aus den Taschen geschleudert werden.
- [ ] **Schritt 3.2:** Nullpunkt-Referenzierung (Homing):
  - Förderband fährt beim Start langsam rückwärts, bis der Referenz-Endschalter auslöst $\rightarrow$ Nullpunkt definiert.
- [ ] **Schritt 3.3:** Integration der Schieberegister-Logik (Array-Shift bei jedem Takt).
- [ ] **Schritt 3.4:** Synchronisationsabgleich:
  - Timing-Abstimmung: Sensor-Messzeit (Integrationszeit TCS34725 ca. $50\,\text{ms}$) $\rightarrow$ Stoppzeit des Bandes optimieren.
- **Meilenstein 3:** Das System durchläuft einen Trockenzyklus ohne Bälle; die Stationen schalten rein softwaregesteuert präzise im Takt.

---

## Phase 4: Validierung, Testmatrix & Fehlerbehandlung
**Ziel:** Nachweis der Sortiergenauigkeit unter realen Betriebsbedingungen.

- [ ] **Schritt 4.1:** Durchführung der standardisierten Testmatrix (100 Durchläufe):
  - 30x Roter Ball
  - 30x Weißer Ball
  - 20x Gemischte Reihenfolge (Rot-Weiß-Rot-Rot-Weiß...)
  - 20x Leertakte dazwischen
- [ ] **Schritt 4.2:** Implementierung der Fehlerbehandlungsroutinen (Safety & Error Handling):
  - Sensor liefert ungültigen Wert $\rightarrow$ Status `UNBEKANNT` $\rightarrow$ Ball wird bis zum Bandende durchgelassen (Ausschuss-Behälter).
  - Blockade-Erkennung Schrittmotor $\rightarrow$ Not-Aus / Stall-Warnung.
- [ ] **Schritt 4.3:** Optimierung der Gesamt-Taktzeit (Ziel: $\le 1{,}5\,\text{Sekunden}$ pro Sortierschritt).
- **Meilenstein 4:** Fehlerrate liegt unter $1\%$; kein Klemmen über 100 Testzyklen.

---

## Phase 5: Dokumentation & NWT-Präsentationsreife
**Ziel:** Wissenschaftliche Aufbereitung für die Jahresarbeit.

- [ ] **Schritt 5.1:** Erstellung des finalen Schaltplans (z. B. in Fritzing oder KiCAD).
- [ ] **Schritt 5.2:** Daten-Logging über ESP32-Schnittstelle (z. B. Ausgabe von Sortierstatistik: Gesamtzahl Bälle, Sortierrate, Farbverteilung via Serieller Monitor / Webserver).
- [ ] **Schritt 5.3:** Nachweis der Modularität im Bericht:
  - Theoretische Dokumentation: „Erweiterung auf 4 Farben durch Hinzufügen von 2 Servos am PCA9685 und 3 Zeilen Code“.
- **Meilenstein 5:** Vollständige Dokumentation, betriebsbereiter Demonstrator für das Kolloquium.

---

## 6. Risikomatrix & Gegenmaßnahmen (Risk Mitigation)

| Risiko / Fehlerquelle | Auswirkung | Präventiv- / Gegenmaßnahme |
| :--- | :--- | :--- |
| **Streulicht verändert RGB-Werte** | Falschauswurf | Vollständig geschlossenes Sensorgehäuse; Nutzung der integrierten Weißlicht-LED als einzige Lichtquelle; Kalibrierung auf relative Farbverhältnisse statt absolute Helligkeit. |
| **Ball verfehlt Auswurfkanal** | Verklemmung des Bands | Trichterförmige Auffanggeometrie; Servo-Hebel mit elastischer Lippe (TPU oder Moosgummi). |
| **Spannungseinbruch bei Servo-Aktivierung** | ESP32 stürzt ab (Brownout Reset) | Getrennte Netzteile/Spannungsregler; $470\,\mu\text{F}$-Pufferkondensator direkt am Stromeingang des PCA9685. |
| **Schrittverlust am Förderband** | Ortsversatz der Bälle zu den Stationen | Formschlüssiger Zahnriemenantrieb; konservative Beschleunigungswerte; TMC2209-Treiber im StealthChop/SpreadCycle-Modus. |
