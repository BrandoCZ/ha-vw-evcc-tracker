# 1. VW EU Data Act Portal: Daten des Autos anfragen

🇬🇧 [English](../en/01-vw-eu-data-act-portal.md)

Nach dem EU Data Act kannst du die Daten anfordern, die dein Auto erzeugt. In diesem Projekt liefert das Portal Akkustand, Reichweite und Kilometerstand an **evcc**, das sie an Home Assistant weitergibt.

> **Grundlage dieser Anleitung:** das Portal im Oktober 2026 (deutsche Oberfläche, Privatkunde mit einem VW ID.7). Menünamen können in deinem Land, deiner Sprache oder bei deiner Marke abweichen. Mit **(ungeprüft)** markierte Punkte sind Annahmen – bitte Korrekturen melden.

## Voraussetzungen

- Ein Volkswagen-Konto, das mit dem Auto verbunden ist (das Konto, das beim Fahrzeug als „Primary user“ erscheint)
- Das Fahrzeug muss im Portal sichtbar sein (siehe [FAQ des Portals](https://eu-data-act.drivesomethinggreater.com/de/en/service/faq.html), Eintrag „Why is my vehicle not visible in the EU Data Act Portal?“)

## Schritte

1. Portal öffnen: <https://eu-data-act.drivesomethinggreater.com> und mit dem Volkswagen-Konto anmelden.
2. **Fahrzeugübersicht** öffnen und das Auto wählen (die Fahrgestellnummer wird angezeigt). **Fahrzeugdetails** öffnen.
3. Auf der Detailseite gibt es drei Anfragearten:
   - **Daten abrufen** – einmalig eine Datei mit historischen Daten innerhalb von 24 Stunden. Wird hier nicht gebraucht.
   - **Benutzerdefinierte Daten abrufen** – laufende Lieferung von Data Clustern deiner Wahl in einer Häufigkeit deiner Wahl. **Diese Anfrage ist die richtige.**
   - **Datenweitergabe anfragen** – Daten an Dritte geben. Wird nicht gebraucht.
4. **Benutzerdefinierte Daten anfragen** wählen und einstellen:
   - **Name:** frei wählbar (z. B. „HA und evcc“).
   - **Data Clusters:** *All Data*. (Eine kleinere Auswahl kann ebenfalls reichen; ob sie alles Nötige für evcc enthält, ist **ungeprüft**.)
   - **Häufigkeit:** die kürzeste verfügbare Stufe. Dieses Projekt nutzt **alle 15 Minuten**. Die möglichen Werte zeigt das Formular; zuerst sahen wir **täglich**.
5. Absenden. Die Anfrage erscheint danach unter **Ihre Dateien** mit Häufigkeit und Datum der nächsten Datei.

## Regeln, die du kennen musst

- **Es kann nur eine benutzerdefinierte Anfrage gleichzeitig aktiv sein.** Um Häufigkeit oder Cluster zu ändern, musst du die laufende Anfrage löschen (*Datenpaket löschen*) und neu anlegen. Dabei kann sich die Kennung ändern, mit der evcc die Daten abholt (**ungeprüft**) – danach in evcc prüfen (siehe Anleitung 2).
- **Dateien werden höchstens 7 Tage aufbewahrt** und müssen **innerhalb von 7 Tagen nach der Erstellung abgerufen** werden, sonst wird die Anfrage abgebrochen. evcc ruft sie laufend ab; wenn evcc und Home Assistant länger als eine Woche ausfallen, die Anfrage prüfen.
- Ein Paket enthält höchstens **30 Dateien**.
- **Wartungsfenster:** Gelegentlich sperrt das Portal das *Anlegen* neuer Anfragen (laufende Lieferungen funktionieren weiter). Ein Hinweis steht auf der Startseite des Portals.
- Ein **Data Dictionary** (PDF) mit allen Datenpunkten je Cluster kann im Portal heruntergeladen werden. Es nennt die vorhandenen Schlüssel (z. B. `state_of_charge`), aber nicht, wie oft ein Wert aktualisiert wird.

## Was zu erwarten ist

- Bei **täglicher** Lieferung kann der Akkustand in Home Assistant bis zu einen Tag hinter dem Auto zurückliegen. Bei uns war das der Grund, warum sich das Dashboard nach einer Fahrt nicht aktualisierte. Bei **alle 15 Minuten** kommen neue Werte etwa 15 bis 30 Minuten nach der Meldung des Autos an.
- Das Portal sendet keine Push-Benachrichtigung: evcc fragt ab. Die Verzögerung setzt sich aus Lieferintervall (Portal) und Abrufintervall (evcc) zusammen.
- Direkt nach einem evcc-Neustart oder bei einer Lieferlücke kann evcc kurz `0` für Akku- oder Kilometerstand melden. Unsere Automationen ignorieren einen Akkustand von 0 % als Startwert.

## Was eine Datendatei enthält

Mit dem Cluster *All Data* enthielt eine Datei für einen ID.7 etwa 100 Einträge (`key`, `dataFieldName`, `value`). Sie enthält keinen Standort und keine Geschwindigkeit. Gesehen im Oktober 2026:

| Thema | Beispiele |
|---|---|
| Akku, Reichweite | `battery_state_report.soc`, `battery_level_HV.value`, `value` (vermutlich Reichweite; **ungeprüft**), `energy_contents.*` (Einheit unklar) |
| Kilometerstand | `mileage.value`, `mileage.state` |
| Laden | `charging_state_report.*` (Modus, Zustand, Szenario), `battery_state_report.charge_power`, `settings.target_soc`, `battery_care_mode.charge_bcam_threshold`, `settings.max_charge_current_ac` |
| Klima | `climatisation_state`, `climatisation_settings.*` (Zieltemperatur, Zonen, Heizung), `remaining_climate_time` |
| Fahrzeugzustand | `locked`, `open`, `parking_brake`, `parking_light_*` |
| Temperaturen | `outdoor_temperature`, `min_temperature`, `max_temperature` |
| Meta | `car_captured_time`, `timestamp`, `update_reason`, `report_type`, `error_code` |

evcc gibt nur Akkustand, Reichweite, Kilometerstand und Ladelimit an Home Assistant weiter. Die übrigen Felder stehen dort nicht zur Verfügung. Die Dateien enthalten die Fahrgestellnummer und eine Benutzerkennung: niemals posten.

## Weiter

[2. evcc: Installation und Konfiguration](02-evcc.md)
