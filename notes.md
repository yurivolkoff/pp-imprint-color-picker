# Prototype Notes — imprint-color-picker
_Date: 2026-05-19 | Page: PDP / "Sending us artwork" modal_

## Task
Component prototype for the imprint color picker that lives inside the artwork-upload modal: pick N colors one-by-one (method-driven max), choose from a curated standard palette, and optionally upgrade individual slots to a Pantone match at +$20 setup per color.

## Source
- Live observed picker: https://admin.promotionpros.com/5-panel-mesh-back-price-buster-cap → Customize → Imprint Details → Upload artwork (observed 2026-05-19)
- Research deliverable: `research/reports/2026-05-19-imprint-color-picker-research.md`

## Design Library
`design/library/promotionpros-pdp.md` — extracted 2026-03-20. Tokens (Lato, navy `#001542`, orange `#FE5000`, body `#525252`, Bootstrap 5 spacing) carried into the prototype `:root`. Library is 60 days old (older than the 14-day rule); deviation justified by: (a) live site is Cloudflare Turnstile-gated so re-extraction would fail without manual capture, (b) nopCommerce theme tokens have not changed in any branch Yuri has surfaced, and (c) this prototype is component-scoped, not page-scoped, so font/color tokens are the relevant slice and they're authoritative.

## Key Design Decisions

- **Per-slot tier toggle** (`Standard / PMS`) replaces the current global PMS checkbox. Rationale: PP buyers routinely mix tiers ("1 standard black + 2 brand-color Pantones") — a global flag can't express that and forces a worst-case upcharge or a worse-case undercharge. Matches the per-color "Edit Colors" model in CustomInk's Design Lab.

- **Slot pattern with `+ Add color (X of Y)`** caps at the active decoration method's max. Replaces the disconnect today between the outer "Number of Imprint Colors" dropdown and the inner `Add Additional Imprint Color (X/4)` button, which can desync.

- **Swatch grid in standard popover, Pantone-code search in PMS popover**. Two distinct interaction patterns matched to two distinct user behaviors: standard buyers recognize by hue, PMS buyers know their code. Avoids forcing one interaction onto both populations.

- **Curated standard palette** — removed Pantone-named colors ("Reflex Blue", "Process Blue", "Rhodamine Red"), removed "Full Color" and "No Colors" (those are decoration modes). The standard list is now ~36 cleanly-named hues across 7 family groups. This is the single biggest fix for the current picker's tier-boundary confusion.

- **Live price impact** — `Pantone match (N × $20)` line item appears in the order summary the moment a slot is upgraded to PMS, with a colored highlight in the breakdown. Customers see the cost before committing to it, not as a post-checkout surprise.

- **Method-aware behavior** — picker reads the decoration method and adapts: max slots (Embroidery 12, Screen Print 6, Transfer 4, 3D Embroidery 12, Pad Print 1), PMS support (`true` for print methods, `thread` for embroidery with explicit "matches closest Madeira/Isacord thread" disclaimer, would be `false` for engraving methods not in this demo). On method downgrade, excess slots are trimmed with a toast; PMS slots auto-downgrade if the new method doesn't support it.

- **Pantone calibration disclaimer** in the PMS popover header. Pantone's own UX guidance: physical Pantone swatches are authoritative, screen rendering is approximate. Disclosure protects the buyer's expectation and PP's downstream artwork-confirmation conversation.

- **Selected-colors chip row** above the slot list, with running summary ("2 standard · 1 PMS · +$20"). One-glance recap that survives popover dismissal. Mirrors the pattern Pantone's own Color Finder uses for multi-selection.

- **Artwork instructions** kept in the modal but **explicitly scoped** to placement/sizing/orientation via inline help text ("Color goes in the picker above."). Removes the today's back-channel where buyers type Pantones into the free-text field and bypass the structured picker.

- **Empty-state microcopy** — "Pick a color or skip — our specialist will confirm with you." Preserves the optionality of the picker (per Yuri's reframe) while reducing the cognitive friction of "do I have to do this now?"

- **Demo control strip** above the modal lets reviewers cycle through states (empty / one standard / mixed / max / Embroidery / Pad Print downgrade) without manually constructing each. Not for production.

## Baymard Compliance

| Rule ID | Rule | Pass/Fail | Notes |
|---------|------|-----------|-------|
| B-PDP018 | Reproduce core content across variations | N/A | Component-only; no cross-variation content here. |
| B-PDP019 | Use swatches (not dropdowns) for color variation | PASS | Standard popover renders an 8-col swatch grid with hover tooltips; current picker uses a text dropdown. |
| B-PDP041 | Complex multi-variable customization should be a separate post-cart process | DEFERRED | Picker still lives in the pre-cart artwork modal per Yuri's "picker only" scope. Reframing the entire decoration flow to post-cart is the broader PDP Stage 2 question, not this component. Flagged for epic discussion. |
| B-PDP042 | When only one option exists, remove the selector | PASS | Pad Print method drops slot cap to 1 and the `+ Add color` button disables; the picker collapses to a single slot. |
| B-PDP013 / #36 | Display price per unit / make price implications clear | PASS | Live breakdown lists each setup, run-charge, and PMS surcharge as separate lines; per-unit price visible. |
| B-PDP021 (family) | Mention price additions for variations | PASS | Each PMS slot shows `+$20` inline on the trigger; chip row totals; breakdown line item. |
| NN/g #1 visibility of system status | — | PASS | Method-cap badge, slot counter, live breakdown, toast on method downgrade. |
| NN/g #4 consistency | — | PASS | Pantone names confined to PMS tier; standard list curated to remove ambiguous Pantone-named entries. |
| NN/g #5 error prevention | — | PASS | Free-text artwork instructions explicitly scoped away from color spec ("Color goes in the picker above"); method change with PMS downgrade is communicated before committing. |
| NN/g #6 recognition over recall | — | PASS | Swatches with hover labels in standard tier; chip row showing selections persists between popover opens. |
| WCAG 2.1 AA color contrast | — | PARTIAL | Disclaimer banner and primary text meet AA. Swatch tooltips on hover only — keyboard users see selected name only after picking. Improvement: persistent labels under swatches or full keyboard list view for accessibility audit. Logged for follow-up. |

## Competitor Summary

| Competitor | Their Approach | PP Delta |
|------------|----------------|----------|
| CustomInk | 43 standard ink colors, Pantone per-color via "Edit Colors" in Design Lab, $20/match, 1 free per $400 spent | PP matches the $20 spec and the per-color granularity. Doesn't replicate the "$400-spend = free PMS" loyalty mechanic — flagged for Mike/Emily as a possible sales-pricing lever later. |
| 4imprint | "As close as possible" free via the artwork-comments field; exact PMS negotiated per-order with customer care | PP's picker provides a structured path the buyer can use without an email round-trip, while preserving the "skip — specialist will confirm" escape hatch via empty-state copy. |
| AnyPromo / Quality Logo Products / Executive Advertising | Checkbox-expands-palette pattern: standard dropdown by default; checking "PMS Match" reveals Pantone options with fee | PP's per-slot toggle is a more granular and discoverable take on the same concept. The checkbox-expand approach can't represent mixed tiers per color, which is the common case. |
| Pantone Color Finder | Search by code / family / hex with chip + name + code + values | PP picker's PMS popover replicates the search interaction; uses a hex approximation (not a Pantone Connect API license) plus the calibration disclaimer to bound the expectation. |

## Known Deviations from Live Site

- **Picker is now a primary structural element of the modal**, not buried after the upload control. On the live site, the dropdown sits below the uploader and is easy to scroll past. Prototype puts uploader → picker → instructions in a clear hierarchy.
- **Two-column modal layout** with live price summary on the right; live picker has no in-modal price feedback at all. Adds 280px of width — fits comfortably in the 880px modal.
- **Method selector inside the modal** for prototype demo purposes — in production this lives upstream in the configurator and the picker just reads from state. The prototype duplicates it so reviewers can demo the method-aware behavior without exiting the modal.
- **Curated palette** (~36 standard colors vs. live's 29 with 2 mode entries). Counts are close; the curation is in *which* names made the cut, not how many.
- **PMS dataset** is ~70 entries representative of common brand colors. Production would source the full Solid Coated + Uncoated + Neons + Metallics tables (~2,000 Coated + ~1,400 Uncoated + extras) from an open Pantone-to-hex lookup.

## Open Questions for Next Iteration

1. Should "skip — specialist will confirm" be an explicit button/checkbox, or remain implicit via the empty-state copy?
2. Should the standard popover include a "request a swatch sample" CTA for buyers who want physical reassurance before committing?
3. Loyalty/volume softener on PMS surcharge (CustomInk's "1 free per $400") — worth exploring with Mike/Emily for promo-pricing strategy.
4. Accessibility: persistent under-swatch labels vs. tooltip-only is a tradeoff between density and clarity — needs a11y audit to decide.
5. Engraving / debossing / laser methods — confirm with Boris whether they should hide the picker entirely or show a "color N/A" state.

## Change Log
- 2026-05-19: Initial prototype. Slot pattern, per-slot tier toggle, curated standard palette, PMS search, method-aware caps, live price summary, calibration disclaimer, chip row summary, demo state controls.
- 2026-05-20: **v2 restructure** in response to Yuri review.
  - **Slots → chips.** Removed the per-color slot rows entirely. Each selection is now a single chip (swatch + name/code + optional `+$20` + ×) in a flex-wrapping row. A single `+ Add color` button sits inline with the chips. Chips are editable (click to reopen the popover pre-selected). Picker section is ~1/3 the vertical height of v1.
  - **Tier choice moved into the popover.** Standard / PMS tabs at the top of the popover header replace the per-slot toggle. Tier decision happens where the new color is chosen, not as a separate state on a slot row.
  - **Modal reframed as the artwork-upload pop-up.** Active decoration-method `<select>` removed from inside the modal. Replaced with a read-only context bar showing Method · PMS support · Preferred colors (X of N max) · "Set on previous step" · `Edit upstream →` link. The user lands here with method + preferred count already decided upstream.
  - **Preferred-count warning banner.** New soft-warning state above the chips: when chips.length > preferred, shows "You preferred N colors. Adding an Nth color raises your price by $X (setup + run-charge + PMS). That's fine — just so you know." Explanatory, not blocking. Method's actual max is still the hard cap.
  - **Price summary now shows delta vs. preferred-count baseline** (e.g. "+$305 vs. 3-color baseline") — gives the buyer a direct read of "what does going over preferred actually cost me?"
  - **Demo controls reorganized** — Method and Preferred Colors are now dropdowns (instead of buttons), to demo upstream-input variation; quick states updated to `Empty / 1 of 3 / 3 of 3 / 4 of 3 (warning) / Mixed / Reset`.
  - **CSS fix:** added `[hidden] { display: none !important }` to override class-level `display: flex` (was leaking PMS filter chips into the Standard tab).
  - All other behavior (swatch grid for standard, PMS code search with fuzzy match, method-aware caps + PMS support semantics, embroidery thread-match disclaimer, downgrade toasts, calibration disclaimer) carried forward unchanged.
- 2026-05-20: **v3 — Pattern A inline picker** in response to Yuri's concerns about click cost / eye-travel and popup-on-popup layering.
  - **Popover removed.** The picker is now an inline section that expands below the chip zone — visually attached, no floating surface, no z-index layer. Zero popup-on-popup. Single surface end-to-end.
  - **Sticky open across multiple picks.** Click `+ Add color` once → picker opens. Click swatches / PMS rows to append chips. Picker stays open. New chips animate in (scale + opacity pop). Close with the **Done picking** button, `Esc`, or by focusing the artwork-instructions textarea (natural progression signal).
  - **Click cost for 3 colors dropped from 6 → 4** (1 Add + 3 picks vs. v2's 3×(Add+pick)). Eye-travel is one vertical column inside the picker section rather than a popover-button zigzag.
  - **Add-color button doubles as picker toggle.** Filled navy state when picker is open. Click again to close. Mirrors Notion/Linear tag-picker pattern.
  - **Edit mode.** Click an existing chip → picker opens with that chip's tier preselected and current color highlighted; picking *replaces* the chip; after replace, picker auto-switches to add mode (so next click appends, not replaces). Mode label at top of picker explicitly shows "Replacing [chip]" vs. "Add color · N of M used".
  - **Auto-close at max.** Picking the last allowed color (when chips.length hits method max) closes the picker automatically — there's nothing left to do.
  - **Wider swatch grid.** Inline picker is the full width of the modal body (~480 px), so the grid runs 10 columns instead of 8, density up ~25%.
  - Visual continuity: chip zone has flat bottom border + picker has flat top border when picker is open, so they read as one component. Subtle shadow under the picker for separation from artwork instructions below.
