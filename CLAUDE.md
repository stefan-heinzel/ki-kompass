# KI-Kompass Gesamtschule NRW – Projektbeschreibung für Claude Code

## Projekt
RAG-basierter KI-Agent in n8n für die 150 Lehrkräfte einer Gesamtschule in NRW. Er beantwortet Fragen zum verbindlichen schulischen KI-Konzept (PDF, 62 Seiten) mit Quellenangabe, ergänzt Landesrecht NRW über eine Websuche und protokolliert anonym für ein Dashboard der Schulleitung.
Capstone-Projekt AI Automations & Agents (MSIT München) und IHK-Prüfung „KI-Tool-Experte/-in“. Team: Stefan Heinzel, Julian Hildebrandt. Abgabe: Do 26.11.2026, Prüfung: Do 03.12.2026.

## Arbeitsregeln
- Sprache: Deutsch (Antworten, Workflow-Namen, Node-Namen, Kommentare).
- Sparsam mit Credits arbeiten: gezielte Aufträge, keine unnötigen Wiederholungen, keine großen Dateien lesen, wenn es nicht nötig ist.
- Vor jeder Änderung an n8n kurz sagen, was du vorhast. Bestehende Workflows nie löschen; nur auf ausdrücklichen Auftrag ändern.
- Workflows nach jeder fertigen Änderung mit den n8n-MCP-Tools validieren und als JSON nach `workflows/` exportieren (Dateiname siehe unten).
- **Live-Stand zuerst:** Stefan und Julian ändern Workflows auch direkt im n8n-Editor. Vor jeder Änderung den aktuellen Stand aus n8n lesen und mit dem Export in `workflows/` vergleichen. Abweichungen (fehlende oder geänderte Parameter, verschobene Verbindungen) zuerst melden und nicht stillschweigend überschreiben.
- **Editor-feste Parameter:** Beim Speichern im Editor entfernt n8n Parameter, die im aktuellen Node-Schema nicht angezeigt werden (Beispiel 18.11.: `initialMessages` bei nicht-öffentlichem Chat Trigger, `topK` der Konzept-Suche). Parameter daher nur mit den Namen und Bedingungen des aktuellen Node-Schemas setzen und nach dem Speichern erneut auslesen.
- **Werkzeugnamen:** Nodes, die einem Agenten als Tool dienen, ohne Bindestrich benennen (z. B. `Konzept_Suche`, `Erlass_Suche`) – n8n nutzt den Node-Namen als Tool-Namen. Nur aktuelle Node-Typen verwenden; nach jedem Bauschritt einen echten Testlauf einplanen, da die Validierung Laufzeitfehler nicht erkennt.
- Julian arbeitet mit Codex; dessen Regeln stehen in `AGENTS.md` und müssen inhaltlich mit dieser Datei übereinstimmen. Regeländerungen in beiden Dateien nachziehen.
- Systemprompts liegen versioniert in `prompts/`. Bei Prompt-Änderungen Workflow und Datei gemeinsam aktualisieren.
- Niemals Schlüssel, Tokens oder Passwörter in Dateien, Commits oder Ausgaben schreiben. Zugangsdaten liegen ausschließlich als Credentials in n8n.
- `_privat/` wird nie committet (steht in `.gitignore`). Daraus nur lesen, nichts kopieren nach `docs/` oder `workflows/`.
- Personenbezogene Daten von Schüler:innen, Eltern oder Lehrkräften kommen in Testdaten und Logs nicht vor.

## Ordner
- `docs/` – Projektdokumente (Erfolgskriterien, Beitragslog, Architektur, Bericht-Entwürfe)
- `workflows/` – n8n-Workflow-Exporte (JSON)
- `tests/` – Testkatalog und Evaluationsergebnisse
- `prompts/` – versionierte Systemprompts
- `_privat/msit-vorgaben/` – Prüfungsvorgaben (README.md = Überblick), nur lesen bei Bedarf
- `_privat/wissensbasis/` – das KI-Konzept als PDF (Wissensbasis)

## Technik
- n8n Cloud: https://ki-kompass.app.n8n.cloud (über den MCP-Server `n8n-mcp` erreichbar)
- Sprachmodell: OpenRouter-Credential „OpenRouter - n8n“ (eigener Key mit Kostenlimit). Modelle: `anthropic/claude-sonnet-5.5` (Agent), `anthropic/claude-haiku-5.5` (Klassifizierung, Guardrails). Nur diese Modelle verwenden.
- Embeddings: Google Gemini, Modell `models/gemini-embedding-001` (Credential vom Typ Google Gemini(PaLM) Api). OpenRouter-Embeddings sind für die Kohorte gesperrt.
- Vector Store: Qdrant Cloud (EU, Frankfurt), Collection `ki_konzept`.
- Websuche: Tavily über HTTP Request mit Header-Auth-Credential, nur Domains `schulministerium.nrw` und `recht.nrw.de`.
- Logging: n8n Data Table `ki_kompass_log` (anonym: keine Namen, keine Fragetexte, keine Session-ID).
- Credentials existieren bereits in n8n und sind mit Julian geteilt. Vorhandene Credentials per Typ/Name auswählen, keine neuen anlegen.

## Workflows
| Kürzel | Name in n8n | Export-Datei | Zweck |
|---|---|---|---|
| WF1 | WF1 · Wissensbasis Ingest | `workflows/wf1_wissensbasis_ingest.json` | PDF per Formular hochladen → Text-Splitter → Gemini-Embeddings → Qdrant |
| WF2 | WF2 · KI-Kompass Chat | `workflows/wf2_ki_kompass_chat.json` | Chat → Guardrails → Themen-Zuordnung → Agent (RAG + Erlass-Suche) → Log |
| WF3 | WF3 · Dashboard | `workflows/wf3_dashboard.json` | Webhook → Data Table → HTML-Dashboard (Chart.js) |

Details zu Nodes und Parametern: `docs/Architektur_KI-Kompass.pdf` (v0.1). Aktueller Stand, Credentials und Startanleitung: `README.md`. Abweichungen (z. B. Gemini statt OpenAI für Embeddings, OpenRouter statt Anthropic direkt) gehen dem Architektur-PDF vor.

## Erfolgskriterien
Siehe `docs/erfolgskriterien.md`.
