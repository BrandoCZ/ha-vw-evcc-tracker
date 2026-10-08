# 3. Telegram bot

🇩🇪 [Deutsch](../de/03-telegram.md)

The Telegram bot lets you enter charging sessions and trips from your phone – **without VPN and without opening your Home Assistant to the internet**. Home Assistant asks Telegram for new messages ("polling"); nothing connects to your home network from outside.

> **What this guide is based on:** Home Assistant 2026.9 with the built-in *Telegram bot* integration. Items marked **(unverified)** are assumptions. **Treat the bot token like a password** – never post it, and never commit it.

## 1. Create the bot

1. In Telegram, open the chat with **@BotFather**.
2. Send `/newbot`, choose a name and a user name (must end in `bot`).
3. BotFather answers with the **API token**. Keep it private.
4. Optional, but recommended – give your bot a command menu with `/setcommands` (choose your bot, then send):
   ```
   charge - Record a charging session
   trip - Record a trip
   status - Battery level and last 30 days
   help - Help and keyboard
   ```
   The command names are the ones used by the automations in this project (`/charge`, `/trip`, `/help`, `/status`). You can rename them later in the automation; the descriptions are yours to translate.

## 2. Add the integration to Home Assistant

1. **Settings → Devices & services → Add integration → Telegram bot.**
2. Enter the API token. Choose the **polling** platform (the alternative, webhooks, needs a publicly reachable HTTPS address – not needed here). **(unverified:** exact wording of the setup screens.)
3. In the integration's options, the **parse mode** can be set. This project uses *markdown*.
4. **Allowed chat IDs:** the bot only reacts to chats you allow. Add your own numeric Telegram chat ID as an *allowed chat ID* entry of the integration. How to find your ID: send a message to a bot such as *@userinfobot*, or look at the Home Assistant log after sending your bot a message **(unverified)**. Without an allowed chat ID, messages from that chat are ignored and no notification entity exists.

What you get after that:

| What | Example | Used for |
|---|---|---|
| Notify entity per allowed chat | `notify.<bot>_<chat title>` | Sending messages to you (reminders, answers) |
| Event entity | `event.<bot>_update` | Shows the last update from Telegram |
| Events on the Home Assistant event bus | `telegram_command`, `telegram_text`, `telegram_callback` | What the automations listen to |

Entity names depend on what you called your bot and the chat. The automations in [guide 4](04-home-assistant.md) use a neutral example name (`notify.id7_telegram_example`) – replace it with the entity ID of your own notify entity.

## 3. What the bot can do

Handled by two automations (see guide 4):

**Direct commands**

| Command | Meaning |
|---|---|
| `/charge <kWh> <price> [provider] [note]` | Record a charging session. The price is either the **total** (`25.52`) or **per kWh** (`0.57/kWh`). |
| `/trip <km> <kWh/100km> [name]` | Record a trip. |
| `/status` | Battery level and the last 30 days. |
| `/help` or `/start` | Help text and the button keyboard. |

**Guided dialog with buttons** – send `/charge` or `/trip` **without values**, or press the keyboard buttons:

- **⚡ Charge:** kWh → provider → price type (total or per kWh) → amount. A quick button for your **favorite price** (`input_number.vwt_favorite_price`) and an **undo** button are included.
- **🛣 Trip:** km → consumption.
- **📊 Status.**

The current dialog step is kept in an input helper and expires after **30 minutes**. All texts in the published automations are **English**; to translate them, edit the messages in `packages/vwt_automations.yaml`. Decimals can be typed with a dot or a comma.

## 4. Reminders from Home Assistant

When the battery level rises by at least 5 %, Home Assistant sends you a message asking you to record the charging session, including a ready-to-copy `/charge` command with the estimated kWh. It ignores a previous value of 0 % (short data gaps after an evcc restart would otherwise look like a charge from 0 to 100 %).

## 5. Test

1. Send `/help` to your bot. You should get the help text and a keyboard.
2. Press **⚡ Charge** and follow the dialog, or send `/charge 40 20.00 Test`.
3. In Home Assistant, the charging log (to-do list) should contain a new entry.
4. Delete the test entry again (dashboard → *Logs* → select entry → delete) so that your totals stay correct.

## Troubleshooting

| Problem | Check |
|---|---|
| Bot does not answer | Is the integration loaded? Is your chat ID an *allowed chat ID*? Home Assistant log (`telegram_bot`). |
| "Unauthorized" or no updates | Token wrong or revoked: create a new one with `/token` at BotFather and update the integration. |
| Commands do nothing | Are the two automations enabled? Check their traces in Home Assistant. |
| Wrong notify entity name | Check the entity ID of your notify entity and adjust it in the automations. |

## Next

[4. Home Assistant](04-home-assistant.md) *(planned)*
