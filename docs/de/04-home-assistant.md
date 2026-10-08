# 4. Home Assistant: Helfer, Skripte, Automationen, Dashboard

🇬🇧 [English](../en/04-home-assistant.md)

Diese Anleitung installiert alles innerhalb von Home Assistant. Vorher muss [Anleitung 2 (evcc)](02-evcc.md) laufen; der Telegram-Teil ([Anleitung 3](03-telegram.md)) ist optional.

> **Getestet mit** Home Assistant 2026.9 auf einer frischen Container-Installation nach den folgenden Schritten (die Oberflächen-Schritte über die entsprechenden API-Aufrufe): Pakete, To-do-Listen, Konfigurationsprüfung, Neustart, Werte setzen, Dashboard speichern, einen Ladevorgang erfassen und wieder löschen (alle Summen gingen auf 0 zurück). Die evcc-Integration und die Karten wurden von Hand installiert, **nicht über HACS** (ungeprüft). Die ursprüngliche deutsche Fassung läuft auf Home Assistant OS erst seit wenigen Tagen. **Nicht getestet:** die Telegram-Nachrichten und -Tasten auf einer frischen Instanz (dort gibt es keinen Bot), die 30-Tage-Werte auf einer frischen Instanz (sie brauchen eine erste Stunde Statistik) und die Knöpfe des Dashboards. Mit **(ungeprüft)** markierte Punkte sind Annahmen.

## Was installiert wird

| Datei | Inhalt |
|---|---|
| `packages/vwt_helpers.yaml` | Eingabe-Helfer, Zähler, Adapter-Sensoren für evcc, berechnete Sensoren, Utility-Meter |
| `packages/vwt_scripts.yaml` | Skripte: Ladevorgang erfassen, Strecke erfassen, Korrektur anwenden, Eintrag löschen |
| `packages/vwt_automations.yaml` | automatische Streckenerkennung, Lade-Erinnerung, 30-Tage-Werte, Korrekturliste, Telegram-Befehle und -Dialog |
| `dashboards/vwt-dashboard.yaml` | Dashboard mit den Reitern *Now*, *Analysis* und *Logs* |

Alle Entitäten beginnen mit `vwt_`. Nichts davon enthält persönliche Daten.

## 1. Voraussetzungen

1. **evcc-Integration für Home Assistant** (`marq24/ha-evcc`, eine eigene Integration). Mit HACS installieren (nach „evcc“ suchen) oder von Hand, dann unter *Einstellungen → Geräte & Dienste* hinzufügen. Die Adresse deines evcc eintragen (z. B. `http://homeassistant.local:7070`). **Nach der Installation der eigenen Integration Home Assistant neu starten** (erst dann lässt sie sich hinzufügen). **Websocket** eingeschaltet lassen: Dann kommen neue Werte praktisch sofort in Home Assistant an.
2. **HACS-Karten** für das Dashboard: *Mushroom*, *card-mod*, *ApexCharts card*, *Plotly graph card*.
3. **Telegram-Bot** (optional): siehe [Anleitung 3](03-telegram.md).

## 2. evcc-Entitäts-IDs herausfinden

Die Entitäts-IDs hängen vom **Titel deines evcc-Ladepunkts** ab. *Entwicklerwerkzeuge → Zustände* öffnen und nach `vehicle_soc` suchen. Beim Ladepunkt-Titel `ID.7 (virtual)` heißen die Entitäten:

```
sensor.evcc_id_7_virtual_vehicle_soc
sensor.evcc_id_7_virtual_vehicle_range
sensor.evcc_id_7_virtual_vehicle_odometer
```

Bei dir können sie anders heißen (Titel, Sprache: „virtuell“ statt „virtual“).

## 3. Pakete aktivieren

In der `configuration.yaml` ergänzen (gibt es schon einen Abschnitt `homeassistant:`, nur die Zeile `packages` hinzufügen):

```yaml
homeassistant:
  packages: !include_dir_named packages
```

Den Ordner `packages` neben der `configuration.yaml` anlegen und die drei Dateien aus dem Ordner `packages/` dieses Repositorys hineinkopieren (mit der App „File editor“, Samba oder SSH).

## 4. Die drei evcc-Entitäts-IDs anpassen

`packages/vwt_helpers.yaml` öffnen, den Abschnitt **ADAPTER SENSORS** suchen und `evcc_id_7_virtual` durch den Anfang deiner eigenen Entitäten ersetzen (Suchen-und-Ersetzen in der ganzen Datei genügt, sechs Stellen). Das ist die einzige Stelle, die deine evcc-Namen kennt; alles andere nutzt `sensor.vwt_soc`, `sensor.vwt_range` und `sensor.vwt_odometer`.

> **Die Sensoren und ihre `name:`-Zeilen nicht umbenennen.** In YAML leitet Home Assistant die Entitäts-ID aus dem *Namen* ab; Skripte, Automationen und Dashboard verlassen sich auf die IDs `sensor.vwt_…`.

## 5. Die beiden To-do-Listen anlegen

*Einstellungen → Geräte & Dienste → Integration hinzufügen → Local To-do.* Zwei Listen mit genau diesen Namen anlegen:

- `VWT Charging log` → Entität `todo.vwt_charging_log`
- `VWT Trip log` → Entität `todo.vwt_trip_log`

## 6. Konfiguration prüfen und neu starten

*Entwicklerwerkzeuge → YAML → Konfiguration prüfen*, danach **Home Assistant neu starten**. Anschließend unter *Entwicklerwerkzeuge → Zustände* nach `vwt_` filtern: Du solltest die Helfer, die Sensoren `sensor.vwt_soc`, `sensor.vwt_range`, `sensor.vwt_odometer` (mit den Werten deines Autos), die Skripte und sechs Automationen sehen.

## 7. Eigene Werte setzen

In *Entwicklerwerkzeuge → Zustände* oder auf einer vorübergehenden Dashboard-Karte setzen:

| Entität | Wert |
|---|---|
| `input_number.vwt_battery_net_kwh` | **nutzbare (netto)** Akkukapazität deines Autos in kWh, für die Verbrauchsberechnung (der ID.7 Pro S hat 86 kWh netto) |
| `input_number.vwt_favorite_price` | dein üblicher Preis pro kWh an deiner Stammsäule (für die Taste „⭐ Favorite“ in Telegram) |
| `input_text.vwt_notify_entity` | Entitäts-ID deiner Telegram-Notify-Entität (nur wenn du Telegram nutzt) |

Die Referenzwerte der automatischen Streckenerkennung (`vwt_last_odometer`, `vwt_last_soc`) füllen sich bei der ersten Änderung des Kilometerstands von selbst.

## 8. Dashboard installieren

1. *Einstellungen → Dashboards → Dashboard hinzufügen →* **Neues Dashboard von Grund auf**, Titel `EV tracker`, **URL `vwt-dashboard`** (die Knöpfe im Dashboard navigieren zu dieser Adresse).
2. Das neue Dashboard öffnen, Stift (Bearbeiten) → drei Punkte → **Rohkonfigurations-Editor**.
3. Inhalt löschen, den vollständigen Inhalt von `dashboards/vwt-dashboard.yaml` einfügen, speichern.

## 9. Testen

1. **Ladevorgang erfassen:** Reiter *Logs* → Knopf *Charging* (oder `script.vwt_record_charge` unter *Entwicklerwerkzeuge → Aktionen* aufrufen) mit 40 kWh und 20 €. Das Ladelog bekommt einen Eintrag, die Summen und die Werte *This month* steigen.
2. **Rückgängig:** Unter *Logs* den Eintrag bei *Correct entry* wählen → *Delete*. Die Summen kehren zu den alten Werten zurück.
3. **Strecken:** ein paar Kilometer fahren. Etwa 15 bis 30 Minuten nachdem das Auto den neuen Kilometerstand gemeldet hat, sollte automatisch ein Streckeneintrag im Fahrtenbuch erscheinen, mit dem Verbrauch aus dem Akkuabfall (bei einer sehr kurzen Strecke ist der Verbrauch grob: 1 % Akku sind schon etwa 0,9 kWh).
4. **Telegram** (falls genutzt): `/help`, danach die Tasten.

## Gut zu wissen

- **Format der Logeinträge:** `2026-09-13 · Aral · 44.78 kWh · 25.52 € · 0.57 €/kWh`. Das Korrektur-Skript liest die Werte aus diesem Text zurück, deshalb Einträge nicht von Hand ändern.
- **Rückdatierte Einträge:** Die Skripte akzeptieren ein Datum. Ein solcher Eintrag zählt in den Summen, aber nicht in den Zählern des *laufenden* Monats oder Jahres.
- **Utility-Meter zeigen „unknown“**, bis sich ihre Quelle zum ersten Mal ändert. Das ist normal.
- **30-Tage-Werte** werden aus der Langzeit-Statistik berechnet; bei einer frischen Installation bleiben sie auf 0, bis die erste Stunden-Statistik existiert.
- **Namen der Automationen** entstehen aus ihren Aliassen (z. B. `automation.vwt_telegram_commands`).
- **Kurze Nullwerte:** evcc kann nach einem Neustart kurz `0` für Akku- oder Kilometerstand melden. Der Kilometerstand-Adapter ignoriert 0; die Lade-Erinnerung ignoriert einen vorherigen Akkustand von 0 %.
- **Sprache:** Alle Namen, Meldungen und das Dashboard sind englisch. Für eine andere Sprache die Texte in den Paketdateien und im Dashboard ändern.

## Aktualisieren / Entfernen

- **Aktualisieren:** Dateien in `packages/` ersetzen, Konfiguration prüfen, neu starten. Summen und Logs bleiben erhalten (sie liegen in den Helfern und den To-do-Listen).
- **Entfernen:** Dateien in `packages/` löschen, die beiden To-do-Listen und das Dashboard entfernen, neu starten.

## Weiter

[5. Fehlersuche](05-troubleshooting.md)
