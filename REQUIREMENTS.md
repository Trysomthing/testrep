# Countdown Web App — Requirements

This file is the source of truth for what this app does. It is kept up to
date alongside the code — whenever a feature is added, changed, or removed,
this file is updated in the same commit/PR. Do not rely on chat history or
commit archaeology to know what the app is supposed to do; read this file.

## What it is

A single self-contained `index.html` — no external libraries, no CDN, no
build step, no backend. Dark glassmorphic aesthetic (gradient text, blurred
orbs, frosted-glass cards). Works on mobile and desktop. Deployed via GitHub
Pages from `master`.

## Core countdown

- Live Days / Hours / Minutes / Seconds countdown to a target date & time.
- **Emphasis**: whichever unit is the largest still-nonzero one gets a
  visually bigger box — wider grid column *and* bigger digits, not just
  bigger text. Once days hits 0, hours becomes the emphasized one, then
  minutes, then seconds. The other boxes shrink and dim slightly.
- Progress bar spanning from the event's creation time to its target time.
- "It's here! 🎉" done state once the countdown reaches zero.

## Multiple events

- An events bar (chips) at the top lets you switch between saved countdowns.
- "+ New" opens a modal to add an event: title (required), date + time
  (required), location (optional), notes (optional), and an optional custom
  color override.
- Each chip has a pencil icon to **edit** that event in place (same modal,
  pre-filled, updates by id — never creates a duplicate) and a "×" to
  **delete** it. The delete icon is hidden when it's the only event left —
  you can never end up with zero events.
- Persisted in the browser's `localStorage` (no backend): the event list and
  which one is currently active both survive reloads.
- Seed/default event on first load: "Something big is coming" — July 29,
  2026, 16:00.

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

**Special case:** the seed event (id `seed-1`, "Something big is coming")
always shows the `love` illustration (couple hugging + heart), regardless
of keyword matching — it's the flagship example the app ships with, not
user-generated content, so it's hardcoded rather than relying on "big" or
"coming" to accidentally match a keyword. This only applies to that one
event id; every other event (including one a user renames to the same
title) goes through normal keyword matching.

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
(`--accent-1/2/3`, `--bg-1/2`), so the title gradient, number gradients,
buttons, progress bar, background orbs, and the illustration's own SVG
gradients all reskin together from the same source.

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
