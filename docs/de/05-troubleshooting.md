# 5. Fehlersuche

🇬🇧 [English](../en/05-troubleshooting.md)

Alles hier wurde im ursprünglichen Setup gesehen (VW ID.7, evcc 0.316, Home Assistant 2026.9). Mit **(ungeprüft)** markierte Punkte sind Annahmen.

## Die Daten in Home Assistant sind alt oder ändern sich nicht

Geh die Kette vom Auto bis zu Home Assistant durch. Der erste Schritt, der alte Werte zeigt, ist der Übeltäter.

| # | Prüfen | Wenn alt / falsch |
|---|---|---|
| 1 | **Die App deines Autos** (z. B. We Connect): aktueller Akkustand und Kilometerstand? | Ist die App auch alt, hat VW noch keine neueren Daten. Warten; das Projekt kann das nicht beschleunigen. |
| 2 | **VW EU Data Act Portal** → Fahrzeugdetails: Die Anfrage existiert, zeigt die gewünschte Häufigkeit und ein Datum für die *nächste Datei* ([Anleitung 1](01-vw-eu-data-act-portal.md)) | Anfrage fehlt oder wurde abgebrochen (Dateien müssen innerhalb von 7 Tagen abgerufen werden): neu anlegen. **Tägliche** Lieferung = bis zu einen Tag Verzögerung: kürzere Häufigkeit wählen (wir nutzen 15 Minuten). |
| 3 | **evcc**: Ladepunkt → *Update behaviour* muss **always** sein (Intervall 15 min) ([Anleitung 2](02-evcc.md)) | Bei *charging* (Standard) oder *connected* fragt evcc ein abgestelltes, nicht angestecktes Auto nie ab. Nach dem Speichern **Restart** drücken. |
| 4 | **evcc-Werte**: `http://<evcc>:7070/api/state` öffnen und `loadpoints[0].vehicleSoc`, `vehicleOdometer` ansehen | Sind sie alt, liegt das Problem vor Home Assistant (Schritte 1–3 oder das evcc-Protokoll: Zeilen `dsg ERROR` mit *timeout* bedeuten, dass der VW-Dienst langsam oder nicht erreichbar war). |
| 5 | **Home Assistant**: *Entwicklerwerkzeuge → Zustände*, nach `vehicle_soc` suchen | Hat evcc den neuen Wert, Home Assistant aber nicht: Integration neu laden, **Websocket** eingeschaltet lassen, Protokoll der Integration ansehen. |

Normale Verzögerung: Neue Werte kommen etwa **15 bis 30 Minuten** nach der Meldung des Autos an (Lieferintervall plus Abrufintervall von evcc). VW sendet nichts per Push.

## Akku- oder Kilometerstand ist kurz 0

Nach einem evcc-Neustart oder einer Lieferlücke von VW kann evcc kurz `0` melden.

- Der **Kilometerstand-Adapter** (`sensor.vwt_odometer`) ignoriert 0 (er ist dann *nicht verfügbar*, bis ein echter Wert kommt); die automatische Streckenerkennung löst beim nächsten echten Wert wieder aus.
- Die **Lade-Erinnerung** ignoriert einen vorherigen Akkustand von 0 %, sonst sähe „0 % → 55 %“ wie eine Ladung aus.
- Siehst du trotzdem eine falsche Meldung „Charging detected“, prüfe, ob deine Kopie von `vwt_automations.yaml` die Bedingung `trigger.from_state.state | float(0) > 0` enthält.

## Die automatische Strecke hat falsche Akkuwerte (z. B. „55 % → 55 %“, 0 kWh)

Der Referenzwert `input_number.vwt_last_soc` wurde überschrieben, bevor die Strecke erfasst wurde (zum Beispiel durch eine falsche Ladeerkennung). So korrigierst du das:

1. Den richtigen Eintrag mit `script.vwt_record_trip` anlegen (*Entwicklerwerkzeuge → Aktionen*): Name, km, kWh (Akkuabfall in % × Nettokapazität / 100), Quelle `automatic`.
2. Den falschen Eintrag löschen: Reiter *Logs* → *Correct entry* → *Delete*.

Ein winziger Akkuabfall (1–2 %) liefert einen sehr groben Verbrauch; das gleicht sich über viele Strecken aus.

## Summen oder Monatswerte sind falsch

- **Logeinträge niemals von Hand ändern.** Das Korrektur-Skript liest die Werte aus dem Eintragstext.
- **Monats-/Jahreszähler** beginnen mit dem ersten Wert, den sie sehen, und zeigen *unknown*, bis sich ihre Quelle zum ersten Mal ändert.
- **Rückdatierte Einträge** (ein Datum in der Vergangenheit) zählen in den Summen, aber nicht in den Zählern des laufenden Monats/Jahres. Haben ältere Buchungen die Zähler verschoben, korrigiere sie mit *Entwicklerwerkzeuge → Aktionen → `utility_meter.calibrate`*.
- **Nach einer manuellen Korrektur** einer Summe: Die Helfer `input_number.vwt_total_*` enthalten die Summen und lassen sich direkt setzen.

## Home Assistant meldet Fehler

| Meldung | Ursache und Lösung |
|---|---|
| *Konfiguration ungültig* oder eine Automation/ein Skript „konnte nicht validiert werden“ | Den Fehler unter *Entwicklerwerkzeuge → YAML → Konfiguration prüfen* lesen. Häufig: Kopierfehler in den Paketdateien oder ein Selektor als `{}` statt leer (`date:` statt `date: {}`). |
| Entitäts-IDs der Skripte weichen ab (z. B. `script.vwt_record_charging_session`) | Home Assistant hat sich eine ID aus einem ersten, ungültigen Laden gemerkt. Die Entität unter *Einstellungen → Entitäten* in `script.vwt_record_charge` / `script.vwt_record_trip` umbenennen. |
| `sensor.vwt_…` existiert nicht | Der *Name* eines Template-Sensors oder Utility-Meters wurde geändert. In YAML kommt die Entitäts-ID aus dem Namen; den ursprünglichen Namen wiederherstellen. |
| Dashboard zeigt „Entität nicht verfügbar“ oder „Custom element doesn't exist“ | Fehlende HACS-Karten (*Mushroom*, *card-mod*, *ApexCharts card*, *Plotly graph card*) oder die Adapter-Sensoren zeigen auf falsche evcc-Entitäts-IDs ([Anleitung 4](04-home-assistant.md), Schritt 4). |
| Reparatur: *„Die Einheit von sensor.xyz hat sich geändert“* (Langzeit-Statistik) | Eine Statistik ohne Einheit, typischerweise nach einem Import von Verlaufsdaten. Unter *Entwicklerwerkzeuge → Statistiken* gibt es evtl. *Problem beheben*; fehlt die Einheit nur, müssen die Statistik-Metadaten angepasst werden (der Autor nutzte den WebSocket-Befehl `recorder/update_statistics_metadata`; **ungeprüft**, ob das auch in der Oberfläche geht). Vorher immer ein Backup anlegen. |

## evcc

| Symptom | Ursache und Lösung |
|---|---|
| Der Modus des Ladepunkts ist nicht *off* | Er kann in der evcc-Oberfläche versehentlich verstellt werden (bei uns stand er einmal auf *smart*). Wieder auf **off** stellen (Entität `select.evcc_…_mode` oder evcc-Oberfläche). Dieses Setup darf nie laden. |
| Einstellungen wirken nicht | Nach dem Speichern zeigt evcc eine *Restart*-Leiste; drücken. |
| Fahrzeug zeigt keine Daten, nachdem die Portal-Anfrage gelöscht und neu angelegt wurde | Das Fahrzeug in evcc öffnen und *Prüfen* drücken (**ungeprüft**). Prüfen, ob die Anfrage im Portal aktiv ist und die erste Datei erzeugt wurde. |
| `dsg ERROR … not available` in den ersten Minuten | Normal, bis die erste Datendatei existiert. |

## Telegram

Siehe die Tabelle am Ende von [Anleitung 3](03-telegram.md). Kurz: Integration geladen? Deine Chat-ID als *Allowed chat ID* eingetragen? `input_text.vwt_notify_entity` auf deine Notify-Entität gesetzt? Automationen aktiv (Ablaufprotokolle ansehen)?

## Wo du nachsehen kannst

- **Home Assistant:** *Einstellungen → System → Protokolle*; *Entwicklerwerkzeuge → Ereignisse/Zustände*; das **Ablaufprotokoll** einer Automation (*Einstellungen → Automationen → … → Abläufe*).
- **evcc:** das Protokoll der App; die lokale API `http://<evcc>:7070/api/state` (nur lesen).
- **VW-Portal:** Fahrzeugdetails → *Ihre Dateien* (Häufigkeit, nächste Datei).

## Ein Problem melden

Bitte angeben: Home-Assistant-Version, evcc-Version, wie beides installiert ist, was du erwartet hast, was passiert ist, und die betreffenden Protokollzeilen. **Entferne** vor dem Posten E-Mail-Adressen, FINs, Chat-IDs und Tokens.
