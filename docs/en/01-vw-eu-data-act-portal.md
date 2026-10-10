# 1. VW EU Data Act portal: request your car's data

🇩🇪 [Deutsch](../de/01-vw-eu-data-act-portal.md)

Under the EU Data Act you can request the data your car generates. In this project the portal delivers the battery level, range and odometer to **evcc**, which hands them to Home Assistant.

> **What this guide is based on:** the portal as seen in October 2026 (German user interface, private customer with a VW ID.7). Menu names may differ in your market, language or brand. Items marked **(unverified)** are assumptions – please report corrections.

## What you need

- A Volkswagen Group account that is linked to your car (the account shown as "Primary user" of the vehicle)
- The vehicle must appear in the portal (see [the portal FAQ](https://eu-data-act.drivesomethinggreater.com/de/en/service/faq.html), entry "Why is my vehicle not visible in the EU Data Act Portal?")

## Steps

1. Open the portal: <https://eu-data-act.drivesomethinggreater.com> and sign in with your Volkswagen account.
2. Open **Vehicle overview** (*Fahrzeugübersicht*) and choose your car (its VIN is shown). Open **Vehicle details** (*Fahrzeugdetails*).
3. On the details page you will find three request types:
   - **Retrieve data** – a one-off file with historical data within 24 hours. Not what we need.
   - **Retrieve custom data** (*Benutzerdefinierte Daten abrufen*) – continuous delivery of data clusters you choose, at a frequency you choose. **This is the one.**
   - **Share your data** – hand data to a third party. Not needed.
4. Choose **Request custom data** (*Benutzerdefinierte Daten anfragen*) and configure it:
   - **Name:** anything you like (for example "HA and evcc").
   - **Data clusters:** *All Data*. (A smaller selection may work too; whether it contains everything evcc needs is **unverified**.)
   - **Frequency:** choose the shortest available. This project uses **every 15 minutes**. The available options are shown in the form; the default we first saw was **daily**.
5. Submit. Your request now appears under **Your files** with its frequency and the date of the next file.

## Rules you must know

- **Only one custom request can be active at a time.** To change frequency or clusters you have to delete the running request (*Delete data package*) and create a new one. Doing so may change the identifier that evcc uses to fetch the data (**unverified**) – check evcc afterwards (see guide 2).
- **Files are kept for 7 days at most** and must be **retrieved within 7 days after creation**, otherwise the request is cancelled. evcc retrieves them continuously; if evcc and Home Assistant are offline for more than a week, check the request.
- A package contains at most **30 files**.
- **Maintenance windows:** from time to time the portal blocks the *creation* of new requests (existing deliveries keep working). A notice is shown on the portal's start page.
- A **data dictionary** (PDF) listing all data points per cluster can be downloaded from the portal. It tells you which keys exist (for example `state_of_charge`); it does not tell you how often a value is updated.

## What to expect

- With **daily** delivery, the battery level in Home Assistant can be up to a day behind the car. In our case this was the reason the dashboard did not update after a drive. With **every 15 minutes** new values arrive within roughly 15 to 30 minutes after the car reported them.
- There is no push notification from the portal: evcc polls. The lag therefore is delivery interval (portal) plus polling interval (evcc).
- Right after a restart of evcc or a gap in delivery, evcc may briefly report `0` for battery level or odometer. Our automations ignore a battery level of 0 % as a starting value.

## What a data file contains

With the cluster *All Data* a file for an ID.7 held about 100 entries (`key`, `dataFieldName`, `value`). It contains no location and no speed. Seen in October 2026:

| Topic | Examples |
|---|---|
| Battery, range | `battery_state_report.soc`, `battery_level_HV.value`, `value` (probably range; **unverified**), `energy_contents.*` (unit unclear) |
| Odometer | `mileage.value`, `mileage.state` |
| Charging | `charging_state_report.*` (mode, state, scenario), `battery_state_report.charge_power`, `settings.target_soc`, `battery_care_mode.charge_bcam_threshold`, `settings.max_charge_current_ac` |
| Climate | `climatisation_state`, `climatisation_settings.*` (target temperature, zones, heating), `remaining_climate_time` |
| Car state | `locked`, `open`, `parking_brake`, `parking_light_*` |
| Temperatures | `outdoor_temperature`, `min_temperature`, `max_temperature` |
| Meta | `car_captured_time`, `timestamp`, `update_reason`, `report_type`, `error_code` |

evcc passes only battery level, range, odometer and charge limit on to Home Assistant. The other fields are not available there. The files carry the VIN and a user ID: never post them.

## Next

[2. evcc: install and configure](02-evcc.md)
