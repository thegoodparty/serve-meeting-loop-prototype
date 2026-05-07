# Serve Meeting Loop Prototype

Single-file HTML prototype of the Serve meeting loop: pre-meeting briefing, in-meeting agenda navigator, and post-meeting constituent recap. The same source document drives two rendering modes (Document and Live), and a single composed recap is sent to constituents.

## Open

```
open index.html
```

The prototype runs as a static file. No build step. Tailwind and React load from CDN.

## Docs

- [`docs/architecture-handoff.md`](docs/architecture-handoff.md) — Locked architectural decisions, data model, and engineering handoff notes for the meeting loop.
- [`docs/gp-webapp-codebase-reference.md`](docs/gp-webapp-codebase-reference.md) — Pointers into the existing GoodParty webapp codebase: routes, shared components, and conventions to match.
- [`docs/win-onboarding-ui-reference.md`](docs/win-onboarding-ui-reference.md) — Reference for the Win onboarding UI patterns the Serve flow should align with.

## Prototype features

- Document and Live mode toggle on the same source briefing.
- 35% / 65% segmentation between briefing summary and per-item detail.
- Pattern B layout with swap behavior between summary and item view.
- Position toggle for the official's stance on each agenda item.
- Live agenda navigator that walks through items during the meeting.
- Inline note capture per agenda item.
- Composed-recap fingerprint that reflects the official's votes, notes, and positions.
- Notify-your-constituents flow at the end of the loop.

## Status

This repo is the source of truth for the v4 prototype. Lovable connection is planned so Kaylee can iterate on the prototype directly. No deployment target is wired up; the prototype runs locally as a static file.
