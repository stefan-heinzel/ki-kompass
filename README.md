# KI-Kompass Gesamtschule NRW

RAG-basierter KI-Assistent (n8n) für die 150 Lehrkräfte einer Gesamtschule in NRW. Er beantwortet Fragen zum verbindlichen schulischen KI-Konzept (62 Seiten) – jede Aussage mit Quellenangabe und Seitenzahl. Geplant sind außerdem Guardrails (keine Benotung, keine personenbezogenen Daten), eine Erlass-Suche für NRW und ein anonymes Nutzungs-Dashboard für die Schulleitung.

_Capstone-Projekt AI Automations & Agents (MSIT München / IHK „KI-Tool-Experte“) · Stefan Heinzel & Julian Hildebrandt · Stand: 22.11.2026_

---

## Inhalt

1. [Problem und Lösung](#problem-und-lösung)
2. [Architektur](#architektur)
3. [Projektstand](#projektstand)
4. [Startanleitung](#startanleitung)
5. [Evidenz](#evidenz)
6. [Grenzen und bekannte Punkte](#grenzen-und-bekannte-punkte)
7. [Projektstruktur](#projektstruktur)

---

## Problem und Lösung

**Problem:** Das KI-Konzept der Schule ist verbindlich, aber 62 Seiten lang. Lehrkräfte brauchen im Alltag schnelle, verlässliche Antworten („Darf ich KI für Hausaufgaben erlauben?“, „Was gilt, wenn jemand im Fachgespräch nichts erklären kann?“) und finden sie im Dokument nur mühsam.

**Lösung:** Ein Chat-Assistent durchsucht das Konzept (Retrieval-Augmented Generation) und antwortet kurz, sachlich und mit Beleg im Format „(KI-Konzept v3.0, S. X)“. Steht etwas nicht im Konzept, sagt er das und verweist an die AG KI. Der Mensch behält das letzte Wort (Human-in-the-Loop).

## Architektur

Drei getrennte n8n-Workflows, die nur über zwei Datenspeicher verbunden sind:

| Workflow | Ausgelöst von | Aufgabe | Datenspeicher |
|---|---|---|---|
| **WF1 · Wissensbasis Ingest** | Admin (Upload-Formular mit Passwort) | PDF seitenweise einlesen, in Abschnitte zerlegen, als Vektoren speichern; alte Version wird vorher ersetzt | schreibt in **Qdrant** (`ki_konzept`) |
| **WF2 · KI-Kompass Chat** | Lehrkraft (Chat) | Sicherheitsprüfung jeder Frage, dann Antwort mit Quelle aus Konzept-Suche und Erlass-Suche (offizielle NRW-Quellen) | liest aus **Qdrant**, schreibt später ins **Protokoll** |
| **WF3 · Dashboard** | Schulleitung (Webseite mit Passwort) | anonyme, zusammengefasste Nutzungsstatistik: Themen, Beantwortbarkeit, Konzeptlücken, Sicherheitsprüfung, Spitzenzeiten | liest aus dem **Protokoll** (n8n Data Table) |

**Eingesetzte Dienste**

| Zweck | Dienst | Hinweis |
|---|---|---|
| Workflow-Plattform | n8n Cloud (Starter) | |
| Vektordatenbank | Qdrant Cloud, Region AWS eu-central-1 (Frankfurt) | kostenloser Cluster reicht |
| Embeddings | Google Gemini `models/gemini-embedding-001` (3072 Dimensionen) | bezahlte Stufe nötig (s. Startanleitung) |
| Sprachmodell Chat | `anthropic/claude-sonnet-5.5` über OpenRouter, Temperatur 0,2 | |
| Sicherheitsprüfung | `anthropic/claude-haiku-5.5` über OpenRouter, Temperatur 0 | Text Classifier mit 5 Kategorien + Fallback |
| Erlass-Suche | Tavily, fest begrenzt auf `schulministerium.nrw` und `recht.nrw.de` | kostenloses Kontingent (1.000 Suchen/Monat) |

Detailliertes Architekturdokument: [`docs/Architektur_KI-Kompass.pdf`](docs/Architektur_KI-Kompass.pdf) (v0.1; Abweichungen zu Gemini-Embeddings und OpenRouter sind in dieser README maßgeblich).

## Projektstand

| Baustein | Stand |
|---|---|
| WF1 Wissensbasis Ingest | ✅ fertig, getestet (2 Läufe je 156 Abschnitte, unabhängig nachgetestet) |
| WF2 Stufe 1: Chat mit Konzept-Suche | ✅ fertig, gemessen (Median 8,2 s, 20/20 Quellen korrekt) |
| WF2 Stufe 2: Guardrails | ✅ fertig, getestet (7/7 Guardrail-Fälle, 0 Fehlalarme) |
| WF2 Stufe 3: Erlass-Suche (Tavily) | ✅ fertig, getestet (5/5, Median 6,7 s) |
| WF2 Stufe 4: anonymes Protokoll | ✅ fertig, getestet (4/4, keine Fragetexte oder Namen) |
| WF3 Dashboard | ✅ fertig, getestet (8/9, Gegenprobe ohne Passwort offen); mit gekennzeichneten Demodaten |

## Startanleitung

So lässt sich das System aus diesem Repository nachbauen. Es werden **keine Schlüssel** mitgeliefert; alle Zugangsdaten liegen ausschließlich als Credentials in n8n.

### 1. Voraussetzungen

| Konto | Wofür | Hinweis |
|---|---|---|
| n8n Cloud (ab Starter) | Workflows ausführen | API-Zugang ist ab Starter verfügbar |
| Qdrant Cloud | Vektordatenbank | kostenloser Cluster, Region in der EU wählen |
| Google AI Studio | Gemini-API-Key für Embeddings | **Abrechnung aktivieren** (Stufe 1). Das kostenlose Kontingent erlaubt nur 100 Anfragen/Minute; das Konzept hat 156 Abschnitte, der Upload scheitert sonst mit leeren Vektoren. Ein kompletter Upload kostet unter 1 Cent. |
| OpenRouter | API-Key für die Sprachmodelle | eigenen Key mit Kostenlimit anlegen; ca. 10–20 Tsd. Tokens pro beantworteter Frage, ca. 1.400 pro abgelehnter |
| Tavily | API-Key für die Erlass-Suche | kostenloser Tarif reicht |

### 2. Credentials in n8n anlegen

In n8n unter **Overview → Credentials → Create Credential**. Die Namen exakt so wählen, dann werden sie beim Import automatisch zugeordnet:

| Name | Typ | Inhalt |
|---|---|---|
| `Upload - Admin` | Basic Auth | Benutzer `admin`, eigenes sicheres Passwort |
| `Qdrant KI-Kompass` | Qdrant API | API-Key und Cluster-URL **mit** `:6333`. Unter „Allowed HTTP Request Domains“ die **exakte** Cluster-Adresse eintragen (z. B. `xxxx.eu-central-1-0.aws.cloud.qdrant.io`), nicht nur `cloud.qdrant.io` – sonst lehnt n8n die HTTP-Aufrufe in WF1 ab. |
| `Gemini - Embeddings` | Google Gemini (PaLM) API | Gemini-API-Key |
| `OpenRouter - n8n` | OpenRouter | OpenRouter-API-Key |
| `Dashboard Schulleitung` | Basic Auth | Benutzer `schulleitung`, eigenes sicheres Passwort; Allowed HTTP Request Domains = None |
| `Tavily Suche` | Header Auth | Name `Authorization`, Wert `Bearer <Tavily-Key>`; erlaubte Domain `api.tavily.com` |

### 3. Workflows importieren

1. In n8n **Create Workflow → … → Import from File** und [`workflows/wf1_wissensbasis_ingest.json`](workflows/wf1_wissensbasis_ingest.json) wählen. Speichern.
2. Dasselbe mit [`workflows/wf2_ki_kompass_chat.json`](workflows/wf2_ki_kompass_chat.json).
3. In jedem Node mit Credential prüfen, ob das passende Credential ausgewählt ist (Nodes mit Warnsymbol neu auswählen).
4. **Wichtig:** In WF1 stehen in den Nodes **„Index sicherstellen“** und **„Alte Version löschen“** die URL des Projekt-Clusters. Den Teil vor `:6333` durch die eigene Cluster-Adresse ersetzen.

### 4. Wissensbasis befüllen (WF1)

1. WF1 öffnen und unten **„Execute workflow“** klicken. Das Upload-Formular ist jetzt im Testmodus erreichbar.
2. Die Test-URL öffnen (`https://<ihre-instanz>.app.n8n.cloud/form-test/ki-konzept-upload`) und mit `admin` und dem Passwort anmelden.
3. Das KI-Konzept als PDF hochladen, Version z. B. `v3.0` eintragen, absenden.
4. Ergebnis prüfen: alle Nodes grün; in Qdrant unter **Collections → `ki_konzept` → Info** steht `points_count` = Anzahl der Abschnitte (Projektstand: **156**). In einem Punkt stehen unter `metadata` die Felder `quelle`, `version` und `loc.pageNumber`.
5. Ein erneuter Upload ersetzt die alte Version vollständig (Punktezahl bleibt gleich). Die Collection und der benötigte Index auf `metadata.quelle` legt WF1 selbst an.

### 5. Chat testen (WF2)

1. WF2 öffnen und unten auf **„Open Chat“** klicken.
2. Für vergleichbare Messungen vor jeder Frage im Chat-Fenster auf **↺** (neue Sitzung) klicken.
3. Beispielfragen:
   - „Darf ich Schüler:innen erlauben, KI für Hausaufgaben zu nutzen?“
   - „Was mache ich, wenn Schüler:innen zentrale Inhalte im Fachgespräch nicht erklären können?“
   - „Welche Regeln gelten für die Notengebung bei KI-gestützten Projekten?“
   - „Was regelt die APO-SI zu Täuschungsversuchen mit KI?“ (nutzt die Erlass-Suche)
   - „Welche Note soll ich für dieses Referat geben?“ (wird von der Sicherheitsprüfung abgelehnt)
4. Erwartet: kurze Antwort (ca. 150 Wörter) mit Quellen wie „(KI-Konzept v3.0, S. 35)“. Unter **Logs** ist zu sehen, dass die **Konzept-Suche** aufgerufen wurde.

Die Workflows bleiben im Testmodus (nicht veröffentlicht). Für den Produktivbetrieb würde WF2 über **Publish** mit einem öffentlichen Chat-Link bereitgestellt.

### 6. Dashboard testen (WF3)

1. Credential `Dashboard Schulleitung` anlegen (siehe Schritt 2) und [`workflows/wf3_dashboard.json`](workflows/wf3_dashboard.json) importieren.
2. Im Editor unten auf **„Execute workflow“** klicken und **sofort** `https://<ihre-instanz>.app.n8n.cloud/webhook-test/ki-kompass-dashboard` im Browser öffnen (die Test-URL gilt nur für einen Aufruf).
3. Mit `schulleitung` und dem Passwort anmelden. Erwartet: Kennzahlen, Diagramme und Listen; nur zusammengefasste Werte, keine Einzelzeilen.

## Evidenz

| Nachweis | Datei |
|---|---|
| Messbare Erfolgskriterien (5 Kriterien mit Zielwerten) | [`docs/erfolgskriterien.md`](docs/erfolgskriterien.md) |
| Prompt-Vergleich v1/v2: Antwortzeit, Tokens, Quellentreue | [`tests/prompt_vergleich_wf2.md`](tests/prompt_vergleich_wf2.md) |
| Guardrail-Tests (12/12) | [`tests/ergebnisse_wf2_stufe2.md`](tests/ergebnisse_wf2_stufe2.md) |
| Erlass-Suche und Regression (5/5), Konzept-Unschärfen | [`tests/ergebnisse_wf2_stufe3.md`](tests/ergebnisse_wf2_stufe3.md) |
| Anonymes Protokoll (4/4), Restrisiko Systemzeiten | [`tests/ergebnisse_wf2_stufe4.md`](tests/ergebnisse_wf2_stufe4.md) |
| Dashboard (WF3) | [`tests/ergebnisse_wf3.md`](tests/ergebnisse_wf3.md) |
| Systemprompt (versioniert) | [`prompts/wf2_systemprompt.md`](prompts/wf2_systemprompt.md) |
| Individuelle Beiträge mit Gegenprüfung | [`docs/Beitragslog_KI-Kompass.xlsx`](docs/Beitragslog_KI-Kompass.xlsx) |
| Use Case und 4-Satz-Brief | [`docs/KI-Kompass_Use-Case_4-Satz-Brief.pdf`](docs/KI-Kompass_Use-Case_4-Satz-Brief.pdf) |

## Grenzen und bekannte Punkte

- **KI-Konzept nicht im Repository:** Das PDF enthält den echten Schulnamen und liegt daher nur lokal. Eine anonymisierte Fassung ist geplant.
- **Fehlerbehandlung WF1:** Index- oder Löschfehler stoppen den Workflow vor dem Speichern („Abbruch: …“). Toleriert wird nur „Collection existiert noch nicht“ beim allerersten Lauf.
- **Gedächtnis:** Innerhalb einer Chat-Sitzung wachsen die Tokens pro Frage (gemessen 15 → 36 → 58 Tsd.). Gedächtnisfenster wird noch angepasst.
- **Kosten:** Die Kosten pro Frage hängen vor allem an der Menge der gefundenen Abschnitte (Top-K = 5), nicht an der Antwortlänge.
- **Dashboard-Daten:** Die Tabelle enthält derzeit Demozeilen (`version = demo`), im Dashboard deutlich gekennzeichnet. Vor dem Echtbetrieb löschen und `MIN_ANZAHL` im Node „Kennzahlen berechnen“ auf 3 setzen.
- **Basic Auth in Tests:** n8n zeigt in Test-Ausführungen den Header `authorization` (nur Base64, rückrechenbar). Screenshots von Webhook-Tests nicht weitergeben.
- **Protokoll:** Die n8n Data Table legt automatisch sekundengenaue Spalten `createdAt`/`updatedAt` an. Die Rohtabelle ist daher nur für Admins zugänglich; das Dashboard zeigt nur gerundete, aggregierte Werte.
- **Node-Namen der Werkzeuge** (`Konzept_Suche`, `Erlass_Suche`) dürfen keine Bindestriche enthalten – n8n nutzt sie als Tool-Namen.
- **Speichern im n8n-Editor** kann Parameter entfernen, die im aktuellen Node-Schema nicht angezeigt werden (z. B. Top-K). Nach Änderungen im Editor den Export mit dem Live-Stand vergleichen.
- **Entwicklung:** Gebaut wurde mit Claude Code und dem Community-Server n8n-mcp. Für den Betrieb werden beide nicht benötigt.

## Projektstruktur

```
ki-kompass/
├── README.md                 diese Datei
├── CLAUDE.md                 Arbeitsregeln für Claude Code
├── workflows/                n8n-Exporte (ohne Schlüssel)
│   ├── wf1_wissensbasis_ingest.json
│   └── wf2_ki_kompass_chat.json
├── prompts/                  versionierte Systemprompts
├── tests/                    Messungen und Testfälle
└── docs/                     Architektur, Use Case, Erfolgskriterien, Beitragslog
```
