# 🏆 Sports Survival Pool

A self-hosting **survival pool** for any group-stage + knockout tournament (World Cup, Euros, Copa América…). Everyone's assigned a team; you're alive as long as your team is — last person standing wins.

You get:
- 🌐 An always-on **website** — live standings, knockout bracket, groups, results, and a live/upcoming match strip
- 🔔 **Slack alerts** — morning fixtures, kick off / half time / full time, and elimination/advancement news
- 🤖 **Fully autonomous** — refreshes itself from live data; nothing to babysit

It runs entirely on **free tiers** (Vercel + GitHub Actions + ESPN's public feed). The only paid bit is a few cents of Anthropic API for the standings brain.

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https://github.com/wahajfar/sports-survival-pool)

---

## Quick start (~15 min)

1. **Use this template** → "Use this template" → create your own repo.
2. **Edit `config.json`** — your pool name, tagline, colours, the competition, and (after step 5) your site URL.
3. **Edit `data/roster.json`** — one entry per player: `name`, `team` (exact national-team name), `group`, `flag` emoji, optional `photo` URL, optional `slackId` (for @-mentions).
4. **Create a Slack Incoming Webhook** (api.slack.com/apps → your app → Incoming Webhooks → add to your channel) and an **Anthropic API key** (console.anthropic.com).
5. **Add two repo secrets** (Settings → Secrets and variables → Actions): `ANTHROPIC_API_KEY` and `SLACK_WEBHOOK_URL`.
6. **Deploy to Vercel** (button above, or import the repo). Put the resulting URL in `config.json` → `siteUrl`.
7. **Actions tab → enable workflows.** Done — it runs itself.

> Want the standings, bracket and alerts to start populating immediately? In the Actions tab, run **"Update World Cup standings"** once manually.

---

## How it works

| Layer | Runs | What it does |
|---|---|---|
| **Website** (`index.html` + `app.js`) | always | Renders `data/standings.json`; falls back to the roster before the first run |
| **Standings updater** (`scripts/update.mjs`) | hourly | Claude + web search → survival standings → site refresh → Slack on real changes |
| **Live watcher** (`scripts/live.mjs`) | every 15 min | ESPN feed → kick off / half time / full time to Slack |
| **Morning fixtures** (`scripts/morning.mjs`) | daily 8am | Posts the day's matchups, tagging the playing owners |

All three are GitHub Actions workflows in `.github/workflows/`. Edit the `cron:` lines to change timing (they're in UTC).

---

## Configuration (`config.json`)

| Field | Meaning |
|---|---|
| `poolName`, `tagline`, `logoText`, `edition` | Branding shown on the site |
| `competition.name` | Used in the standings prompt (e.g. "UEFA Euro 2028") |
| `competition.espnLeague` | ESPN league slug for live data — `fifa.world` (World Cup), `uefa.euro`, `conmebol.america`, etc. |
| `competition.format` | One sentence describing the format, fed to the standings AI |
| `competition.finalLabel` | Footer text, e.g. "Final: July 19, 2026" |
| `siteUrl` | Your deployed URL (used in Slack links) |
| `theme` | `bg`, `card`, `ink`, `accent`, `highlight`, `live` hex colours |

## Roster (`data/roster.json`)
`team` must match the national-team name the data feed uses (common aliases like Türkiye/Turkey, Ivory Coast/Côte d'Ivoire are handled). Set `slackId` (a Slack member ID like `U0123…`) to have alerts @-mention people.

## Use it for a different tournament
Set `competition.espnLeague` (live + morning posts work instantly for any ESPN soccer competition) and `competition.name` + `competition.format` (for the standings updater). The standings logic is tuned for **group-stage + knockout** tournaments; for other formats, review the prompt in `scripts/update.mjs`.

## Cost
Vercel, GitHub Actions, and the ESPN feed are free. The hourly standings run uses ~2¢ of Anthropic API each → roughly $10–15 over a month-long tournament. Drop the updater to every 2 hours to halve it.

## Branding
Default font is Montserrat (free). To use a custom brand font, drop `Display.otf` into `assets/fonts/`. Colours come from `config.json` → `theme`.
