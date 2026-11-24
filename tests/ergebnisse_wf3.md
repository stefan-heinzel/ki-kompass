# WF3 · Testergebnisse Dashboard

**Datum:** 22.11.2026 · **Tester:** Stefan · **System:** WF3 · Dashboard – Webhook (GET, Basic Auth „Dashboard Schulleitung“) → Data Table `ki_kompass_log` lesen → Kennzahlen berechnen (nur Aggregate) → HTML-Seite (Chart.js) ausliefern
**Datenbasis:** 4 echte Testzeilen (v3.0) + 36 Demozeilen (version = „demo“, plausibel erfunden, im Dashboard als „Enthält Demodaten“ gekennzeichnet)

## Ergebnisse

| ID | Prüfung | Erwartet | Ergebnis | Bestanden |
|---|---|---|---|---|
| D01 | Aufruf mit korrektem Passwort (Test-URL) | Dashboard wird angezeigt | Seite mit allen Bausteinen geladen | ✅ |
| D02 | Hinweis Demodaten | Banner sichtbar, solange Zeilen mit version = demo existieren | „Enthält Demodaten – die Zahlen sind nicht echt.“ | ✅ |
| D03 | Kennzahl-Kacheln | Fragen, Anteil beantwortbar, Median Antwortzeit, Erlass-Suchen, Daumen-hoch-Quote | 40 · 79 % · 12,9 s · 7 · 79 % (19 Bewertungen) | ✅ |
| D04 | Diagramme | Themen, Wochentage, Stunden | alle drei gerendert (Chart.js) | ✅ |
| D05 | „Nicht im Konzept geregelt“ | Stichworte mit Häufigkeit, absteigend | u. a. „KI bei Förderplänen“ 3×, echter Fall „Private KI-Nutzung Lehrkräfte“ 1× | ✅ |
| D06 | Sicherheitsprüfung | Anzahl je Ablehnungsgrund | 5 Kategorien angezeigt | ✅ |
| D07 | Keine Einzelzeilen | nur Zusammenfassungen, kein Zeitstempel einzelner Fragen | erfüllt | ✅ |
| D08 | Hell/Dunkel-Umschalter | Wechsel funktioniert | funktioniert | ✅ |
| D09 | Aufruf ohne bzw. mit falschem Passwort (privates Fenster) | Zugriff verweigert | Zugriff verweigert (Gegenprüfung Julian, 23.11.) | ✅ |

## Befunde

1. **Test-Webhook:** Die Test-URL lauscht nur kurz und nur nach Klick auf „Execute workflow“ (unten im Editor). „Listen for test event“ im Node-Fenster testet nur den Webhook-Node selbst – der restliche Workflow läuft dann nicht.
2. **Sicherheit Basic Auth (wichtig):** In Test-Ausführungen zeigt n8n den Header `authorization` an. Er ist nur Base64-kodiert, Benutzername und Passwort lassen sich daraus zurückrechnen. Folgen: Passwort ändern, da es in Test-Screenshots sichtbar war; Screenshots und Logs von Webhook-Tests nicht weitergeben; Basic Auth nur über HTTPS (gegeben).
3. **Beschriftungen** im Kasten Sicherheitsprüfung nach erstem Test präzisiert („Benotungsanfrage abgelehnt“ statt „Benotung durch KI“, missverständlich).
4. **Credential „Dashboard Schulleitung“** mit „Allowed HTTP Request Domains = None“: wird nur für eingehende Prüfung genutzt, kann nicht an fremde Adressen gesendet werden.
5. **Datenschutz:** Lücken-Stichworte werden HTML-escaped; Konstante MIN_ANZAHL (Test: 1, empfohlen im Echtbetrieb: 3), damit seltene Einzelfragen nicht zuordenbar sind.
6. **Vor Echtbetrieb:** Demozeilen löschen (Filter version = demo), MIN_ANZAHL auf 3, WF3 nach Absprache veröffentlichen.
