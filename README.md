<div align="center">

# 🐟 Chipilito

Ein schlanker, selbstgehosteter KI-Chatbot für Node.js – läuft wahlweise
komplett lokal mit [Ollama](https://ollama.com), mit Google Gemini oder mit
jedem OpenAI-kompatiblen Anbieter (z. B. [OpenRouter](https://openrouter.ai),
LM Studio, Groq). Kein Framework-Overhead, keine externe Datenbank – nur
Node.js, SQLite und ein einziger Server-Prozess.

<p>
  <img alt="Node" src="https://img.shields.io/badge/node-%3E%3D18-339933?logo=node.js&logoColor=white">
  <img alt="License" src="https://img.shields.io/badge/license-MIT-blue">
  <img alt="Version" src="https://img.shields.io/badge/version-2.0.0-d97757">
</p>
</div>
---

## ✨ Features

- 💬 **Mehrere KI-Anbieter** – Ollama (lokal), Google Gemini oder ein
  beliebiger OpenAI-kompatibler Endpoint, jederzeit im Modell-Umschalter
  wechselbar. Modelle lassen sich mit ⭐ favorisieren (kontogebunden
  gespeichert).
- 📌 **Chat-Verwaltung** – Anheften, Umbenennen, Löschen und Gruppieren von
  Chats in **Projekte** über ein ⋮-Kontextmenü, ähnlich wie man es aus
  modernen Chat-Oberflächen kennt.
- 📁 **Projekte** – Chats lassen sich Projekten zuordnen (und wieder
  entfernen), Projekte selbst können umbenannt oder gelöscht werden.
- 📎 **Datei-Uploads** – per Klick oder **Drag & Drop** direkt in den
  Chat-Bereich. Unterstützt PDF, Word (.docx), Excel, Bilder und mehr.
- 📄 **Export** – Antworten lassen sich als PDF, Word oder Excel exportieren.
- 📦 **ZIP-Download für Mehrdatei-Antworten** – enthält eine Antwort mehrere
  Code-Blöcke (z. B. bei einer Projektkorrektur), bietet die App automatisch
  einen "Als ZIP herunterladen"-Button an, inkl. automatischer
  Dateinamen-Erkennung.
- 🌍 **Mehrsprachig** – Deutsch, Englisch, Französisch, Spanisch.
- 🔐 **Eigene Konten** – einfache Benutzerverwaltung (Name + Passwort) mit
  kontoübergreifender Chat-Synchronisierung über SQLite.
- 🎤 **Spracheingabe** über die Web Speech API des Browsers.
- 🐳 **Docker-ready** – inkl. `docker-compose.yml` mit optionalem
  Ollama-Container.

---

## 📸 Screenshot

*(Füge hier gerne einen Screenshot deiner Instanz ein, z. B.
`![Chipilito](docs/screenshot.png)`)*

---

## 🚀 Schnellstart

### Voraussetzungen

- [Node.js](https://nodejs.org) ≥ 18
- Optional: [Ollama](https://ollama.com) für lokale Modelle, oder ein API-Key
  für Gemini/OpenRouter

### Installation

```bash
git clone <dein-repo-url>
cd chipilito
npm install
cp .env.example .env   # Werte nach Bedarf anpassen (siehe unten)
npm start
```

Die App läuft danach unter **http://localhost:3000**.

### Entwicklung (Auto-Reload)

```bash
npm run dev
```

---

## ⚙️ Konfiguration (`.env`)

| Variable | Beschreibung | Default |
|---|---|---|
| `CHATLITE_PROVIDER` | `ollama`, `gemini` oder `openai_compatible` | `ollama` |
| `OLLAMA_URL` | URL des Ollama-Servers | `http://localhost:11434` |
| `OLLAMA_MODEL` | Standard-Ollama-Modell | `gemma2` |
| `OLLAMA_NUM_CTX` | Kontextfenster-Größe für Ollama | `21586` |
| `GEMINI_API_KEY` | API-Key für Google Gemini | – |
| `OPENAI_COMPATIBLE_URL` | Basis-URL des OpenAI-kompatiblen Anbieters | – |
| `OPENAI_COMPATIBLE_API_KEY` | API-Key dafür | – |
| `OPENAI_COMPATIBLE_MODEL` | Standardmodell dafür | – |
| `PORT` | Server-Port | `3000` |
| `DB_PATH` | Pfad zur SQLite-Datei (relativ wird relativ zum Projektordner aufgelöst) | `./chipilito.db` |
| `MAX_FILE_SIZE` | Maximale Upload-Größe in Bytes | `5242880` (5 MB) |

Eine vollständige Vorlage liegt in [`.env.example`](.env.example).

> **Modellwahl im laufenden Betrieb:** Der `CHATLITE_PROVIDER` in der `.env`
> setzt nur den *Standard*. Im Modell-Umschalter der App kann jederzeit
> zwischen allen konfigurierten Anbietern und Modellen gewechselt werden
> (inkl. Live-Abfrage aller verfügbaren Modelle beim OpenAI-kompatiblen
> Anbieter, z. B. der komplette OpenRouter-Katalog).

---

## 🐳 Docker

```bash
docker compose up -d
```

Startet Chipilito **und** einen lokalen Ollama-Container. Daten (SQLite-DB,
Ollama-Modelle) liegen in benannten Docker-Volumes und überleben Neustarts.
Für NVIDIA-GPU-Beschleunigung von Ollama die entsprechenden Zeilen in
`docker-compose.yml` einkommentieren.

Nur die App ohne Compose bauen:

```bash
docker build -t chipilito .
docker run -p 3000:3000 -v chipilito_data:/app/data chipilito
```

---

## 🗂️ Projektstruktur

```
chipilito/
├── server.js              # Dünner Einstiegspunkt: erstellt den HTTP-Server,
│                           # verdrahtet die Routen-Module
├── server/
│   ├── config.js           # Zentrale Konfiguration (liest .env)
│   ├── http-utils.js        # readJsonBody, sendJson, serveStatic, ...
│   ├── router.js            # Schlanker Routen-Matcher (kein Express)
│   ├── file-extraction.js   # PDF/Word/Excel/ZIP-Textextraktion für Uploads
│   ├── providers/
│   │   ├── ollama.js
│   │   ├── gemini.js
│   │   └── openaiCompatible.js
│   └── routes/
│       ├── auth.js, models.js, chats.js, projects.js
│       ├── favorites.js, chat-completion.js, export.js
├── db.js                   # SQLite-Zugriff (Nutzer, Chats, Projekte, Favoriten)
├── export.js                # PDF/Word/Excel-Dokumentengenerierung
├── public/
│   ├── index.html
│   ├── app.js               # Gesamte Frontend-Logik (kein Build-Schritt nötig)
│   └── styles.css
├── .env.example
├── Dockerfile
├── docker-compose.yml
└── CHANGELOG.md
```

Kein Bundler, kein Build-Schritt fürs Frontend – `public/` wird direkt vom
Server ausgeliefert. Das Backend ist in kleine, fokussierte Module aufgeteilt
(siehe oben) statt einer einzigen großen `server.js`.

---

## 🔌 API-Übersicht

Alle Endpoints erwarten (außer Auth) den Header `x-api-key` mit dem beim
Login erhaltenen Schlüssel.

| Methode | Pfad | Zweck |
|---|---|---|
| `POST` | `/api/auth/register` / `/api/auth/login` | Konto anlegen / einloggen |
| `GET`/`PUT` | `/api/auth/profile` | Profil lesen / Namen ändern |
| `GET` | `/api/models` | Verfügbare Modelle je Anbieter |
| `POST` | `/api/chat` | Nachricht senden (Streaming-Antwort) |
| `GET`/`PUT`/`DELETE` | `/api/chats/:id` | Chat laden/speichern/löschen |
| `PUT` | `/api/chats/:id/pin` \| `/rename` | Chat anheften / umbenennen |
| `POST` | `/api/chats/:id/addToProject` | Projekt zuordnen (`project_id: null` entfernt) |
| `GET`/`POST` | `/api/yumProjects` | Projekte auflisten / anlegen |
| `PUT`/`DELETE` | `/api/yumProjects/:id` | Projekt umbenennen / löschen |
| `GET`/`PUT` | `/api/favorites` | Favorisierte Modelle laden / speichern |

---

## 🛠️ Tech-Stack

- **Backend:** Node.js (ESM), reines `http`-Modul (kein Express im
  Request-Pfad), [better-sqlite3](https://github.com/WiseLibs/better-sqlite3)
- **Frontend:** Vanilla JS, kein Framework, [highlight.js](https://highlightjs.org)
  für Syntax-Highlighting, [JSZip](https://stuk.github.io/jszip/) für
  ZIP-Downloads
- **Export:** [pdfkit](https://pdfkit.org), [docx](https://docx.js.org),
  [exceljs](https://github.com/exceljs/exceljs)
- **Datei-Parsing:** [pdf-parse](https://www.npmjs.com/package/pdf-parse),
  [mammoth](https://github.com/mwilliamson/mammoth.js) (Word),
  [adm-zip](https://github.com/cthackers/adm-zip)

---

## 📄 Lizenz

MIT – siehe [`LICENSE`](LICENSE) (bzw. nach Bedarf ergänzen).

## 📝 Changelog

Alle Änderungen sind in [`CHANGELOG.md`](CHANGELOG.md) dokumentiert.
## 👥 Mitwirken & Support

Beiträge, Fehlerberichte und Feature-Anfragen sind herzlich willkommen! Erstelle dazu einfach ein Issue oder einen Pull Request.

*Copyright (C) Noel Joan - 2026. Alle Rechte vorbehalten.*
