# 3. Telegram-Bot

🇬🇧 [English](../en/03-telegram.md)

Mit dem Telegram-Bot trägst du Ladevorgänge und Strecken vom Handy aus ein – **ohne VPN und ohne dein Home Assistant ins Internet zu öffnen**. Home Assistant fragt bei Telegram nach neuen Nachrichten („Polling“); von außen verbindet sich nichts mit deinem Heimnetz.

> **Grundlage dieser Anleitung:** Home Assistant 2026.9 mit der eingebauten Integration *Telegram bot*. Mit **(ungeprüft)** markierte Punkte sind Annahmen. **Behandle den Bot-Token wie ein Passwort** – nie posten, nie committen.

## 1. Bot anlegen

1. In Telegram den Chat mit **@BotFather** öffnen.
2. `/newbot` senden, einen Namen und einen Benutzernamen wählen (muss auf `bot` enden).
3. BotFather antwortet mit dem **API-Token**. Privat halten.
4. Optional, aber empfehlenswert – Befehlsmenü mit `/setcommands` anlegen (Bot wählen, dann senden):
   ```
   charge - Ladevorgang erfassen
   trip - Strecke erfassen
   status - Akku und letzte 30 Tage
   help - Hilfe und Tastenleiste
   ```
   Die Befehlsnamen sind die der Automationen in diesem Projekt (`/charge`, `/trip`, `/help`, `/status`). Du kannst sie später in der Automation umbenennen.

## 2. Integration in Home Assistant einrichten

1. **Einstellungen → Geräte & Dienste → Integration hinzufügen → Telegram bot.**
2. API-Token eintragen. Als Plattform **Polling** wählen (die Alternative, Webhooks, braucht eine von außen erreichbare HTTPS-Adresse – hier nicht nötig). **(Ungeprüft:** genaue Bezeichnungen der Einrichtungsdialoge.)
3. In den Optionen der Integration lässt sich der **Parse-Modus** einstellen. Dieses Projekt nutzt *markdown*.
4. **Erlaubte Chat-IDs:** Der Bot reagiert nur auf Chats, die du erlaubst. Trage deine eigene numerische Telegram-Chat-ID als *Allowed chat ID* in die Integration ein. So findest du sie: eine Nachricht an einen Bot wie *@userinfobot* senden oder nach einer Nachricht an deinen Bot im Home-Assistant-Protokoll nachsehen **(ungeprüft)**. Ohne erlaubte Chat-ID werden Nachrichten dieses Chats ignoriert, und es gibt keine Benachrichtigungs-Entität.

Danach hast du:

| Was | Beispiel | Wofür |
|---|---|---|
| Notify-Entität je erlaubtem Chat | `notify.<Bot>_<Chat-Titel>` | Nachrichten an dich senden (Erinnerungen, Antworten) |
| Event-Entität | `event.<Bot>_update` | zeigt das letzte Update von Telegram |
| Ereignisse auf dem Home-Assistant-Ereignisbus | `telegram_command`, `telegram_text`, `telegram_callback` | worauf die Automationen hören |

Die Entitätsnamen hängen davon ab, wie du Bot und Chat genannt hast. Die Automationen in [Anleitung 4](04-home-assistant.md) verwenden einen neutralen Beispielnamen (`notify.id7_telegram_example`) – ersetze ihn durch die Entitäts-ID deiner eigenen Notify-Entität.

## 3. Was der Bot kann

Zwei Automationen übernehmen das (siehe Anleitung 4):

**Direktbefehle**

| Befehl | Bedeutung |
|---|---|
| `/charge <kWh> <Preis> [Anbieter] [Notiz]` | Ladevorgang erfassen. Der Preis ist entweder der **Gesamtpreis** (`25.52`) oder **je kWh** (`0.57/kWh`). |
| `/trip <km> <kWh/100km> [Name]` | Strecke erfassen. |
| `/status` | Akkustand und die letzten 30 Tage. |
| `/help` oder `/start` | Hilfetext und Tastenleiste. |

**Geführter Dialog mit Tasten** – `/charge` oder `/trip` **ohne Werte** senden oder die Tasten der Tastenleiste drücken:

- **⚡ Charge (Laden):** kWh → Anbieter → Preisart (Gesamt oder je kWh) → Betrag. Dazu eine Schnelltaste für deinen **Favoriten-Preis** (`input_number.vwt_favorite_price`) und eine **Rückgängig**-Taste.
- **🛣 Strecke:** km → Verbrauch.
- **📊 Status.**

Der aktuelle Dialogschritt steht in einem Eingabe-Helfer und verfällt nach **30 Minuten**. Alle Texte in den veröffentlichten Automationen sind **englisch**; zum Übersetzen die Meldungen in `packages/vwt_automations.yaml` bearbeiten. Dezimalzahlen lassen sich mit Punkt oder Komma eingeben.

## 4. Erinnerungen von Home Assistant

Steigt der Akkustand um mindestens 5 %, schickt Home Assistant dir eine Nachricht mit der Bitte, den Ladevorgang zu erfassen – samt fertigem `/charge`-Befehl mit der geschätzten kWh-Menge. Ein vorheriger Wert von 0 % wird ignoriert (kurze Datenlücken nach einem evcc-Neustart würden sonst wie eine Ladung von 0 auf 100 % aussehen).

## 5. Testen

1. `/help` an den Bot senden. Du solltest den Hilfetext und eine Tastenleiste bekommen.
2. **⚡ Charge** drücken und dem Dialog folgen, oder `/charge 40 20.00 Test` senden.
3. In Home Assistant sollte das Ladelog (To-do-Liste) einen neuen Eintrag enthalten.
4. Den Test-Eintrag wieder löschen (Dashboard → *Protokolle* → Eintrag wählen → löschen), damit deine Summen stimmen.

## Fehlersuche

| Problem | Prüfen |
|---|---|
| Bot antwortet nicht | Ist die Integration geladen? Ist deine Chat-ID eine *Allowed chat ID*? Home-Assistant-Protokoll (`telegram_bot`). |
| „Unauthorized“ oder keine Updates | Token falsch oder widerrufen: bei BotFather mit `/token` einen neuen erzeugen und in der Integration aktualisieren. |
| Befehle bewirken nichts | Sind die beiden Automationen aktiv? Ablaufprotokolle in Home Assistant ansehen. |
| Falscher Name der Notify-Entität | Entitäts-ID der Notify-Entität prüfen und in den Automationen anpassen. |

## Weiter

[4. Home Assistant](04-home-assistant.md) *(geplant)*
