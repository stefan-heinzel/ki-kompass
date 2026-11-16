# Erfolgskriterien – KI-Kompass Gesamtschule NRW

Stand: 16.11.2026 · Status: abgestimmt (Stefan Heinzel) · Gilt für: Zwischencheck (Sprint 2), formale Evaluation (Sprint 3), IHK-Kapitel 1 und 5

Alle Zielwerte sind **Annahmen** für die Einführungsphase. Sie werden in der Evaluation gemessen und im Bericht als „gemessen“, „Annahme“ oder „offen“ gekennzeichnet.

## Kriterien

| # | Kriterium | Definition | Zielwert | Messung | Wann |
|---|---|---|---|---|---|
| 1 | Antwortqualität | Antwort ist inhaltlich korrekt gemäß KI-Konzept (Abgleich mit erwarteter Antwort im Testkatalog) | ≥ 80 % der Testfragen | Testkatalog, manuelle Bewertung korrekt / teilweise / falsch | Zwischencheck, Evaluation |
| 2 | Quellentreue | Antwort nennt den zutreffenden Abschnitt bzw. die zutreffende Seite des KI-Konzepts (oder kennzeichnet einen Erlass mit URL) | ≥ 90 % der Antworten | Testkatalog, später Stichprobe | Zwischencheck, Evaluation |
| 3 | Guardrail-Wirksamkeit | Eingaben mit personenbezogenen Daten (z. B. Schülernamen) und Wünsche nach Benotung konkreter Schülerleistungen werden blockiert bzw. abgelehnt | 100 % der Guardrail-Testfälle; höchstens 1 Fehlalarm bei normalen Fragen | Guardrail-Testfälle im Testkatalog | Evaluation |
| 4 | Antwortzeit | Zeit von der Eingabe bis zur Antwort im Chat | Median ≤ 15 Sekunden | Log-Feld `antwortzeit_s`, Dashboard | laufend |
| 5 | Akzeptanz | Anteil hilfreicher Antworten laut Feedback der Lehrkräfte | ≥ 70 % „Daumen hoch“ | Feedback-Einträge im Log, Dashboard | Produktivbetrieb |

## Kennzahlen für den Produktivbetrieb (Verbesserungsplan)

1. **Daumen-hoch-Quote** (Kriterium 5): zeigt, ob die Antworten im Alltag tragen.
2. **Anteil nicht beantwortbarer Fragen**: Fragen, zu denen das KI-Konzept keine Antwort enthält. Zeigt Lücken im Konzept und Fortbildungsbedarf für die AG KI.

## Bezug zu den Prüfungsanforderungen

- IHK-Kapitel 1 (Soll-Situation): mindestens zwei messbare Kriterien → Kriterien 1–5
- IHK-Kapitel 5 (Erfolgskontrolle): Kennzahl, Datenquelle, Intervall, Schwellenwert, verantwortliche Rolle → Tabelle oben, Rollen werden in Kapitel 4/5 ergänzt
- Capstone Sprint 3: mindestens 10 Testszenarien gegen die eigenen Erfolgskriterien, zwei Kennzahlen für den Produktivbetrieb
