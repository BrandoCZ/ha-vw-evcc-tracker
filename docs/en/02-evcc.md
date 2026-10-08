# 2. evcc: install and configure

🇩🇪 [Deutsch](../de/02-evcc.md)

evcc fetches the car's data from the VW EU Data Act portal and makes it available to Home Assistant. In this project evcc is **only a data source** – there is no wallbox.

> **What this guide is based on:** evcc 0.316 running as an app (add-on) on Home Assistant OS, configured through the web interface (no `evcc.yaml`). Items marked **(unverified)** are assumptions. Never post your VW password or e-mail address in issues or screenshots.

## 1. Install evcc

- **Home Assistant OS / Supervised:** install the evcc app from the app store. The app used here comes from a community app repository; see the evcc documentation for the current repository (**unverified:** exact repository URL).
- **Other systems:** install evcc as described in the [evcc documentation](https://docs.evcc.io) (Docker, Linux package, …). Everything below is done in the evcc web interface and is the same.

On first start without a configuration file, evcc runs in *database-only mode*: you configure it entirely in the web interface (**More → Configuration**). Without a charge point it reports "meter-only mode" – that is normal until step 3.

## 2. Add the vehicle

Prerequisite: you have created the data request in the portal ([guide 1](01-vw-eu-data-act-portal.md)). evcc's own help text says the same: without the activated continuous data request the portal returns no data.

1. **More → Configuration → Vehicles → Add vehicle.**
2. **Manufacturer:** *Volkswagen EU Data Act* (covers e-Golf, e-Up, ID family).
3. **Title:** for example `ID.7`.
4. **Username / Password:** the credentials of your Volkswagen account (the one you also use for the Volkswagen app / We Connect and the portal).
5. **Vehicle Identification Number:** optional; only needed if you have several vehicles at the same manufacturer.
6. **Battery capacity:** optional. We entered **91 kWh** (the value evcc shows for the ID.7 Pro S; it is the gross capacity, **unverified**). Home Assistant uses its own net capacity setting for the consumption calculation.
7. **Validate & save.** If validation fails, check the data request in the portal and the credentials.

The vehicle card in evcc now shows *Charge*, *Odometer*, *Range* and *Charge limit* once the first data file has arrived. Right after the data request is created this can take up to the delivery interval.

## 3. Add a virtual charge point

evcc needs a charge point to keep polling the vehicle. We create a **demo charger that never charges**.

1. **More → Configuration → Charging points & heaters → Add charging point.**
2. **Charger:** *Demo charger*. **Title:** for example `ID.7 (virtuell)` / `ID.7 (virtual)`.
3. **Default mode: Off.** The mode must stay **Off**.
4. **Phases / current:** any plausible values (we used 3-phase, 6 A to 16 A). They are never used.
5. **Default vehicle:** your vehicle (this switches auto-detection off).
6. **Update behaviour: always**, **Update interval: 15 minutes.**
7. **Save**, then press **Restart** when evcc asks for it.

> **Warning – never use a real charger type here.** In particular do not choose a type that can control the vehicle through the manufacturer API ("Vehicle API-only charger"-style types). Such a type could, for example, stop a charging session at a public charger. The demo charger cannot.

### About "Update behaviour"

| Setting | evcc polls the vehicle … | Works here? |
|---|---|---|
| charging (default) | only while charging | **No** – the car never charges via evcc |
| connected | only while plugged in | **No** – it is never plugged in |
| always | at the configured interval | **Yes** |

evcc shows a red warning for *always* (the vehicle battery may drain, some manufacturers may prevent charging, API misuse). In this setup the data comes from the VW portal's cloud delivery, not from waking the car, but **we have not verified what exactly happens on the vehicle side** – use it at your own risk and switch back to *charging* if you are unsure.

### Choosing the interval

The portal delivers every 15 minutes. Polling more often than that brings no new data; polling less often adds delay. **15 minutes** is the sensible value.

## 4. Check that it works

- evcc web interface → the vehicle card shows values, and the charge point shows *not connected* in mode *off*.
- Local API (read-only): `http://<evcc>:7070/api/state` → `loadpoints[0]` contains `vehicleSoc`, `vehicleRange`, `vehicleOdometer`, `mode: "off"`.
- evcc log after a restart should contain a line like `poll mode '{always 15m0s}' may deplete your battery …` – that is the expected warning, not an error.

## Known quirks

- **First data takes a while** after creating the data request. Log lines such as `dsg ERROR … not available` in that time are normal.
- **Short `0` values:** after an evcc restart or a delivery gap the battery level or odometer can briefly be `0`. Home Assistant automations should ignore these (ours do).
- **The mode can be changed by accident** in the evcc web interface (we found the charge point on *smart* once). If you use a Home Assistant integration, watch the mode entity; it must be *off*.
- **Restart after settings changes:** evcc shows a *Restart* bar after saving charge point settings. The new settings apply only after that.
- **Changing the portal request** (delete and recreate) may change what evcc fetches. If data stops afterwards, open the vehicle in evcc and press *validate* (**unverified** whether this is needed).

## Next

[3. Telegram bot](03-telegram.md) · [4. Home Assistant](04-home-assistant.md)
