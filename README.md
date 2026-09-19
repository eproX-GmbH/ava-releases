# AVA

[![Latest Release](https://img.shields.io/github/v/release/eproX-GmbH/ava-releases?include_prereleases&label=release&color=00c0a7)](https://github.com/eproX-GmbH/ava-releases/releases/latest)
[![Service Health](https://img.shields.io/website?url=https%3A%2F%2Fava-db-gateway.fly.dev%2Fhealth&label=db-gateway&up_message=operational&down_message=offline)](https://ava-db-gateway.fly.dev/health)
[![Master-Data](https://img.shields.io/website?url=https%3A%2F%2Fava-master-data.fly.dev%2Fhealth&label=master-data&up_message=operational&down_message=offline)](https://ava-master-data.fly.dev/health)

> Sales Intelligence als Desktop-App: Firmen aus Deutschland, Österreich und dem Vereinigten Königreich recherchieren, beobachten und ins eigene CRM bringen. KI läuft auf dem Rechner des Nutzers oder mit eigenem Schlüssel.

AVA verdichtet öffentliche Unternehmensdaten zu einem vollständigen Firmenbild: amtliches Register, Jahresabschlüsse und Bekanntmachungen, Firmenwebsite, Ansprechpartner mit Herkunftsnachweis und eine KI-Bewertung gegen das eigene Idealkundenprofil. Danach beobachtet AVA die Firmen weiter und meldet, was sich ändert: Insolvenzen, Löschungen, Geschäftsführerwechsel, neue Jahresabschlüsse, Website-Änderungen, LinkedIn-Signale.

Bedient wird AVA über einen Chat-Agenten mit rund 250 Werkzeugen, über die Firmenansichten in der App, über Telegram unterwegs und über gespeicherte Workflows. Alles Rechenintensive (Browser-Automatisierung, Extraktion, KI-Aufrufe) läuft **lokal auf der Maschine des Nutzers**. Die Cloud ist Substrat: Anmeldung, Stammdaten, Verarbeitungsstand, geteilte Korpora.

**In diesem Repository liegen ausschließlich die fertigen Installationsdateien.** Der Quellcode ist nicht öffentlich.

## Inhalt

- [Download](#download)
- [Was AVA heute kann](#was-ava-heute-kann)
- [Architektur](#architektur)
- [Auslieferung](#auslieferung)
- [Sicherheit und Datenschutz](#sicherheit-und-datenschutz)
- [Lizenz](#lizenz)

## Download

Am einfachsten über [ava.bi](https://www.ava.bi) — dort führt der Knopf direkt zur passenden Datei für dein System. Alternativ unter [Releases](https://github.com/eproX-GmbH/ava-releases/releases) die neueste Fassung wählen:

- **Mac mit Apple Silicon:** die Datei mit `arm64` im Namen, Endung `.dmg`
- **Mac mit Intel:** die Datei mit `x64` im Namen, Endung `.dmg`
- **Windows:** `AVA-Setup-<Version>.exe`

Nach der Installation mit Konto anmelden oder registrieren; beim ersten Start führt ein Assistent durch die Wahl der KI (lokales Modell oder eigener Schlüssel).

Die App prüft selbst auf neue Fassungen und meldet sich, wenn eine bereitsteht. Jeder Download und jeder Neustart wird vorher bestätigt; stille Aktualisierungen gibt es nicht.

## Was AVA heute kann

### Firmen aufnehmen und recherchieren

- **Import** per Excel/CSV, einzeln per Name und Ort, aus HubSpot oder über den Firmen-Radar. Jede Firma wird öffentlichen Quellen zugeordnet (Fuzzy-Suche über den Stammdaten-Index).
- **Sechs Producer** je Firma, die sich gegenseitig anstoßen. Status pro Firma und Stufe live als Matrix, mit Pause, Löschen, Wiederholen je Stufe, Logs und Screenshots je Lauf.

| Producer | Quelle | Ergebnis |
|---|---|---|
| `structured-content` | Handelsregister (Registerportal), Firmenbuch (AT) und Companies House (UK) über master-data | Stammdaten, Sitz, Geschäftsführung bzw. Officers; für deutsche HRB-Firmen zusätzlich die Gesellschafterliste (siehe Verflechtungen) |
| `company-publication` | Unternehmensregister / Bundesanzeiger | Jahresabschlüsse, Lagebericht, Bekanntmachungen; Kennzahlen per Lazy-RAG auf Abruf statt Voranalyse |
| `website` | Websuche + Firmenwebsite | Beste Treffer-Website, Inhalte, Stellenanzeigen, eingesetzte Systeme aus der Datenschutzerklärung |
| `company-profile` | Website | Firmenprofil (Angebot, Branche, Größe, Standorte) |
| `company-contact` | Website, Impressum | Ansprechpartner und Kontaktwege mit Beleg, Herkunftsnachweis und Art.-14-Hinweis |
| `company-evaluation` | alles oben | KI-Bewertung gegen das Idealkundenprofil, Best-Match-Ranking, Angebotsvergleich |

- **Länder:** Deutschland (Handelsregister, Insolvenzbekanntmachungen), Österreich (Firmenbuch über JustizOnline, Ediktsdatei) und Vereinigtes Königreich (Companies House, Gazette). Länderfilter in Firmensuche und „Meine Firmen“, Landeschip und amtliche Kennung je Firma. Schweiz ist bewertet, aber nicht umgesetzt.
- **Stammdaten aktuell halten (Register-Delta):** Neueintragungen, Änderungen und Löschungen kommen täglich aus den Registern. Die Arbeit teilen sich Betreiber-Worker in der Cloud und Nutzer, die „Stammdaten mitpflegen“ einschalten (Opt-in, höchstens 60 Abfragen je Stunde). Geänderte Registerblätter markieren die betroffene Firma als veraltet.
- **Insolvenzen und Firmenstatus:** gezielte Prüfung je Pool-Firma alle 30 Tage; Insolvenz, Löschung, Löschungsankündigung und Liquidation erscheinen als Chip in den Tabellen, als Warnung in jedem Chat-Werkzeug und als Meldung des Firmenstatus-Wächters.
- **Firmen-Verflechtungen (Deutschland):** Gesellschafterlisten werden über den Registerordner geladen, mit dem KI-Modell des Nutzers gelesen (zwei unabhängige Lesungen, harter Qualitätsfilter, Bild-Modell ab Stufe A Pflicht) und zu Beteiligungen, Personen und gemeinsamen Adressen verdichtet. Firmen-Gesellschafter werden rekursiv nachgezogen (Besuchsliste, Notbremse Tiefe 6 / 200 Firmen, abschaltbar). Reiter „Verflechtungen“ mit Netzgrafik, Gesellschaftertabelle und Personenseite; Meldung „Gesellschafterwechsel“. Als Org-Feature abschaltbar. Stand: umgesetzt, Ende-zu-Ende-Erprobung läuft.
- **Neue Firmen finden (Firmen-Radar):** Scan in einer Region aus öffentlichen Firmeneinträgen, KI-geplanter Web-Recherche und unverarbeitetem Registerbestand; Mini-Profile, Score gegen das Idealkundenprofil, Import erst nach Entscheidung des Nutzers. Automatik täglich oder wöchentlich als Opt-in.
- **Idealkundenprofil (ICP):** aus der eigenen Website und bis zu fünf Kunden-Websites abgeleitet oder als Fragebogen; bleibt lokal.

### Beobachten und melden

- **Heartbeat und Meldungen:** neue Veröffentlichungen, Profiländerungen, Bewertungs-Auffälligkeiten, Firmenstatus, Gesellschafterwechsel; Glocke in der App, OS-Benachrichtigung, Telegram. Ruhezeiten, dringende Meldungen umgehen sie.
- **Beobachtungsregeln (Watches)** mit eigener Bewertungsvorschrift in Nutzersprache.
- **Website-Überwachung** beliebiger URLs mit KI-Zusammenfassung der Änderung und Beweis-Screenshot; Bot-Schutz wird abgewartet, nie umgangen.
- **LinkedIn (Opt-in, eigenes Konto):** Feed-Beobachter für Signale zu Firmen im Bestand, Personen-Watchlist, Personen-Radar aus Beitrags-Engagement. Bildanalyse per Vision-Modell.
- **Safe Browsing** vor Website-Abrufen.

### Kontakte und Kommunikation

- **Ansprechpartner** mit Beleg, Herkunft und Löschfunktion; Datenschutzhinweise (Art. 14) als Text exportierbar.
- **E-Mail-Muster:** Adressen nach dem Muster der Firma ableiten und gegen den Mail-Server prüfen (Hintergrund mit Vorrang für Chat, Last und Akku; verifizierte Adressen werden nie erneut geprüft, Bounces und Antworten fließen zurück).
- **Mail-Postfach** anbinden (IMAP): lesen, antworten, weiterleiten, archivieren, Triage mit Kontext je Firma.
- **Telegram:** Meldungen, Kurzprofile und Freigaben aufs Handy, vollwertiger Chat mit Sprachnachrichten und Bildern.

### Arbeiten mit AVA

- **Chat-Agent** mit rund 250 Werkzeugen in 36 Gruppen, Tool-Suche, Gedächtnis über Gespräche, Nutzerprofil, Vorschlags-Chips für die nächsten Schritte, Sprachmodus (lokale Whisper-Transkription).
- **Workflows:** Abläufe aus dem Gespräch speichern, als Diagramm kontrollieren, per Zeitplan oder Ereignis ausführen (neuer Radar-Treffer, eingehende Mail, Import fertig). Schreibende Schritte nur nach Freigabe; Freigaben in App, Meldungen oder Telegram; Laufhistorie und Audit.
- **Skills:** wiederverwendbare Routinen per Slash-Befehl, mit Trust-Modell.
- **Integrationen:** HubSpot (lesen, anlegen, verknüpfen), Notion, Obsidian. Teilen von Recherchen, Radar-Firmen und Workflows mit der Organisation.
- **KI-Modelle:** lokal (Ollama, kuratierte Modelle mit Hardware-Prüfung) oder mit eigenem Schlüssel bei OpenAI, Anthropic, Google, Mistral, DeepSeek, xAI, Qwen; ChatGPT-Abo und Anthropic-Abo per OAuth nutzbar. Jedes Modell hat eine Qualitätsstufe (S/A/B/C), die entscheidet, ob ein Ergebnis bestehende Daten überschreiben darf und welche Funktionen es freischaltet. Getrenntes, günstigeres Modell für die Hintergrundverarbeitung, Token-Limit je Tag, Verbrauchsübersicht.

### Organisationen

- Mandanten mit Mitgliedern, Beitrittsanfragen, Vorgaben je Organisation: Anbieter-Sperre, vorgegebene Modelle, Organisationsschlüssel über den Gateway-Proxy, Limits, abschaltbare Module (LinkedIn, Bildanalyse, Kontakt-Recherche, Mail, Telegram, Workflows, Vorschläge, Stammdaten mitpflegen, Verflechtungen). Abgeschaltetes verschwindet vollständig aus der App.
- Seat-Abrechnung je Belegungsmonat (Stripe), Kontingente je Plan mit Vorprüfung vor jedem Import, Verbrauch je Mitglied.
- Mehrere Konten auf einem Gerät (Account-Spaces), Audit-Protokoll für sicherheits- und kostenrelevante Aktionen.

## Architektur

```
┌────────────────────────────────────────────────┐   ┌────────────────────────────────┐
│ Desktop-App (macOS arm64/x64, Windows x64)     │   │ Cloud-Substrat (Fly.io, EU)    │
│                                                │   │                                │
│  Chat-Agent · Firmen · Radar · Workflows       │   │ db-gateway                     │
│  Meldungen · Mail · Telegram · Organisation    │   │  Auth, Policy                  │
│                                                │   │  Persist-Bus mit Tier-Gate     │
│  6 Producer-Subprozesse (lokal)                │◄──┤  Register-Delta-Queue, Cron    │
│  Register-Delta-Worker (Opt-in)                │AMQP  Verflechtungen-Kontexte      │
│  LLM lokal (Ollama) oder eigener Schlüssel     │   │  Operator-Proxies (Suche, CRM) │
│  Whisper-Sidecar für Sprache                   │   │  Abrechnung (Stripe)           │
│  Hintergrund-Browser, gehärtet                 │   │                                │
└────────────────────────────────────────────────┘   │ master-data                    │
                                                     │  Stammdaten DE/AT/UK, Suche    │
                                                     │  Personen, Beteiligungen,      │
                                                     │  Adressen, Insolvenz-Ereignisse│
                                                     │                                │
                                                     │ Register-Worker des Betreibers │
                                                     │  (nur öffentliche Register)    │
                                                     └────────────────────────────────┘
```

**Compute-Lokalität ist Invariante**: jeder LLM-Aufruf und jeder Web-Abruf für die Recherche läuft auf der Maschine des Nutzers. Cloud-seitig läuft Substrat, die Register-Delta-Worker des Betreibers (nur öffentliche Register, kein LLM) und die wenigen Dienste, die einen Betreiber-Schlüssel brauchen (Websuche, CRM-OAuth-Austausch, optionaler Organisationsschlüssel-Proxy).

**Persist-Bus mit Tier-Gate:** Producer schreiben nicht direkt in die Datenbank, sondern schicken Ereignisse an den Gateway. Der prüft Mandant, Modul-Freigabe der Organisation und die Qualitätsstufe des Modells und verwirft Rückschritte („einer verarbeitet, alle profitieren“, aber nie mit schlechteren Daten).

## Auslieferung

Der Desktop-Release entsteht aus einem Tag `v0.1.X` über GitHub Actions: macOS arm64 und x64 (signiert, notarisiert), Windows x64 (Azure Artifact Signing), OTA-Updates über den integrierten Updater. Die Dateien in diesem Repository stammen ausschließlich aus diesen Läufen.

## Sicherheit und Datenschutz

- Schlüssel und Tokens liegen im Schlüsselbund des Betriebssystems; eigene KI-Schlüssel überschreiten nie die Grenze zur Oberfläche und werden nicht über den Chat gesetzt.
- Hintergrund-Browser sind gehärtet: keine Downloads außer den amtlichen Registerdateien (XML, Gesellschafterlisten als PDF/TIFF, geprüft an den Magic Bytes, nach dem Lesen gelöscht), keine Web-Berechtigungen, Producer nur auf Loopback.
- Personendaten: Ansprechpartner mit Herkunftsnachweis, Art.-14-Hinweis und Löschfunktion; Aufbewahrungsfristen per Cron; Geburtsdaten aus Gesellschafterlisten nur als Jahr nach außen, intern mit Hash für eine spätere Entfernung.
- Alle Ausgaben von Modellen und externen Quellen werden vor der Übernahme gegen Schemata geprüft (Yup). „Lieber keine Daten als falsche Daten“ ist Regel, nicht Ausnahme.

## Lizenz

AVA steht unter der AVA Source-Available License der eproX GmbH. Das ist **keine Open-Source-Lizenz** im Sinne der OSI. Der Lizenztext liegt beim Quellcode; rechtlich maßgeblich ist allein er. Kurz zusammengefasst:

- **Erlaubt:** Quellcode lesen, lokal bauen und ausführen, ändern, unentgeltlich forken; Server-Komponenten lokal zum Entwickeln und Testen betreiben; Nutzung im Rahmen des Free-Plans oder eines Bezahlplans laut [AGB](https://www.ava.bi/agb).
- **Nicht erlaubt:** die App über den Free-Plan hinaus selbst betreiben (eigene Gateway-/Worker-Instanz produktiv, fremdes Backend, Umgehen von Plan-/Kontingent-Prüfungen); geschäftliche Nutzung über den Free-Plan hinaus ohne Bezahlplan; kommerzielle Weiterverbreitung (Verkauf, Hosting für Dritte, Einbau in kostenpflichtige Produkte).

Drittkomponenten behalten ihre jeweiligen Lizenzen. Fragen zu Bezahlplänen und kommerzieller Lizenzierung: [info@eprox-gmbh.de](mailto:info@eprox-gmbh.de).

---

_Fragen, Feedback, Bugs:_ [info@eprox-gmbh.de](mailto:info@eprox-gmbh.de)

---

_Fragen, Feedback, Bugs:_ [info@eprox-gmbh.de](mailto:info@eprox-gmbh.de)
