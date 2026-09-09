> **Reference for the myiDecide Presentation Builder skill.** Two documents:
> the ELEMENT CONTRACT (what an element is, how its parts group, the slot
> rules) followed by the DESIGN PLAYBOOK (zones, type roles, gaps, colour
> roles). Read when composing or re-aligning slides.

# Element contract — what a slide is made of

Every slide in this deck was built from a **template** that references
**element variations** from a fixed library. Each slide's plan carries a
`BUILT FROM` brief naming exactly which ones. Read it before you judge or
change a layout — it is the difference between a design choice and a defect.

Full schema: `assets/docs/template-schema.md`. Library: `assets/json/elements.json`.

## The one rule

> **Anything that must move or disappear together is ONE element with several
> parts — never two separate things.**

A **slot** is the smallest thing you may address: you can move it, drop it,
count it, animate it. A **part** has no independent existence. A button's label
is not a thing you can position — it is part of the button.

So:

- A label outside its pill is a **defect**, never a style.
- An eyebrow drifting away from its headline is a **defect** — they are one
  `heading.kicker`.
- A photo, its scrim and its label in an `action.tile` are one button. The click
  belongs to the photo layer; the label never carries it.
- A list row's detail belongs to its title (tight gap ~16px). Rows separate from
  each other on the stack gap (~37px). A detail spaced like its own row is a
  defect.

## Grouping is stated, not inferred

Every part carries `idecide/group` + `idecide/groupRole` metadata, written when
the slide is built and preserved through saves and reloads. You never have to
work out which blocks belong together from their positions or sizes — ask.

A `move` acts on the whole element (every sibling by the same delta) and a
`delete` removes the whole element. Deleting a wired button (`btn:`, `btnurl:`,
`btnfinish:`) is refused outright by the build — except on the cover, where a
wired button is the duplicate to remove (next section).

### A subordinate line hangs off the text directly above it (binding, 2026-09-09)

A detail line pairs with its title, not with whatever else shares the
title's row. On a numbered step the row is [big numeral, title] and the
detail sits under the TITLE; spacing it off the row's ink bottom hung it
16 px under the numeral's glyph — a full line below its title — and "Your
laptop / Wherever you work" read as two list items (Duolingo "English Test
- 3", Bren: "make sure these lines are being properly grouped and share
tighter spacing, and then keep the wider spacing between the groups of
paired text"). The packer now pairs a subordinate row with the previous
row's text that shares its horizontal span (the title), and only with the
whole row when nothing does; a plated or ringed row is still spaced by its
box. Pair gap inside a unit (`PAIR_GAP`, 16 px), unit gap between units
(`GAPT`) — the pair reads as one object, the list as its members.

## The cover has exactly one action element (binding, 2026-09-03)

Slide 1 is advanced by the platform on **any click**; it never carries a
button action (verified-platform-facts §13). So the cover is built with
exactly one thing that reads as clickable: the **static** "Click anywhere to
Begin" pill (`btn-static`), which the composer draws itself. Concretely:

- the composer strips every wired item (`target` / `url` / `finish`) from a
  `kind: "cover"` plan before its renderer runs, and never builds the
  bottom-left nav-button row there; the old top-right "accent chip" on the
  cover is gone for the same reason;
- the wire pass skips the first slide of the deck (and any `kind: "cover"`
  plan), and removes an action record it finds on a `btn:`-named block there,
  so decks built before this rule are healed on the way past;
- `setAction` and a targeted `addButton` on the cover are refused with the
  rule spelled out; `delete` of a wired button on the cover is **allowed**,
  because it was never the way forward.

A cover with one static pill is complete. Two pills on a cover is a defect
(three tester builds shipped one — `btn-static` beside a wired `btn:Welcome`
built from the planner's `copy.items`).

## What a slide is made of is recorded, not guessed

Each slide's build writes a ledger — one entry per element, with its id and its
parts — and the log line `built (<slide>): heading.kicker · action.rect×6 ·
contact.stack` says exactly what was made. If you are told an element is
missing, that is measured from the ledger, not inferred from a screenshot.

## Parts are listed bottom → top

That order is the z-order, the build order and the reveal order at once. In
`action.rect [background(click)+icon?+label]` the background is underneath and
carries the click; `?` marks an optional part.

## Anchors

| Anchor | Behaviour |
|---|---|
| `canvas` | Pinned to the slide, outside the flow. |
| `zone.*` | Packed inside that zone. |
| `corner` | Pinned to a corner inset. |
| **`under:<el>`** | **Bound to that slot — moves with it, drops with it.** |
| `stack-foot` / `column-foot` / `above-band` | Sender placement from the layout's own skeleton. |

If a slot is anchored `under:` another, you may not move one without the other,
and you may not keep it when the parent is dropped.

## What you may change when content doesn't fit

In this order — stop as soon as it fits:

```
1  drop:decor          motifs, logo, chip rows
2  drop:support.line   the supporting line, then a kicker's eyebrow part
3  swap:heading.long   move the headline into the 130-char band
4  shrink:heading      within its band
5  yield:media         photo band → 30% floor · side panel → 22% floor
6  scale:group         the WHOLE set together, never one element (28px floor)
7  tighten:rhythm      the stack gap gives LAST
```

**Never dropped, at any rung:** anything marked `REQUIRED` in the brief — every
`action.*` button, every menu row, the headline, the stage field. Navigation is
never sacrificed and a button label is never truncated. If buttons don't fit,
shrink the group, shrink the heading, or narrow the media — in that order.

## Sizing

- `hug` — a solo button/chip measures its own label's ink and fits it.
- `share` — siblings take **one** width (the widest label's) so they read as a
  set. Adding or removing a sibling re-computes that shared width for all.
- `fill` — consumes its zone.

## Alignment

`align` is a **block property** — where the block sits in its zone. It is never
a font setting. Centring a group means moving the blocks, not switching the text
box to centre-aligned.

The whole set is middle-aligned in the room it *actually* has — bounded by the
photo band or side panel, not the canvas.

### The template's arrangement decides the axis (binding, 2026-09-04)

Every variation declares an arrangement — `left-aligned`, `centered` or
`right-aligned` (`skel.arr.align`, lifted into `spec.align`). That axis governs
**everything in the content zone**: text, button groups, Back pills, stat units,
charts, photo-card strips and the sender block. Left stays left, right stays
right, centred stays centred. Only an Edit-step request changes it; a
reviewer who finds a left-aligned layout centred (or the reverse) has found a
defect, not a choice.

Concretely: `text()` takes the declared axis when its caller says nothing (the
old canvas-centre heuristic survives only for plans with no arrangement);
every renderer records its content zone (`zoneSet`); the nav-pill row is built
inside that zone on that axis as ONE set (it is never pinned bottom-left);
`alignAxis` moves whole rows onto the axis — left ink edges to the zone's left
edge, right edges to its right edge, centres to its centre.

### Which side the photo goes (binding, 2026-09-04)

The skeleton's `side` / `cues` say where the photo sits; the spec prose is read
only for a phrase that names the PHOTO's side ("photo left", "portrait right").
"photo left, 3 compact rows right" contains both words — grepping the prose
for "right" first drew a photo-LEFT template mirrored. One helper (`photoLeft`)
answers for every split renderer.

### A deck alternates its layouts (binding, 2026-09-05)

The axis rule above says a template's arrangement is honoured; it does not say
every template should be left-aligned. The library is 314 left / 60 centred /
1 right, so a picker that only avoided repeats let the majority speak and deck
202 came out left end to end. Variety is an objective of the template picker
(`assignVariations`): among the candidates that already pass every hard rule
(capacity, photo, freshness across the last two builds, adjacent signature),
the ones that differ from the previous slide — a different axis, the photo on
the other side, a different layout family — and that pull the deck toward a
mix while its running left share is high are preferred, and the seeded draw
decides between the leaders, so a deck is still reproducible from its
`styleSeed`. Parallel beats (answers, menus) still share one variation. The
build log reports the axis mix ("axis mix: 29 left / 19 centred-or-right").
None of this changes an axis after binding — that is still an Edit request.

### The kind outranks the category (binding, 2026-09-05)

The outline model can bind a slide to any category; the renderer is chosen by
what the slide IS first. A `question`, `menu` or `hamburger` is rendered by K
(wired buttons), a `cta` by N, the `cover` by O — whatever category the
outline named — and `enforceDeckStructure` coerces the category to the kind's
family (question → 4, cta → 15, menu/hamburger → 2, cover → 1) before the
picker runs, so the variation it binds can hold the buttons. The composer
renders any interactive kind through K as a last net. Deck 202's "Finish Up -
3" was a question bound to category 14: it drew as display rows with no
buttons and the outcome fork was a dead end.

Within a category the parsed skeleton, not the category number, picks the
renderer: category 6 with items goes by `arr.itemsMode` (rows → G, cards →
F, chips → H, else A); category 7 by `layout` (column → C photo band,
layered → A over a full-bleed photo, else B side panel); category 3 and 10
already did. A renderer is never asked to draw a stage it has no code for.

## What sits inside a button is centred (binding, 2026-09-04)

The label of a button with no icon is centred across the plate's inner width.
With an icon, the icon + gap + label are paired as ONE object and that pair is
centred inside the plate — on every button: pills, rect rows, stacked menu
rows, K tiles. (Icons in a stacked menu therefore do not line up in a column;
accepted.) Never set a label's text alignment to Left or Right to "align it
with" its icon — the pair is measured (ink width, `Auto` width trick, position
restored) and placed.

## No answer key (binding, 2026-09-04)

A button is drawn in the contrast (featured) colour ONLY when its label is a
way-forward — Move Ahead, Finish Up, Continue, Next — and NEVER on a
`question` slide, whatever the item's target. Every answer on a question wears
the same plate colour: the viewer is not told which one is right. If buttons
on a slide differ in colour, either exactly one is the way-forward button or
every button has its own colour by design.

## Buttons only where the viewer must leave by hand (binding, 2026-09-04)

On a content slide that auto-advances, the items in `copy.items` are
**display** — photo cards, rows, chips — paired with the script. Their targets
are dropped and logged ("rendered as display — the slide moves on by itself").
Pills are built only on a slide the viewer must leave: `autoAdvance:false`, or
a sub-fork with its Back item. Category 14 explainers (renderer P) draw their
items as `paneltile.photo` tiles: photo + scrim + icon + label, one object,
never a `btn:` name. When such a template names `action.rect` REQUIRED, the
display units built from the same items satisfy the contract and the audit
says so.

## Graphic elements sit in the flow like anything else (binding, 2026-09-04)

SVG icons, infographic parts, charts, images, lotties and videos follow the
template's spacing and positioning rules and are grouped with the text they
belong to — they are never dropped at a fixed spot after the renderer runs.

- **`stat.ring` is one unit**: ring graphic + figure + label. The figure is
  drawn first and MEASURED; the ring is sized to hug that ink (≥180px, ≤ the
  zone; the digits use ≤78% of the diameter), and when even the largest ring
  cannot hold the digits the type comes down (floor 64px) — the figure is the
  content, the ring is its frame. The ring is inserted BEHIND the digits; the
  label sits under the ring on the slide's axis. The ring chart is uploaded
  without its own label/caption (the figure IS the label) and with a track
  visible on its field (ink at 12% on light fields, white at 22% on dark).
- Other chart types are a `chart.flat` unit placed in the stack at the
  column's alignment, never at a fixed y.
- A row whose graphic ENCLOSES its text (a ring around a figure, a disc, a
  plate) is spaced by the graphic's box, exactly like a plated button row —
  never by the text's ink.
- **Animated icons (lotties) are icons.** Where the animated library has a
  confident match for an icon concept, the build places a looping animation
  in the icon's slot — same size box (the animation keeps its own aspect
  inside it, Contain), same group role `icon`, same z-order plate → icon →
  label, same well — recoloured to the brand before upload: the drawing's
  ink → brand ink on light fields, white on dark fields; its accent → the
  brand accent; whites transparent — except the wired *flat* drawing, the
  designed one, which is placed exactly as Lordicon drew it (its colours and
  whites untouched). The file is a 60 s seamless loop; the
  block's duration is the slide's, so the animation is trimmed to the slide
  and loops within it. No match → the SVG glyph, exactly as before. A
  recolour re-places the animation in the new tone; a lottie is never a
  background or a photo.
- **Lotties by default, SVG for the gaps. The order is the remote library →
  the SVG glyph (2026-09-06).** The library is the whole Lordicon *wired*
  collection plus the *system* family — 3,687 icons harvested under Bren's
  PRO licence, pre-processed by the library generator (pinch variation, 60 s
  `layers` loop, repainted to the library's colour language) and hosted at
  `https://idecide.com/lottie-library/` (`LOTTIE_REMOTE` in the panel; the
  catalogue is probed through `version.json`, cached in `chrome.storage`,
  files fetched per build). History, so old logs make sense: 2026-09-04 a
  548-file bundled library; 2026-09-05 a live `api.lordicon.com` search with a
  packaged key in front of it; 2026-09-06 both removed — "after we have the
  new library build, we can remove our original self hosted lotties, and the
  lordicon api approach and just use our new self hosted lordicon files".
  No key, no attribution (PRO), no Settings step.
- **Which drawing where (Bren 2026-09-06/08).** Every icon has up to four
  drawings — "think of the wired versions as more visual, stylized
  versions, and the system versions as more simplified versions for button,
  or pill elements": wired **outline** (2-tone, recoloured to the brand) on
  light fields and wired **flat** (the designed drawing — "intentional
  colors that should not be modified when used", placed untouched) on
  dark/photo fields — "the theme decides", a hex tone by its lightness — for
  standalone icons: above text, or alone on screen; the 1-tone **system
  outline** (light) / **system solid** (dark), recoloured, for button and
  pill icons. The panel uploads the wired build under
  `<tone>:<concept>` and the system build under `<tone>:<concept>#ui`; every
  button/pill glyph is named `btn/icon`, so `icon()` asks for `#ui` there and
  falls back to the wired drawing when the icon has no system version. A
  Lordicon animation is drawn 28 % larger than the glyph's box because its
  canvas keeps a margin, so it reads the size of the glyph it replaces.
- **One icon style per group (binding, Bren 2026-09-09).** "Elements of the
  same group type on the same slide should attempt to use the same icon
  style … if there are multiple buttons on screen, they should all try to
  use the same icon style, like a system solid, outline or 2 tone icon. If
  we are listing 3 icons above 3 lines of text, these should all try to use
  the same style as well … if no icon of the same style is available, it can
  then fall back to another style, or then fall back to the static svg icons
  as the last resort." The tone rule above already gives one style per
  family per field; what broke it was availability — one member of a row
  without a flat (or a system solid) took the other style alone, and the row
  read as two styles (Duolingo "How A Lesson Works - 3": a designed yellow
  puzzle beside white glyphs). The panel decides per slide, per group — the
  wired buttons of a menu/question/CTA are one group, the display icons
  (rows, cards, chips, steps, `copy.icon`) another
  (`decideSlideIconStyles`): every animated member has the rule's style →
  nothing changes; not all, but every one has the family's other style → the
  GROUP takes that style (uploaded as `<tone>:<concept>@<style>`, written to
  the plan as `_iconStyle.ui` / `.wired`, and `icon()` tries that key first);
  no style they all share → the rule stands and the odd member falls back on
  its own — the other style, then the SVG glyph. An icon the library does
  not know is SVG whatever the group does, and never pulls the group off the
  rule. The log line is `icon styles (<slide>): …`.
- **The cover's pill arrow is animated too (2026-09-09).** The "Click
  anywhere to Begin" pill is composed, not planned, so no plan ever asked the
  library for its arrow; the panel now prefetches the library's exact
  `arrow-right` with the SVG warm-up and the pill asks for it by that name
  (`btn/icon` → the system drawing). The SVG fallback is still the `arrow`
  glyph.
- **The match is precision-first, never fuzzy:** a name word must match —
  the file name or one of the item's **aliases** (the builder's Lucide-style
  spellings: `chevrons-down` → `two-chevrons-down`, `refresh-cw` →
  `arrow-rotate-right`) — every concept word must be a tag, no foreign name
  word, motif categories only on request; a refined name falls back to its
  base word (`calendar-check` → `calendar`, `shopping-bag` → `bag`);
  directions are meaningful (`arrow-left` is not `arrow`). The detail prompt
  tells the copy model the vocabulary is the Lordicon wired collection with
  exact names. The files are the `layers` loop build — the editor's player
  freezes a time-remapped loop after its first cycle. The library step never
  fails silently: the log says how many icons animated and which source,
  what stayed SVG, and when the remote catalogue was unreachable (the cached
  copy is used, or the step is skipped with one line). The cover-guard
  leaves glyphs and animations alone (they are Contain, never Cover).
- **An icon knows what it is, so an edit can change any part of it (Bren
  2026-09-08).** Every placed icon carries marks — `idecide/icon` (concept),
  `idecide/iconTone`, and for an animation `idecide/lottie` (the catalogue
  id), `idecide/iconStyle` (`outline` / `flat` / `system-outline` /
  `system-solid`), `idecide/iconAccent` (`brand`, a hex override, or `none`
  for a designed drawing), `idecide/iconSwaps` (colours swapped on request)
  and `idecide/iconColors` (the hexes it shows) — and `inspect` reports them
  as `layers[].icon`. The edit ops read them: **setIcon** puts another
  concept in the same box (tone, style, family kept — a button icon stays a
  system drawing), **setIconStyle** swaps the drawing (solid ↔ outline
  within the family, wired ↔ system, flat, animated ↔ static), and
  **setIconColor** takes a `tone` (ink), an `accent` (the 2-tone drawing's
  second colour) or `from`/`to` swaps (one specific colour for another — the
  one way a designed flat drawing is ever repainted, because it was asked
  for). Each such drawing is an upload of its own, keyed
  `<tone>:<concept>[#ui|@<style>][+<accent>][~<from>><to>…]` (`lottieKeyFor`
  in the pipeline); the panel asks `IDP.iconNeeds` which keys a turn's ops
  want, fetches those files from the library, paints and swaps them
  (`ensureLottieNeeds`) and uploads them before the ops run. The deck-wide
  `recolor` sweep follows an animation's tone AND accent, keeps its style,
  and leaves designed drawings alone. The 28 % overscan applies to every
  Lordicon drawing an op places or re-places (glyph → animation grows the
  box, animation → glyph shrinks it), so sizes read the same either way.

### A kicker is one object, recorded once (binding, 2026-09-05)

An eyebrow on a headline is `heading.kicker` — one group with the parts
`eyebrow` and `headline` — whether the renderer built it through
`EL.heading` or drew two text blocks that `headBind` paired from geometry.
The headline's solo ledger entry goes in before the pairing so the pair
replaces it; a slide never records `heading.standard×2` for one heading, and
the audit reads every heading for the eyebrow part before it reports a kicker
short of one.

## One shortcode per slide (binding, 2026-09-04)

A `[viewer-*]` / `[sender-*]` token appears at most ONCE on a slide. The
canonical contact lines (`token-sender`, `token-email`, the `contact:` group)
win; otherwise the first in reading order keeps it and the later text loses
the token — a block left empty is destroyed and removed from the ledger. The
cover greeting is drawn by the builder, so a plan that also puts
`[viewer-name-first]` in the headline gets it once, not twice.

## The ☰ opens the Hamburger Menu (binding, 2026-09-04)

`api.slides.setMenuSlide(id, true)` is UNIQUE — setting it on one slide unsets
every other. So the flag is set on the **Hamburger Menu** slide only, never on
menu-kind slides in general, and `assertMenuSlide` re-reads the live list and
re-sets it after the shells pass, at the end of every navigated build, after a
restore and at the start of every edit commit. The revision step can ask for
it with `{"hamburger": true}`.

## Measure only what has stopped moving (binding, 2026-09-04)

The engine lays text out with whatever face it has at that instant; a display
font still downloading measures as the fallback face. Every pass that reads a
text frame — the packer, the axis pass, the overlap audit — runs only after
`settleText` has seen two consecutive samples (frame height + visible line
count, every text block) agree, up to 1.5s. Deck 200's question headline
measured 2 lines at creation and was 3 once Poppins arrived; the buttons had
already been spaced under the short number. Two text blocks in one column are
always two rows (stacked text never legitimately overlaps), and rows inside a
unit are verified against the real frame above them after placement.

## Timing follows the same structure

Slots enter in stack order, ~0.18s apart, 1–1.5s each, EaseOutQuint (never bare `EaseOut` — the builder shows it as a raw i18n key). Parts inside one
slot enter 0.08s behind their own background — one object arriving, not four.
Stage media gets 2–4s. Every layer runs to the last frame plus a 0.3s tail.

## The background plate is ALWAYS the backmost layer of its group (binding)

A button — or any plated composite — stacks in exactly this z-order, bottom
to top: **background plate → icon → label**. The plate is created FIRST so it
lands lowest, it carries the click action (it covers the whole tappable
area), and nothing in the group may ever sit beneath it. When repairing or
hand-assembling a button, verify the plate's child index is the minimum of
its group before committing — a plate inserted above its icon covers the
glyph and reads as a broken control (live lesson, sweetgreen Back buttons,
2026-08-24). Entrance animations follow the same order: plate first, then
icon, then label, 0.08s apart.

## Text boxes hug their words — except where a shortcode lives (binding)

Every text box is trimmed to its measured ink plus slack, by
`tightenTextBoxes()`. This is not cosmetic: **alignment cannot be judged, by a
human or by the overlap passes, while boxes are wider than their content.** A
box the width of its column tells you nothing about where the words actually
sit.

The measurement is the engine's, not an estimate: flip `widthMode` to `Auto`,
read `getFrameWidth`, restore mode/width/position (the restore of position is
required — measuring moves the block). Slack scales with type size: 14% for
display (≥44pt), 10% for body, 18px floor. If the trim causes the frame height
to grow, the line wrapped and the box is put back exactly as it was — a wide
box costs nothing, an orphaned word costs a rebuild.

**The exception: shortcodes.** `[sender-email]`, `[viewer-name-first]` and
friends are PLACEHOLDERS. At play time they become real values that are
usually longer than the token. Trimming to the token's ink therefore
guarantees the live value wraps, or gets auto-shrunk by the player — the same
defect, one step later. Measured on deck 186: `[sender-email]` inks at
**201px** as a token and **354px** as an average real address, a 153px
shortfall on every contact block.

So a token-bearing line is measured a **second** time with each token replaced
by a sample of the average real value, and the box reserves the WIDER of the
two. Averages live in `TOKEN_AVG` in composer.js and are tunable:

Averages are **rounded up to the nearest 5** — the reserve is a cushion, so
erring wide costs a few pixels while erring narrow costs a wrap. The table is
keyed by the FIELD, not the full token: `viewer-` / `sender-` (and any future
role prefix) are stripped, and both roles share the larger reserve, since a
viewer's name is no shorter than a sender's.

| field | reserved chars | example |
|---|---|---|
| `name-first` | 10 | Michael |
| `name` | 20 | Katherine Brooks |
| `email` | 25 | katherine@brookslaw.com |
| `phone` | 15 | (555) 123-4567 |
| `scheduling-url` | 30 | |
| `company-name` | 20 | |
| `current-date` | 20 | Wednesday, 12 March |

So `[viewer-email]` and `[sender-email]` both reserve 25 characters. A token
with no table entry is left exactly as written — no reserve, no guessing.

The sample string uses mid-width letters, so it is neither the widest case
("MMMM") nor the narrowest ("iiii").


---

# iDecide Slide Design System — Composition Playbook
**v5 — full library: 15 categories × 25 variations; skeletons in strict §12 vocabulary with arrangement/distinct differentiators.**

**Purpose.** This document is the authoring source of truth. It describes the template library the way a designer would brief it — skeletons, proportions, capacities, and variables — so the builder composes a *bespoke* layout that fits the specific content, while still cycling through the full range of 375 structurally distinct compositions.

> **The one rule that matters:** *Anything that must move or disappear together
> is ONE element with several parts — never two separate things.* A slot is the
> smallest thing the layout may address; a part (a button's label, a kicker's
> eyebrow, a row's detail) has no independent existence.
>
> **The second rule:** *The template is already chosen.* Each slide arrives
> bound to one of 375 templates, each built from named element variations.
> Write copy TO the slots it declares — read the slide's BUILT FROM brief
> first. Don't pick a different look and don't squeeze content past a cap:
> the caps are what keep a slide legible at phone size.
>
> **What that rule does NOT mean.** The brief governs COPY. It is not a list of
> what the slide is permitted to contain. Footage always lands — a photo zone
> holds it, a layout with none gets it as a scrimmed background. A grid is
> arithmetic: `grid: {cols, rows}` on the plan draws that grid whatever the
> template's own zones say, and every panel of a multi-panel grid gets its own
> clip. A button can be built and wired on any slide. Never leave something out
> of a slide, and never tell the client it cannot be done, because the bound
> template has no slot with that name.

---
## PART I — GLOBAL SYSTEM

### 1. Canvas, units, margins
- Canvas **1558 × 720** (mobile landscape, full-screen on a phone, paired with voiceover). Slides are **visual aids**, not documents.
- All geometry here is **fractions of the canvas** and **relationships**, never fixed pixel boxes. Convert at build time.
- **Safe inset 4–7%** (≈62–110 px) for all text. Full-bleed *images* reach the edge; full-bleed *text* never does.
- **Slot rhythm — two gaps, both always in play:** the STACK gap (W×0.024 ≈ 37px)
  between slots, the TIGHT gap (W×0.010 ≈ 16px) between parts inside one slot (a
  title and its detail, a numeral and its label, [sender-name] and
  [sender-email]). Never one uniform gap — equal spacing makes a detail read as
  its own item.
- **Panel seams** in multi-panel layouts are hairline (≈0.3%) — panels read as one composition, not separate cards.
- Optical balance beats mathematical centering: headlines over photos sit slightly above center; stat blocks sit slightly left of center in splits.

### 2. Type roles, sized by content length
Assign a **role**, then choose a size inside its band based on actual length. Longer content → lower end, or a different role.

| Role | Size band | Character budget |
|---|---|---|
| Hero statement | 92–200 | ≤ 28 chars (160+ only for 1–2 words) |
| Slide headline | 54–80 | 28–70 chars |
| Long-sentence headline | 44–58 | 70–130 chars |
| Stat numeral | 110–260 | ≤ 6 glyphs |
| Subhead / lead-in | 34–46 | ≤ 70 chars |
| Body | 28–40 | ≤ 160 chars per block |
| Card / item title | 30–40 | ≤ 22 chars — buttons, 1–3 words |
| Card / item body | 28–32 | ≤ 55 chars (dropped before anything else) |
| Eyebrow / label | 28–34 | ≤ 24 chars, UPPERCASE, tracking 3–6 |
| Button / chip label | 28–34 | ≤ 22 chars (≤ 18 in a circle action), never wraps, never truncated |
| Attribution / fine print | 28–32 | ≤ 80 chars |

**Gutters are balanced too.** A side photo/video column is not fixed: when
the copy runs long, the PANEL gives width — it stays anchored to its canvas
edge and its inner edge moves so the gutter between the widest line and the
video equals the margin between the canvas and the text. Text only gives way
once the panel has reached its floor. This is the horizontal mirror of a
photo band giving height to the stack above it.

**Paired lines are one object.** A list item that runs to two lines — a
label over its detail — binds on a TIGHT gap so it reads as a single unit;
the uniform gap then separates whole units, not lines. Equal spacing between
every line makes the second line look like its own list item. A headline is
never half of a pair: it leads the slide. Measure the tight gap ink-to-ink,
because an icon well beside a label is taller than the words it sits next
to.

**Stacks are flex columns.** Elements stacked vertically — eyebrow,
headline, subhead, button, contact block — sit on ONE repeated gap, not a
different gap per pair, and the finished group is middle-aligned in the space
it actually has (on a cover with a photo band that is canvas-top to band-top,
not the whole canvas). Lines that belong to a single unit (a contact block's
name and address) use a tighter gap so the unit reads as one thing. When the
room runs short the unit sheds its least load-bearing line before anything
overlaps.

## Grouped elements (the element system)

A slide is not a pile of blocks. It is a handful of GROUPS, and the layout
engine moves, spaces, centres and animates each group as one object. Write
copy to the group, not to the block.

| Group | Its parts, bottom → top | Holds |
|---|---|---|
| Button / action | background (carries the click) · icon · label | label ≤22 chars, never wraps |
| Circle action | circle · icon · label | 1–3 words, 2 short lines max |
| Chip | pill · icon · label | ≤22 chars |
| List row (listrow.icon) | row background(click, on menus) · icon well · icon · title · detail? | title ≤22, detail ≤55; title→detail on the TIGHT gap, row→row on the STACK gap |
| Card | card bg · icon · title · body | body drops past 4 cards |
| Sender block | lead · [sender-name] · [sender-email] | fixed content; lead drops when tight |
| Stat | numeral · label | numeral ≤6 chars |
| Quote (quote.mark) | motif/quote-mark · quote · attribution | the mark is a PART of the quote object, not droppable decor |
| Person | plate · portrait · name · role · facts | facts are mini-pairs |
| Process step | step.numbered: plate? · step-num · label · body? — OR step.icon: plate? · step/icon · label · body? (never both) | steps share one width; body drops first |
| Chart | frame · shapes · labels | one per slide |

Three rules follow from this and they are binding:

1. **A group's parts are never separated.** A label belongs to its button, a
   detail belongs to its title, [sender-name] belongs with [sender-email].
   Inside a group the spacing is tight; between groups it is even.
2. **REQUIRED slots are never dropped to make room** — every action button,
   every menu row, the headline, the stage field. When a slide is crowded the
   fit ladder runs, in this exact order and no other:
   drop:decor (motifs, logo, chip rows) → drop:support.line (then a kicker's
   eyebrow PART) → swap:heading.long (into the 130-char band) → shrink:heading
   → yield:media (a band to its 30% floor, a side panel to 22%) → scale:group
   (the whole set together, 28px floor, never one element) → tighten:rhythm
   (the stack gap gives LAST — it is what makes the set read).
3. **Stack order is reveal order.** Backgrounds appear, then icons, then
   titles, then details. Write copy knowing the label lands before its
   detail.

Full definitions, including internal spacing and what happens when a member
is added or removed: `assets/docs/element-contract.md` (the binding rules) and
`assets/docs/template-schema.md` (the schema). `element-system.md` is the
background essay — useful, superseded wherever the two disagree.

**Options are load-bearing.** A menu or question option is navigation, not
decoration: it is never dropped, never truncated, and never pushed off the
canvas. When a slide is crowded the fit ladder above runs and the options all
stay — the set scales together before anything is lost.
This is why option labels are held to 1–3 words: every extra word on a label
steals room from the option below it.

**Hard floor 28 px.** Headlines ≤ 3 lines, body blocks ≤ 4 lines. If it won't
fit, you do not change the layout and you do not cut items — you write shorter
copy to the slot's cap, and the build runs its fit ladder:
   drop:decor (motifs, logo, chip rows) → drop:support.line (then a kicker's
   eyebrow PART) → swap:heading.long (into the 130-char band) → shrink:heading
   → yield:media (a band to its 30% floor, a side panel to 22%) → scale:group
   (the whole set together, 28px floor, never one element) → tighten:rhythm
   (the stack gap gives LAST — it is what makes the set read). A three-word line in a body-sized slot is equally wrong: **promote it** to Hero statement.

**Typefaces:** one DISPLAY face (headlines, numerals, quotes; 600–800, italic = warm accent where the face has it) + one clean BODY face (body, labels, UI; 400–800). The pairing is a per-brand design decision — serif + sans is one classic option, not a rule: a modern brand may pair a strong sans display with a lighter sans; a heritage brand a serif with a humanist sans; a bold brand a condensed display. Never a third family. Numerals always in the display face — that's what makes stats feel designed.

**Band layouts pack evenly.** With a photo band top or bottom, the text
stack distributes in the remaining zone: equal padding above and below,
equal glyph-measured gaps between rows (font leading varies — space is
computed from font size × lines, never from frame boxes). When the stack
needs more room, the ladder has already swapped and shrunk the HEADLINE — only
then does the footage yield (a band to its 30% floor, a side panel to 22%), and
only after that is the whole set scaled together (28px floor). Text never
crowds the footage and never crams. Text never touches the footage and never crams. Solid
background colors live on the CANVAS itself, not on a redundant
background shape.

**Footage floors.** Background clips and stills always have a short side of
at least 720px; interactive slides (questions, menus, CTAs, the cover) hold
their graphics ≥15 seconds and take clips ≥15s long, so the video plays on
while the viewer reads and decides.

**Button labels are centred by their plate, not by an align setting.** A solo
button hugs its label's ink (pad max(24, h×0.3), icon gap 18), so there is no
dead gap to correct; buttons in a set share one width, so a short label centres
in the shared plate. Never set a button label's text alignment — align is a
block property and the parts of a button are not independently positionable.
A featured button never shares its field's color: on a primary field it
features in accent.

**Buttons, pills and chips — inner padding is non-negotiable.** Any element
that puts text or an icon inside a shaped background (menu rows, option
cards, CTA circles/squares/bars, decorative pill chips, credential chips)
keeps visible breathing room on every side: ≥24px horizontal, and the label
never touches the shape edge (circles inset labels ~14% of the diameter).
When space runs short the type scales with the rest of the set (28px floor) and
the rhythm gap gives LAST — padding never shrinks and a label is never
truncated. A solo button hugs its label's ink; buttons that appear together
share ONE width (the widest label's), so a longer label widens the whole set. Labels never overflow their background.

### 3. Color roles
Map the brand palette onto: **Ink** (text on light) · **Surface-light** (default light field) · **Surface-tint** (cards/wells on light) · **Primary** (solid fields, featured buttons, key numerals) · **Primary-deep** (dark fields, photo scrims) · **Accent** (warm secondary — featured item, one underline, one numeral, a signature) · **On-dark** / **On-dark-muted**.

Max **2 background field colors** per presentation (plus photos). **Accent on ≤1 element per slide.** Never mid-tone text on mid-tone fill. A single 2-stop gradient is allowed for scrims and occasional fields; nothing more elaborate.

### 4. Photography
- **The imagery floor: at most FOUR slides in the whole deck may carry no image or video.** These play
  full-screen on a phone, where a slide with nothing behind the type reads as one that failed to load.
  Spend those four deliberately — on the densest beats, where a background would compete with the text.
  Everything else, menus and questions included, gets a full-bleed motion background or a framed image/video.
- **Every image unique** — no repeats, and watch for near-duplicates from one shoot.
- Source ~2400 px wide for full-bleed and half-panels, ~1600 for small slots. Never upscale.
- **Fit:** people/scenes → cover (keep faces out of crop and out from under text). Logos, product, diagrams, QR → contain on a contrasting field.
- **Casting** matches the audience the script implies, consistently.
- **Text over photo always needs a scrim:** left-aligned text → horizontal gradient ~90% at the text edge fading to ~20%; centered text → vertical gradient dark top and bottom, ~55% mid; text panel → flat semi-opaque field 78–95%. Never blur. Stack: image → scrim → content.
- Busy photo? Put text in a solid card beside it or a band across the lower third instead of a gradient.

### 5. Icons
- **Every list item, menu option, benefit card, button, and contact line gets an icon.** A bare bullet is a missed opportunity.
- **One concept, one icon** — shield=protection, heartbeat=health, home=housing, people=family, calendar=scheduling, document=paperwork, check=confirmation, arrow=progression, pin=location, phone/mail=contact. Never repeat a generic glyph down a list.
- Single-weight line icons, consistent stroke deck-wide. Bare on color, or inside a soft tinted rounded container ≈1.7–2× the icon's optical size.
- Sizes: inline with a row ≈0.9–1.2× row text · card icon ≈1.2–1.5× card title · hero/feature icon 3–5× body.
- Stroke: ink on light, on-dark over dark/photo. Tint the container, not the icon. Never emoji.

### 6. Interactive units (buttons, menu rows, chips, tiles)
Always **one grouped unit** of stacked layers:
1. **background layer** — fill + corner radius + padding = the hit target (pill radius = half height; card radius 14–28).
2. **icon layer** — bare, or in a rounded icon container.
3. **text layer** — the label (never wraps; widen the unit instead).
4. There is no chevron/affordance part — don't invent one. The parts an element
   has are exactly the ones its library entry lists.
Featured/selected = solid Primary or Accent + on-dark text. Default = Surface-light or low-alpha fill + ink text. Keep layers grouped so downstream wiring and animation can target them.

### 7. Composing a slide (procedure)
1. **Read the content** — counts, numbers, items, photo availability, whether a choice is being made.
2. **Classify the beat** — open · divider · emotional · explanation · proof/stat · list · story/quote · person · process · choice · close.
3. **Read the slide's BUILT FROM brief** — it names the element variations, their
   parts, their counts (×N min-max) and which slots are REQUIRED. The template
   is already assigned by the seeded rotation; you do not choose or change it,
   and you cannot split a slide (its identity comes from the script row). If
   the content is too big for the slots, write less of it.
4. **Assign type roles**, size by actual length (§2).
5. **Fill the slots** — one copy value per part the brief lists, within its cap.
   Zones, gaps and positions are resolved by the packer (tighten → clamp →
   balance); never state pixel positions.
6. **Icons** for every discrete item (§5).
7. **Color roles**, contrast check, accent once (§3).
8. **Photo pass** — unique, high-res, cast, scrimmed (§4).
9. **Rhythm check** against neighbours (§8).
10. **Polish check** (§9).

### 8. Rhythm across a presentation (50–150+ slides)
- **Never the same variation twice in a row** — and never the same *skeleton* twice in a row either (a photo-left split followed by a photo-right split is still repetition).
- **Any variation appears at most 3 times**; prefer 1–2. With 375 available, a 100-slide deck should use ~60+ distinct variations.
- **Alternate value:** no more than 2 photo-heavy slides consecutively without a solid-field or light-card slide between, and vice versa.
- **Alternate density:** dense (cards, grid, list) → sparse (hero, statement, stat).
- **Parallel beats stay identical:** one divider variation, one transition/gate variation, one menu-revisit variation per presentation. Consistency where the viewer needs orientation; variety everywhere else.
- **At most 4 bare slides** in the deck — count before delivering.
- **Voiceover pairing:** on-screen text ≈ one third of what is spoken. If the slide reads like a transcript, cut it.

### 9. Polish checklist (fail any → fix before moving on)
- Nothing below 28 px · headline ≤3 lines · body ≤4 lines · no orphan last word.
- Every slot's parts read as ONE object: label inside its plate, eyebrow with
  its headline, detail on the tight gap under its title, anchored slots with
  their parent.
- Every REQUIRED slot in the brief is present, un-truncated and on-canvas.
- No text touching an edge; margins consistent deck-wide.
- Every text element's size matches its role **and** its actual length.
- Every list/menu/button item has a distinct, relevant icon.
- All text over photography sits on a scrim or solid field and passes contrast.
- One primary action per interactive slide.
- No duplicate or soft images.
- Accent used once; ≤2 background field colors deck-wide.
- Skeleton differs from the previous slide; variation not already used 3×.

### 10. Building in myiDecide (img.ly CE.SDK v1.74.1)
- Page **1558 × 720**, one page per slide.
- Blocks: `//ly.img.ubq/graphic` rects for fields, cards, bands, scrims, button backgrounds; `//ly.img.ubq/text` for every text element; image fills for photos; vector/graphic for icons.
- **One string per text block** — headline, eyebrow, and subhead are separate blocks (inline bold within a sentence is fine; a *label* is its own block).
- **Auto-height, not shrink-to-fit**, so overflow is visible during design and fixed by editing copy or size.
- **Buttons = grouped** background(click) → icon? → label, named by element and
  index (`btn-1/bg`, `btn-1/icon`, `btn-1/label`). The shared prefix and the
  group mark are what tell every later pass they are ONE object.
- **Scrims** = graphic blocks with a 2-stop linear gradient or flat semi-transparent fill, directly above the image block, below all content.
- **Keep effects simple:** solid or single 2-stop gradient fills, at most one soft shadow per element, plain rect/rounded-rect/circle. No blur/backdrop, blend modes, masks, or layered shadows.
- **Name blocks by role** (`headline`, `eyebrow`, `photo-hero`, `scrim`, `btn-1/bg`, `btn-1/icon`, `btn-1/label`).
- **Fonts:** platform-available families only; map/embed both display serif and UI sans so text survives export/import.
- Personalization tokens are literal bracketed strings in their own text blocks.

### 11. Failure modes this document prevents
| Symptom | Root cause | Correct move |
|---|---|---|
| Paragraph crammed where a numeral belongs | Reusing a slot instead of choosing a role | Re-classify; use a Split or List variation with Body role |
| Three words floating in a body box | Role mismatch | Promote to Hero statement on a Statement variation |
| Headline running 5 lines | Size copied, not chosen | Drop to long-sentence band or rewrite shorter |
| Text unreadable over a photo | Missing scrim | Add gradient/flat scrim or move text to a solid field |
| Seven items in a 3-slot layout | Item count past the slot's count.max | Write to the declared count — merge or cut CONTENT ideas. If the beat is genuinely a set of N things, say so with `grid: {cols, rows}` and it is drawn as N panels; otherwise it is two beats |
| Every slide looks the same | — | Templates are assigned by the seeded rotation (no back-to-back skeleton, ≤3 uses per deck), so variety is handled for you — but `grid` and `photoZone` are still yours to set when the content asks for them |
| Slide reads like the narration | Copy dumped from script | Keep the key phrase; let voiceover carry the rest |
| Repeated or soft photo | No image ledger / low-res source | Track every image; source at 2400 px |

---

### 12. Skeleton vocabulary (machine-parseable — use these phrasings EXACTLY)
Every *Skeleton:* line is built ONLY from these phrases so the parser extracts every field. Never paraphrase them.

**Layout mode (exactly one, first):** "side-by-side zones (row)" | "stacked zones (column)" | "grid N cols (2fr+1fr) x M rows" | "layered over full-bleed image" | "single field"

**Photo zones:** "full-bleed photo" | "photo column left|center|right NN% w" | "photo band top|middle|bottom NN% h" | "zone left|right NN% w containing N photo columns side-by-side" | "zone top|bottom NN% h containing N stacked photo cells" | "N photo cells NN% w x NN% h" | "one large photo panel left NN% w full height + N cells right NN% w x NN% h each" | "N color panels (checkerboard)" | "circular portrait" | "logo/mark slot (NNNpx)" | "no photo - solid field" | "no photo - gradient field"

**Bands & panels:** "header band top NN% h (tone)" | "caption band bottom NN% h (tone)" | "content panel left|right|top|bottom NN% w|h (tone)"

**Overlays (absolute layers, after flow zones):** "horizontal scrim from left|right" | "vertical scrim" | "flat tint scrim NN%" | "top|bottom band overlay (tone)" | "corner card top|bottom-left|right ~NN% w (tone)" | "centered overlay NN% w" | "side rail left NNNpx" | "edge tab right NNNpx"

*These flags describe the STAGE so the parser can extract it. They do NOT
declare content — what a slide holds comes from its element brief
(listrow.icon, card.photo, chip.pill, motif.glyph, stat.ring…). Where a flag
and the brief disagree, the BRIEF wins.*

**Arrangement clause (one per skeleton, before field):** "arrangement: <flags>" where flags are comma-separated from: "content centered" | "content left-aligned" | "content right-aligned" | "items as N stacked rows" | "items as N cards in a row" | "items as N pill chips" | "items in N-col grid of M" | "display numeral" | "giant glyph backdrop" | "oversized quote mark" | "italic script accent" | "circular portrait" | "star rating motif" | "progress ring" | "bar chart rows" | "play-button motif"

**Field & inset (always last):** "field: white|light-grey|ink-dark|near-black|primary|primary-light|slate|accent|pale-blue|gradient primary-deep" | "inset NNpx NNpx" | "edge-to-edge zones"

**Action targets (Category 15):** "circle (NN% of canvas height diameter)" | "square NN% w x NN% h of canvas" | "rectangle (NN% of canvas height)" | "N full-height column buttons" | "N quadrant/grid panel buttons"

**Position cues** close every *Notes:* line as "Position cues: photo left; band bottom; text right." — renderers read side/position words there.

**Uniqueness rule:** within a category no two variations may share the tuple (layout mode, photo mode, first photo fraction, photo count, band position, scrim direction, arrangement flags, field tone, action targets). The skeleton signature MUST include the arrangement clause — it is what separates otherwise-similar single-field or repeated-row layouts.

---

---

*PART II (the 15×25 variation library) ships separately: the planning call receives a condensed category catalog, and each slide's build receives its locked variation's full spec (skeleton + arrangement + distinct + holds + type ceiling + position cues).*
