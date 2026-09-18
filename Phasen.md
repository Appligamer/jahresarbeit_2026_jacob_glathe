+-------------------------------------------------------------+
|                      PHASE 1: TRANSPORT                     |
|  - Schrittmotor fährt exakt S_step vor.                     |
|  - Band stoppt und arretiert (Haltemoment aktiv).           |
+------------------------------+------------------------------+
                               |
+------------------------------v------------------------------+
|                      PHASE 2: SHIFT                         |
|  - Daten-Array wird um 1 Position nach rechts verschoben.   |
|  - Index 0 wird temporär auf LEER gesetzt.                  |
+------------------------------+------------------------------+
                               |
+------------------------------v------------------------------+
|                      PHASE 3: ERKENNUNG                     |
|  - Farbsensor an Station 0 misst RGBC-Werte.                |
|  - Klassifikations-Logik bestimmt: LEER, ROT, WEISS etc.     |
|  - Ergebnis wird in Index 0 geschrieben.                    |
+------------------------------+------------------------------+
                               |
+------------------------------v------------------------------+
|                      PHASE 4: AUSWURF                       |
|  - ESP32 iteriert durch Array-Indizes:                      |
|    * Wenn Index 1 == ROT   -> Aktiviere Servo 1             |
|    * Wenn Index 2 == WEISS -> Aktiviere Servo 2             |
|  - Alle relevanten Servos fahren zeitgleich aus.            |
|  - Verweilzeit (Dwell Time: ca. 250ms).                     |
|  - Servos fahren in Grundstellung zurück.                   |
|  - Ausgeworfene Slots im Array werden auf LEER gesetzt.     |
+------------------------------+------------------------------+
                               |
+------------------------------v------------------------------+
|                      PHASE 5: BEREIT                        |
|  - Bereit für nächsten Zyklus (Freigabe oder Wartetakt).    |
+-------------------------------------------------------------+
