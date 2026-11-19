# WF2 · Testergebnisse Stufe 3 (Erlass-Suche) und Regression

**Datum:** 19.11.2026 · **Tester:** Stefan · **System:** WF2 Stufe 3 – Sicherheitsprüfung (Haiku), Agent (Sonnet) mit Konzept_Suche (Qdrant, Top-K 5) und Erlass_Suche (Tavily, nur schulministerium.nrw und recht.nrw.de), Systemprompt v3
**Durchführung:** jede Frage in neuer Chat-Sitzung; Konzeptquellen im KI-Konzept v3.0 (PDF) nachgeschlagen, Erlassquelle auf schulministerium.nrw aufgerufen

## Ergebnisse

| ID | Kurzbeschreibung | Werkzeuge | Zeit | Tokens | Quellen korrekt | Bestanden |
|---|---|---|---|---|---|---|
| T12 | APO-SI und Täuschung mit KI | Konzept_Suche, Erlass_Suche | 14,3 s | 14.540 | 3 offizielle NRW-Quellen + S. 35, 37 | ✅ |
| T13 | Private KI-Nutzung am Wochenende (nicht beantwortbar) – Lauf 1 | Konzept_Suche | 4,5 s | 15.953 | 4 / 4 (S. 33) | ✅ |
| T13 | – Lauf 2 (Wiederholung) | Konzept_Suche | 9,6 s | 17.440 | – | ✅ |
| T04 | Regeln Notengebung (Regression) | Konzept_Suche | 5,5 s | 16.421 | 6 / 6 (S. 22, 40, 47, 48) | ✅ |
| T02 | KI für Hausaufgaben (Regression) | Konzept_Suche | 6,7 s | 10.603 | 6 / 6 (S. 8, 11, 32, 42) | ✅ |

**Antwortzeit (alle Läufe):** Median **6,7 s** (Ziel ≤ 15 s erfüllt).

## Befunde

1. **Erlass-Suche (T12):** Agent formuliert die Suchanfrage selbst („APO-SI Täuschungsversuch KI“); Treffer ist der offizielle MSB-Handlungsleitfaden 2023. Antwort gibt nur Gefundenes wieder, trennt Landesrecht und Konzept, kennzeichnet externe Quellen und stellt korrekt fest, dass die APO-SI keine KI-spezifische Regel enthält. Nachweis: Tool-Input und -Output im n8n-Log.
2. **Kennzeichnung:** Prompt-Vorgabe „Offizieller MSB-Erlass NRW“ wird auch für Verordnung (APO-SI) und Handreichung (Leitfaden) verwendet → Formulierung im Prompt auf „Offizielle NRW-Quelle“ verallgemeinern (optional).
3. **Quellenformat (T04):** Seit Prompt v3 durchgehend vollständig „(KI-Konzept v3.0, S. X)“; Länge ca. 170 Wörter (v2: 230). Formatmängel vom 18.11. behoben.
4. **T13:** Keine erfundene Regel; klare Abgrenzung privat/dienstlich mit Verweis an die AG KI.
5. **Streuung der Antwortzeit:** Gleiche Frage (T13) 4,5 s bzw. 9,6 s bei ähnlichen Tokens → Latenz hängt von der Auslastung beim Modellanbieter ab. Bewertung deshalb über den Median mehrerer Fragen.
6. **Konzept-Unschärfe Nr. 4 (T02, Hinweis an AG KI):** Ampelstufe Gelb ist widersprüchlich definiert – S. 11 „mit Kennzeichnung“, S. 42 (Klassenplakat) „nur mit Erlaubnis der Lehrkraft“.

## Behobene Bau-Fehler auf dem Weg zu Stufe 3 (für Kapitel 5)

| Fehler | Ursache | Erkannt durch | Lösung |
|---|---|---|---|
| „The name of this tool is not a valid alphanumeric string“ | n8n nutzt den Node-Namen als Tool-Namen; Bindestrich unzulässig | Erster Testlauf (Validierung grün) | Nodes in `Erlass_Suche` / `Konzept_Suche` umbenannt |
| „toolHttpRequest has a supplyData method but no execute method“ | Veralteter LangChain-Node, in der n8n-Version nicht mehr lauffähig | Zweiter Testlauf (Validierung grün) | Ersetzt durch `n8n-nodes-base.httpRequestTool` v4.5 mit `$fromAI` |

Lehre: Statische Validierung erkennt Laufzeitfehler nicht – jeder Bauschritt braucht einen echten Testlauf.

## Konzept-Unschärfen gesamt (Stand 19.11.)

| # | Fundstelle | Unschärfe | Gefunden in |
|---|---|---|---|
| 1 | S. 15 vs. S. 48 | Veto-Regel: „mangelhaft – ungenügend“ vs. „mangelhaft“ | F2 / T03 |
| 2 | – | Hausaufgaben: Lern- oder Prüfungssituation? nicht eindeutig | T02 (18.11.) |
| 3 | S. 29 → Anhang 8 (S. 60) | Verweis auf Nachteilsausgleich ins Leere | T16-K |
| 4 | S. 11 vs. S. 42 | Ampel Gelb: „mit Kennzeichnung“ vs. „nur mit Erlaubnis“ | T02 (19.11.) |
