# ha-vw-evcc-tracker

🇬🇧 [English version](README.md)

Kilometerstand, Akkustand, Verbrauch und Ladevorgänge eines **Elektroautos aus dem Volkswagen-Konzern** (gebaut und getestet mit einem VW ID.7) in **Home Assistant** verfolgen – mit dem offiziellen **VW EU Data Act Portal** als Datenquelle und **evcc** als Brücke.

> **Status: in Arbeit, zum Feedback geteilt.** Das Projekt war ursprünglich ein privates Setup und wird zu einer Anleitung aufbereitet. Es ist ein Beispiel, kein fertiges Produkt: **kein Support, Nutzung auf eigene Gefahr.** Ideen, Korrekturen und Fragen sind sehr willkommen – bitte ein Issue oder eine Discussion eröffnen.

## Was es kann

- Liest Akkustand (SoC), Reichweite und Kilometerstand des Autos aus dem EU Data Act Portal von VW (Lieferung alle 15 Minuten) über evcc.
- Zeigt sie in Home Assistant an und berechnet Strecken und Verbrauch automatisch, sobald sich der Kilometerstand ändert.
- Führt Ladevorgänge (öffentliches Laden) und Strecken in zwei To-do-Listen, mit Monats- und Jahressummen, Kosten und 30-Tage-Ansicht.
- Ladevorgänge und Strecken lassen sich per **Telegram-Bot** (auch unterwegs, ohne VPN) oder per Dashboard-Knopf eintragen.
- Dashboard mit drei Reitern (Jetzt · Auswertung · Protokolle) für Handy, Tablet und Desktop, im hellen und dunklen Modus.

## Funktionsweise

```
VW EU Data Act Portal (Datenanfrage, alle 15 min)
        │
        ▼
      evcc  (virtueller Ladepunkt, Demo-Wallbox, Modus „Aus“)
        │
        ▼
 Home Assistant  ──►  Automationen / Skripte / Helfer  ──►  Dashboard
        ▲
        └── Telegram-Bot (Ladevorgänge und Strecken erfassen)
```

Wichtig: Dieses Setup hat **keine Wallbox**. evcc dient nur als Datenquelle für das Auto. Der virtuelle Ladepunkt muss eine *Demo-Wallbox im Modus „Aus“* bleiben – niemals einen Ladepunkt-Typ verwenden, der das Fahrzeug steuern kann.

## Anleitungen

| # | Anleitung | Stand |
|---|---|---|
| 1 | [VW EU Data Act Portal: Daten des Autos anfragen](docs/de/01-vw-eu-data-act-portal.md) · [EN](docs/en/01-vw-eu-data-act-portal.md) | Entwurf |
| 2 | [evcc: Installation und Konfiguration](docs/de/02-evcc.md) · [EN](docs/en/02-evcc.md) | Entwurf |
| 3 | [Telegram-Bot](docs/de/03-telegram.md) · [EN](docs/en/03-telegram.md) | Entwurf |
| 4 | [Home Assistant: Helfer, Skripte, Automationen, Dashboard](docs/de/04-home-assistant.md) · [EN](docs/en/04-home-assistant.md) | Entwurf |
| 5 | [Fehlersuche](docs/de/05-troubleshooting.md) · [EN](docs/en/05-troubleshooting.md) | Entwurf |

## Voraussetzungen

- Home Assistant (getestet mit 2026.9 auf Home Assistant OS)
- evcc (getestet mit 0.316)
- Ein Fahrzeug aus dem VW-Konzern mit Online-Verbindung und Zugang zum EU Data Act Portal
- HACS-Karten für das Dashboard: Mushroom, card-mod, ApexCharts-Card, Plotly-Graph-Card
- Optional: ein Telegram-Bot zum Erfassen unterwegs

## Lizenz

[MIT](LICENSE). Volkswagen, evcc, Home Assistant und Telegram sind Marken ihrer jeweiligen Inhaber; dieses Projekt steht mit keinem von ihnen in Verbindung.
