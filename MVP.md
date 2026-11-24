# MVP — Event "secretary" concept

This file tracks the product direction discussed for turning the countdown
app into a small-event helper (RSVP + "what to bring" + automated
reminders), separate from `REQUIREMENTS.md`, which documents what the
shipped app currently does. Nothing here is built yet unless a section
says so.

See `reports/Event planning industry opportunity.md` (and the matching
published doc) for the market research this is based on. The short
version: don't build a general event-planner platform — stay a free,
lightweight, single-event tool, and lean into the RSVP/reminders angle
instead of weddings/corporate/vendor-marketplace territory.

## The concept

You (the organizer) prep an event once, in the app. Two guests get a
link by email/SMS, RSVP, and optionally claim a "bring" item — all
without installing anything. The app follows up with scheduled
reminders, and *those* reminders (not the first invite) are what invite
the guest to install the app themselves.

A rendered sequence diagram of this flow was published here:
https://claude.ai/artifact/Nqf9HFLT9Tw5WU9yQaoLAw

## MVP — required to make the concept real

1. **Server-side event storage.** The one unavoidable architecture
   change. Today an event lives only in the creator's browser
   (`localStorage`); a shareable event needs a canonical copy on a server
   so every guest, and the organizer's own other devices, see the same
   state.
2. **Shareable guest link.** A public, no-login URL per event (reusing
   the existing share flow) showing the countdown, time/location, and
   the plan.
3. **RSVP.** Yes/no/maybe (+ guest count), tied to a lightweight guest
   identity (name + email/phone, no account).
4. **"What to bring" claiming.** Extends the existing Assignments
   feature so items can be left unassigned and a guest can claim one
   from the link, not just the organizer assigning by name.
5. **Guest contact capture.** An email (and optionally phone) collected
   at RSVP time — the only way to reach people later.
6. **Scheduled reminders, email-only to start.** Fires at fixed offsets
   (1 week / 1 day / day-of) to guests, and separately to the organizer
   ("3 people haven't responded yet").
7. **Organizer dashboard.** The existing single-page view, now showing
   live RSVP/claim status pulled from the server instead of only local
   data.
8. **i18n carried through.** Guest-facing pages and reminder emails
   reuse the EN/HE + RTL system already built.

### Demo/preview of this surface (no backend yet)

A front-end-only preview of steps 3–4 (RSVP + open/claimable bring
items) has been added to the single-file app so the shape of the data
and UI can be reviewed before any backend is built. It's local-only —
one browser, no real invites sent, no second device involved — see
`REQUIREMENTS.md` for exactly what it does and doesn't do.

## Beyond MVP — defer or drop

- **Push notifications** (installed PWA) and **WhatsApp/SMS
  reminders** — real reach upgrades, but paid/complex; add once email
  reminders prove people act on them.
- **Co-organizers / multi-editor** access to one event.
- **Nudge automation** ("auto-remind non-responders") as a smarter
  layer on top of scheduled reminders.
- **Guest comments/coordination thread** per event.
- Anything that starts to look like a planner platform — budget
  tracking, vendor/gift marketplace, seating charts, registries,
  B2B/white-label for venues. The research's whole point was to stay
  out of that territory; these are explicitly out of scope, not just
  "later."

Not in scope at all: Instagram as a delivery channel (no API for
unsolicited DMs to arbitrary users).
