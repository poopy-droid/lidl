# Lidl Connect Datenverbrauchs-Benachrichtigung 📱

Ein Skript, das sich automatisch in dein **Lidl-Connect-Konto** einloggt, deinen Datenverbrauch überwacht und **automatisch +1 GB nachbucht**, sobald 80 % deines Volumens verbraucht sind — plus Benachrichtigungen über **Telegram** und/oder **Discord**.

So gehst du nie wieder unerwartet ohne Datenvolumen aus.

> ⚠️ **Hinweis:** Dieses Skript dient ausschließlich zu Demonstrationszwecken. Der Einsatz automatisierter Skripte/Bots zur Automatisierung ist laut Lidl-Richtlinien strikt untersagt. Verstöße können zu einem **sofortigen Ausschluss bzw. zur Kündigung** führen. Du nutzt es auf eigene Verantwortung.

---

## ✨ Funktionen

- **Automatische Anmeldung** – Playwright-Login in dein Lidl-Connect-Konto (keine manuelle Eingabe nötig)
- **Datenabruf** – liest verbleibendes Tarif- und Refill-Volumen live aus dem Lidl-Dash-Board
- **Automatische Nachbuchung** – +1 GB, ab 80 % Verbrauch; Zyklus wiederholt sich, bis das Nachfüll-Kontingent deines Plans aufgebraucht ist
- **Adaptive Intervalle** – Check- und Keep-Alive-Intervalle skalieren nach verbliebenem Volumen **und** deiner Internet-Geschwindigkeit
- **Benachrichtigungssystem** – Status-, Refill- und Fehlermeldungen per Telegram und/oder Discord
- **Automatische Updates** – prüft optional GitHub und aktualisiert sich selbst
- **Watchdog** – 30-Sekunden-Heartbeat; erkennt hängende Prozesse, startet Browser neu
- **Robuste Fehlerbehandlung** – Retry, 10-min-Cool-down, Browser-Restart bei wiederholten Fehlern

---

## 🔁 Wie die automatische Nachbuchung funktioniert

1. Das Skript liest nach jedem Check dein **Tarifvolumen** und dein **Refill-Volumen** aus der Lidl-Seite.
2. Ab **80 % Verbrauch** des Gesamtvolumens klickt es `Refill aktivieren` → **+1 GB**.
3. Verbraucht das 1 GB auf → nächster Check → **nächstes 1 GB** → und so fort.
4. Die **verbliebenen 20 % deines Tarifs bleiben unangetastet** — das Refill-Volumen wird zum eigentlichen Datenpuffer.
5. Fehlversuch (z. B. Refill-Kontingent aus, UI geändert) → **10 Minuten Cool-down**, dann neuer Versuch.
6. Der Zyklus endet, wenn dein Plan keine Nachfüllungen mehr erlaubt — danach zählt dein unangetasteter Tarif-Buffer weiter.

Der Fortschritt zur 80 %-Schwelle erscheint als Bar im Log und in den Benachrichtigungen, z. B.:

```text
⏳ WAITING FOR 80%
used 18.2/26.0 GB (70%)
[████████████████░░]
88% of 80% used · to 80%: 2.6 GB
```

---

## 📊 Adaptive Intervalle

Check- und Keep-Alive-Intervalle sind **kein fester Ticker** — sie skalieren nach:

1. **Verbliebenem Datenvolumen** (mehr Daten → längere Wartezeit)
2. **Internet-Geschwindigkeit** über `INTERNET_SPEED_MBPS` (Platzhalter: 500 Mbps)
   - Skalierungsfaktor: `500 / speed`, eingezogent auf **0.5 – 3.0**
   - Schnelleres Netz → kürzere Intervalle · Langsameres Netz → längere Intervalle

| Verfügbare Daten | Check-Intervall (bei 500 Mbps) |
|:---|---:|
| ≥ 10 GB  | 15–30 min  |
| ≥ 5 GB   | 10–15 min  |
| ≥ 3 GB   | 5–7,5 min  |
| ≥ 2 GB   | 2,5–4 min  |
| ≥ 1,2 GB | 1,5–2,5 min |
| ≥ 1 GB   | 1–1,5 min  |
| < 1 GB   | 1 min      |

**Keep-Alive** (Session halten): Basis 2 min je 25 GB, max. 30 min, sinkt linear auf **30 s** an, wenn die 80 %-Schwelle erreicht ist. Jitter +0–50 % — Intervalle werden nur verlängert, nie verkürzt.

---

## 📦 Installation

### Voraussetzungen

- **Node.js** (Version 16 oder höher) + npm
- Ein **Lidl-Connect-Konto**
- (Optional) Ein **Telegram-Bot** und/oder ein **Discord-Webhook**

### Schritte

```bash
# 1. Repo klonen (dein Fork):
git clone https://github.com/DEIN-BENUTZER/lidl
cd lidl

# 2. Abhängigkeiten installieren:
npm install

# 3. Playwright-Browser installieren:
npx playwright install

# 4. Konfiguration anlegen:
cp .env.example .env
```

### `.env` konfigurieren

Pflichtfeld `RUFNUMMER` + `PASSWORD` ausfüllen, den Rest nach Bedarf (siehe [Konfiguration](#-konfiguration-env)).

---

## ⚙️ Konfiguration (`.env`)

| Variable | Standard | Beschreibung |
|---|:---|---|
| `RUFNUMMER` | – *(Pflicht)* | Lidl-Login, mit `0` am Anfang |
| `PASSWORD` | – *(Pflicht)* | Lidl-Login-Passwort |
| `BROWSER` | `firefox` | `firefox`, `webkit` oder `chromium` |
| `TELEGRAM_ALLOW` | `false` | Telegram-Benachrichtigungen aktivieren |
| `TELEGRAM_TOKEN` | – | Token deines Telegram-Bots |
| `TELEGRAM_CHAT_ID` | – | Chat-ID für die Nachrichten |
| `DISCORD_ALLOW` | `false` | Discord-Benachrichtigungen aktivieren |
| `DISCORD_WEBHOOK_URL` | – | Webhook-URL für Discord |
| `KILL_EXISTING_PROCESSES` | `true` | Beendet **alle** Browser-Prozesse beim Start (Achtung: nicht parallel mit anderen Browser-Skripten laufen!) |
| `KILL_SCRIPT_INSTANCES` | `true` | Beendet alte `script.js`-Instanzen (hilft bei 100 % CPU-Hängen) |
| `AUTO_UPDATE` | `true` | Prüft GitHub nach neuerer Version, aktualisiert sich |
| `SLEEP_MODE` | `smart` | `random` · `fixed` · `smart` — Wartezeit-Modus der Hauptschleife |
| `SLEEP_TIME` | – | Festwert in Sekunden für `SLEEP_MODE=fixed` (min. 60 s) |
| `INFO_LEVEL` | `info` | `info` · `warn` · `error` — Log-/Benachrichtigungsdetails (akzeptiert auch den alten Namen `INFOLEVEL`) |
| `INTERNET_SPEED_MBPS` | `500` | Deine Internet-Geschwindigkeit in Mbps für die adaptive Intervall-Skalierung |
| `HEADLESS` | *(headless)* | Auf `false` setzen, wenn du das Browser-Fenster sehen willst |

---

## 💻 Nutzung

**Standard (headless):**

```bash
node script.js
```

**Windows-Start (headless, Browser-Fenster bleibt geschlossen):**

```
start.bat
```

**Längerer Betrieb (Linux):**

```bash
nohup node script.js &
```

**Docker:**

```bash
docker build -t lidl-extender .
docker run -d --name lidl-extender --hostname lidl-extender --restart unless-stopped lidl-extender
```

Das Skript meldet sich ein, liest den aktuellen Verbrauch, sendet die Status-/Refill-Benachrichtigung und wiederholt den Zyklus mit adaptiv berechneten Intervallen.

---

## 🐛 Troubleshooting

| Problem | Ursache / Lösung |
|---|---|
| `ENV Fehler: RUFNUMMER oder PASSWORD fehlt` | `.env` fehlt oder Pflichtfelder leer — `.env.example` kopieren und ausfüllen |
| `Zu viele NaN-Fehler` | Lidl-UI geändert oder Session abgelaufen — Skript startet den Browser automatisch neu und loggt sich wieder ein |
| `Refill-Aktivierung fehlgeschlagen` | Kein Nachfüll-Kontingent mehr, Button/Text auf der Lidl-Seite geändert — 10 min Cool-down, dann neuer Versuch |
| 100 % CPU, kein Fortschritt | Setze `KILL_SCRIPT_INSTANCES=true`, Skript startet neu |
| Browser-Fenster soll sichtbar sein | `HEADLESS=false` (oder `start.bat` für den headless-Modus) |
| Telegram/Discord nichts empfangen | `TELEGRAM_ALLOW`/`DISCORD_ALLOW` auf `true`, Token/Webhook prüfen |

---

## 📂 Projektstruktur

```
├── .env.example          # Beispiel für die Umgebungsvariablen
├── .env                  # Persönliche Zugangsdaten (nicht hochladen!)
├── CHANGELOG.md          # Versionsverlauf
├── Dockerfile            # Docker-Build-Konfiguration
├── LICENSE               # GNU Public License v3.0
├── package.json          # Abhängigkeiten und Metadaten (v1.2.4)
├── package-lock.json     # Abhängigkeitsversionen
├── README.md             # Diese Datei
├── script.js             # Hauptskript
└── start.bat             # Windows-Start (Headless)
```

---

## 📝 Version & Changelog

Aktuelle Version: **1.2.4**

Alle Änderungen sind detailliert in der [CHANGELOG](CHANGELOG.md) nachvollziehbar.

---

## 📄 Lizenz

Dieses Projekt steht unter der **GNU General Public License v3.0**.

---

## 💖 Danke

Danke, dass du dir dieses Projekt angesehen hast!
Ich hoffe, es hilft dir, deinen Datenverbrauch immer im Blick zu behalten.
