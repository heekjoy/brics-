# BRICS 24H Market Clock

A market24hclock-style 24-hour dial built around **your** day (Asia/Bangkok), with a BRICS /
geopolitics / markets news desk, market highlights and the Crypto Fear & Greed Index.

```
market24-brics/
├─ index.html                the site (single file, no build step, no dependencies)
├─ update.ps1                daily data refresh -> data/brief.js + data/brief.json
├─ refresh-and-open.cmd      double-click: refresh data, then open the site
├─ install-daily-update.cmd  double-click ONCE: refresh every morning at 07:15
├─ data/                     generated snapshot (safe to delete, regenerated each run)
└─ .github/workflows/        optional: refresh daily in the cloud
```

## Editing it without touching code

Click **⚙ Customize** in the header:

- **Daily Routine** — start time, name, colour and note per block. Each block runs until the next
  one starts, so the 24 hours always add up and the ring can never have a gap. Save and the dial,
  timeline and "Right Now" panel update instantly.
- **Flights** — one editable list *per month*, because Qatar Airways schedules shift between
  seasons. Pick the month, edit flight number, airports, times and operating days, or copy a month
  you already filled in.

Everything is saved in your browser (localStorage), so it survives reloads and doesn't need a
rebuild. "Reset to default" restores what's coded in `index.html`.

## Start here

Double-click **`refresh-and-open.cmd`**, or:

```bash
powershell -ExecutionPolicy Bypass -File update.ps1
```

then open `index.html`. It works straight from disk (`file://`) — no web server needed.

## What updates when

| Piece | Source | Refresh |
|---|---|---|
| Clock, schedule, exchange sessions | your browser | every second, DST handled automatically |
| Crypto Fear & Greed + 30-day history | alternative.me | live in the browser, every 5 min |
| Bitcoin / Ethereum price | CoinGecko | live in the browser, every 5 min |
| Indices, gold, Brent, DXY, US 10Y, BRICS FX | Yahoo Finance | each `update.ps1` run |
| BRICS / Geopolitics / Markets headlines | Google News, Al Jazeera, BBC World, Cointelegraph | in-browser every 30 min *when a public proxy answers*, otherwise each `update.ps1` run |
| Flight status | your saved schedule (or AeroDataBox with a key) | recalculated every minute |

Sentiment and crypto are fetched by the page itself (those APIs allow browser calls), so they stay
current even if you forget to run the updater. News and index quotes need `update.ps1`, because
news sites don't allow direct browser fetches.

## Run it automatically every day

**Easiest:** double-click **`install-daily-update.cmd`** once. It registers a 07:15 daily task and
runs it immediately. Remove it any time with `schtasks /Delete /TN "BRICS Clock Daily Update" /F`.

**Same thing by hand** — runs at 07:15 so the brief is ready at breakfast:

```bash
schtasks /Create /TN "BRICS Clock Update" /SC DAILY /ST 07:15 /TR "powershell -ExecutionPolicy Bypass -File \"C:\Users\DELL\Downloads\market24-brics\update.ps1\""
```

Remove it later with `schtasks /Delete /TN "BRICS Clock Update" /F`.

**Or host it free on GitHub Pages** — push this folder to a repo, enable Pages, and the included
workflow (`.github/workflows/daily-brief.yml`) refreshes the snapshot every day at 00:15 UTC and
commits it. Nothing to run on your machine.

## Putting it on Google Sites

Google Sites can't host HTML/JS files itself, and its "Embed code" box can't reach `data/brief.js`.
So host the folder somewhere that serves files, then embed that URL:

1. **Publish the folder** — GitHub Pages is the best fit because the included workflow keeps the
   news fresh without your PC being on. Push this folder to a repo, then
   *Settings → Pages → Deploy from a branch → `main` / `/ (root)`*. Set
   *Settings → Actions → General → Workflow permissions* to **Read and write** so the daily job can
   commit its snapshot. Your URL becomes `https://<user>.github.io/<repo>/`.
2. **Embed it** — in Google Sites: *Insert → Embed → By URL*, paste the URL, **Insert**, then drag
   the frame to the height you want.
3. **Use the view modes** so each embed is one screen instead of one long scroll:

   | URL | Shows |
   |---|---|
   | `.../?view=clock` | dial + Now/Next + Fear & Greed + timeline |
   | `.../?view=markets` | market cards + BRICS FX + session table |
   | `.../?view=flights` | the Qatar Airways watchlist |
   | `.../?view=news` | the three news desks |
   | `.../` | everything |

Put the clock embed in a full-width section at the top of the page and the news embed below it.

## Making it yours

Everything you'd want to change sits in one config block at the top of the `<script>` in `index.html`:

- **`SCHEDULE`** — your daily blocks. `s`/`e` are 24h Bangkok times, `hex` is the ring colour,
  `note` is the line shown under "Right Now". Blocks may cross midnight (Sleep does).
- **`SESSIONS`** — exchanges drawn as bands around the dial, in each venue's own local time.
- **`TZ`** — change if you move; the whole dial follows.

News desks live in the `$Feeds` array in `update.ps1` — each entry is a category (`brics`, `geo`,
`markets`) plus an RSS URL. `G-News 'your query when:2d'` builds a Google News search feed, so you
can target any topic. `-PerDesk 25` keeps more headlines per column.

## Notes

- Times on the dial are **Bangkok**; the small grey `08z` labels under each hour are the UTC
  equivalent, and every card in the top bar is a live world clock.
- Exchange sessions use regular cash hours and are dimmed when closed (weekends included). Holidays
  are *not* tracked — a holiday will still show as "open".
- Market data is informational only, not investment advice.
- **Flight status without an API key is computed from the times you saved, not from a live feed** —
  it tells you where a flight *should* be, and the **Live ↗** link opens the real position. For true
  live status, get a free RapidAPI AeroDataBox key and paste it into ⚙ Customize → Flights; it is
  stored in your browser only. Delays and cancellations never appear in schedule mode.
- Routes shipped as defaults were checked against public trackers (QR836 Doha–Bangkok, QR416
  Doha–Beirut, QR002 London–Doha, QR1427 Doha–Addis Ababa, QR976 Doha–Hanoi, QR151 Doha–Madrid).
  **QR1238 could not be verified** and ships blank — fill it in from your ticket.
