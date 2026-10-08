# 2. evcc: Installation und Konfiguration

🇬🇧 [English](../en/02-evcc.md)

evcc holt die Daten des Autos aus dem VW EU Data Act Portal und stellt sie Home Assistant bereit. In diesem Projekt ist evcc **nur Datenquelle** – es gibt keine Wallbox.

> **Grundlage dieser Anleitung:** evcc 0.316 als App (Add-on) auf Home Assistant OS, konfiguriert über die Weboberfläche (ohne `evcc.yaml`). Mit **(ungeprüft)** markierte Punkte sind Annahmen. Poste niemals dein VW-Passwort oder deine E-Mail-Adresse in Issues oder Screenshots.

## 1. evcc installieren

- **Home Assistant OS / Supervised:** die evcc-App aus dem App-Store installieren. Die hier genutzte App stammt aus einem Community-Repository; das aktuelle Repository steht in der evcc-Dokumentation (**ungeprüft:** genaue Repository-Adresse).
- **Andere Systeme:** evcc nach der [evcc-Dokumentation](https://docs.evcc.io) installieren (Docker, Linux-Paket, …). Alles Weitere geschieht in der evcc-Weboberfläche und ist gleich.

Beim ersten Start ohne Konfigurationsdatei läuft evcc im *Datenbank-Modus*: Die gesamte Einrichtung erfolgt in der Weboberfläche (**Mehr → Konfiguration**). Ohne Ladepunkt meldet evcc „meter-only mode“ – das ist bis Schritt 3 normal.

## 2. Fahrzeug anlegen

Voraussetzung: Die Datenanfrage im Portal ist angelegt ([Anleitung 1](01-vw-eu-data-act-portal.md)). Der Hilfetext von evcc sagt dasselbe: Ohne aktivierte laufende Datenanfrage liefert das Portal keine Daten.

1. **Mehr → Konfiguration → Fahrzeuge → Fahrzeug hinzufügen.**
2. **Hersteller:** *Volkswagen EU Data Act* (gilt für e-Golf, e-Up, ID-Familie).
3. **Titel:** z. B. `ID.7`.
4. **Benutzername / Passwort:** die Zugangsdaten deines Volkswagen-Kontos (dasselbe wie für die Volkswagen-App / We Connect und das Portal).
5. **Fahrgestellnummer:** optional; nur nötig, wenn du mehrere Fahrzeuge desselben Herstellers hast.
6. **Batteriekapazität:** optional. Wir haben **91 kWh** eingetragen (der Wert, den evcc für den ID.7 Pro S zeigt; es ist die Bruttokapazität, **ungeprüft**). Home Assistant nutzt für die Verbrauchsberechnung eine eigene Einstellung mit der Nettokapazität.
7. **Prüfen und speichern.** Schlägt die Prüfung fehl, Datenanfrage im Portal und Zugangsdaten kontrollieren.

Die Fahrzeugkarte in evcc zeigt *Ladestand*, *Kilometerstand*, *Reichweite* und *Ladelimit*, sobald die erste Datei angekommen ist. Direkt nach dem Anlegen der Datenanfrage kann das bis zu einem Lieferintervall dauern.

## 3. Virtuellen Ladepunkt anlegen

evcc braucht einen Ladepunkt, um das Fahrzeug laufend abzufragen. Wir legen eine **Demo-Wallbox an, die nie lädt**.

1. **Mehr → Konfiguration → Ladepunkte & Heizgeräte → Ladepunkt hinzufügen.**
2. **Ladegerät:** *Demo charger*. **Titel:** z. B. `ID.7 (virtuell)`.
3. **Standardmodus: Aus.** Der Modus muss auf **Aus** bleiben.
4. **Phasen / Strom:** beliebige plausible Werte (bei uns 3-phasig, 6 A bis 16 A). Sie werden nie benutzt.
5. **Standardfahrzeug:** dein Fahrzeug (schaltet die automatische Erkennung ab).
6. **Update behaviour: always**, **Update interval: 15 Minuten.**
7. **Speichern**, danach **Restart** drücken, sobald evcc dazu auffordert.

> **Warnung – niemals einen echten Ladegerätetyp verwenden.** Insbesondere keinen Typ, der das Fahrzeug über die Hersteller-API steuern kann („Vehicle API-only charger“ und ähnliche). So ein Typ könnte z. B. eine laufende Ladung an einer öffentlichen Säule stoppen. Die Demo-Wallbox kann das nicht.

### Zur Einstellung „Update behaviour“

| Einstellung | evcc fragt das Fahrzeug ab … | Funktioniert hier? |
|---|---|---|
| charging (Standard) | nur während des Ladens | **Nein** – das Auto lädt nie über evcc |
| connected | nur wenn angesteckt | **Nein** – es ist nie angesteckt |
| always | im eingestellten Intervall | **Ja** |

evcc zeigt bei *always* eine rote Warnung (Fahrzeugakku kann entleert werden, manche Hersteller verhindern dann das Laden, Missbrauch der API). In diesem Setup stammen die Daten aus der Cloud-Lieferung des VW-Portals und nicht daraus, das Auto zu wecken, aber **wir haben nicht geprüft, was auf Fahrzeugseite genau passiert** – Nutzung auf eigene Gefahr, bei Unsicherheit zurück auf *charging* stellen.

### Intervall wählen

Das Portal liefert alle 15 Minuten. Häufigeres Abfragen bringt keine neuen Daten, seltenes verlängert die Verzögerung. **15 Minuten** ist der sinnvolle Wert.

## 4. Funktion prüfen

- evcc-Weboberfläche → die Fahrzeugkarte zeigt Werte, der Ladepunkt steht auf *nicht verbunden* im Modus *Aus*.
- Lokale API (nur lesen): `http://<evcc>:7070/api/state` → `loadpoints[0]` enthält `vehicleSoc`, `vehicleRange`, `vehicleOdometer` und `mode: "off"`.
- Das evcc-Protokoll enthält nach einem Neustart eine Zeile wie `poll mode '{always 15m0s}' may deplete your battery …` – das ist die erwartete Warnung, kein Fehler.

## Bekannte Eigenheiten

- **Die ersten Daten brauchen Zeit** nach dem Anlegen der Datenanfrage. Protokollzeilen wie `dsg ERROR … not available` sind in dieser Zeit normal.
- **Kurze `0`-Werte:** Nach einem evcc-Neustart oder einer Lieferlücke können Akku- oder Kilometerstand kurz `0` sein. Automationen in Home Assistant sollten das ignorieren (unsere tun es).
- **Der Modus kann versehentlich** in der evcc-Oberfläche geändert werden (bei uns stand der Ladepunkt einmal auf *smart*). Wer eine Home-Assistant-Integration nutzt, sollte die Modus-Entität im Blick behalten; sie muss *off* sein.
- **Neustart nach Änderungen:** Nach dem Speichern der Ladepunkt-Einstellungen zeigt evcc eine *Restart*-Leiste. Die neuen Einstellungen gelten erst danach.
- **Anfrage im Portal ändern** (löschen und neu anlegen) kann ändern, was evcc abholt. Kommen danach keine Daten mehr, das Fahrzeug in evcc öffnen und *Prüfen* drücken (**ungeprüft**, ob das nötig ist).

## Weiter

[3. Telegram-Bot](03-telegram.md) *(geplant)* · [4. Home Assistant](04-home-assistant.md) *(geplant)*
