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

`memorial → birthday → travel → tennis → soccer → basketball → sports → work → love → generic (fallback)`

Order matters — `memorial` is checked first on purpose so a serious event
can never be miscategorized by an incidental word match further down the
list. The first theme with a matching keyword wins; if none match, the
event falls back to `generic`.

Sport-specific events (tennis, football/soccer, basketball) each get
their own ball/equipment illustration rather than one shared "sports"
picture — a generic sports image reading as a *specific other* sport
(e.g. a tennis racket showing up for a football event) looked exactly as
wrong as the generic fallback did before `sports` existed at all. Any
other sport/competition keyword falls through to a neutral trophy.

| Theme | Keywords | Palette | Illustration |
|---|---|---|---|
| `memorial` | cemetery, funeral, memorial, gravesite, grave, wake, vigil, remembrance, burial, headstone, condolence, mourning, in memory | muted slate / silver / lavender | lit candle with flowers |
| `birthday` | birthday, bday, born | warm orange / yellow / pink | balloon bouquet + confetti |
| `travel` | trip, travel, flight, vacation, holiday, cruise, journey, getaway | sky blue / indigo / yellow | paper airplane + motion trails + clouds |
| `tennis` | tennis | tennis-ball green / white / court blue | tennis racket + ball with motion lines |
| `soccer` | football, soccer | pitch green / white / sky blue | soccer ball (pentagon pattern) + grass |
| `basketball` | basketball | ball orange / white / red | basketball + hoop |
| `sports` (catch-all) | sports, sport, match, tournament, championship, playoff, baseball, hockey, golf, marathon, race, game day, gym, workout | gold / white / rose | trophy |
| `work` | launch, deadline, work, meeting, interview, exam, presentation, project, conference, release, demo, graduation | emerald / blue / purple | rocket with flame + stars |
| `love` | love, anniversary, wedding, valentine, honeymoon, engagement, propose, proposal, date night, romance, marry, married | purple / pink / blue | couple hugging + pulsing heart |
| `generic` (fallback) | *(none — used when nothing else matches)* | purple / pink / blue (same as `love`) | hourglass with falling sand |

Covering every possible event type exactly isn't realistic for a fixed
keyword list — when you hit a gap like `sports`/`tennis` was, say so and
it gets added, the same way this one was (and the same way the generic
`sports` bucket was then split into per-sport themes once a mismatch
showed up there too).

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

## Assignments

A second panel, right below Milestones, for delegating tasks to specific
people — e.g. after sharing the countdown with family/friends, "assign"
someone a job rather than tracking your own to-dos.

- Each event has an `assignments` array: `{ id, person, task, completed }`.
  `person` is freeform text (no contact list/picker — just a name typed in)
  and `task` is what they're meant to do.
- Structurally and visually a twin of the Milestones panel, down to reusing
  the same CSS classes: a `done/total` counter, a checkbox per row to mark
  an assignment done/not-done, a "×" to delete one, and the same
  show-only-when-there's-something-to-show behavior — no bordered panel
  chrome at all when there are zero assignments and the add form is
  closed, just a plain "+ Add assignment" button (sitting directly under
  "+ Add milestone"). The panel (and its header/border) reappears once
  there's an assignment or the add form is open, same rules as Milestones.
- Adding one asks for two fields: who (`person`) and what (`task`); both
  are required. No due date (unlike milestones) — kept deliberately
  simple for now.
- Each row shows the task as the primary (bold) line and the assigned
  person as the secondary line underneath — you're scanning for what
  needs doing first, who's doing it second.
- The seed event ships with 2 example assignments (1 already completed);
  a newly-created event starts with none.
- Unlike milestones, assignments do **not** drive the illustration
  decorations or any other gamification — they're a separate, independent
  checklist.
- **Open/claimable items**: `person` may be left blank when adding an
  assignment (the "Who…" field says so). A blank-person row renders a
  "Claim" button instead of a name; clicking it opens an inline
  name field + confirm button, and submitting it sets `person` to that
  name. This is a local stand-in for a guest claiming a "what to bring"
  item from their own RSVP link once that link exists — see `MVP.md`.
  There is no real multi-device sharing here yet: claiming only changes
  the data in the current browser.

## Guests (RSVP) — local preview, not yet shareable

A third panel, between Milestones and Assignments, previewing the planned
RSVP surface described in `MVP.md`. **This is a local-only mockup**: it
does not send invites, does not involve a second device or browser, and
nothing here talks to a server. Its purpose is to let the shape of the
data and UI be reviewed before the real backend (shareable event link,
email invites, scheduled reminders) is built.

- Each event has a `guests` array: `{ id, name, rsvp }`, where `rsvp` is
  one of `pending` / `going` / `maybe` / `declined`.
- Same chromeless panel pattern as Milestones/Assignments (reusing the
  same CSS): hidden chrome with just a "+ Add guest" button when there
  are no guests and the add form is closed; full panel once there's a
  guest or the form is open.
- Adding a guest asks only for a name (no email/phone — that capture step
  is part of the real RSVP flow in `MVP.md`, not this preview).
  New guests start as `pending`.
  - Each row shows the guest's name and a colored RSVP badge
  (Pending / Going / Maybe / Can't go). Tapping the badge cycles it to the
  next status, in that order, looping back to Pending.
- The counter badge shows `{going}/{total}`.
- The seed event ships with 2 example guests (one Going, one Maybe); a
  newly-created event starts with none.
- Guests do not drive illustrations/gamification, same as Assignments.

## Top row layout

The events dropdown trigger, the Add to Calendar icon, the Share icon,
and the language toggle all sit together in one row at the top of the
page (`.top-row`), not spread between the top and a button row near the
bottom. The events trigger shrinks to share the row with the icons, but
its dropdown panel still spans the full row width (positioned against
`.top-row`, not the trigger itself) so the event list stays comfortably
wide. The Calendar dropdown menu opens anchored to its own trailing edge
(`inset-inline-end`, which follows reading direction) rather than
centered under the button, so it can't run off the side of the screen
now that the button sits near the edge of a narrow row.

## Language & RTL (English / Hebrew)

Full bilingual UI, not just RTL-for-content: a pill-shaped toggle button
(`#langToggle`, in the top row) switches the app's own chrome — labels,
placeholders, aria-labels, toasts, confirmations, error messages, date
formatting — between English and Hebrew, and flips the whole page to a
right-to-left layout when Hebrew is active. The toggle always shows the
*other* language's own name (e.g. it reads "עברית" while in English), so
it's legible regardless of which language is currently showing.

- **Seed/demo content is never auto-translated.** Event titles,
  milestone/assignment text a user types, and location/notes are left
  exactly as authored — only the app's own interface strings are
  localized. Proper nouns ("Google Calendar", "Apple / Outlook (.ics)")
  are also left untranslated.
- **String system:** a `STRINGS = { en: {...}, he: {...} }` dictionary
  (plain strings plus parameterized function entries for strings that
  interpolate an event/milestone title, a count, etc.) and a `t(key,
  ...args)` lookup helper, with an automatic fallback to English for any
  missing key.
- **Static markup** is translated via `data-i18n` / `data-i18n-placeholder`
  / `data-i18n-aria-label` / `data-i18n-title` attributes, applied by
  `applyStaticTranslations()`.
- **Dynamic strings** (toasts, `confirm()` prompts, the Add/Edit modal's
  title and submit-button swap, form validation errors, milestone due-date
  phrases, aria-labels built from a live title, the share text,
  `document.title`) all go through `t()` at the point they're built,
  rather than being translated after the fact.
- **Locale-aware dates:** the three `toLocaleString`/`toLocaleDateString`
  call sites (target date line, event-chip dates, milestone due dates)
  use the active language's locale code (`en-US` / `he-IL`) instead of
  the browser default, so Hebrew mode shows Hebrew month names.
- **Persistence & detection:** the chosen language is saved to
  `localStorage` (`countdown_lang_v1`) and restored on load; with no
  saved choice yet, it's detected from `navigator.language` (Hebrew if it
  starts with `he`, English otherwise). A small inline script in `<head>`
  (before the main stylesheet/script) sets `<html lang>`/`dir`
  synchronously from that same logic, so the page never flashes in the
  wrong direction before the full script runs.
- **RTL layout:** toggling Hebrew sets `document.documentElement.dir =
  'rtl'`. Most of the layout (flex rows, the countdown grid including its
  per-unit emphasis sizing) mirrors automatically under `dir="rtl"` with
  no JS changes. A handful of physical left/right CSS properties were
  converted to logical ones (`text-align: start` instead of `left`, the
  Calendar menu's `inset-inline-end` instead of `right`) plus one explicit
  `[dir="rtl"]` override for the Past-events disclosure triangle, which
  otherwise points the wrong way. The decorative corner shapes (`.orb1/2/3`)
  are intentionally left un-mirrored — purely cosmetic, not worth the
  complexity.
- **Switching language re-renders** the events dropdown, the active event
  display, milestones, and assignments in place, so anything already on
  screen picks up the new language immediately — not just content
  rendered after the switch.

## Add to Calendar

Icon-only button (calendar icon, no visible label — `aria-label`/`title`
carry "Add to Calendar" for accessibility), in the top row next to the
events dropdown and Share. Opens a dropdown with two
options, both driven by the currently active event
(title, date/time, location, description; duration defaults to 1 hour):
- **Google Calendar** — opens a prefilled event-creation link in a new tab
  (includes the viewer's timezone).
- **Apple / Outlook (.ics)** — downloads a standards-compliant
  `VCALENDAR`/`VEVENT` file.

## Export

**Currently hidden from the UI** (the "Export" button/dropdown is in the
DOM with the `hidden` attribute, not removed — a deliberate, easily-reversed
toggle, not a deleted feature). Re-enable by dropping that attribute from
`.export-wrap` in `index.html`. The feature code, its tests, and the
description below are otherwise unchanged and current.

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

**Share** is its own standalone icon-only button next to Add to Calendar
(not a dropdown item; Export, also normally in that row, is currently
hidden — see above) — uses `navigator.share()` with the image attached as
a file where supported (mobile Chrome/Safari, opens the native share
sheet: WhatsApp, Telegram, Email, etc.). Falls back to copying the image
to the clipboard plus a toast message on browsers without file-sharing
support (most desktop browsers).

All exports (PNG, GIF, clipboard copy, share) hide the events dropdown,
the calendar/export/share buttons themselves, the footer, and any open
modal from the captured output.

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
