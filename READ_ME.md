# PS5 Stock Bot (Canada)

Checks **Best Buy, Amazon.ca, Walmart.ca, Costco.ca and EB Games** for the
**PS5 Pro, PS5 Slim and PS5 Slim Digital** about once a minute, and emails you
the moment one is in stock **sold by the store itself**. Marketplace and
third-party sellers (the scalper listings on Amazon and Walmart) are ignored.

It runs free on GitHub Actions, so your computer doesn't need to be on.

---

## Setup (about 2 minutes, no accounts, nothing to pay)

### 1. Get the alerts on your phone
1. Install the free **ntfy** app ([Android](https://play.google.com/store/apps/details?id=io.heckel.ntfy) /
   [iPhone](https://apps.apple.com/app/ntfy/id1625396347)). No sign-up needed.
2. Tap **+** (Subscribe to topic) and enter exactly: `ps5-alerts-0f701b06d64b`
3. In the app, allow notifications. Stock alerts are sent as *urgent* so they get through.

That's it - the topic name is already in `.github/workflows/check-stock.yml`.

> This repo is public, so anyone reading the workflow can see that topic name. The worst they
> could do is see the same alerts or send you junk pushes. To keep it private, make up a new name,
> add it as a repository secret called `NTFY_TOPIC` (Settings → Secrets and variables → Actions),
> and subscribe to that name in the app instead.

### 2. (Optional) Email too
Email alerts still work if you want them as well: add the secrets `SMTP_USER` (Gmail address),
`SMTP_PASS` (a Gmail [app password](https://myaccount.google.com/apppasswords), not your normal one)
and `EMAIL_TO`. Skip this if the phone push is enough.

### 3. Code on GitHub
The code and `.github/workflows/check-stock.yml` must be on the `main` branch.

### 4. Turn it on and test it
1. Go to the **Actions** tab → enable workflows if asked.
2. Click **PS5 stock check → Run workflow**, tick **Just send a test alert**, Run.
   Your phone should buzz within a minute. If nothing is set up, the run turns red.
3. Run it again **without** the box ticked. Open the run → the **Summary** shows a
   table of every store/console and what the bot saw.

After that it runs on its own every 5 minutes. Done!

---

## What the statuses mean (in the run Summary)

| Status | Meaning |
|---|---|
| 🟢 IN_STOCK | Sold by the store itself — you got an alert |
| ⚪ OUT_OF_STOCK | Not available |
| 🟡 THIRD_PARTY | Only a marketplace seller has it — ignored on purpose |
| 🟠 UNKNOWN | Page loaded but the bot couldn't tell (e.g. Walmart in stock but seller unclear — no alert, to be safe) |
| 🔴 BLOCKED / ERROR | The store blocked the check or the page changed |

You only get **one alert per restock** (not one every minute). If a store blocks
the bot for ~5 hours straight, you'll get one heads-up alert about it.

## Things to know

- **Some stores fight bots.** Amazon, Walmart and Costco sometimes block requests
  coming from GitHub's servers. The bot uses a browser-like connection to reduce this,
  but if a store shows 🔴 constantly, see "Running on your own computer" below — a home
  internet connection gets blocked much less.
- **GitHub schedules can lag** a few minutes when GitHub is busy. Each run checks
  about 4 times to make up for it.
- **GitHub pauses scheduled workflows after 60 days with no repo activity.** If that
  happens you'll get an email from GitHub — just click to re-enable.
- **Costco** needs a membership to buy online.
- **PlayStation Direct** doesn't sell to Canada, and **The Source** is now Best Buy, so
  neither is included.
- This bot only **alerts** you — it doesn't buy anything. Be ready to check out fast
  (stay logged in to each store, payment saved).

## Add or remove consoles

Edit `products.yaml` on GitHub (click the file → pencil icon). Each entry is just a
store, a name and a link. Best Buy needs the "Web Code" from the product page, Amazon
the ASIN (the `B0…` code in the URL). Delete an entry, or add `enabled: false`, to stop watching it.

## Optional settings

Add these under **Settings → Secrets and variables → Actions → Variables tab**:

- `REMIND_HOURS` — re-email you every N hours while something stays in stock (default 0 = off)
- `HEALTH_ALERT_AFTER` — how many failed rounds before the "store is blocking" email (default 60)

## Running on your own computer (optional)

```bash
pip install -r requirements.txt
export SMTP_USER=you@gmail.com SMTP_PASS=yourapppassword EMAIL_TO=you@gmail.com
python bot.py --test-email          # check email works
python bot.py --dry-run             # one check, print results, no email
python bot.py --loop-minutes 600    # keep checking every minute for 10 hours
```

Run the tests with `pip install pytest && pytest -q`.
