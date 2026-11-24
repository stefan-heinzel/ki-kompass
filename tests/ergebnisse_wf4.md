# WF4 · Testergebnisse Chat-Oberfläche für Lehrkräfte

**Datum:** 22.–23.11.2026 · **Tester:** Stefan · **System:** WF4 · KI-Kompass Oberfläche (eigene Chat-Seite, aus n8n per Webhook ausgeliefert) → Frage-Weiterleitung (serverseitig, Basic Auth „Chat Lehrkräfte“) → WF2 (Sicherheitsprüfung, Agent, Protokoll); Feedback-Webhook schreibt anonyme Zeile in `ki_kompass_log`

## Entscheidung Zugang (23.11.)

Für Prototyp und Demo **ohne Anmeldung** auf der Seite. Absicherung: Pfad mit zufälligem Suffix (nicht im Repo dokumentiert, Export enthält Platzhalter `<SUFFIX>`), Kostenlimit am OpenRouter-Key (5 $), Veröffentlichung nur zu Test- und Demozeiten. WF2 selbst bleibt geschützt (Weiterleitung serverseitig mit Basic Auth). Im Echtbetrieb: Anmeldung über die schulische Plattform (z. B. LOGINEO NRW).

## Ergebnisse (Endstand 23.11.) – 7 / 7 bestanden

| ID | Prüfung | Erwartet | Ergebnis | Bestanden |
|---|---|---|---|---|
| O01 | Seite aufrufen | Chat-Seite mit Datenschutzhinweis und Beispielfragen | angezeigt | ✅ |
| O02 | Beispielfrage „Dürfen Schüler:innen KI für Hausaufgaben nutzen?“ | Antwort mit Quellen | 10,7 s; Quellen S. 8, 11, 32, 42; Widerspruch S. 11/42 benannt | ✅ |
| O03 | Darstellung | Markdown sicher gerendert (DOMPurify), kein ungefiltertes HTML | Fett, Listen korrekt; DOMPurify-Test: Script, Bild, Event-Handler, javascript:-Links, iframe, style werden entfernt | ✅ |
| O04 | 👍-Feedback | Status 200, Bestätigung, nur einmal pro Antwort | „Danke für Ihr Feedback“ | ✅ |
| O05 | Benotungsfrage „Welche Note soll ich für dieses Referat geben?“ | Ablehnung | 2,0 s, Ablehnungstext | ✅ |
| O06 | Hell/Dunkel | Umschalter funktioniert | funktioniert | ✅ |
| O07 | Dashboard zählt Feedback als Bewertung, nicht als Frage | Daumen-Quote steigt, Fragen gesamt nur um echte Fragen | wie erwartet (Produktions-URL WF3) | ✅ |

## Fehlerfälle auf dem Weg (für Kapitel 5)

| # | Symptom | Ursache | Erkannt durch | Lösung |
|---|---|---|---|---|
| 1 | 401 „Authorization is required!“ beim Chat-Aufruf | Browser sendet gespeicherte Basic-Auth-Daten nicht an den WF2-Chat-Endpunkt | Netzwerk-Reiter (Status, Response) | Frage-Weiterleitung über WF4 (serverseitig) |
| 2 | Auch Weiterleitung 401, vorher CORS-Preflight | n8n liefert Webhook-HTML in einer Sandbox aus → Seite hat Origin „null“, Anmeldedaten werden bei fetch nie automatisch gesendet (Schutzfunktion von n8n Cloud) | Preflight im Netzwerk-Reiter | Entscheidung: Demo ohne Anmeldung, CORS für die zwei Webhooks offen |
| 3 | CORS-Fehler nach Einschränkung auf die eigene Domain | Origin der Sandbox-Seite ist „null“, nicht die n8n-Domain | Netzwerk-Reiter | allowedOrigins „*“ (unkritisch, keine Anmeldedaten) |
| 4 | Antwort leer (`{"output":""}`) nach 0,4 s | WF2-Chat-Trigger im Modus „Using Response Nodes“ liefert nur `executionStarted`/`resumeToken`, Antwort asynchron | n8n-Ausführungsprotokoll von WF4 | WF2 auf „Respond to Webhook“ umgestellt; Reihenfolge Antwort → Auswertung → Protokoll bleibt |

Lehre: Jeder Fehler wurde mit Netzwerk-Reiter bzw. Ausführungsprotokoll eingegrenzt, nicht durch Raten.

## Hinweise

- Seit der Umstellung von WF2 auf „Respond to Webhook“ ist der Test-Chat im n8n-Editor („Open Chat“) ggf. nicht mehr nutzbar; Tests laufen über die Oberfläche.
- Vor der Präsentation WF2, WF3, WF4 veröffentlicht prüfen; danach WF4 wieder auf „unpublished“.
