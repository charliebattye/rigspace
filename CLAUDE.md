# RigSpace

RigSpace is a capture log and storage forecast for multi-day location shoots.
It records the Streams and PSC media banked on each RX day and projects the run
rate forward against the storage quotas available.

It is a sibling to CodecCalc and deliberately shares its visual world. CodecCalc
sizes drives before a shoot; RigSpace records what landed. Keep them recognisably
the same instrument.

Charlie owns this tool. Act as a calm guide as well as a developer: explain the
immediate outcome in plain English, and ask one short question at a time when a
decision is genuinely required.

## The shape of the project

The whole tool is one file: `index.html`. It carries its own CSS and JavaScript,
and has no build step, no dependencies, no package manager and no server. Its
only external requests are the three Google Fonts families in the head.

To preview a change, open `index.html` in a browser and reload.

Keep it that way unless there is a real reason not to — the same rule CodecCalc
holds itself to, and for the same reason.

The file opens with `<meta charset>` and `<meta name="viewport">` but **no**
`<!DOCTYPE>`, `<html>`, `<head>` or `<body>` tags. That is deliberate and load
bearing: it lets the identical file both open standalone and be published as a
Claude Artifact, which wraps it in its own document skeleton. Do not add the
wrapper tags.

All non-ASCII characters are written as HTML entities in the markup and `\uXXXX`
escapes in the script, so the file renders correctly however it is served.

## Where it runs, and where the data goes

| Context | Storage | Visible to |
|---|---|---|
| Published Claude Artifact | the artifact's shared database (`db` capability) | the whole team |
| Local file or static hosting | `localStorage` | that browser only |

Capabilities are requested through `useCap()`, a wrapper over `claude.use()`
that resolves `null` outside the artifact viewer instead of throwing. Every
capability path must tolerate that null — the page renders first from local
state and lights the shared store up afterwards, never the other way round.

Declared capabilities: `db` (the shared log) and `downloads` (CSV export).

`saveFile()` tries the `downloads` capability first and falls back to an
`<a download>`. Both halves are needed: inside the viewer the sandbox makes the
anchor inert, and outside it there is no capability. It reports the outcome next
to the button — an early version reported into a hint element far down the page,
which made a working export look like a dead button.

CSV import accepts what export writes — eight columns, both modes, header row —
so the round trip is lossless. Shorter rows still land in the active mode. Keep
those two formats in step: `importText()` distinguishes them by looking for
`shot` / `to shoot` in column three.

## The data model

One document per RX day, at `days/<RX id>`:

    { rx:'RX04A', num:4, part:'A', date:'2026-08-14',
      main:   { entered:true,  streams:5.02, psc:1.61 },
      parity: { entered:false, streams:0,    psc:0    } }

And one settings document at `meta/settings` holding `project`, `basis`, and a
pool pair per mode.

Three things about this shape matter:

- **The schedule is the log.** A row exists for every RX day in the project.
  `entered:false` means the day has not been shot *in that mode* yet, and the
  count of those is what the forecast runs over. There is no separate
  "days remaining" figure to keep in step, because there cannot be one that
  drifts.
- **`entered` is per mode, the row is not.** Main can be a day ahead of Parity.
  Deleting a row removes the RX day from both modes; the RX-days control will
  not trim a day that has figures in either.
- **RX ids parse to `{num, part}`** and sort on `num * 100 + part`. A split RX
  is two RX days and one RX, so per-day and per-RX means legitimately differ.
  The projection uses the per-day one.

`normaliseRow()` and `normaliseSettings()` migrate older stored shapes forward.
Leave them in place; a store written by an earlier version still exists.

## The forecast

The mean of the last N recorded RX days (N from the basis control), carried
forward over the days with no figures in that mode. Streams and PSC are
projected separately so the mix travels with them.

The band is one standard deviation of a daily figure widened as **sd x sqrt(k)**
over k days — the spread of a *total*, not one day's spread repeated k times.
Do not "simplify" this to linear; it was linear once and made the band
implausibly wide over a long schedule.

Two refusals are deliberate:

- It does not forecast below **three** recorded days. Two points is a line, not
  a trend.
- It never runs past the days in the log. If you want a longer projection, add
  the RX days — the tool will not invent them.

## The colour constraint

Series identity: **Streams is amber `#FF764A`, PSC is green `#73FFC5`** —
CodecCalc's own two accents.

CodecCalc's red `#FF4433` sits only **Delta E 8.2** from that amber (5.6 under
simulated deuteranopia), far below the 15 floor at which two marks stay
tellable apart. So red is never a chart mark here. The pool limit is a neutral
hairline; red appears in exactly one role, the point where the projection
crosses the pool, always with a label beside it.

If you introduce a new colour, check it against both series before using it.
The pair passes every other gate on the `#14171C` panel: CVD Delta E 19.2,
normal-vision 34.4, contrast 6.8 and 14.4.

## Before calling a change done

- Open `index.html` and exercise **both** modes. They must show their own pools,
  their own day counts and their own totals from one shared schedule.
- Check the RX-days control both ways: raising it adds days, lowering it removes
  only days with no figures in either mode and says so when it cannot.
- Check the log table shows the active mode's three columns beside the other
  mode's two, and that a day shot in one mode but not the other reads correctly.
- Check the pool charts, including the overflow case — set a pool low enough
  that the projection crosses it and confirm the crossing is marked.
- Check the layout at phone width, and that the two pool fields wrap as a pair.
- Confirm no new external request has been introduced.

## Publishing

The repo is the source of truth. The running tool is a Claude Artifact, because
the shared database is what makes it usable by more than one person — static
hosting would trap each person's log in their own browser.

To update the live tool, publish `index.html` to the existing artifact rather
than creating a new one, so the log and everything in it survives:

    https://claude.ai/code/artifact/6e285dab-5962-4993-9fe0-84887800fb35

Publishing without that URL makes a *second* artifact with an empty database,
which is the one mistake here that loses data.
