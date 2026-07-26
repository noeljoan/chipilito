# Beitragen zu Chipilito

Danke für dein Interesse an Chipilito! Diese Datei beschreibt kurz, wie du Änderungen beisteuern kannst.

## 🚀 Lokal einrichten

```bash
git clone https://github.com/yourusername/chipilito.git
cd chipilito
npm install
cp .env.example .env   # Werte nach Bedarf anpassen
npm start
```

Der Server läuft danach unter `http://localhost:3000`. Für Änderungen an lokalen KI-Antworten zusätzlich [Ollama](https://ollama.com) installieren und ein Modell laden (`ollama pull gemma2`).

## 🐛 Bugs melden

Bitte über ein GitHub Issue melden, mit:
- kurzer Beschreibung des Problems
- Schritten zum Reproduzieren
- erwartetes vs. tatsächliches Verhalten
- falls hilfreich: Konsolen-Ausgabe (Server-Terminal und/oder Browser-Konsole `F12`)

## 💡 Neue Features vorschlagen

Gerne zuerst ein Issue mit der Idee eröffnen, bevor viel Code geschrieben wird – so lässt sich früh klären, ob das Feature ins Projekt passt.

## 🔧 Pull Requests

1. Repository forken
2. Feature-Branch erstellen: `git checkout -b feature/meine-neue-funktion`
3. Änderungen committen: `git commit -m 'Kurze, klare Beschreibung der Änderung'`
4. Branch pushen: `git push origin feature/meine-neue-funktion`
5. Pull Request öffnen, mit kurzer Beschreibung **was** sich ändert und **warum**

## 🎨 Code-Stil

- 2 Leerzeichen Einrückung, kein Tab
- `const`/`let` statt `var`
- Kommentare auf Deutsch (passend zum Rest des Projekts), Variablennamen/Funktionsnamen auf Englisch
- Neue Provider/Endpunkte nach Möglichkeit dem bestehenden Muster in `server.js` folgen (klare Fehlermeldungen, kein stiller Absturz)
- Vor dem PR kurz manuell durchtesten: Login/Registrierung, ein Chat mit dem Standard-Provider, ggf. betroffene Exportformate

## 🌍 Übersetzungen

Neue UI-Texte immer in **allen vier Sprachen** (DE/EN/FR/ES) ergänzen – die Übersetzungstabellen liegen am Anfang von `public/app.js`. Ein PR, der nur eine Sprache abdeckt, wird wahrscheinlich um Ergänzung der anderen drei gebeten.

## ✅ Checkliste vor dem PR

- [ ] Server startet ohne Fehler (`npm start`)
- [ ] Betroffene Funktion manuell getestet
- [ ] Keine Secrets (API-Keys, Passwörter) versehentlich committet
- [ ] `.env` nicht mit eingecheckt (nur `.env.example` bei neuen Variablen aktualisieren)
- [ ] Übersetzungen ergänzt, falls neue UI-Texte hinzugekommen sind

## Fragen?

Einfach ein Issue eröffnen – auch für Rückfragen, nicht nur für fertige Bugs/Vorschläge.
