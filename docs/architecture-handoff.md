# Meeting Loop Architecture: Handoff for Refinement

**Date:** 2026-05-06
**Status:** Architecture locked; UX details open for refinement
**Audience:** Refining team. Refine before engineering builds.

## TL;DR

One product, one document per meeting, two rendering modes. Document mode is the pre-meeting briefing (auto-drafted top 3 plus 7 swappable alternatives). Live mode is the in-meeting assistant (35% segment only). Same data, different chrome, time-based switch. Recap is a derived composition, not a separate data object.

## Locked decisions

### 1. Single product, two rendering modes

The pre-meeting briefing and the in-meeting assistant are the same product, the same document per meeting, with two rendering modes that share data. Time triggers the switch from Document mode to Live mode; manual override is allowed. The EO's job is one continuous flow (prep, show up, follow, capture, leave), so two separate products would fracture it. The document-style UI direction also requires a single artifact that progressively reveals live affordances.

### 2. Draft by default, swap if needed (Pattern B)

The EO clicks Prepare and lands on a finished briefing already drafted on our top 3 picks. They read first, then customize only if our picks are wrong. The swap interaction pulls from a list of about 7 alternatives, and the draft re-renders. We do not ask the EO to pick first and draft after. This pattern is approved on the condition that the swap interaction is clean. If it feels clunky, the pattern fails.

### 3. Data model

Meeting contains an ordered list of Issues, structured to Robert's Rules sections. Each Issue carries:
- Briefing content (background, recommendation, rationale, sources)
- Agenda timing (order, expected time block)
- Live state (whether the meeting is currently on this issue)
- User state (planned position, notes, captured outcome, position-change timestamps)

Issue is the atomic unit, consistent with the PISS reframe.

### 4. Document mode (pre-meeting reading experience)

- Auto-drafted top 3 issues, picked by platform alignment with agenda relevance
- 7 other issues available to swap in or out
- Full agenda collapsed below the top 3
- Position toggle on each issue (Yes / No / Abstain / Undecided), optional
- Save as PDF
- Notion / Google Docs aesthetic, no card UI, no nested accordions

### 5. Live mode (in-meeting assistant, 35% segment only for V1)

- Annotated agenda scroll-synced to meeting time
- Full agenda expanded as titles plus timestamps
- Prepped content blooms on the 3 priorities when the meeting lands on them
- Position toggle live with timestamps so the EO can update positions in real time
- Quick-capture for notes (photo, type, paste)
- Mobile-first chrome
- Auto-activates at meeting start; manual toggle override available

### 6. Recap (third surface, composed, not stored)

Drafted from briefing content plus position changes plus notes. Reviewable and shareable. Does not have its own data model; it is a composition over Document and Live state.

### 7. Top-3 pick logic for V1

Platform alignment crossed with agenda relevance. Pick the 3 agenda items most aligned with the EO's stated platform priorities. If fewer than 3 align, fill with the most publicly substantive items (resolutions, final approvals), skipping consent and proclamations. Constituent signal becomes the third leg in V2 once the Constituents object holds more than poll responders. The UI must be transparent about the pick rationale ("We picked these because they map to what you ran on. Here are 7 others if you want to swap.").

### 8. Vote capture via EO self-record

Each issue has a position toggle (Yes / No / Abstain / Undecided). Optional pre-meeting, updatable during discussion, expected after. Every change is timestamped so the recap can narrate position evolution. No automated capture from minutes in V1 because public minutes lag up to a month. The toggle's optional nature satisfies the May 6 morning concern about pre-meeting position capture being premature: the affordance exists throughout the flow rather than as a forced step.

### 9. Segmentation locked for V1

- 35% with published agendas: Document mode plus Live mode
- 65% without published agendas: Document mode only, with an empty Robert's Rules template option
- No Live mode fallback for the 65% in V1

## Constraints from May 6 (must respect)

- PISS reframe: Issue is the unit of value, not artifacts like meetings or recaps
- Document UI direction: no card UI, no nested accordions, Notion / Google Docs aesthetic
- Drop "campaign" from EO-facing copy; use "issues" or "governance priorities"
- Robert's Rules format is the agenda backbone
- Trust ladder: GoodParty↔EO loop earns the right before EO↔constituent loop
- Recording is optional for a willing minority, not in V1 scope; flag as future enhancement
- 1:1 constituent communication is out of scope
- Pre-meeting position capture as a forced step is OUT; position toggle as an optional affordance is IN

## Open questions for refinement

1. **Swap UX:** How does the EO see the 7 alternatives? What is the interaction model (modal, inline list, drag, side panel)? How does the draft re-render so it does not feel clunky? This is the highest-risk UX detail because the architecture is approved only if execution is clean.
2. **Position toggle UI:** Where does it live in Document mode versus Live mode? How does the timestamp surface (always visible, hover, audit log)? How does change history render in the recap?
3. **Edge case, zero platform matches:** What does Document mode show when the EO's platform yields no agenda alignment? It still needs to ship a useful briefing.
4. **Edge case, dense agenda:** How does Live mode navigate efficiently when the agenda has 15+ items? Jump-to, search, collapse non-priority sections?
5. **Post-meeting Document state:** Does the document collapse to a third visual state, or does it just accumulate annotations on the existing doc? Implications for the recap composition.
6. **Off-meeting access:** What does the EO see if they open the briefing during a meeting they are not attending live? Likely a manual toggle to Document mode, but specify the default.

## Pointers to load

- `serve-product/Constituent-Communication/2026-05-06-jam-feedback-and-v3-plan.md` (morning leadership jam plan)
- `meeting-notes/digests/2026-05-06-digest.md` (full digest of all four May 6 meetings, including the afternoon reconvene that locked the Document and Live split)
- `CONTEXT.md` (current sprint state, refreshed today)
- `serve-product/strategy/serve-strategy.md` (canonical Serve strategy v8)
- `meeting-notes/granola/meetings/2026-05-06_*` (four Granola files for raw context)
