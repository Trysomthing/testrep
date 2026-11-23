# Countdown Web App — Requirements

This file is the source of truth for what this app does. It is kept up to
date alongside the code — whenever a feature is added, changed, or removed,
this file is updated in the same commit/PR. Do not rely on chat history or
commit archaeology to know what the app is supposed to do; read this file.

## What it is

A single self-contained `index.html` — no external libraries, no CDN, no
build step, no backend. Works on mobile and desktop. Deployed via GitHub
Pages from `master`.

### Visual language: "Arcade Pop"

Bold, chunky, sticker-styled — not glassmorphic. The defining elements:

- A fixed **ink** color (`--ink`, a near-black) used everywhere as a thick
  (2–3px) outline plus a flat, hard-edged offset drop shadow (no blur) —
  `var(--shadow-x) var(--shadow-y) 0 var(--ink)` — on cards, buttons,
  chips, menus, modals and the illustration itself. This reads as a sticker
  or arcade-badge look, not a soft shadow.
- Flat, saturated fills instead of gradients or translucent glass: no
  `backdrop-filter`, no gradient-clipped text. Headings and the "done"
  message are solid text with a small hard `text-shadow` offset in ink
  instead of a gradient fill.
- The four countdown tiles each get a different flat accent color, cycling
  `--accent-1 → --accent-2 → --accent-3 → --accent-1` across
  Days/Hours/Minutes/Seconds, with dark ink numerals/labels for contrast
  (not light text on dark glass).
- Small decorative "sticker" shapes (`.orb1/2/3`) drift gently in the
  background — flat colored circles/squares with an ink border, no blur —
  replacing the old soft glow orbs. They intensify slightly as the event
  gets closer (`time-mid`/`time-close`), same trigger as before.
- Interactive elements (buttons, chips) "press" on hover: they translate
  toward their shadow and the shadow shrinks, rather than lifting up.
- Illustrations (all themes) use flat accent-color fills with a thick ink
  stroke instead of soft gradient fills, and the whole illustration gets a
  hard ink drop-shadow — a cut-out sticker look. The generic hourglass's
  thin frame lines stay a plain flat-colored stroke (no ink border — it's
  a line drawing, not a filled shape).
- **Exception — the `memorial` theme opts out of the playful motion**,
  though it keeps the same sticker-outline structure and its own muted
  palette: no bob animation on the illustration, no drifting background
  sticker shapes (`body[data-theme="memorial"]` dampens both), no confetti
  burst on completion, and the done-state message reads "The day is here"
  instead of "🎉 It's here!". `applyTheme()` sets `document.body.dataset
  .theme` to the active theme's key so this (and any future per-theme
  exception) can hook off it.

## Core countdown

- Live Days / Hours / Minutes / Seconds countdown to a target date & time.
- **Emphasis**: whichever unit is the largest still-nonzero one gets a
  visually bigger box — wider grid column *and* bigger digits, not just
  bigger text. Once days hits 0, hours becomes the emphasized one, then
  minutes, then seconds. The other boxes shrink and dim slightly.
- Progress bar spanning from the event's creation time to its target time.
- "It's here! 🎉" done state once the countdown reaches zero.
- When the active event has a location set, a small pin-icon line under the
  target date shows the location text as a link that opens a Google Maps
  search for it (`https://www.google.com/maps/search/?api=1&query=...`) in
  a new tab — plain URL, no API key, no external script. Hidden entirely
  when the event has no location. This is intentionally lightweight (no
  live Google Places autocomplete) to keep the app dependency-free and
  key-free, per this file's "What it is" section.

## Multiple events

- A single dropdown trigger at the top shows the active event's title and
  date. Tapping it opens a panel listing every saved event, instead of the
  old horizontal row of chips (which got busy and hard to scan once there
  were more than a couple of events).
- Events in the panel are grouped by how soon they are, computed from
  calendar dates (not raw time, so "today" means today regardless of the
  event's time of day):
  - **Today** — target date is today.
  - **This week** — 1–7 days out.
  - **Later** — more than 7 days out.
  - **Past** — the target date has already passed. Collapsed by default
    behind a "Past (N)" toggle (a native `<details>`), shown de-emphasized,
    most-recently-ended first — so events you can no longer act on don't
    compete for attention with the ones you can, but are still one tap
    away. Within every other group, events are sorted soonest-first.
  - A group header is only shown when it has events in it.
- "+ New countdown" at the top of the panel opens a modal to add an event:
  title (required), date + time (required), location (optional), notes
  (optional), and an optional custom color override.
- Each row shows the event's title and date, a pencil icon to **edit** it
  in place (same modal, pre-filled, updates by id — never creates a
  duplicate), and a "×" to **delete** it. The delete icon is hidden when
  it's the only event left — you can never end up with zero events.
  Clicking a row's title switches to that event and closes the panel;
  clicking elsewhere, or pressing Escape, closes it without switching.
- Persisted in the browser's `localStorage` (no backend): the event list and
  which one is currently active both survive reloads.
- Seed/default event on first load: "Only 16 KM away" — October 3, 2026,
  21:10 — with a custom color (`#fb7185`, coral) set, producing a
  coral/warm-gold/lavender palette instead of the theme-default purple/
  pink/blue.

## Illustration & theme system

Each event's title is lowercased and keyword-matched (substring match)
against a fixed list of themes, checked **in this priority order**:

`memorial → birthday → travel → work → love → generic (fallback)`

Order matters — `memorial` is checked first on purpose so a serious event
can never be miscategorized by an incidental word match further down the
list. The first theme with a matching keyword wins; if none match, the
event falls back to `generic`.

| Theme | Keywords | Palette | Illustration |
|---|---|---|---|
| `memorial` | cemetery, funeral, memorial, gravesite, grave, wake, vigil, remembrance, burial, headstone, condolence, mourning, in memory | muted slate / silver / lavender | lit candle with flowers |
| `birthday` | birthday, bday, born | warm orange / yellow / pink | balloon bouquet + confetti |
| `travel` | trip, travel, flight, vacation, holiday, cruise, journey, getaway | sky blue / indigo / yellow | paper airplane + motion trails + clouds |
| `work` | launch, deadline, work, meeting, interview, exam, presentation, project, conference, release, demo, graduation | emerald / blue / purple | rocket with flame + stars |
| `love` | love, anniversary, wedding, valentine, honeymoon, engagement, propose, proposal, date night, romance, marry, married | purple / pink / blue | couple hugging + pulsing heart |
| `generic` (fallback) | *(none — used when nothing else matches)* | purple / pink / blue (same as `love`) | hourglass with falling sand |

Important: **the illustration shape always comes from keyword matching
only.** Nothing about custom colors below ever changes which picture is
drawn — only its colors.

Every event, including the seed event, is re-matched against its *current*
title every time it's displayed — so editing an event's title (including
the seed event) always updates its illustration. There is no hardcoded
exception for any event id.

### Manual color override

In the "New Countdown" / "Edit Countdown" form, checking "Use a custom
color" and picking one color derives a full palette from it:
- Two more accent colors via hue rotation (+55°/−70° from the picked hue —
  matching the spread of the original hand-picked purple/pink/blue palette,
  not a harsh 120° primary-color triad).
- A matching dark background tone (same hue, low lightness).

This overrides the *matched theme's colors* — it does not change which
illustration is shown; that's still whatever `pickTheme(title)` returned.

All theme/custom colors are applied via CSS custom properties
(`--accent-1/2/3`, `--bg-1/2`), so the countdown tiles, buttons, progress
bar, background sticker shapes, and the illustration's own flat SVG fills
all reskin together from the same source.

## Gamification: milestones & ambient time stages

Two independent layers, both driven by data already on the event (no new
storage beyond the event object):

### Milestones checklist

- Each event has a `milestones` array: `{ id, title, completed, dueDate }`.
  `dueDate` is an optional `YYYY-MM-DD` string.
- A "Milestones" panel (between the progress bar and the Add to
  Calendar/Export buttons) shows a checklist for the active event's
  milestones, a `done/total` counter, and a checkbox to toggle each one
  done/not-done plus a "×" to delete it. Changes save to `localStorage`
  immediately.
- **The bordered panel itself is only shown when there's something to put
  in it.** With zero milestones and the add form closed, there's no panel
  chrome at all (no border, no background, no "Milestones" header) — just
  a plain, borderless "+ Add milestone" button sitting in that spot. The
  full panel (border, shadow, header/count) reappears the moment either
  condition flips: the add form is opened, or the event has at least one
  milestone. Deleting the last milestone (with the form closed) collapses
  it back down to just the button.
- Adding one: tapping "+ Add milestone" reveals the title input, optional
  due-date picker, a submit button, and a cancel ("×") button; submitting
  or cancelling collapses the form back to just the toggle (and, per
  above, the whole panel if that leaves zero milestones). It also
  auto-collapses (discarding anything half-typed) whenever you switch to
  a different event, so you never find yourself mid-edit on the wrong
  event's form.
- The seed event ships with 4 example milestones (2 already completed,
  each with a due date); a newly-created event starts with none.
- When a milestone has a due date, it's shown under the title as:
  - an incomplete milestone past its due date — `"<date> · Nd overdue"`;
  - due today — `"<date> · due today"`;
  - due in the future — `"<date> · Nd left"`;
  - a completed milestone — just `"Due <date>"` (no countdown, since it's
    done). Days are calculated from calendar dates only, ignoring time of
    day. A milestone with no due date shows no badge at all.

### Milestone-driven decorations

- Completion percentage (`completed / total` milestones) adds small SVG
  decoration motifs onto the event's illustration, one at a time as
  thresholds are crossed: **25%** → a sparkle, **50%** → a flower (bottom
  left), **75%** → a second flower (bottom right), **100%** → a ring glow
  around the whole illustration.
- An event with zero milestones shows no decorations at all (no implicit
  0% state).
- Decorations reuse the theme's current accent colors (`var(--accent-2/3)`)
  so they always match the active illustration/palette, and fade in with a
  short transition when a new one appears.

### Time-driven ambient stage

- As the countdown progresses from creation time to target time, `<body>`
  gets one of three classes reflecting how close the event is:
  `time-early` (elapsed < 50%), `time-mid` (50–85%), `time-close` (≥85%,
  including after the event has passed). This brightens/intensifies the
  background orbs and starfield (including a few extra twinkling stars) as
  the event approaches — a subtle ambient cue independent of milestones.
- When the countdown reaches zero, a one-time confetti burst (24 falling
  pieces in the current theme's colors) fires across the screen in addition
  to the existing "It's here!" state. It fires only once per page load per
  event, not on every tick.

### Export interaction

The milestones panel, the extra twinkle stars, and any in-flight confetti
are all hidden during Image/GIF export (same mechanism as the existing
export hide-list) — the decoration motifs baked into the illustration
itself are *not* hidden, since they're part of the artwork being exported.

## Add to Calendar

Dropdown with two options, both driven by the currently active event
(title, date/time, location, description; duration defaults to 1 hour):
- **Google Calendar** — opens a prefilled event-creation link in a new tab
  (includes the viewer's timezone).
- **Apple / Outlook (.ics)** — downloads a standards-compliant
  `VCALENDAR`/`VEVENT` file.

## Export

Dropdown with four options, all client-side, no server/library:
- **Save as Image (PNG)** — renders the page (minus buttons/chrome) to a
  canvas via an SVG `foreignObject` + `<img>` + `<canvas>` technique.
  **Known engine limitation, worked around:** `border`, `box-shadow`, and
  `text-shadow` silently fail to rasterize through that `<img>`-decode
  path once the page is this complex (verified against a hand-written
  minimal reproduction of the same technique, which renders them fine —
  and the exact same generated SVG also renders them fine when navigated
  to directly as a document; only decoding it as an `<img>` for
  `drawImage` drops them). Flat fills, text, and the illustration's own
  native SVG `stroke`/`fill` all rasterize correctly regardless. Since
  export already hides everything except the countdown tiles, the eyebrow
  badge, and the progress bar, `captureToCanvas()` redraws just those
  elements' ink borders and hard shadows natively on the canvas
  afterward (`measureInkFrames`/`drawInkShadows`/`drawInkBorders`),
  measured from the *live* DOM with `.exporting` also forcing
  `transition:none; animation:none` on every element first — otherwise a
  transition still mid-flight (e.g. exporting immediately after switching
  to an event with different emphasis) would leave the hand-measured
  geometry mismatched against the settled snapshot. The clone also only
  captures `document.body`, not `<html>`, so the live theme/custom-color
  CSS variables (set on `document.documentElement`) are explicitly
  re-declared in the exported SVG's own stylesheet — without that, the
  export silently reverted to the stylesheet's default purple/pink/blue
  regardless of the actual active theme. `text-shadow` on the title and
  the done-state message is not reconstructed (minor, accepted gap —
  everything else about the export matches the live page).
- **Save as GIF (animated)** — captures ~5 live frames (~450ms apart,
  showing the countdown actually ticking) and encodes them with a
  hand-written GIF89a encoder: fixed 256-color palette (3/3/2-bit RGB
  buckets), a real dictionary-based LZW compressor, NETSCAPE loop
  extension. Fully self-contained.
- **Copy Image to Clipboard** — copies a PNG snapshot via the Clipboard API.
- **Share… (WhatsApp, Telegram, Email)** — uses `navigator.share()` with
  the image attached as a file where supported (mobile Chrome/Safari,
  opens the native share sheet). Falls back to copying the image to the
  clipboard plus a toast message on browsers without file-sharing support
  (most desktop browsers).

All four hide the events bar, the calendar/export buttons themselves, the
footer, and any open modal from the captured output.

## Known limitations (accepted tradeoffs, not bugs)

- Theme keyword matching is inherently limited to the lists above — any
  event type not covered falls back to the neutral `generic` hourglass by
  design, rather than guessing.
- GIF export's fixed 256-color palette means visible color banding on
  smooth gradients — an accepted tradeoff for a dependency-free encoder.
- `navigator.share()` with file attachments is mobile-browser-only today;
  desktop browsers always take the clipboard-copy fallback path.

## Maintenance

When you add, change, or remove a feature in this app: update this file in
the same commit/PR. If a section above no longer matches the code, fix the
section — don't leave it stale.
