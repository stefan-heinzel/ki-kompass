# AGENTS.md – Regeln für KI-Assistenten (Codex u. a.)

Dieses Repository wird von zwei Personen mit unterschiedlichen KI-Assistenten bearbeitet: Stefan Heinzel mit Claude Code, Julian Hildebrandt mit Codex, Marcel Jahn (seit 20.11.). **Für alle Assistenten gelten dieselben Regeln.** Die vollständige Projektbeschreibung steht in [`CLAUDE.md`](CLAUDE.md), der aktuelle Projektstand in [`README.md`](README.md). Bei Widersprüchen gilt `CLAUDE.md`.

## Projekt in einem Satz
RAG-basierter KI-Assistent in n8n für die 150 Lehrkräfte einer Gesamtschule in NRW: beantwortet Fragen zum schulischen KI-Konzept mit Quellenangabe, prüft jede Frage vorab (Guardrails), sucht Landesrecht NRW über eine Erlass-Suche und protokolliert anonym für ein Dashboard der Schulleitung. Capstone-Projekt MSIT / IHK-Prüfung „KI-Tool-Experte/-in“.

## Verbindliche Regeln
1. **Sprache:** Deutsch – Antworten, Dateien, Workflow- und Node-Namen.
2. **Keine Geheimnisse:** Niemals Schlüssel, Tokens oder Passwörter in Dateien, Commits oder Ausgaben schreiben. Zugangsdaten liegen ausschließlich als Credentials in n8n.
3. **`_privat/` ist tabu für Git:** Der Ordner (KI-Konzept als PDF, Prüfungsvorgaben) wird nie committet und nie nach `docs/`, `tests/` oder `workflows/` kopiert. Er existiert nur lokal bei Stefan.
4. **Keine personenbezogenen Daten:** Keine echten Namen, Kontaktdaten oder Gesundheitsangaben von Schüler:innen, Eltern oder Lehrkräften in Testdaten, Logs oder Dokumenten. Testfälle nutzen erfundene Daten.
5. **Workflows:** Bestehende Workflows nie löschen. Vor einer Änderung den aktuellen Stand in n8n mit dem Export in `workflows/` vergleichen; Abweichungen zuerst melden („Live-Stand zuerst“). Nach jeder Änderung validieren und den Export aktualisieren. Nichts veröffentlichen (Publish), ohne dass beide zugestimmt haben.
6. **Werkzeugnamen in n8n:** Nodes, die einem KI-Agenten als Werkzeug dienen, brauchen Namen ohne Bindestrich (z. B. `Konzept_Suche`, `Erlass_Suche`).
7. **Prompts:** Systemprompts liegen versioniert in `prompts/`. Bei Prompt-Änderungen Workflow und Datei gemeinsam aktualisieren und die Wirkung in `tests/` dokumentieren.
8. **Beitragslog (`docs/Beitragslog_KI-Kompass.xlsx`):** Excel-Dateien kann Git nicht zusammenführen. Immer erst `git pull`, dann bearbeiten, dann sofort committen und pushen. Nie gleichzeitig mit der anderen Person bearbeiten – vorher kurz absprechen. In Excel bearbeiten (nicht Numbers), damit Auswahllisten und Formeln erhalten bleiben.
9. **Vor jedem Push:** `git pull`, Dateien auf Schlüssel prüfen, aussagekräftige deutsche Commit-Nachricht.
10. **Echte Dateien statt Links:** Dokumente immer als Datei ins Repo legen – keine Verknüpfungen (`.url`) oder Links auf Chat- oder Cloud-Ablagen (laufen ab, enthalten Zugriffsschlüssel).
11. **Ehrlich dokumentieren:** Testergebnisse nur eintragen, wenn der Lauf tatsächlich stattgefunden hat; Fehler und Abweichungen werden dokumentiert, nicht geglättet.

## Ordner
| Ordner | Inhalt |
|---|---|
| `workflows/` | n8n-Exporte (JSON, ohne Schlüssel): WF1 Wissensbasis, WF2 Chat, WF3 Dashboard |
| `prompts/` | versionierte Systemprompts |
| `tests/` | Testkatalog, Messungen, Testergebnisse |
| `docs/` | Erfolgskriterien, Beitragslog, Architektur, Use Case, Bericht-Entwürfe |

## Zuständigkeiten (Stand 19.11.2026)
- **Stefan:** Workflows WF1–WF3, Tests der Workflows, README.
- **Julian:** Testkatalog und formale Evaluation, Kapitel 1–3 und Evaluationsbericht der KI-Einsatz-Planung, Gegenprüfung.
- Der aktuelle Arbeitsplan liegt außerhalb des Repos (geteiltes Sheet).
