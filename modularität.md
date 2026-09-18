---

# DATEI 2: `HARDWARE_BOM.md`

```markdown
# Hardware-Stückliste (Bill of Materials - BOM) & Pinout

## 1. Elektronische Kernkomponenten

| Pos. | Komponente | Spezifikation / Modell | Anzahl | Schnittstelle / Betriebsspannung | Funktion |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1.1** | Mikrocontroller | ESP32 NodeMCU DevKit C V4 | 1 | 3.3V Logik (Versorgung: 5V Micro-USB) | Zentrale Steuereinheit |
| **1.2** | Farbsensor | Adafruit/AZ-Delivery TCS34725 | 1 | I2C (3.3V), integr. Neutralweiß-LED | Farberkennung an Station 0 |
| **1.3** | PWM-Servotreiber | PCA9685 (16-Kanal, 12-Bit) | 1 | I2C (3.3V Logik, 5V–6V Servo-VCC) | Modulare Aktor-Ansteuerung |
| **1.4** | Schrittmotor | NEMA 17 (17HS4401 o.ä., 1.5A, 42Ncm)| 1 | 12V Bipolarschrittmotor | Taktantrieb des Förderbands |
| **1.5** | Schrittmotortreiber| TMC2209 oder A4988 | 1 | STEP/DIR Interface (12V Motor, 3.3V Logik)| Präzise Schrittmotorsteuerung |
| **1.6** | Servomotoren | SG90 (Micro) oder MG90S (Metallgetr.)| 2+ | PWM (5V), Anschluss an PCA9685 | Auswurfstößel je Station |
| **1.7** | Optischer Endstopp | TCST2103 oder mechanischer Endschalter| 1 | Digital Input (3.3V) | Referenzfahrt (Homing) des Bands |

---

## 2. Spannungsversorgung & Passive Bauteile

| Pos. | Komponente | Spezifikation | Anzahl | Funktion |
| :--- | :--- | :--- | :--- | :--- |
| **2.1** | Schaltnetzteil | 12V DC / min. 3A | 1 | Hauptenergiequelle (Schrittmotor) |
| **2.2** | DC-DC Step-Down Regler | LM2596 (12V $\rightarrow$ 5V, min. 3A) | 1 | Stabile 5V-Versorgung für Servos |
| **2.3** | Elektrolytkondensator | $100\,\mu\text{F}$ bis $470\,\mu\text{F}$ (min. 25V) | 2 | Pufferung Motor- und Servospannung |
| **2.4** | Pegelwandler (Optional)| Logic Level Shifter 3.3V $\leftrightarrow$ 5V | 1 | Schutz der I2C-Leitungen bei 5V-Pullups |
| **2.5** | Terminal Block / PCB | Schraubklemmen & Lochrasterplatine | - | Sichere Stromverteilung & Common-GND |

---

## 3. Mechanische Komponenten & Struktur

| Pos. | Komponente | Spezifikation | Anzahl | Bemerkungen |
| :--- | :--- | :--- | :--- | :--- |
| **3.1** | Rahmenprofil | Aluminium-Konstruktionsprofil 2020 | ca. 1.5 m | Gestell für Band und Stationen |
| **3.2** | Antriebselement | Zahnriemen GT2 ($6\,\text{mm}$ Breite) + Pulleys | 1 Set | Spielfreier Riementrieb |
| **3.3** | Kugellager | 608ZZ (Skateboard-Lager) | 4 | Lagerung der Antriebs- & Umlenkachsen |
| **3.4** | 3D-Druck-Elemente | PLA oder PETG Filament | ca. 500 g | Bandsegmente, Trichter, Auswurfhebel |
| **3.5** | Sensor-Optikgehäuse | 3D-Druck (Schwarz, matt) | 1 | Blende zur Abschirmung von Umgebungslicht |

---

## 4. Vollständige Pin-Belegungsmatrix (ESP32)

Um Pin-Konflikte (Boot-Pins, interne Funktionen) auszuschließen, ist folgende feste Verdrahtung definiert:

```text
ESP32 NodeMCU V4
┌─────────────────────────────────────────────────────────┐
│ Pin       │ Funktion         │ Ziel-Komponente          │
├───────────┼──────────────────┼──────────────────────────┤
│ GPIO 21   │ I2C SDA          │ TCS34725 & PCA9685 (SDA) │
│ GPIO 22   │ I2C SCL          │ TCS34725 & PCA9685 (SCL) │
│ GPIO 26   │ STEP (Takt)      │ Motortreiber STEP        │
│ GPIO 27   │ DIR (Richtung)   │ Motortreiber DIR         │
│ GPIO 25   │ ENABLE (Freigabe)│ Motortreiber EN          │
│ GPIO 33   │ SENSOR_INT       │ TCS34725 LED-Steuerung   │
│ GPIO 34   │ LIMIT_SWITCH     │ Homing-Endschalter (IN)  │
│ 3V3       │ VCC (Logik)      │ TCS34725, ESP32 Logik    │
│ GND       │ Common Ground    │ Alle Komponenten (Masse) │
└─────────────────────────────────────────────────────────┘
