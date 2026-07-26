# CHANGELOG - Chipilito Chatbot

## [2026-07-25] - Backend in Module aufgeteilt (v2.1-Vorbereitung)

`server.js` (zuletzt ~960 Zeilen) war eine einzige lange Datei mit
Routing-if-Kette, KI-Provider-Logik und Datei-Extraktion vermischt. Jetzt
aufgeteilt in:

```
server.js               # dünner Einstiegspunkt
server/config.js          # zentrale Konfiguration (.env)
server/http-utils.js       # readJsonBody, sendJson, serveStatic, ...
server/router.js           # schlanker Routen-Matcher
server/file-extraction.js  # PDF/Word/Excel/ZIP-Textextraktion
server/providers/          # ollama.js, gemini.js, openaiCompatible.js
server/routes/             # auth.js, models.js, chats.js, projects.js,
                            # favorites.js, chat-completion.js, export.js
```

**Verhalten wurde 1:1 übernommen** (keine beabsichtigten Änderungen an
Request/Response-Formaten) - mit zwei Ausnahmen, die dabei als echte,
vorher unentdeckte Bugs aufgefallen sind und mitbehoben wurden:

- **`POST /api/export` hatte gar keine Route.** Die Export-Funktionen
  (`generatePDF`/`generateDocx`/`generateXlsx`) waren in der alten `server.js`
  zwar importiert, aber nie an eine Route gehängt - der Export-Button in der
  Oberfläche konnte dadurch nie funktioniert haben. Jetzt in
  `server/routes/export.js` implementiert.
- Tote/unbenutzte Funktionen (`isFusion360Request`, `FUSION_360_GUIDANCE`,
  ungenutzter `db`-Import) beim Umzug entfernt.

Der neue Code wurde nicht nur per `node --check` (Syntax), sondern mit
Mock-Abhängigkeiten tatsächlich **gestartet und die Routen per `curl`
durchgetestet** (Auth-Fehler, Chats, Favoriten, Projekte, Export,
unbekannte Route → 405, fehlende Datei → 404) - dabei wurde ein
zusätzlicher Bug in der `__dirname`-Berechnung gefunden und behoben
(zeigte auf den falschen Ordner für `public/`).

---

## [2026-07-24] - Bessere Dateinamen-Erkennung + Gemini-Bug behoben

**Deine eigentliche Frage** (generische Namen wie `code-1.js` statt echter
Dateinamen in der ZIP): zwei Verbesserungen, beide nicht kompliziert.

1. **Das Modell wird jetzt gebeten, Dateinamen zu nennen.** Neuer
   System-Prompt-Hinweis (`FILE_NAMING_HINT` in `server.js`, für alle drei
   Anbieter Ollama/Gemini/OpenAI-kompatibel), der das Modell bittet, vor
   jedem Code-Block den Dateinamen als eigene **fett** formatierte Zeile zu
   schreiben (`**app.py**`). Das ist der zuverlässigste Hebel, da die
   Erkennung im Frontend nur greifen kann, wenn der Name überhaupt im Text
   auftaucht - eine Garantie gibt es trotzdem nicht (hängt vom jeweiligen
   Modell ab, wie gut es Systemanweisungen befolgt).
2. **Erkennung im Frontend robuster gemacht**: `extractCodeBlocks()` schaut
   jetzt bis zu 3 Zeilen zurück (statt nur 1) und überspringt dabei
   Leerzeilen, außerdem werden jetzt auch Formulierungen wie "Datei: app.py"
   oder "File: app.py" erkannt, nicht nur "**app.py**"/"`app.py`".

**Nebenbei gefunden:** Bei der Gemini-Anbindung war die API-URL komplett
kaputt (`https://googleapis.com{GEMINI_KEY}` - fehlendes `$`, falscher Pfad,
kein Modellname) - Gemini konnte dadurch nie funktioniert haben. Behoben:
korrekte URL (`generativelanguage.googleapis.com`, mit `alt=sse` für
sauberes Streaming), konfigurierbares `GEMINI_MODEL` (Default
`gemini-2.0-flash`) und richtige SSE-Zeilen-Verarbeitung.

---

## [2026-07-24] - Konto-Menü (Einstellungen/Hilfe/Abmelden)

Das bereits vorhandene Klick-Dropdown beim Benutzernamen unten links enthielt
bisher nur "Abmelden". Ergänzt um zwei weitere Einträge, wie im
Referenz-Screenshot:

- **Einstellungen** – als deaktivierter Menüpunkt ("Bald verfügbar"), Platz
  für eine spätere Einstellungsseite.
- **Hilfe** – öffnet das GitHub-Repo (`https://github.com/noeljoan/chipilito`)
  in einem neuen Tab.
- **Abmelden** – unverändert vorhandene Funktion, jetzt optisch mit den
  anderen beiden Einträgen vereinheitlicht (Icon + Trennlinie).

---

## [2026-07-24] - Letzte "Yum-Projekt"-Reste im Wortlaut entfernt

Zwei Fehlermeldungen in `db.js` sagten noch "Yum-Projekt nicht gefunden" -
jetzt "Projekt nicht gefunden". Genau diese Meldung dürfte auch den Fehler
beim "Aus Projekt entfernen" ausgelöst haben, falls dabei noch die alte
`db.js` (vor der letzten Änderung) lief - die neue Version behandelt
`project_id: null` (Entfernen) als eigenen Fall, bevor überhaupt nach einem
Projekt gesucht wird, und sollte den Fehler dann nicht mehr auslösen.

---

## [2026-07-24] - Chats wieder aus einem Projekt entfernen

Chats, die einem Projekt zugeordnet sind, zeigen im ⋮-Menü jetzt **"Aus
Projekt entfernen"** statt "Zum Projekt hinzufügen" - landen danach wieder
unter "Aktuelle", ohne gelöscht zu werden.

- `addChatToProject()` in `db.js` akzeptiert jetzt auch `projectId = null`
  und hebt dann einfach die Zuordnung auf (kein neuer Endpoint nötig, gleiche
  Route `POST /api/chats/:id/addToProject` mit `{ project_id: null }`).
- Menüpunkt im Frontend wechselt automatisch je nachdem, ob der Chat aktuell
  einem Projekt zugeordnet ist oder nicht.

---

## [2026-07-24] - Projekte umbenennen/löschen

Die Projekt-Überschriften in der Sidebar (unter "Projekte") haben jetzt beim
Hovern ein eigenes kleines ⋮-Menü mit **Umbenennen** und **Löschen** - analog
zum bestehenden Menü bei einzelnen Chats.

- **Backend**: neue Funktionen `renameYumProject()`/`deleteYumProject()` in
  `db.js` sowie `PUT`/`DELETE /api/yumProjects/:id` in `server.js`. Die
  Projekt-Liste ist wie bisher kontoübergreifend gemeinsam (kein
  Besitzer-Konzept je Projekt) - jeder eingeloggte Nutzer kann jedes Projekt
  umbenennen/löschen.
- **Löschen ist nicht destruktiv für Chats**: Wird ein Projekt gelöscht,
  bleiben die zugeordneten Chats erhalten - nur die Projekt-Zuordnung wird
  aufgehoben (sie tauchen danach unter "Aktuelle" auf), das Projekt selbst
  verschwindet aus der Liste.

---

## [2026-07-24] - Drag & Drop für Datei-Anhänge

Dateien lassen sich jetzt per Drag & Drop irgendwo im Chat-Bereich fallen
lassen (nicht nur über den 📎-Button) - funktioniert genauso wie eine über
den Datei-Dialog ausgewählte Datei (gleiche Größenprüfung, gleiche
Bild-Vorschau). Während des Ziehens erscheint ein Overlay ("Datei hier
ablegen") als visuelles Feedback.

- `handleFileSelect()` wurde in eine gemeinsame `processSelectedFile()`
  aufgeteilt, die jetzt sowohl vom `<input type="file">` als auch vom
  `drop`-Event genutzt wird.
- Neues Overlay-Element (`#dropOverlay`) in `index.html` + CSS dafür.
- Unterstützt weiterhin nur eine Datei gleichzeitig (wie der 📎-Button auch) -
  bei mehreren fallen gelassenen Dateien wird die erste genommen.

---

## [2026-07-24] - ZIP-Download als echte Datei-Karte (wie im Referenz-Screenshot)

Statt eines kleinen Buttons in der Aktionsleiste zeigt eine Antwort mit
mehreren Code-Blöcken jetzt eine richtige Karte unter der Nachricht: Icon +
Dateiname (`<Chat-Titel>.zip`) + Typ "ZIP" + "Herunterladen"-Button - optisch
angelehnt an die Datei-Karten, wie man sie aus Claude.ai kennt. Funktional
unverändert (weiterhin clientseitiges Bündeln der Code-Blöcke per JSZip),
nur die Darstellung wurde professioneller gemacht.

---

## [2026-07-24] - Datenbank-Pfad-Bug (echte Ursache für "Favoriten verschwinden")

Gefunden: `DB_PATH` (in `db.js`) und der `public/`-Pfad (in `server.js`)
wurden relativ zu `process.cwd()` aufgelöst - also relativ zu dem Ordner, aus
dem heraus `npm start` gerade aufgerufen wird. Wurde die App mal aus einem
anderen Verzeichnis gestartet (andere Verknüpfung, IDE, Terminal-Tab, ...),
hat Node eine **andere/neue** `chipilito.db` gelesen bzw. angelegt - alle
Daten (Favoriten, aber potenziell auch Chats/Projekte) sahen dann aus, als
wären sie beim Neustart verschwunden, obwohl die alte Datei einfach nur an
einem anderen Ort lag.

- Beide Pfade werden jetzt relativ zur jeweiligen Datei selbst aufgelöst
  (`import.meta.url`), unabhängig vom Arbeitsverzeichnis beim Start.
- Beim Start wird jetzt geloggt, welche DB-Datei verwendet wird
  (`[Chipilito] Verwende Datenbank: ...`) - zur Kontrolle.
- Ein per `.env`/`DB_PATH` gesetzter **relativer** Pfad wird ebenfalls relativ
  zur Datei aufgelöst, nicht mehr relativ zu `process.cwd()`. Absolute Pfade
  in `DB_PATH` funktionieren weiterhin unverändert.

---

## [2026-07-24] - Download mehrerer Dateien als ZIP (z. B. Projektkorrektur)

Antworten mit **mehreren Code-Blöcken** (z. B. wenn die KI mehrere Dateien
eines Projekts korrigiert) bekommen jetzt einen zusätzlichen Button
**"Als ZIP"** unter der Nachricht, der alle Code-Blöcke dieser Antwort
gebündelt als echte `.zip`-Datei herunterlädt.

- Dateinamen werden nach Möglichkeit automatisch erkannt: entweder direkt in
  der Code-Zaun-Zeile (` ```python app.py `) oder in der Zeile unmittelbar
  davor (`**app.py**`, `` `app.py` ``, `### app.py`, `app.py:`). Ohne
  erkennbaren Namen wird ein generischer Name nach Sprache vergeben
  (`code-1.py`, `code-2.js`, ...).
- Läuft komplett im Browser (per JSZip, neu via CDN eingebunden), kein
  Server-Roundtrip nötig - der Button erscheint automatisch, sobald eine
  Antwort (auch während/nach dem Streamen) 2 oder mehr Code-Blöcke enthält.
- Der einzelne Download-Button pro Code-Block (aus dem letzten Update) bleibt
  natürlich weiterhin bestehen, für den Fall, dass nur eine Datei gebraucht
  wird.

---

## [2026-07-24] - Favoriten pro Konto gespeichert + Sidebar-Gruppierung (Projekte/Aktuelle)

**Favoriten (⭐ im Modell-Umschalter) verschwanden:** Sie lagen bisher nur im
Browser-`localStorage`, komplett unabhängig vom Server. Wirken Favoriten nach
einem Server-Neustart "verschwunden", liegt das fast immer daran, dass die
App über eine andere Adresse aufgerufen wurde als beim Setzen der Favoriten
(z. B. eine neue ngrok-URL bei jedem Tunnel-Neustart) - `localStorage` ist
strikt pro Adresse getrennt, ein Server-Neustart selbst löscht es nicht.
- Neue Spalte `favorite_models` in der `users`-Tabelle + `GET`/`PUT
  /api/favorites`: Favoriten werden jetzt zusätzlich am Konto gespeichert und
  bei Login/Session-Wiederherstellung geladen - bleiben so unabhängig von
  Adresse/Browser erhalten, solange man eingeloggt ist. `localStorage` bleibt
  als schneller lokaler/Gast-Fallback bestehen.

**Sidebar-Gruppierung wie im Referenz-Screenshot:** Die Chat-Liste ist jetzt
in Abschnitte gegliedert:
- **"Projekte"**: Chats werden nach zugeordnetem Yum-Projekt gruppiert, mit
  eigener Überschrift je Projekt.
- **"Aktuelle"**: alle nicht zugeordneten Chats darunter (nur als eigene
  Überschrift, wenn auch mindestens ein Projekt vorhanden ist - sonst gäbe es
  nur eine einzige, überflüssige Überschrift).
- `renderSidebar()` wurde dafür in eine `buildChatItem()`-Hilfsfunktion
  (einzelner Chat-Eintrag) und die neue Gruppierungslogik aufgeteilt.

---

## [2026-07-24] - Download-Button für Code-Blöcke

Jeder Code-Block (z. B. ```python ... ```) in einer KI-Antwort hat jetzt neben
dem Kopieren-Button auch einen **Download-Button** (⬇️). Ein Klick lädt genau
diesen Codeblock als Datei herunter, mit passender Endung je nach Sprache
(z. B. `.py` für Python, `.js` für JavaScript, `.sql`, `.html`, `.sh`, ...
unbekannte Sprachen fallen auf `.txt` zurück).

- `public/app.js`: `renderCodeBlockHtml()` rendert den zusätzlichen Button,
  `downloadCodeBlock()` erzeugt die Datei clientseitig per `Blob` + temporärem
  `<a download>` (kein Server-Roundtrip nötig).
- Übersetzt in DE/EN/FR/ES (`downloadTitle`).

---

## [2026-07-24] - Zurückgerudert: "Neuer Chat" & Löschen wieder vereinfacht

Der letzte Versuch (unbenutzte Chats nicht synchronisieren + ⋮-Menü dafür
ausblenden) war zu fehleranfällig und hat dazu geführt, dass "Neuer Chat"
manchmal gar nichts mehr tat und Chats sich nicht löschen ließen. Deshalb
zurückgebaut auf eine einfachere, robustere Variante:

- `initNewChat()` legt jetzt **immer** garantiert einen neuen, aktiven Chat an
  (kein "Wiederverwendungs"-Sonderfall mehr, der fehlschlagen konnte) - war
  der bisherige Chat nie benutzt (keine eigene Nachricht), wird er einfach
  verworfen statt als Karteileiche liegen zu bleiben.
- Das ⋮-Menü (Anheften/Umbenennen/Zum Projekt/Löschen) ist wieder für
  **jeden** Chat sichtbar, nicht nur für "benutzte".
- Die eine wirklich sinnvolle Verbesserung bleibt erhalten: Löscht man den
  aktiven Chat und es gibt noch einen anderen vorhandenen Chat, wechselt die
  App zu diesem, statt einen neuen leeren zu erzeugen.

---

## [2026-07-24] - Löschen des aktiven Chats: zu vorhandenem Chat statt neuem leeren

Beim Löschen des gerade aktiven Chats wurde bisher **immer** automatisch ein
neuer, leerer Chat erzeugt – selbst wenn noch andere, echte Chats vorhanden
waren. Das wirkte so, als würde nach dem Löschen sofort "wieder ein Chat
auftauchen". `handleDeleteChat()` wechselt jetzt stattdessen zu einem noch
vorhandenen echten Chat (falls einer existiert). Nur wenn wirklich **kein**
anderer Chat mehr übrig ist, wird ein neuer leerer angelegt – das ist dann
unvermeidbar, da immer ein aktiver Chat zum Schreiben vorhanden sein muss.

---

## [2026-07-24] - "Neuer Chat lässt sich nicht löschen" behoben

Jeder Klick auf "Neuer Chat" (bzw. `initNewChat()`, u. a. auch beim Login und
nach dem Löschen des aktiven Chats aufgerufen) hat den neuen, komplett leeren
Chat sofort zum Server synchronisiert. Löschte man diesen leeren Chat, wurde
automatisch ein Ersatz-Chat erzeugt (damit immer ein aktiver Chat existiert)
– der sah identisch aus ("New chat"), es wirkte also so, als würde das
Löschen nichts bewirken.

- `initNewChat()` synchronisiert einen frischen, noch unbenutzten Chat jetzt
  nicht mehr zum Server und legt auch keinen weiteren leeren Chat an, solange
  der aktuell aktive noch unbenutzt ist (nur Begrüßung, keine eigene
  Nachricht) – man bleibt einfach im selben leeren Chat.
- Der Sidebar-Eintrag für einen neuen Chat bleibt weiterhin sichtbar (klares
  Feedback beim Klick auf "Neuer Chat"), aber das ⋮-Menü (Anheften/
  Umbenennen/Zum Projekt/Löschen) erscheint erst, sobald der Chat eine erste
  eigene Nachricht enthält – vorher ergeben diese Aktionen ohnehin keinen
  Sinn, da noch nichts zu löschen/umzubenennen da ist.
- Ein Chat wird erst "real" (auf dem Server gespeichert), sobald die erste
  eigene Nachricht gesendet wird.

---

## [2026-07-24] - Login-Fehler "The quota has been exceeded." behoben

Beim Anmelden (und an sechs weiteren Stellen) wurde der komplette
Chat-Datensatz zusätzlich unkomprimiert in `localStorage` gecacht (fürs
Offline-Funktionieren). Bei vielen/großen Datei-Anhängen in Chats kann das
~5–10MB-Limit von `localStorage` im Browser überschritten werden – der
Browser wirft dann einen `QuotaExceededError` ("The quota has been
exceeded."), der bisher ungefangen bis in den Login-Handler durchschlug und
die Anmeldung komplett abbrechen ließ.

- Neue Funktion `saveChatsToLocalCache()` fängt diesen Fehler jetzt ab und
  loggt nur eine Warnung in die Konsole – Login/Senden/Speichern funktionieren
  weiter, nur der lokale Offline-Cache wird für diesen einen Schreibvorgang
  übersprungen (die Chats selbst sind ja bereits auf dem Server gespeichert).
- Betroffene Stelle war u. a. `loadChatsForCurrentScope()`, die direkt nach
  dem Login aufgerufen wird.

Falls die Meldung weiterhin auftaucht bzw. der Browser-Speicher dauerhaft
voll ist: im Browser die Website-Daten für `localhost:3000` löschen
(Chats bleiben erhalten, da sie über den Account auf dem Server liegen).

---

## [2026-07-23] - Projekt-Badge, Wortlaut & Sprachwechsel-Fixes

- **"Zum Projekt" statt "Yum-Projekt"**: Der Text zum Anlegen eines neuen
  Projekts hieß "Yum-Projekt hinzufügen" – heißt jetzt "Neues Projekt
  erstellen" (die Aktion im ⋮-Menü selbst hieß schon vorher korrekt
  "Zum Projekt hinzufügen").
- **Projekt-Zugehörigkeit war nirgends sichtbar**: `handleAddToProject` schrieb
  bisher den Platzhalter-String `'project'` in den Chat statt der echten
  Projekt-ID/des Namens – dadurch tauchte die Zuordnung nirgends in der
  Oberfläche auf. Jetzt zeigt jeder zugeordnete Chat in der Sidebar einen
  kleinen 📁-Badge mit Projektnamen (Projektnamen werden per
  `GET /api/yumProjects` gecacht).
- **Sprachwechsel blieb teilweise auf Deutsch**: Modell-Gruppen im
  Modell-Umschalter ("Ollama (lokal)", "OpenAI-kompatibel" etc.) waren fest
  auf Deutsch verdrahtet statt übersetzt zu werden; Buttons an bereits
  angezeigten Nachrichten (Kopieren/Bearbeiten/Export) wurden beim
  Sprachwechsel nicht neu gerendert. Beides jetzt behoben: `setLanguage()`
  lädt die Modell-Liste neu und rendert den aktiven Chat neu. Außerdem
  fehlende fr/es-Übersetzungsschlüssel ergänzt (`chatRenameTitle`,
  `chatRenamePrompt`) und den Tooltip "Andere Aktionen" übersetzbar gemacht.
- Nebenbei: ungültiger `//`-Kommentar in `styles.css` (kein gültiges
  CSS-Kommentarformat) entfernt.

---

## [2026-07-23] - OpenAI-kompatible Modelle im Modell-Umschalter

Der Modell-Umschalter im Frontend konnte schon immer eine ganze Liste von
OpenAI-kompatiblen Modellen (z. B. OpenRouter) anzeigen, `GET /api/models`
lieferte dafür aber immer `openaiCompatible: null` – nur das eine feste
Modell aus `OPENAI_COMPATIBLE_MODEL` war nutzbar.

- **Backend** (`server.js`): `/api/models` fragt jetzt bei gesetzter
  `OPENAI_COMPATIBLE_URL` zusätzlich `GET {url}/models` beim Anbieter ab und
  liefert die komplette Modell-Liste (`openaiCompatible.models`). Schlägt die
  Abfrage fehl (z. B. keine Internetverbindung), fällt es automatisch auf das
  eine Modell aus `OPENAI_COMPATIBLE_MODEL` zurück – kein Verhaltensbruch.
- Ausgewähltes Modell wird wie bisher pro Chat-Anfrage mitgeschickt
  (`provider` + `model` im Body von `POST /api/chat`).

---

## [2026-07-23] - Drei-Punkte-Menü für Chat-Aktionen fertiggestellt

Das Frontend (`public/app.js`, `public/styles.css`) hatte das Drei-Punkte-Menü
(⋮) bereits vollständig gebaut, die zugehörigen Backend-Routen fehlten aber
komplett. Ergänzt:

- **Backend** (`server.js`): neue Endpoints
  - `GET /api/chats` – Chats des Kontos laden (inkl. `pinned`, `yum_project_id`)
  - `PUT /api/chats/:id` – Chat speichern/aktualisieren (Titel + Nachrichten)
  - `DELETE /api/chats/:id` – Chat löschen
  - `PUT /api/chats/:id/pin` – Anheften umschalten (angeheftete Chats erscheinen oben)
  - `PUT /api/chats/:id/rename` – Chat umbenennen
  - `POST /api/chats/:id/addToProject` – Chat einem Yum-Projekt zuordnen
  - `GET /api/yumProjects` / `POST /api/yumProjects` – Yum-Projekte auflisten/anlegen
- **Datenbank** (`db.js`): `listChats` liefert jetzt `pinned`/`yum_project_id` und
  sortiert angeheftete Chats nach oben; neue Funktionen `pinChat` und
  `addChatToProject`.
- **Frontend** (`public/app.js`):
  - drei **Syntax-Fehler behoben**, die das Laden von `app.js` im Browser komplett
    verhindert haben (fehlende `}` im Übersetzungsobjekt, ein verwaister
    `catch`-Block ohne zugehörige Funktion, zwei Template-Strings ohne Backticks)
  - beim Laden vom Server bleiben `pinned`/`yum_project_id` jetzt erhalten
    (vorher gingen sie bei jedem Reload verloren)
  - "Yum-Projekt hinzufügen" kann jetzt auch ein neues Projekt anlegen, statt nur
    zwischen bereits vorhandenen zu wählen
- Ungenutzten, unvollständigen Entwurf `index.js` entfernt (war nirgends
  eingebunden, `server.js` ist der tatsächliche Einstiegspunkt)

---

## [2026-07-22] - Projektinitialisierung
- Chipilito Chatbot v2.0.0 gestartet
- SQLite-Datenbank (`chipilito.db`) erstellt
- Server läuft auf `http://localhost:3000`
- KI-Provider: Ollama (Standard-Modell: `gemma2`)