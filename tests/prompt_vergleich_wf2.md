# WF2 · Prompt-Vergleich (Längenvorgabe)

**Ziel:** Wirkung einer Längenvorgabe im Systemprompt auf Antwortzeit, Tokenverbrauch und Quellentreue messen.
**Bezug:** Erfolgskriterien 2 (Quellentreue ≥ 90 %) und 4 (Antwortzeit Median ≤ 15 s), siehe `docs/erfolgskriterien.md`.

## Methode

- Workflow: WF2 · KI-Kompass Chat, Stufe 1 (Agent + Konzept-Suche, Modell anthropic/claude-sonnet-5.5, Temperatur 0,2, Top-K 5)
- Jede Frage in einer **neuen Chat-Sitzung** (Gedächtnis leer), Test im n8n-Editor
- Gemessen: Gesamtzeit und Tokens laut n8n-Logs; Seitenangaben im KI-Konzept v3.0 nachgeschlagen
- Einzige Änderung zwischen v1 und v2: Längenvorgabe im Abschnitt „Formatierung“ des Systemprompts

## Testfragen

| # | Frage | Kategorie |
|---|---|---|
| F1 | Darf ich Schüler:innen erlauben, KI für Hausaufgaben zu nutzen? | Unterrichtseinsatz |
| F2 | Was mache ich, wenn Schüler:innen zentrale Inhalte im Fachgespräch nicht erklären können? | Leistungsbewertung |
| F3 | Welche Regeln gelten für die Notengebung bei KI-gestützten Projekten? | Leistungsbewertung |

## Ergebnisse

| # | v1 Zeit | v1 Tokens | v1 Quellen korrekt | v2 Zeit | v2 Tokens | v2 Quellen korrekt |
|---|---|---|---|---|---|---|
| F1 | 16,9 s | 14.929 | 7 / 7 | 7,7 s | 13.544 | 6 / 6 |
| F2 | 14,4 s | 14.936 | – | 8,2 s | 14.531 | 6 / 6 |
| F3 | 19,5 s | 21.525 | – | 9,8 s | 19.960 | 8 / 8 |
| **Median / Summe** | **16,9 s** | **51.390** | | **8,2 s** | **48.035** | **20 / 20** |

v1 und v2 gemessen am 17.11.2026 (Stefan), Quellen im PDF geprüft. Quellenprüfung F1: alle Seitenangaben (S. 6, 8, 11, 32, 34, 35, 39) im PDF bestätigt; PDF-Seite = aufgedruckte Seitenzahl.

## Nebenbefund: Tokens wachsen innerhalb einer Sitzung

Dieselben drei Fragen nacheinander in **einer** Sitzung: 14.929 → 35.754 → 58.093 Tokens. Das Gedächtnis schickt den bisherigen Verlauf (offenbar inkl. gefundener Konzept-Abschnitte) bei jeder Frage mit. Folge: Kosten steigen mit der Gesprächslänge. → Maßnahme nach dem Prompt-Vergleich prüfen (Gedächtnisfenster verkleinern / nur Frage+Antwort speichern).

## Auswertung v1 → v2

- **Antwortzeit:** Median 16,9 s → 8,2 s (−51 %). Erfolgskriterium 4 (≤ 15 s) mit v2 erfüllt.
- **Tokens:** 51.390 → 48.035 (−7 %). Kosten sinken kaum: Der Großteil sind Eingabe-Tokens (gefundene Konzept-Abschnitte), nicht die Antwort. Kostenhebel liegt bei Top-K bzw. Abschnittslänge, nicht beim Prompt.
- **Quellentreue v2:** 20 / 20 Seitenangaben im PDF bestätigt. Kleinigkeit F3: „Fachgespräch zählt am meisten“ mit S. 22 belegt, die Gewichtung selbst steht auf S. 23/48. Ab der dritten Quelle teils nur „(S. 48)“ statt „(KI-Konzept v3.0, S. 48)“.
- **Länge:** Antworten ca. 140–190 Wörter, mit Angebot für Details.
- **Qualitativ (F2):** Der Assistent legt einen Widerspruch im Konzept offen (S. 15 „mangelhaft – ungenügend“ vs. S. 48 „mangelhaft“) und verweist auf die Fachkonferenz, statt ihn zu glätten. → Hinweis an AG KI; Demo-Kandidat.
- **Entscheidung:** Prompt v2 wird beibehalten.
