# 5. Troubleshooting

🇩🇪 [Deutsch](../de/05-troubleshooting.md)

Everything here was seen in the original setup (VW ID.7, evcc 0.316, Home Assistant 2026.9). Items marked **(unverified)** are assumptions.

## The data in Home Assistant is old or does not change

Work through the chain from the car to Home Assistant. The first step that shows old values is the culprit.

| # | Check | If it is old / wrong |
|---|---|---|
| 1 | **Your car's own app** (e.g. We Connect): current battery level and odometer? | If the app is also old, VW has no newer data yet. Wait; nothing in this project can speed it up. |
| 2 | **VW EU Data Act portal** → vehicle details: the request exists, shows the frequency you want and a *next file* date ([guide 1](01-vw-eu-data-act-portal.md)) | Request missing or cancelled (files must be retrieved within 7 days): create it again. **Daily** delivery = up to a day of delay: choose a shorter frequency (we use 15 minutes). |
| 3 | **evcc**: charge point → *Update behaviour* must be **always** (interval 15 min) ([guide 2](02-evcc.md)) | With *charging* (default) or *connected* evcc never polls a parked car that is not plugged in. After saving, press **Restart**. |
| 4 | **evcc values**: open `http://<evcc>:7070/api/state` and look at `loadpoints[0].vehicleSoc`, `vehicleOdometer` | If these are old, the problem is before Home Assistant (steps 1–3, or the evcc log: lines `dsg ERROR` with *timeout* mean the VW service was slow or unreachable). |
| 5 | **Home Assistant**: *Developer tools → States*, search `vehicle_soc` | If evcc has the new value but Home Assistant does not: reload the evcc integration, keep **websocket** on, check the integration's log. |

Normal delay: new values arrive roughly **15 to 30 minutes** after the car reported them (delivery interval plus evcc polling interval). There is no push from VW.

## Battery level or odometer is 0 for a moment

After an evcc restart or a gap in VW's delivery, evcc can briefly report `0`.

- The **odometer adapter** (`sensor.vwt_odometer`) ignores 0 (it becomes *unavailable* until a real value arrives); the automatic trip detection triggers again on the next real value.
- The **charge reminder** ignores a previous battery level of 0 %, otherwise "0 % → 55 %" would look like a charge.
- If you see a false "Charging detected" message anyway, check that your copy of `vwt_automations.yaml` contains the condition `trigger.from_state.state | float(0) > 0`.

## The automatic trip has the wrong battery values (e.g. "55 % → 55 %", 0 kWh)

The reference value `input_number.vwt_last_soc` was overwritten before the trip was recorded (for example by a false charge detection). Fix it:

1. Create the correct entry with `script.vwt_record_trip` (*Developer tools → Actions*): name, km, kWh (battery drop in % × net capacity / 100), source `automatic`.
2. Delete the wrong entry: *Logs* tab → *Correct entry* → *Delete*.

A tiny battery drop (1–2 %) gives a very rough consumption figure; it evens out over many trips.

## Totals or monthly values are wrong

- **Never edit log entries by hand.** The correction script reads the values from the entry text.
- **Monthly/yearly meters** start from the first value they see and show *unknown* until their source changes the first time.
- **Back-dated entries** (a `date` in the past) count in the totals but not in the current month/year meters. If you have older bookings that shifted them, correct the meters with *Developer tools → Actions → `utility_meter.calibrate`*.
- **After a manual fix** of a total, recalculate: the helpers `input_number.vwt_total_*` hold the totals and can be set directly.

## Home Assistant complains

| Message | Cause and fix |
|---|---|
| *Configuration invalid* or an automation/script "could not be validated" | Read the error in *Developer tools → YAML → Check configuration*. Common cause: a copy-paste error in the package files, or a selector written as `{}` instead of an empty value (`date:` not `date: {}`). |
| Entity IDs of scripts differ (for example `script.vwt_record_charging_session`) | Home Assistant remembered an ID from a first, invalid load. Rename the entity in *Settings → Entities* to `script.vwt_record_charge` / `script.vwt_record_trip`. |
| `sensor.vwt_…` does not exist | The *name* of a template sensor or utility meter was changed. In YAML the entity ID comes from the name; restore the original name. |
| Dashboard shows "Entity not available" or "Custom element doesn't exist" | Missing HACS cards (*Mushroom*, *card-mod*, *ApexCharts card*, *Plotly graph card*) or the adapter sensors point to wrong evcc entity IDs ([guide 4](04-home-assistant.md), step 4). |
| Repair: *"The unit of sensor.xyz has changed"* (long-term statistics) | A statistics series without unit, typically after importing history. *Developer tools → Statistics* may offer *Fix issue*; if the unit is simply missing, the statistics metadata has to be updated (the author used the WebSocket command `recorder/update_statistics_metadata`; **unverified** whether the UI can do it). Always make a backup first. |

## evcc

| Symptom | Cause and fix |
|---|---|
| Charge point mode is not *off* | It can be changed by accident in the evcc web interface (we found it on *smart* once). Set it back to **off** (the entity `select.evcc_…_mode` or the evcc UI). This setup must never charge. |
| Settings do not apply | evcc shows a *Restart* bar after saving; press it. |
| Vehicle shows no data after the portal request was deleted and re-created | Open the vehicle in evcc and press *validate* (**unverified**). Check the portal request is active and the first file has been created. |
| `dsg ERROR … not available` in the first minutes | Normal until the first data file exists. |

## Telegram

See the table at the end of [guide 3](03-telegram.md). In short: integration loaded? Your chat ID in *allowed chat IDs*? `input_text.vwt_notify_entity` set to your notify entity? Automations enabled (check their traces)?

## Where to look

- **Home Assistant:** *Settings → System → Logs*; *Developer tools → Events/States*; the **trace** of an automation (*Settings → Automations → … → Traces*).
- **evcc:** the app's log; the local API `http://<evcc>:7070/api/state` (read-only).
- **VW portal:** vehicle details → *Your files* (frequency, next file).

## Reporting a problem

Please include: Home Assistant version, evcc version, how you installed both, what you expected and what happened, and the relevant log lines. **Remove** e-mail addresses, VINs, chat IDs and tokens before posting.
