# Win Onboarding UI Matching Guide

A reference for the v4 constituent-communication prototype. Win is the GoodParty product for independent candidates. Its onboarding flow has been polished and is the look-and-feel v4 should adopt so a single user moves between the two surfaces without a visual seam. Repo: `~/Code/gp-webapp/`. Read-only.

This guide also documents the briefing detail page (already-shipped Serve doc-style rendering) and the poll-send "review then schedule then send" pattern Bryan called out as the tail of the recap notify-constituents flow.

## 1. Visual tokens

All extracted from `~/Code/gp-webapp/app/globals.css` and `~/Code/gp-webapp/styleguide/tailwind-theme.css`.

**Color palette.** Defined as CSS custom properties at `:root`; the prototype should restate them as CSS variables.

```css
/* Brand */
--goodparty-blue:  #0048c2;
--goodparty-red:   #dc1438;
--goodparty-cream: #fcf8f3;

/* Primary (webapp dark indigo) */
--primary:         #242d3d;
--primary-dark:    #0d1528;
--primary-light:   #484e55;
--primary-background: #f7fafb;

/* Secondary (yellow) — Win primary CTA color */
--secondary:       #ffc523;
--secondary-dark:  #dba219;
--secondary-light: #ffe291;

/* Components: input-active = the selection blue */
--components-input-active: #2563eb;  /* selected card border + stepper fill */
--blue-500:        #1b6afc;
--blue-600:        #1351d8;
--blue-700:        #0d3cb5;

/* Base surface and border (used by cards, inputs) */
--base-surface:    #ffffff;
--base-border:     #d4d4d4;
--base-foreground: #0a0a0a;
--base-muted-foreground: #737373;

/* Slate (used by stepper inactive + sticky header borders) */
--slate-100:       #f1f5f9;
--slate-200:       #e2e8f0;
--slate-300:       #cbd5e1;

/* Status */
--success: #30a541; --error: #e00c30; --warning: #ff9800;

/* Issue palette (briefing detail) */
--brand-midnight-50:  #ecf5ff; --brand-midnight-900:  /* dark navy */;
--brand-halo-green-50:#ddf2e8; --brand-halo-green-900:/* dark teal */;
--brand-waxflower-50: /* peach */; --brand-waxflower-900: /* dark terracotta */;
--brand-bright-yellow-50:#fffadf; --brand-bright-yellow-900:#93640b; /* tip callout */
```

Dark mode is supported via shadcn/ui `[data-slot]` scoping but Win onboarding renders in light mode only. The prototype can ignore dark-mode pairs.

**Typography.** Body font is Outfit (loaded via `next/font` at `app/layout.tsx:12`, exposed as `--outfit-font`). Fallback stack: `-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif`.

| Use | Class | Resolved size |
|---|---|---|
| H1 (step title) | `text-4xl font-bold sm:text-5xl` | 36 / 48 px, weight 700, tracking -0.02em (set on `h1, h2` in globals) |
| H2 (briefing hero) | `text-2xl font-semibold` | 24 px, weight 600 |
| H3 (issue headline) | `text-lg font-semibold leading-7` | 18 px, line-height 28 px |
| Step description | `text-lg sm:text-base text-muted-foreground` | 18 / 16 px |
| Body | `text-sm leading-normal` | 14 px |
| Eyebrow / label caps | `text-xs font-semibold uppercase tracking-wide` | 12 px, semibold, all-caps, letter-spacing 0.05em |
| Caption | `text-xs text-muted-foreground` | 12 px |
| Button text-large | `button-text-large` (Outfit 500, 16/24, ls 0.025em) | per `styleguide/typography.css:35-48` |

**Spacing.** Container max-width is `max-w-4xl` (56rem / 896 px). Side padding is `px-4 sm:px-8`. Vertical rhythm between sections inside a step is `space-y-8` (32 px); inside a section, `space-y-4` (16 px); inside a card row group, `gap-3` (12 px). Footer is a fixed 80 px bar (`h-20`).

**Border radius.** `rounded-md` 6 px (inputs), `rounded-lg` 8 px (radio cards, alerts), `rounded-xl` 12 px (welcome cards, whyWeAsk aside), `rounded-2xl` 16 px (briefing big card + accordion), `rounded-full` (all CTA buttons, stepper segments, filter pills).

**Shadow.** Base inputs use `shadow-xs`. Buttons inherit shadow from their `--component-button-*` tokens; in practice the rendered Win onboarding has near-zero elevation and relies on borders, not shadow. The prototype should keep shadow minimal.

## 2. Component anatomy

### Stepper (top progress bar)

`styleguide/components/ui/stepper.tsx:1-63`. Used by `OnboardingFlow.tsx:887-892`.

- Variant: `bar`. Equal-width segments laid out as `display: grid` with `grid-template-columns: repeat(N, minmax(0,1fr))` and `gap-3`.
- Each segment: `h-1.5 rounded-full`. Filled (`index < currentStep`): `bg-components-input-active` (#2563eb). Unfilled: `bg-slate-200`.
- Above the bar: a right-aligned `Step {n} of {N}` label, `text-sm font-medium text-muted-foreground`.
- Wrapper is fixed: `fixed top-14 left-0 right-0 z-10 border-b border-slate-100 bg-base-surface`, with inner `mx-auto w-full max-w-4xl px-4 py-4 sm:px-8` (`OnboardingFlow.tsx:885-893`).

For v4: render the stepper as a fixed top bar, equal segments, blue fill on completed segments, slate-200 on remaining. Match the right-aligned counter.

### Footer (sticky CTA bar)

`OnboardingFlow.tsx:982-1007`. Note: the older `app/shared/stepper/StepFooter.tsx` is still used by some flows (poll onboarding) and pairs `variant="ghost"` Back with `variant="secondary"` (yellow) Next. The newer Win onboarding uses `variant="ghost"` Back with `variant="default"` (dark indigo) primary.

- Wrapper: `fixed inset-x-0 bottom-0 border-t border-base-border bg-base-surface`.
- Inner: `mx-auto flex h-20 w-full max-w-4xl items-center justify-between px-4 sm:px-8`.
- Order: Back left, primary right.
- Primary `<Button variant="default" size="large">` resolves to `h-12 px-6 py-3 rounded-full`, dark indigo background, white text. Disabled: opacity drops, no hover.
- Back `<Button variant="ghost" size="large">`: transparent background, primary color text, no border. Disabled when `previousStep` is null.

For v4: ghost back left, dark-indigo pill primary right, both `h-12 px-6 rounded-full`. Disable primary when validation fails.

### RadioCardGroup and RadioCardItem

`app/onboarding/components/RadioCardGroup.tsx` wraps `styleguide/components/ui/radio-group.tsx:38-71`.

- Group: `RadioGroup` is a `grid gap-3` of items.
- Item is a `<Label>` styled as a card: `flex cursor-pointer items-start gap-3 rounded-lg border border-base-border bg-base-surface p-3 transition-colors`.
- Selected state via CSS `has()`: `has-[[data-state=checked]]:border-components-input-active has-[[data-state=checked]]:ring-1 has-[[data-state=checked]]:ring-components-input-active`. The whole card gets a 1px blue ring plus blue border; background does not change.
- Internal radio dot: `aspect-square size-4 rounded-full border bg-white`. When checked: `data-[state=checked]:border-4 data-[state=checked]:border-components-input-active` (no center dot; the thick blue ring is the indicator).
- Title: `text-base font-medium text-foreground`. Description: `text-xs text-muted-foreground`. Stack with `gap-0.5`.

For v4: replicate as a plain HTML radio with a sibling label. Use `:has()` if you can target modern browsers, otherwise toggle a `data-checked` attribute via JS and apply the same border + ring.

### whyWeAsk aside

Defined inline in `OnboardingFlow.tsx:201-212` and positioned in the layout at `OnboardingFlow.tsx:954-977`.

- On `md+`: floats to the right of the form using `md:fixed md:top-36 md:w-[280px]` with `right: max(2rem, calc((100vw - 56rem)/2 + 2rem))`. The grid swaps to `md:grid-cols-[minmax(0,1fr)_280px]` only when `whyWeAsk` exists.
- On mobile: stacks below the form (single column).
- Card: `rounded-xl border border-base-border p-5 flex flex-col gap-2`.
- Title row: `text-xs font-semibold tracking-widest text-muted-foreground uppercase` (default copy: "Why this matters").
- Body: `text-sm text-foreground`.
- No icon, no background tint, no indent. Just a bordered box.

For v4: same treatment. On wider screens, two-column with the aside pinned right; on mobile, stack.

### OfficeSelectionStep (give-and-get input)

`app/onboarding/components/OfficeSelectionStep.tsx`. The "give" is the zip code; the "get" is the list of offices.

- Outer: `space-y-6 text-left`.
- Zip input is an `InputWithButton` (label "Zip code", inline "Search" button, numeric inputMode, 5-digit pattern).
- Below the input, results render as a `RadioGroup` of `RadioCardItem`s, grouped by year. Year header: `flex items-center gap-3` with text label and a `h-px flex-1 bg-base-border` divider (a horizontal rule that fills remaining row width).
- Above the list, a count line: "{n} offices showing. Please select yours." in `text-sm text-muted-foreground` (line 407-411).
- Empty state: `rounded-xl border border-dashed border-base-border px-4 py-8 text-center text-base text-foreground` (line 192-195).
- Skeleton: stack of `h-19.5 w-full rounded-md` skeletons with a row of pill skeletons above.
- "I don't see my office" escape hatch: `<Button variant="link" size="small">` centered below results.

For v4 audience picker: mirror this exactly. Zip-style input on top, count label, list of selectable cards grouped by section header with horizontal-rule dividers, a "this list looks wrong" link below.

### ManualOfficeEntryStep

`app/onboarding/components/ManualOfficeEntryStep.tsx:38-120`. A plain stacked form: each field is a `flex flex-col gap-1.5` block with `<Label>` then `<Input>` or `<Select>`. Outer is `space-y-6 text-left`. Labels are sentence case, no asterisk for required (validation-time errors only).

### CTA buttons

`styleguide/components/ui/button.tsx:12-44`.

- All variants are `rounded-full` pills with `border` and consistent height-by-size: `xSmall h-6`, `small h-8`, `medium h-10`, `large h-12`. Padding scales: large is `px-6 py-3`.
- `variant="default"`: dark indigo (`button-primary`), white text. Win onboarding primary.
- `variant="secondary"`: yellow (`#ffc523`) with dark text. Used by older poll onboarding's `StepFooter`.
- `variant="ghost"`: transparent, dark text, no visible border. Back button.
- `variant="outline"`: transparent, dark text, dark border. Used for Download in briefing detail (custom Tailwind classes there: `border-foreground rounded-full px-8 h-10`).
- `variant="link"`: underlined link styled as a button.
- Hover increases opacity / darkens via component tokens. Disabled drops to `opacity-disabled` token.
- Loading: spinner replaces icon; `data-loading="true"` keeps text color readable.

For v4: dark-indigo pill primary, ghost back, yellow only if you want to mirror the older Serve poll-send flow. Pick one.

### PriorityIssueCard (briefing)

`app/dashboard/briefings/[date]/components/PriorityIssueCard.tsx`.

- No outer card wrapper; sits inside the briefing page's big rounded-2xl card with siblings separated by the parent's flow.
- Padding: `px-6 py-4`.
- Issue badge: a 16x16 `rounded-sm` colored square with the issue number in white, paired with an uppercase eyebrow "Priority Issue {n} of {total}".
- Headline: `text-lg font-semibold text-card-foreground leading-7 mb-3`.
- Body sections stack with `flex flex-col gap-3`:
  - Agenda Item: small uppercase eyebrow in issue color, then sentence body `text-sm`.
  - Before {weekday}: same pattern.
  - "Say this in the room" callout: `px-6 py-4 border-l-4` with `border-l-{color}` and `bg-{color}-50` (color cycles through midnight, halo-green, waxflower per `issue-colors.ts`). Body is italic.
  - Tip callout: same shape but always `border-l-brand-bright-yellow-900 bg-brand-bright-yellow-50` regardless of the issue color.
- Feedback row: "Was this helpful?" with thumbs-up / thumbs-down inline icon buttons, replaced by a thank-you message after click.
- Footer CTA: `Read the full briefing` as a small dark-blue pill (`bg-blue-600 text-white rounded-full px-6 h-10`) with right arrow icon.

For v4 recap rendering: reuse this shape one-for-one. Issue palette cycles every 3 items.

### FullAgendaAccordion

`app/dashboard/briefings/[date]/components/FullAgendaAccordion.tsx`.

- Outer: `bg-card rounded-2xl border border-border overflow-hidden`.
- Header button: `flex w-full items-center justify-between px-4 sm:px-8 py-5 text-left`. Left side combines title "Full Agenda" + ` ${count} items including ${first-summary-token}`. Right side is `LuChevronDown` / `LuChevronUp`.
- Open state: `border-t border-border` separator, optional italic summary, then a `divide-y divide-border` list. Each item has an optional monospace number in muted color (`min-w-8`), the title, and an optional `Priority` pill (`rounded-sm bg-brand-midnight-50 px-1.5 py-0.5 text-[10px] font-semibold text-brand-midnight-700`).
- Item numbering uses the agenda's native string (e.g., "1.", "2.a"); render as-is, no transformation.

For v4 if the recap surfaces "items not on the priority list": use this accordion shape so the parity carries.

### PreviewStep + PickSendDateStep + PollPreview (notify-constituents tail)

The poll onboarding's review-schedule-send flow is the pattern Bryan called out.

- `PickSendDateStep.tsx`: centered layout, H1 "When should we send your poll?" (`text-2xl md:text-4xl font-semibold`), supporting paragraph in muted text, then a `PollScheduledDateSelector` in `w-full items-center flex flex-col gap-8 mt-8`.
- `PreviewStep.tsx`: H1 "Review your SMS poll" (same H1 treatment), supporting paragraph, then `<PollPreview>` which stacks two `MessageCard`s.
- `PollPreview.tsx`: first card `Outreach Summary` lists Audience, Send Date, Estimated Completion, Cost as a bulletless list of `text-sm font-normal` lines with bold values. A `note` line below: "You can add more recipients after launch." Second card `Preview` renders a `TextMessagePreview` (an iMessage-style bubble) constrained to `max-w-xs`.
- Edit affordance: in this older flow, edits happen by going Back through the stepper. There is no per-row edit pencil. The footer's `StepFooter` is the navigation contract.

For v4 recap notify-constituents tail: split into two screens to mirror Bryan's "review then schedule then send."

1. **Review.** Recap content card + an Outreach Summary card (Audience size, channel, cost, estimated completion). Stacked, single column, max-width matches step body.
2. **Schedule.** Date picker centered with one supporting line of copy.
3. **Send.** Inline confirmation on the schedule screen's primary CTA ("Schedule send" or "Send now"), then a success state. Do not introduce a separate confirmation page; the primary button completes the flow.

## 3. Microcopy patterns

**whyWeAsk formula.** Lead-in noun phrase + reason + concrete benefit. Uses "you" and "we" interchangeably. Examples from `onboardingConfig.ts`:
- "Knowing whether you're already on the ballot lets us tailor your timeline and the next steps in your campaign plan."
- "We use this to find the district you're running in, pull registered voter data, historical voter turnout, partisan data, and local issues to build your campaign plan."

**Step headlines.** Question form, conversational. "Are you already on the ballot?" "What office are you running for?" "When should we send your poll?" Plain English, contractions allowed, no jargon.

**Step descriptions.** One sentence, declarative, value-forward. "We tailor your strategy to where you actually are in your campaign." "We'll use this to analyze local voter data, trends, & news to create your campaign plan."

**Button labels.** Primary advances with `Continue` until the final step, where it becomes a verb-noun action: `Agree & Create My Plan`. Secondary actions are short and verb-led: `Search`, `Back`, `I don't see my office`, `Read the full briefing`.

**Empty states.** Helpful and actionable, not apologetic. "Enter your zip code to see offices." "We couldn't find any offices for that ZIP. Try a different ZIP or enter your office manually below."

**Error messages.** Plain, action-oriented. "We couldn't load offices for that ZIP code. Try again." Snackbars surface API failures, never modals.

## 4. Layout patterns

- Top-of-page sticky stepper bar (h ~ 60 px) + main content area `max-w-4xl mx-auto px-4 sm:px-8 pt-28 pb-6` + bottom sticky footer bar (`h-20`). Total page is `min-h-screen bg-base-surface`.
- Two-column on `md+` only when `whyWeAsk` exists: `grid-cols-[minmax(0,1fr)_280px] md:items-start`. Otherwise single column.
- Welcome step is the only `text-center` variant; every other step is left-aligned.
- Briefing detail page uses a different shell: a sticky page header, then a gray content area `bg-muted px-4 py-8 sm:px-8 md:px-16 lg:px-32`, with content centered at fixed `width: 680px`. The big card is `bg-card rounded-2xl border border-border overflow-hidden`.

## 5. Interaction patterns

- **Forward / back navigation.** Footer-anchored. `goBack` returns to previous visible step; `goNext` validates, persists, then advances. `OnboardingFlow.tsx:486-860`.
- **Auto-save.** Each step writes its answer to campaign state on advance via `updateCampaign(...)`. There is no explicit "Save" button.
- **Validation timing.** Per-step `isValid({ answers })` runs continuously; the primary button is disabled until it returns true. No inline error text on individual fields during typing; errors appear on submit (zip not 5 digits) or when the API returns an error (snackbar).
- **Skip and conditional steps.** `shouldSkip({ answers })` removes a step from the visible sequence (e.g., manual-office path skips `path-to-victory` and `voter-demographics`). The stepper recomputes total steps based on the visible set, so skip changes are reflected immediately.
- **Loading states.** Skeletons (matching the final shape) for race lookups; spinners inside buttons via `loading` prop on `<Button>`. Page transitions auto-scroll to top (`useEffect(() => window.scrollTo(0, 0), [activeStepId])`).
- **Tracking.** Every step transition fires an `EVENTS.Onboarding.*` Amplitude event with relevant context. The v4 prototype does not need to implement tracking, but the schema (one event per significant transition with object props) is the GoodParty house style.

## 6. Two surfaces, same look-and-feel

To make v4 feel like Win onboarding, the prototype must adopt these specific tokens and components:

- Outfit body font with the system fallback stack. Apply tracking `-0.02em` to h1 and h2.
- The exact CSS variables in section 1, with `--components-input-active: #2563eb` driving stepper fill and selected-card ring.
- Top stepper bar: equal-segment grid, `h-1.5 rounded-full`, blue fill on completed segments, slate-200 on the rest, with right-aligned "Step n of N" label above. Fixed-position, full-width, `border-b border-slate-100`.
- Bottom footer bar: fixed, `h-20`, `border-t border-base-border`, `max-w-4xl` inner, ghost Back left and dark-indigo pill primary right. Both `size="large"` (h-12 px-6).
- Content area: `max-w-4xl px-4 sm:px-8`, `pt-28 pb-6`, `space-y-8` between sections.
- H1 for each step: `text-4xl font-bold sm:text-5xl`. Description below: `text-lg sm:text-base text-muted-foreground`.
- Selectable cards (audience, channel, schedule presets): RadioCardItem shape — `rounded-lg border p-3`, blue 1px ring + blue border on selection, no fill. Title `text-base font-medium`, description `text-xs text-muted-foreground`.
- Whenever a step prompts for data, render an aside on `md+` titled with the small uppercase eyebrow "Why this matters", same `rounded-xl border p-5` treatment, single short paragraph in `text-sm`. On mobile, stack below.
- All CTAs are `rounded-full` pills with the variant + size taxonomy from `button.tsx`. No square corners on buttons anywhere.
- Recap content rendering reuses the briefing detail card structure: a single `rounded-2xl border` card with stacked priority sections inside, each using the issue-color rail + "Say this in the room" callout pattern. The bright-yellow tip callout is reserved for tactical advice.
- Items beyond the priority list collapse into the FullAgendaAccordion shape, not a separate page.
- Notify-constituents tail uses the poll-send pattern: review screen with stacked summary card and content preview, then a schedule screen with a centered date picker, then send completes inline.
- Microcopy follows the question-headline + value-forward-description + plain-language-CTA convention. No "Welcome!" preambles; jump straight into the question or the action.

When all of the above is true, a constituent comm v4 step and a Win onboarding step should be visually indistinguishable apart from content.
