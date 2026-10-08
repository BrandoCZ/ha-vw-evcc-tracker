# 4. Home Assistant: helpers, scripts, automations, dashboard

🇩🇪 [Deutsch](../de/04-home-assistant.md)

This guide installs everything inside Home Assistant. You need [guide 2 (evcc)](02-evcc.md) running first; the Telegram part ([guide 3](03-telegram.md)) is optional.

> **Tested with** Home Assistant 2026.9 on a fresh container installation, following the steps below (the UI steps were done through the equivalent API calls): packages, to-do lists, configuration check, restart, setting values, saving the dashboard, recording a charging session and deleting it again (all totals returned to 0). The evcc integration and the cards were installed by hand, **not through HACS** (unverified). The original German version has been running on Home Assistant OS for only a few days so far. **Not tested:** the Telegram messages and buttons on a fresh instance (no bot there), the 30-day values on a fresh instance (they need a first hour of statistics), and the buttons of the dashboard. Items marked **(unverified)** are assumptions.

## What gets installed

| File | Contents |
|---|---|
| `packages/vwt_helpers.yaml` | input helpers, counter, adapter sensors for evcc, calculated sensors, utility meters |
| `packages/vwt_scripts.yaml` | scripts to record a charging session / a trip, apply corrections, delete an entry |
| `packages/vwt_automations.yaml` | automatic trip detection, charge reminder, 30-day values, correction list, Telegram commands and dialog |
| `dashboards/vwt-dashboard.yaml` | dashboard with the tabs *Now*, *Analysis* and *Logs* |

All entities start with `vwt_`. Nothing contains personal data.

## 1. Prerequisites

1. **evcc integration for Home Assistant** (`marq24/ha-evcc`, a custom integration). Install it with HACS (search for "evcc") or manually, then add it under *Settings → Devices & services*. Enter the address of your evcc (for example `http://homeassistant.local:7070`). **Restart Home Assistant after installing the custom integration** (before you can add it). Keep **websocket** switched on: it makes new values arrive in Home Assistant practically at once.
2. **HACS cards** for the dashboard: *Mushroom*, *card-mod*, *ApexCharts card*, *Plotly graph card*.
3. **Telegram bot** (optional): see [guide 3](03-telegram.md).

## 2. Find your evcc entity IDs

The entity IDs depend on the **title of your evcc charge point**. Open *Developer tools → States* and search for `vehicle_soc`. With the charge point title `ID.7 (virtual)` the entities are:

```
sensor.evcc_id_7_virtual_vehicle_soc
sensor.evcc_id_7_virtual_vehicle_range
sensor.evcc_id_7_virtual_vehicle_odometer
```

Yours may differ (title, language: "virtuell" vs "virtual").

## 3. Enable packages

Add this to `configuration.yaml` (if you already have a `homeassistant:` section, add only the `packages` line):

```yaml
homeassistant:
  packages: !include_dir_named packages
```

Create the folder `packages` next to `configuration.yaml` and copy the three files from this repository's `packages/` folder into it. (Use the File editor app, Samba or SSH.)

## 4. Adapt the three evcc entity IDs

Open `packages/vwt_helpers.yaml`, find the section **ADAPTER SENSORS** and replace `evcc_id_7_virtual` by the prefix of your own entities (a search-and-replace over the whole file is enough, six places). This is the only place that knows your evcc names; everything else uses `sensor.vwt_soc`, `sensor.vwt_range` and `sensor.vwt_odometer`.

> **Do not rename** the sensors or their `name:` lines. In YAML, Home Assistant derives the entity ID from the *name*; the scripts, automations and dashboard rely on the IDs `sensor.vwt_…`.

## 5. Create the two to-do lists

*Settings → Devices & services → Add integration → Local To-do.* Create two lists with exactly these names:

- `VWT Charging log` → entity `todo.vwt_charging_log`
- `VWT Trip log` → entity `todo.vwt_trip_log`

## 6. Check the configuration and restart

*Developer tools → YAML → Check configuration*, then **restart Home Assistant**. Afterwards *Developer tools → States*, filter `vwt_`: you should see the helpers, the sensors `sensor.vwt_soc`, `sensor.vwt_range`, `sensor.vwt_odometer` (with your car's values), the scripts and six automations.

## 7. Set your values

In *Developer tools → States* or on a temporary dashboard card, set:

| Entity | Value |
|---|---|
| `input_number.vwt_battery_net_kwh` | **usable (net)** battery capacity of your car in kWh, for the consumption calculation (the ID.7 Pro S has 86 kWh net) |
| `input_number.vwt_favorite_price` | your usual price per kWh at your regular charger (used for the "⭐ Favorite" button in Telegram) |
| `input_text.vwt_notify_entity` | entity ID of your Telegram notify entity (only if you use Telegram) |

The reference values for the automatic trip detection (`vwt_last_odometer`, `vwt_last_soc`) fill themselves with the first odometer change.

## 8. Install the dashboard

1. *Settings → Dashboards → Add dashboard →* choose **New dashboard from scratch**, title `EV tracker`, **URL `vwt-dashboard`** (the buttons on the dashboard navigate to this URL).
2. Open the new dashboard, click the pencil (edit) → three dots → **Raw configuration editor**.
3. Delete the content, paste the complete content of `dashboards/vwt-dashboard.yaml`, save.

## 9. Test it

1. **Record a charging session:** *Logs* tab → *Charging* button (or call `script.vwt_record_charge` in *Developer tools → Actions*) with 40 kWh and 20 €. The charging log gets an entry; the totals and *This month* values rise.
2. **Undo:** in *Logs*, choose the entry under *Correct entry* → *Delete*. The totals return to their old values.
3. **Trips:** drive a few kilometers. Within about 15 to 30 minutes after the car reported the new odometer, a trip entry should appear automatically in the trip log with consumption derived from the battery drop (a very short trip gives a rough consumption: 1 % battery is already about 0.9 kWh).
4. **Telegram** (if used): `/help`, then the buttons.

## Good to know

- **Format of the log entries:** `2026-09-13 · Aral · 44.78 kWh · 25.52 € · 0.57 €/kWh`. The correction script reads the values back from this text, so do not edit entries by hand.
- **Back-dated entries:** the scripts accept a `date`. Such an entry counts in the totals but not in the *current* month or year meters.
- **Utility meters show "unknown"** until their source changes for the first time. That is normal.
- **30-day values** are calculated from the long-term statistics; on a fresh installation they stay at 0 until the first hourly statistics exist.
- **Entity names of automations** come from their aliases (for example `automation.vwt_telegram_commands`).
- **Short zero readings:** evcc may report `0` for the battery level or odometer for a moment after a restart. The odometer adapter ignores 0; the charge reminder ignores a previous battery level of 0 %.
- **Language:** all names, messages and the dashboard are English. For another language, edit the texts in the package files and the dashboard.

## Update / remove

- **Update:** replace the files in `packages/`, check the configuration, restart. Your totals and logs are kept (they live in the helpers and the to-do lists).
- **Remove:** delete the files in `packages/`, remove the two to-do lists and the dashboard, restart.

## Next

[5. Troubleshooting](05-troubleshooting.md)
