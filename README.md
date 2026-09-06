# RigSpace

RigSpace is a capture log and storage forecast for multi-day location shoots.
It records how much Streams and PSC media lands on each RX day, and projects
the run rate forward against the storage quotas you have available.

It is a sibling to [CodecCalc](https://charliebattye.github.io/codeccalc/) and
shares its look. CodecCalc sizes cards and drives *before* a shoot; RigSpace
records what actually arrived, and tells you whether the drives will hold.

## What it does

- **One RX day per line item.** `RX01`, `RX02`, and split days as `RX04A` /
  `RX04B`. A line item with no figures against it is a day still to shoot, and
  those are what the forecast runs over — nothing has to be kept in step by hand.
- **Two modes, one schedule.** `MAIN` and `PARITY` share the RX days, their
  splits and their dates. Media sizes and both storage pools are held per mode,
  because the parity set is its own copy on its own drives.
- **Two pools, two charts.** Streams and PSC are forecast separately against
  their own quotas, each on its own scale, and the chart marks the RX day where
  the projection crosses the pool.
- **Figures are decimal TB** (1 TB = 1000 GB), matching how drives are sold.

## Running it

Open `index.html` in a browser. There is no build step, no dependency and no
server — the same arrangement as CodecCalc. Its only external requests are the
three Google Fonts families in the head.

## Where the data lives

RigSpace runs in two places, and stores its data differently in each:

| Where | Storage | Who sees it |
|---|---|---|
| Published as a Claude Artifact | the artifact's shared database | everyone on the team who opens it |
| Opened from a file, or GitHub Pages | `localStorage` | that one browser only |

The page asks for its capabilities through `claude.use()`, which resolves `null`
anywhere but the artifact viewer. It detects this and falls back, so the file
still works standalone — but a shared log needs the artifact.

The status line at the foot of *Project & storage* always says which of the two
you are in, so it is never a guess.

## Saving

There is nothing to press. Every entry writes as you make it — to the shared
database in the artifact, to `localStorage` otherwise. The example data the page
opens with is the one exception: it is never written, and the status line says so
until you enter a real RX day.

**Export CSV** writes both modes to one file. **Import CSV** reads that same file
back, header row and all, restoring both modes and which days are still to shoot.
So the round trip is lossless, and a CSV is a real backup rather than a report —
useful for handing the log to a producer and taking it back afterwards.

Shorter rows still import, landing in the mode you are on:

    RX01	2026-08-11	4.12	0.86
