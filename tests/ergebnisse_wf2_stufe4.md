# WF2 · Testergebnisse Stufe 4 (anonymes Protokoll)

**Datum:** 20.11.2026 · **Tester:** Stefan · **System:** WF2 Stufe 4 – nach jeder Antwort bzw. Ablehnung ein Eintrag in der n8n Data Table `ki_kompass_log`; Auswertung (Thema, Beantwortbarkeit, Lücken-Stichwort) per Information Extractor mit Haiku, nur bei erlaubten Fragen
**Durchführung:** jede Frage in neuer Chat-Sitzung im n8n-Editor, danach Tabelle geprüft

## Protokollfelder

| Feld | Inhalt |
|---|---|
| zeitpunkt | Eingang der Frage, auf die volle Stunde abgerundet (Europe/Berlin) |
| kategorie | Ergebnis der Sicherheitsprüfung |
| thema | Leistungsbewertung · Ampel & Unterricht · Datenschutz & Tools · Eltern & Schüler · Erlasse NRW · Sonstiges (bei Ablehnung „–“) |
| beantwortbar | Konzept regelt die Frage ja/nein |
| luecke_stichwort | nur wenn nicht beantwortbar: 2–4 allgemeine Wörter |
| erlass_suche | Erlass_Suche wurde aufgerufen ja/nein |
| antwortzeit_s | Sekunden bis zur Antwort im Chat |
| version | Konzeptversion (v3.0) |
| feedback | leer (folgt mit Weboberfläche) |

Nie gespeichert: Fragetext, Antworttext, Sitzungs-ID, Namen. Spalte `tokens` bewusst weggelassen (Datenminimierung; Kosten pro Key in der OpenRouter-Übersicht).

## Ergebnisse

| ID | Frage (Kurz) | kategorie | thema | beantwortbar | luecke_stichwort | erlass_suche | antwortzeit_s | Bestanden |
|---|---|---|---|---|---|---|---|---|
| T02 | KI für Hausaufgaben | erlaubt | Ampel & Unterricht | ja | – | nein | 13,5 | ✅ |
| T12 | APO-SI und Täuschung | erlaubt | Leistungsbewertung | ja | – | **ja** | 15,5 | ✅ |
| T13 | Private Nutzung am Wochenende | erlaubt | Sonstiges | **nein** | „Private KI-Nutzung Lehrkräfte“ | nein | 9,9 | ✅ |
| T06 | Name + Notenfrage | personenbezug | – | – | – | nein | 1,4 | ✅ |

**4 / 4 bestanden:** je Frage genau ein Eintrag; Chat zeigt weiterhin die Antwort (Antwort wird vor Auswertung und Protokoll gesendet); in keiner Zeile Fragetext oder Name (T06: „Max Müller“ erscheint nirgends).

## Befunde

1. **Thema bei T12:** als „Leistungsbewertung“ statt „Erlasse NRW“ eingeordnet – inhaltlich vertretbar (Täuschung/Bewertung). Entscheidung: `thema` beschreibt den Inhalt, der Bedarf an Landesrecht wird über `erlass_suche` gezählt. Keine Änderung.
2. **Restrisiko Systemzeiten:** Die Data Table legt automatisch `createdAt`/`updatedAt` mit sekundengenauer Zeit an (nicht entfernbar). Damit wird die Rundung von `zeitpunkt` in der Rohtabelle umgangen. Bewertung: geringes Risiko (kein Name, kein Fragetext). Maßnahmen: Zugriff auf die Tabelle nur für Admins; Dashboard zeigt ausschließlich gerundete, aggregierte Werte. Verbesserung für später: Einträge gesammelt (z. B. stündlich) schreiben.
3. **Ausführungsdaten:** Erfolgreiche Produktions-Ausführungen werden nicht gespeichert (enthielten sonst Fragetexte); Fehler-Ausführungen werden zur Fehlersuche gespeichert → im Datenschutzkapitel mit Aufbewahrungsfrist dokumentieren.
4. **Robustheit:** Auswertung und Protokoll laufen mit „On Error: Continue“ – ein Fehler beim Protokollieren verhindert die Antwort nicht.
5. **Hinweis Test:** Die Data-Table-Ansicht aktualisiert sich nicht automatisch (neu laden).
