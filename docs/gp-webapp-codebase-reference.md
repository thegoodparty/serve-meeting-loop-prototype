# gp-webapp Codebase Reference for v4-loop

**Date:** 2026-05-07
**Audience:** Claude agent team building the v4 constituent-communication prototype in a parallel terminal.
**Companion docs:** `win-onboarding-ui-reference.md` (visual and interaction tokens), `../2026-05-06-meeting-loop-architecture-handoff.md` (nine locked architecture decisions).

---

## Purpose

This is the gp-webapp ground truth for the v4 constituent-communication prototype. Use it to align visual language with the existing Win product onboarding, reuse components rather than reinvent them, and avoid spawning a third onboarding scaffold. The companion `win-onboarding-ui-reference.md` (sibling file) carries the visual and interaction tokens; this doc covers architecture, reusable patterns, and recommendations.

---

## Why this exists (scoping quotes)

At the May 6, 2026 13:30 PT leadership reconvene (Grain meeting id `dadf47da-3c95-4422-8f5b-2569cda71eac`), Žak Tomich and Bryan Levine both pointed at gp-webapp as the source of truth.

> **Žak (~28 min):** "We just did an onboarding that we're excited about for Win. Is there any reason this wouldn't look and feel like that? Why would it be two different experiences for the same user?"

> **Bryan (~70 min):** "We actually have all this in a lot of the Serve flows already, like the flow that we're describing and you send a poll. We do very similar stuff. We probably just need to make sure there's consistency there where, okay, here's what we think you should send. Is this good? Yes. Okay, when do you want to send it? Great."

> **Bryan:** "Kaylee's done some designs like that before."

The ask is two-fold: (1) match Win onboarding look and feel, (2) reuse the existing Serve poll-send "review, schedule, send" pattern for the recap notify-constituents tail.

---

## Tech stack

- Framework: Next.js 15 (App Router), React 19, TypeScript
- Styling: Tailwind 4, Radix primitives wrapped in `@styleguide`
- Forms: React Hook Form + Zod
- Data: TanStack Query
- Auth: Clerk
- Instrumentation: Amplitude + Sentry
- Stepper, RadioCardItem, Card, Progress, Sheet, Drawer, Tabs already abstracted in `styleguide/components/ui/`

---

## Repo and clone

- GitHub: `https://github.com/thegoodparty/gp-webapp`
- Local clone: `~/Code/gp-webapp/`

---

## Reusable patterns

| v4 piece | Existing component | File path | Adapt notes |
|---|---|---|---|
| Multi-step linear flow with progress | `OnboardingFlow` + `Stepper` (variant=bar) + `StepFooter` | `app/onboarding/components/OnboardingFlow.tsx`, `styleguide/components/ui/stepper.tsx`, `app/shared/stepper/StepFooter.tsx` | Define v4 steps in the `onboardingConfig.ts` shape (`id`, `title`, `whyWeAsk`, `isValid`, `shouldSkip`) |
| Give-and-get friction (capture meeting dates, unlock briefing) | `OfficeSelectionStep` plus `whyWeAsk` aside | `app/onboarding/components/OfficeSelectionStep.tsx`, `OnboardingFlow.tsx:201` | Swap office-search internals for meeting-date capture, keep aside copy structure as the "why we ask" explainer |
| Position toggle (Yes / No / Abstain / Undecided) | `RadioCardGroup` + `RadioCardItem` | `app/onboarding/components/RadioCardGroup.tsx` | Use as is with four options. Capture timestamp on selection in parent state, similar to `OnboardingAnswers` |
| Document mode briefing rendering | `BriefingDetailPage` + `PriorityIssueCard` | `app/dashboard/briefings/[date]/components/BriefingDetailPage.tsx`, `PriorityIssueCard.tsx` | Direct reuse. PriorityIssueCard already renders a rounded-2xl card with issue-colored "Say this in the room" rail and tip callout |
| Robert's Rules agenda accordion | `FullAgendaAccordion` | `app/dashboard/briefings/[date]/components/FullAgendaAccordion.tsx` | Use as is. Already numbered, hierarchical, priority-flagged |
| Recap "review, schedule, send" tail | `PreviewStep` + `PickSendDateStep` + `PollPreview` | `app/polls/onboarding/components/steps/PreviewStep.tsx`, `PickSendDateStep.tsx`, `app/dashboard/polls/components/PollPreview.tsx` | Adapt to recap content. Bryan called out this exact shape |
| Step state persistence | `OnboardingProvider` + typed `OnboardingAnswers` | `app/polls/contexts/OnboardingContext.tsx`, `app/onboarding/components/onboardingTypes.ts`, `app/onboarding/shared/ajaxActions.ts` | Reuse the `data.onboarding` partial-write pattern |

---

## What to build fresh

- **Document and Live mode toggle:** use `Tabs` from styleguide as the primitive. Small.
- **Swap affordance on `PriorityIssueCard`:** PriorityIssueCard already has thumbs state; extend that interaction. Small.
- **Position-history strip with timestamps:** new state shape. Small.
- **Composed-recap fingerprint widget:** "Built from your briefing, [N] position changes, and [N] notes." Net new. Small.
- **Long-press or inline edit on draft content for the swap UX:** not in gp-webapp. Build fresh. Load-bearing.

---

## Recommendation

Extend the Win onboarding scaffolding at `app/onboarding/`, not the Polls onboarding at `app/polls/onboarding/`. The Win flow is newer, uses the unified `Stepper`, has the `whyWeAsk` aside that matches Žak's give-and-get framing, and uses `RadioCardGroup` cleanly. The Polls flow is older and uses a separate `StepIndicator`.

Reuse `PreviewStep` + `PickSendDateStep` + `PollPreview` for the recap notify-constituents tail. This is Bryan's explicit reference and avoids reinventing the audience-selection, schedule, and send composition.

---

## Anti-patterns

- Do not spawn a third onboarding scaffold. Two already exist (Win, Polls); we are picking Win and consolidating.
- Do not rebuild `BriefingDetailPage`, `PriorityIssueCard`, or `FullAgendaAccordion`. These render in production.
- Do not reinvent audience selection. `PollPreview` already covers it.
- Do not use rectangle CTA buttons or filled radio cards. Win uses `rounded-full` pills and 1px blue ring selection without fill.

---

## Companion docs

- Visual and interaction matching depth: `win-onboarding-ui-reference.md` (sibling file in this folder).
- Architecture handoff with the nine locked decisions (Pattern B, modes, segmentation, swap, fingerprint): `../2026-05-06-meeting-loop-architecture-handoff.md`.
