# Paper Desk & Bull Run

Two takes on the same idea, both single self-contained HTML files with no build
step and no dependencies.

- **`trader.html` — Paper Desk.** A practice trading terminal. Real tickers,
  simulated prices, $500 to start, and a career layered on top.
- **`index.html` — Bull Run.** A skill-based trading roguelite. Nine sessions to
  turn $1,000 into $1,000,000.

---

# Paper Desk

Open `trader.html`. You get $500, eight markets, and an order ticket.

## What it is

A paper trading simulator with a progression system. You post margin, pick your
leverage, optionally attach a stop loss and take profit, and try to compound $500
into something worth bragging about.

**Prices are simulated, and the app says so on every screen.** A published page
cannot reach a live price feed, so instead each real ticker is driven by a
generator tuned to that asset's temperament — GME lurches, SPY crawls, BTC sits
in between. Measured over a four-hour window: SPY moves ~2%, BTC ~3.5%, GME ~8%,
and every asset stays within roughly ±10% of its headline price. The feed sits
behind one `Feed` class, so a websocket adapter can replace it without the rest
of the app noticing.

## The mechanics are the real ones

| | |
|---|---|
| Margin & leverage | Up to 10×, raised to 50× by perks |
| Liquidation | At 90% of margin lost, priced and drawn on the chart before you commit |
| Costs | 0.06% fee per side, a spread on entry, 0.04%/hour funding on borrowed size |
| Orders | Market entry, stop loss and take profit set as a % of margin |
| Book | Three concurrent positions, five with perks |

Stops fill at market, so a gap can blow through one — same as the real thing.

## The career

Closed trades pay XP; XP pays levels; levels unlock perks you buy with cash.
Twelve perks, each stacking with a rising price: fee rebates, +5% on winners,
leverage upgrades, a signal package that draws a moving average and RSI, free
stops, yield on idle cash, liquidation cover. Rank is set by your best equity,
from Retail up to Legend at $1M.

## Losing everything

Drop below $25 and the account closes. Cash and perks go with it; your level,
career stats and best-ever account survive. A fresh stake unlocks on a cooldown,
or immediately if you sit through a sponsor break.

That break is a **labelled placeholder, not an advert** — a countdown with no
network request, no tracking and nothing sold. It exists so the flow can be
tested, and so there is one obvious function to replace (`rewardedBreak()`) if
this is ever monetised.

## Notes

Canvas candlestick chart with volume, moving average, RSI, crosshair, and entry /
stop / target / liquidation lines. Full light and dark themes across all three
viewer states. Everything persists to `localStorage`, wrapped so a blocked store
still gives a working app. Note the file deliberately ships without a doctype so
it can be published as an artifact; because a standalone open then lands in quirks
mode, table colour and font are set explicitly rather than inherited.

---

# Bull Run

A skill-based trading roguelite. You start with **$1,000** and nine sessions to turn it
into **$1,000,000**. One live tape, a leveraged account, and a desk that sells you an
edge between days.

Open `index.html` in any browser. No build step, no dependencies, one file.

## How it plays

Hold `W` / `↑` / `space` to be long, `S` / `↓` to be short — or hold the on-screen
buttons on a touchscreen. Your position is open while the key is down, and you bank
the profit or loss the moment you let go. Entries cost a spread, so churn is not free.

Each session has a quota. Clear it and you pick an upgrade; miss it and the desk
reassigns your capital. Fall through 25% of your morning balance mid-session and the
risk desk closes you out on the spot.

Headlines break on the wire about a second before they hit the tape, and they name
their direction. That second is the whole game.

## The three systems

**The tape.** Price is a log random walk with regime-switching drift, volatility
clustering, news impulses, and a mean-reversion leash. The leash is what matters:
without it the drift regimes compound into one-way escalators, and with it they
turn over into readable waves — rallies, pullbacks, bases, breakouts. Trend
magnitude and reversion strength are balanced so price roams roughly ±35%, which
keeps the y-axis stable enough to read at a glance.

**The sweep.** Excess capital is banked by the firm overnight — you carry at most
125% of the quota you just cleared. Without this the economy explodes: one good
session compounds into numbers that make every later quota irrelevant. With it,
a monster session buys a head start, never a free ride, and every day stays a
real test.

**The desk.** Twenty-eight upgrades across size, tempo, execution, risk, income,
economy and information. They stack, they interact, and several are deliberately
double-edged: leverage and Volatility Swap both raise your ceiling and measurably
lower your survival odds. Tax Haven widens the sweep itself, from 125% of quota to
225% fully stacked, which bends the whole economy in your favour.

**The book.** Every session after the first runs under a condition you choose at the
desk — Quiet Book, Chop, Trend Day, Thin Liquidity, Heavy Wire, The Grind. Conditions
never move the quota; they change the shape of the tape. Chop suits a scalping kit and
starves a swing kit, Trend Day does the reverse, Heavy Wire doubles the headlines and
so rewards anyone who built for the wire. So the shop is two decisions, not one: what
edge you buy, and what market you take it into.

Session one is always the Quiet Book, at a $1,050 quota and a long clock. It is the
on-ramp, and roughly 95% of runs survive it.

Clear session nine and you can stop at the million or keep trading — the quota keeps
multiplying by 2.8 and the only thing left is the number you stopped at.

## Tuning

The quota ladder was derived rather than guessed. A headless harness plays the real
market code with bots at three skill levels and several shopping strategies, which
gives a distribution of achievable per-session multiples. The ladder is set against
those percentiles, accounting for the 1.25× carry cap — the requirement on session
*n* is `target[n] / (target[n-1] × 1.25)`.

Resulting ladder, and what each session actually demands of you:

| Session | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
|---|---|---|---|---|---|---|---|---|---|
| Quota | $1,050 | $1,600 | $2,800 | $5,500 | $12,000 | $30,000 | $85,000 | $280,000 | $1,000,000 |
| You must make | 1.05× | 1.22× | 1.40× | 1.57× | 1.75× | 2.00× | 2.27× | 2.64× | 2.86× |

Measured run win rates: **~32%** for a sharp player, **~16%** for a competent one,
**~5%** for a sloppy one. Session-one clear rate is ~95% at every skill level; the
curve for a sloppy player falls to ~60% by session nine. Builds differentiate by playstyle rather than dominating —
a scalping kit lifts a fast in-and-out player from 75% to 82% per session and *hurts*
a swing player, while the leverage and volatility upgrades trade win rate for ceiling.

## Notes

Rendering is Canvas 2D on a fixed 60Hz accumulator. The chart tints with your live
P&L, so the whole screen is the feedback. Sound is synthesised with WebAudio and
starts muted until you interact. Progress and preferences persist in `localStorage`,
wrapped so a blocked store degrades to a normal game. Honors
`prefers-reduced-motion`.
