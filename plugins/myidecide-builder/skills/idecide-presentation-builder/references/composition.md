> **Reference for the myiDecide Presentation Builder skill.** Two documents:
> the ELEMENT CONTRACT (the binding rules every drawn slide obeys — grouping,
> the cover, axis, buttons, graphics in the flow, shortcodes, measuring)
> followed by the DESIGN PLAYBOOK (2.1.1, template-free: type roles and weights, colour
> roles, the fields, interactive units, the composing procedure, rhythm of
> compositions and of devices across a long deck, the polish checklist, and
> what the test builds, the exemplar slides and decks 305/306 taught). Read
> when composing or re-aligning slides.

# Element contract — the binding rules the builder enforces

**2.0 (2026-09-15, template-free).** Every slide is composed by the design
pass as a SCENE — a stage and an ordered list of elements with their own
geometry (the contract: `assets/prompts/slide_detail_system.md`; the design
system: `assets/prompts/playbook.md`) — and drawn by `inject/scene.js`
through the composer's primitives. This document is what those primitives
and the passes after them GUARANTEE, whatever the scene says: how parts
group, what may never be dropped, how buttons are built and wired, how icons
are chosen, marked and edited, how text is measured. It is not a list of
slots to fill; the scene decides what a slide holds. The template library
(templates, variations, the element library, the BUILT FROM briefs) is
retired — `docs/archive/retired-2026-09-15-templates/` keeps it for decks
built before 2.0, which still carry their bound template and render through
the composer's archetypes.

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

## What happens when content doesn't fit (2.0)

A scene is a designed, FIXED layout (the page is stamped `idecide/layout:
fixed`). The builder does not re-pack it; it measures and protects it:

```
1  measure     text frames are read after the fonts settle (never estimated)
2  tighten     every text box is trimmed to its words (shortcodes reserved)
3  reflow      a stack re-flows on the measured heights; a free-standing
               text that wrapped taller pushes what sits below it in its
               column down by the difference
4  separate    no two buttons may overlap; overlapping text pairs are pushed
               apart by the measured overlap
5  fit         nothing leaves the canvas
```

What that means for a design: a column written too long runs off the bottom,
it is not silently thinned. The fix is the designer's (shorter copy, a wider
box, a smaller role, one element fewer) or the reviewer's (`sceneEdits` /
`scene`). **Never dropped by any pass:** every wired button, every menu row,
the headline, the stage. Navigation is never sacrificed and a button label is
never truncated. Decks built before 2.0 keep the old fit ladder (drop decor →
drop the support line → longline → shrink → yield media → scale the group →
tighten the rhythm) inside the composer's archetypes.

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

### The scene's column decides the axis (binding, 2026-09-04 → 2.0)

A composition puts its content column left, centred or right, and that axis
governs **everything in the column**: text, button groups, Back pills, stat
units, graphics, photo-card strips and the sender block. Left stays left,
right stays right, centred stays centred. Only an Edit-step request changes
it — as a `scene` redesign or `sceneEdits` that move every element of the
column together; a reviewer who finds a left-aligned layout centred (or the
reverse) has found a defect, not a choice. `text()` never invents an axis: a
scene text element states its `align`, and the default is Left.

### A deck alternates its layouts (binding, 2026-09-05 → 2.0)

Variety is the design pass's job and the validator's check: the outline
states one layout intent per slide, no two neighbouring content slides may
share an intent + field + photo treatment, no intent appears more than four
times, and the design pass names each composition's `family` and does not
repeat its neighbour's. The autoScene fallback (a plan with no scene)
alternates its axis and photo side by a hash of the slide name, so even a
deck whose design pass failed does not read as one template.

### The kind outranks everything (binding, 2026-09-05 → 2.0)

What a slide IS decides its buttons: a `question`, `menu`, `hamburger` or
`cta` — and any slide that waits for the viewer (`autoAdvance: false`, the
sub-fork Back) — gets a real, wired button for EVERY wired item, whatever the
scene declares. A scene that leaves a wired item without a `button` element
gets a default button row appended by the normaliser (logged). The cover
gets exactly one static pill. A content slide that auto-advances gets no
buttons: its wired items are drawn as display rows (logged) — the click lives
on the slide after it.

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
a sub-fork with its Back item. An explainer or action-steps slide before a
CTA draws its items as display cards or rows (a `list` element, or media
panels), one object each, never a `btn:` name; the audit accepts those display
units in place of buttons on an auto-advancing slide and says so.

## Graphic elements sit in the flow like anything else (binding, 2026-09-04)

SVG icons, infographic parts, charts, images, lotties and videos follow the
scene's spacing and positioning and are grouped with the text they belong to
— they are never dropped at a fixed spot after the renderer runs. In 2.0 the
data graphics (bars, hbars, dots, ring, stat, progress, and the charts.js
types) are `graphic` elements the scene places and the renderer draws as
native blocks, each grouped with its labels.

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
- **Which drawing where — the 2-tone wired outline, everywhere (binding,
  Bren 2026-09-09).** Every icon has up to four drawings: the WIRED family's
  **outline** (2-tone, recoloured to the brand — every one of the library's
  3,687 icons has it) and **flat** (the designed drawing — "intentional
  colors that should not be modified when used", placed untouched), and the
  1-tone SYSTEM family's **system outline** / **system solid** (a few hundred
  each). "Add a preference to favor using the 2-toned wired outline version
  of the icons. Let it be able to use the other styles, and ensure that the
  user can still ask to use one of the other styles instead, but default to
  using the 2 toned versions. These match the style of presentations a
  little better and feel more intentionally branded" — asked whether
  buttons too: "Everywhere, buttons included." So the build places the
  wired outline for standalone icons, icons above text, and button/pill
  icons alike, on every field; the tone decides only the paint (ink on
  light fields, white on dark, a hex when asked). The panel uploads it under
  `<tone>:<concept>`; a button/pill glyph is still named `btn/icon` (its
  group for the style rule below and for the edit ops), but `icon()` no
  longer prefers a `#ui` drawing there. The other three styles stay
  reachable two ways: a client asks by name in an edit turn (`setIconStyle`
  / `addIcon` with `style` — "flat"/"designed", "system"/"1-tone"/"solid",
  "system-outline" — uploaded as `<tone>:<concept>@<style>`), and the
  ladder below falls back to them when an icon has no outline. History, so
  old decks and logs read right: 2026-09-05/06 "the theme decides" — outline
  on light fields, flat on dark/photo (`lordiconFieldStyleFor`, still what
  "flat" resolves to) — and buttons took the system drawing (system-outline
  light / system-solid dark, `lordiconSystemStyleFor`, still what "system"
  resolves to) under `<tone>:<concept>#ui`; one flag in each of the three
  files (`LORDICON_UI_SYSTEM` / `LOTTIE_UI_SYSTEM` / `UI_SYSTEM_DEFAULT`,
  pinned equal) puts buttons back on that family. A Lordicon animation is
  drawn 28 % larger than the glyph's box because its canvas keeps a margin,
  so it reads the size of the glyph it replaces.
- **The stroke weight (Bren 2026-09-09).** Lordicon draws the wired icons at
  three weights — light, regular, bold — and in the source every stroke
  width is an expression, `value / 2 ×` the control layer's stroke menu
  (1 · 2 · 3). The harvest baked that menu at 2, so **every library file is
  the REGULAR weight** and that is what a build places ("regular is probably
  a good default"). The other two are the same drawing with every stroke
  width scaled — ×0.5 and ×1.5, what the expression would have computed —
  applied panel-side by `IDP_LOTTIE.setStroke` and uploaded under
  `<tone>:<concept>^light` / `^bold`. A client gets one by asking
  (`setIconStyle` with `stroke`: light / regular / bold, or "bolder" /
  "thinner", which step one from where the icon is). The weight belongs to
  the stroke-drawn WIRED family: the system drawings are filled paths (the
  harvested system-outline calendar has no stroke shape at all), so the op
  says so rather than scaling a stray outline. A placed icon carries
  `idecide/iconStroke` when it is not regular, and `inspect` reports
  `icon.stroke`.
- **One icon style per group (binding, Bren 2026-09-09).** "Elements of the
  same group type on the same slide should attempt to use the same icon
  style … if there are multiple buttons on screen, they should all try to
  use the same icon style, like a system solid, outline or 2 tone icon. If
  we are listing 3 icons above 3 lines of text, these should all try to use
  the same style as well … if no icon of the same style is available, it can
  then fall back to another style, or then fall back to the static svg icons
  as the last resort." The default above already gives one style per group
  per field; what broke it was availability — one member of a row without
  the style in play took another alone, and the row read as two styles
  (Duolingo "How A Lesson Works - 3": a designed yellow puzzle beside white
  glyphs; YETI's menu: one system-solid glyph among wired ones). The panel
  decides per slide, per group — the wired buttons of a menu/question/CTA
  are one group, the display icons (rows, cards, chips, steps, `copy.icon`)
  another (`decideSlideIconStyles`): every animated member has the default
  → nothing changes; not all, but every one has the next style on the
  LADDER (`lordiconLadderFor` — a button: outline → the system drawing the
  field picks → the other system style → flat; a display icon: outline →
  flat → the system pair) → the GROUP takes that style (uploaded as
  `<tone>:<concept>@<style>`, written to the plan as `_iconStyle.ui` /
  `.wired`, and `icon()` tries that key first); no style they all share →
  the default stands and the odd member falls back on its own — down the
  same ladder, then the SVG glyph. An icon the library does not know is SVG
  whatever the group does, and never pulls the group off the default. The
  log line is `icon styles (<slide>): …`.
- **Three concepts per icon (binding, Bren 2026-09-09).** "Does the lottie
  icon search currently only search for 1 topic match per icon? … let's
  widen it to 3 possible icon concepts per lottie search, to widen the
  possible matches, and to give a little diversity to lists that might land
  on the same icon choices per list item." A plan's item (and `copy.icon`)
  carries `icon` — the first choice — plus `iconAlts`, two MORE things that
  could draw the same line, each a fair reading on its own and never a
  respelling of the first ("Designed in-house": `pencil-ruler`, then
  `house`, `glasses`; "Thirty days": `calendar`, then `truck`, `watch`).
  Before a slide's icons are fetched, `chooseSlideIcons` settles ONE concept
  per slot in slide order: the first candidate the library HAS whose drawing
  no earlier slot on that slide already took; when every known candidate is
  taken, the first known one (a repeat still animates); when the library
  knows none, the first choice stays and the slot falls to its SVG glyph.
  The pick is written back to `icon`, so everything downstream still reads
  one name, and the candidates + reason to `_iconChoice`, which a re-run
  reads back so the original order still decides. This is what stopped
  Warby Parker's "Five days" and "Thirty days" both drawing the same
  calendar. Log line `icons (<slide>): "<label>": <first> → <chosen> (why)`.
- **The cover's pill arrow is animated too (2026-09-09).** The "Click
  anywhere to Begin" pill is composed, not planned, so no plan ever asked the
  library for its arrow; the panel now prefetches the library's exact
  `arrow-right` with the SVG warm-up and the pill asks for it by that name.
  The SVG fallback is still the `arrow` glyph.
- **The match is precision-first, never fuzzy:** a name word must match —
  the file name or one of the item's **aliases** (the builder's Lucide-style
  spellings: `chevrons-down` → `two-chevrons-down`, `refresh-cw` →
  `arrow-rotate-right`) — every concept word must be a tag, no foreign name
  word, motif categories only on request; a refined name falls back to its
  base word (`calendar-check` → `calendar`, `shopping-bag` → `bag`);
  directions are meaningful (`arrow-left` is not `arrow`). The detail prompt
  tells the copy model the vocabulary is the Lordicon wired collection with
  exact names. The files are the `layers` loop build — the Builder's player
  freezes a time-remapped loop after its first cycle. The library step never
  fails silently: the log says how many icons animated and which source,
  what stayed SVG, and when the remote catalogue was unreachable (the cached
  copy is used, or the step is skipped with one line). The cover-guard
  leaves glyphs and animations alone (they are Contain, never Cover).
- **An icon knows what it is, so an edit can change any part of it (Bren
  2026-09-08).** Every placed icon carries marks — `idecide/icon` (concept),
  `idecide/iconTone`, and for an animation `idecide/lottie` (the catalogue
  id), `idecide/iconStyle` (`outline` / `flat` / `system-outline` /
  `system-solid`), `idecide/iconStroke` (the weight, only when it is not
  regular), `idecide/iconAccent` (`brand`, a hex override, or `none`
  for a designed drawing), `idecide/iconSwaps` (colours swapped on request)
  and `idecide/iconColors` (the hexes it shows) — and `inspect` reports them
  as `layers[].icon`. The edit ops read them: **setIcon** puts another
  concept in the same box (tone and drawing kept), **setIconStyle** swaps
  the drawing (solid ↔ outline within the family, wired ↔ system, flat,
  animated ↔ static) and sets the **stroke weight** (light / regular / bold,
  or "bolder" / "thinner" — valid on its own, no style needed), and
  **setIconColor** takes a `tone` (ink), an `accent` (the 2-tone drawing's
  second colour) or `from`/`to` swaps (one specific colour for another — the
  one way a designed flat drawing is ever repainted, because it was asked
  for). Each such drawing is an upload of its own, keyed
  `<tone>:<concept>[#ui|@<style>][+<accent>][~<from>><to>…][^<stroke>]` (`lottieKeyFor`
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

# myiDecide Slide Design System — Composition Playbook
**2.1.1 — template-free. The designer composes every slide as a SCENE from loose parts, in the deck's own design language; the builder draws it, measures it, animates it.**

**Purpose.** This document is the design source of truth for every slide the
extension builds. Since 2026-09-15 there is no template library: the design
pass writes each slide's composition itself — a stage and an ordered list of
elements with their own geometry on the 1558×720 canvas — and the builder
renders it through primitives that enforce the deck's mechanical rules. What
follows is what a designer must know to compose well for this canvas, and
what the builder guarantees so the designer does not have to.

> **The one rule that matters:** *Anything that must move or disappear together
> is ONE element with several parts — never two separate things.* A button is
> its plate, its icon and its label; a menu row is its numeral, icon, label,
> chevron and divider; a stat is its figure and its caption. Compose these
> units from loose parts — a `repeat` stamps one designed cell per item and
> makes each cell one unit (and one click target when the item is wired);
> a `group` id does the same for a one-off composite. The button / list /
> graphic / sender elements are the builder's own looks — shortcuts, not
> the standard (2.1, after the exemplar decks).
>
> **The second rule:** *Design for a phone.* The deck plays full-screen,
> landscape, on a screen about 7 inches wide. Big type, one idea, generous
> air, strong contrast. Anything that only reads on a laptop is a defect.
>
> **The third rule:** *Vary the deck — in composition AND in detail.* No two
> neighbouring slides share a composition; a composition family appears at
> most four times in a deck; the axis, the photo side, the field and the
> density alternate. Each deck invents its own DESIGN LANGUAGE (the outline's
> `theme.language`: the brand's world, four to six devices, its fields, type
> and motion) and composes from it; a signature detail appears on at most two
> slides unless that language names it (2.1.1). Consistency where the viewer
> needs orientation (type, colours, the button voice, section intros, the
> menu pair) — variety everywhere else.

---
## PART I — GLOBAL SYSTEM

### 1. Canvas, units, margins
- Canvas **1558 × 720** (mobile landscape, full-screen on a phone, paired with voiceover). Slides are **visual aids**, not documents.
- Scenes state geometry in **pixels** on that canvas (or "NN%" of it). The builder clamps to the canvas and measures every text height itself.
- **Safe inset 72–86px** for all text and buttons (the exemplar decks sit their left column at x=72 and end the right column at 1490). Full-bleed *media* reaches the edge; readable content never does.
- **Two gaps, both always in play:** the STACK gap (37px) between slots, the TIGHT gap (16px) between parts inside one slot (a title and its detail, a numeral and its label, [sender-name] and [sender-email]). Never one uniform gap — equal spacing makes a detail read as its own item. Stacks (`stack` on the elements) space a column on the STACK gap for you.
- **Panel seams** in multi-panel layouts are hairline or a consistent 16–24px — panels read as one composition, not separate cards.
- **Spacing inside a container is measured against the container**, not the canvas: a card, a tile, a 2×2 cell states the same two gaps and the same inset against the room it actually has (`metrics(box)` in the composer; `gridCells` hands each cell its inner rect). Pass the canvas and you get the numbers above; pass a cell and you get that cell's.
- Optical balance beats mathematical centering: headlines over photos sit slightly above center; stat blocks sit slightly left of center in splits.

### 2. Type roles, sized by content length
Assign a **role**; the builder picks a size inside its band from the text's actual length and the box width, and shrinks toward the band's floor until the line limit holds. Longer content → lower end, or a different role.

| Role | Size band | Character budget |
|---|---|---|
| hero | 92–200 | ≤ 28 chars (160+ only for 1–2 words) |
| headline | 54–80 | 28–70 chars |
| longline | 44–58 | 70–130 chars (a quote, a long sentence) |
| numeral (the hero figure) | 110–260 | ≤ 6 glyphs |
| stat (a figure inside a unit: a value, a price, a row numeral, a chevron glyph) | 40–96 | ≤ 8 glyphs |
| subhead | 34–46 | ≤ 70 chars |
| body | 28–40 | ≤ 160 chars per block |
| button (item titles, chips, labels) | 28–34 | ≤ 22 chars, 1–3 words, never wraps |
| eyebrow | 27–34 | ≤ 24 chars, UPPERCASE |
| fine (detail, attribution, captions) | 28–32 | ≤ 80 chars |

**Hard floor 28px — and 28 is the WORKING size of every secondary line** (eyebrows, captions, meta, button labels) in the exemplar decks; subheads and row labels 30–34; figures in units 44–72; headlines 56–110. **Write the size you mean**: a `size` inside the role's band is honoured exactly. Headlines ≤ 3 lines (4 for a stacked one-word-per-line hero), body ≤ 4 lines, no orphan last word (the builder binds the last two). A three-word line in a body-sized box is wrong: promote it to hero. A 90-character headline is a longline, not a wrapped hero.

**Weights.** The body face has weights and the exemplars use two or three on one slide: eyebrows BOLD (the default), row labels MEDIUM, captions and button labels SEMIBOLD, subheads REGULAR. `weight` on any text element; `lh` (line-height 0.82–0.95 for a headline that breaks, 1.05 for a subhead), `ls` (letter-spacing 0.2–0.3 on spaced caps), a `"\n"` for a deliberate break (one thought per line), `runs` for a two-tone line.

**Condensed display faces** (Oswald, Barlow Condensed, Roboto Condensed, Stint Ultra Condensed) run about 15–20% narrower and read smaller: give them a size at the top of the band or a `size` override, and check the phone.

**Paired lines are one object.** A list item that runs to two lines — a label over its detail — binds on the TIGHT gap; the STACK gap separates whole units. A headline is never half of a pair: it leads the slide.

**Stacks are flex columns.** Elements stacked vertically — eyebrow, headline, subhead, list, buttons, contact block — sit on ONE repeated gap and the finished column is placed in the space it actually has (`stacks.<id>.valign` + `box`). When the room runs short, shed the least load-bearing line (the support line, a detail) before anything overlaps — the builder pushes lower elements down rather than let text collide, so a column written too long runs off the bottom.

**Typefaces:** one DISPLAY face (headlines, numerals, stats, quotes; 600–800) + one clean BODY face (body, labels, UI — in its weights). The pairing is a per-brand design decision. Never a third family. Numerals and stats always in the display face — that is what makes figures feel designed. A display face may lack a glyph: smart quotes fold to straight ones; ★ renders in most faces, ♡ does not.

### 3. Color roles
Map the brand palette onto: **ink** (text on light) · **surfaceLight** (default light field) · **surfaceTint** (cards/wells on light) · **primary** (solid fields, featured buttons, key numerals) · **primaryDeep** (dark fields, photo tints/scrims) · **accent** (warm secondary — featured item, one underline, one numeral, bars in a chart, the "on" dots) · **onDark** / **onDarkMuted**.

Two field colours per presentation, plus footage, plus the BRAND colour as a tinted field (footage under primary at 0.85+). **The accent is the brand's voice — use it wherever it means something** (2.1): several parts of one unit may carry it, the figures, the eyebrow, the bar that wins, one word of a two-tone line. A status colour (a green "// CORRECT", a coral "NOT QUITE") is a hex. Never mid-tone text on mid-tone fill; mute white by alpha (0.8–0.92), not by grey. A single 2-stop gradient is allowed for scrims, dashes, glow strips, bar fills and occasional fields. Use the role TOKENS in scenes; a hex for a colour the theme has no role for.

### 4. Footage and photography
- **Four premium fields, alternated:** full-bleed footage under a tint of the field colour (0.55–0.72) with a scrim under the copy; footage under the BRAND colour at 0.85+; a framed clip panel (radius 28, a tint, a 2px accent stroke at 0.33); and a clean SOLID light field with no footage at all where a big soft disc + lottie, a stat trio or tiles carry the slide. A designed solid field stays solid — the builder no longer ghosts a clip behind it (2.1). What reads as "failed to load" is a bare headline over nothing; a designed solid slide does not. The outline still gives every slide a hint, so footage is always one word away.
- **Every hint unique** — no repeats, and watch for near-duplicates from one shoot.
- **Fit:** people/scenes → cover-crop (faces out from under text). Logos, product, diagrams → contain on a contrasting field.
- **Casting** matches the audience the script implies, consistently. A named person is never a stock face.
- **Text over footage always needs a tint and/or a scrim:** left-aligned text → `scrim:"left"` (dark at the text edge fading out); centred text → `scrim:"center"` (dark top and bottom, lighter middle); a lower third → `scrim:"bottom"`; plus a tint of the field colour (0.35–0.65 keeps the motion visible, 0.7+ makes footage a texture). Never blur.
- Busy footage? Put text in a solid panel beside it (a side column or a card with alpha 0.9) or a band across the lower third instead of a gradient.
- **Multi-panel sets** (2–6 photographable things side by side) are the deck's most premium layouts: hairline seams, each panel its own clip, titles on a bottom band or under the tiles.

### 5. Icons
- **Every list item, menu option, benefit card, button and contact line gets an icon.** A bare bullet is a missed opportunity.
- **One concept, one icon** — shield=protection, home=housing, users=family, calendar=scheduling, document=paperwork, check=confirmation, arrow=progression, map=location, phone/mail=contact. Never repeat a generic glyph down a list.
- The icons are ANIMATED: the builder places the library's 2-tone wired-outline drawing, painted in the brand colours, wherever the name matches; an unknown name falls back to the static glyph. Use plain, common nouns.
- Sizes: inline with a row ≈ 36–44 · card icon ≈ 40–56 · hero/feature icon 3–5× body. Bare on colour, or inside a soft tinted rounded well ≈ 1.7× the icon (`well:true`).
- Tone: ink on light, white over dark/footage, accent only for the one feature icon. Tint the well, not the icon. Never emoji.
- **Which drawing where — the 2-tone wired outline, everywhere (binding, Bren 2026-09-09).** Standalone icons, icons above a line of text and button/pill icons alike are placed as the library's 2-tone wired outline drawing, painted in the brand colours (ink on light fields, white on dark, a hex when asked). The flat (designed) and 1-tone system drawings exist and a client can ask for them in an edit turn (`setIconStyle`); one icon style per group on a slide. An icon knows what it is, so an edit can change any part of it: every placed icon carries `idecide/icon`, `idecide/iconTone`, and for an animation `idecide/lottie`, `idecide/iconStyle`, `idecide/iconStroke`, `idecide/iconAccent`, `idecide/iconSwaps`, `idecide/iconColors` — the full rules are in `assets/docs/element-contract.md`.

### 6. Interactive units (buttons, menu rows, tiles, chips)
Always **one grouped unit** of stacked layers: plate (fill + corner radius = the hit target; carries the click) → icon → label. The builder builds them from the `button` element; you place the box.
- **EXACTLY ONE animated icon per button** — the label's concept, else an arrow (a check on finish/agree labels, a left arrow on Back). Never two.
- **Text-style (backgroundless) buttons** are legitimate on quiet designs: the builder lays a fully transparent plate under the whole box and puts the action on it, so the click never misses. Give them a clear 64px row and an underline or a leading icon so they read as actions.
- **Padding is non-negotiable:** ≥ 24px horizontal inside a plate, label never touching the edge, label never wrapping — widen the plate or shorten the label. Buttons in a set share ONE width; a solo pill may `fit:true` to hug its label.
- **Two button voices (2.1):** the primary action is a SOLID pill (accent or white plate, dark label, 64–84 tall); the secondary is a GHOST (`style:"ghost"` — fill off, a 2px accent stroke, label in the accent). The exemplars type the arrow into the label ("Buy now  →", "Try again  ↺") with `icon:false`; either the typed glyph or the one animated icon — never both.
- **A designed row is a button too:** a `repeat` of wired items makes every cell one click target (an invisible plate under the whole cell carries the wiring and the track mark) — a menu row of numeral · icon · label · chevron · divider is a button, and nothing in it can separate.
- **Sizes:** 68–84px tall (the builder raises anything under 64; 72–84 for menu rows), 520–680px wide in a column, ≥ 490px for full rows, 14–18px between rows. Glass plates carry a hairline stroke on both fields; the icon inside a 72px+ plate is 40px.
- **Featured** = solid primary (accent on a primary field) + onDark text — only for Move Ahead / Finish Up / Continue / Next. Question answers are all equal.
- **The cover has exactly one action element**, the static "Click anywhere to Begin" pill; the platform advances slide 1 on any click.

### 7. Composing a slide (procedure)
1. **Read the content** — counts, numbers, items, whether a choice is being made, what the footage can show.
2. **Classify the beat** — open · divider · emotional · explanation · proof/stat · list · story/quote · person · process · choice · close.
3. **Read the beat's shape** (the outline's intent names it — never a composition) and decide the composition yourself, from the deck's DESIGN LANGUAGE: the stage (one of the fields the language alternates), the grid (one column, an unequal split, equal cells, a shift), where the copy sits, what the substance is built from — a `repeat` of designed cells, a native graphic, a quote, a hero figure — and which of the language's devices supplies the detail.
4. **Assign type roles**, size by actual length (§2). Put the copy column in a stack.
5. **Place every element**: back-to-front, pixels on the canvas, safe inset kept. Buttons for every wired item on a slide that waits for the viewer.
6. **Icons** for every discrete item (§5).
7. **Color roles**, contrast check, accent once (§3).
8. **Footage pass** — unique hint, cast, tinted and scrimmed (§4).
9. **Rhythm check** against neighbours (§8) — name the composition family.
10. **Polish check** (§9).

### 8. Rhythm across a presentation (50–150+ slides)
- **Never the same composition twice in a row** — and a mirrored split (photo left after photo right) is still a split: change the family, not just the side.
- **Any family appears at most 4 times**; prefer 1–2. A 50-slide deck should show a dozen distinct families.
- **Alternate value:** no more than 2 footage-heavy slides consecutively without a solid or light-panel slide between, and vice versa.
- **Alternate density:** dense (cards, rows, graphic) → sparse (hero, statement, stat).
- **Parallel beats stay identical:** one section-intro composition, one menu composition (First and Return share footage and layout), one CTA family per deck. Orientation where the viewer needs it; variety everywhere else.
- **Rhythm of devices (2.1.1):** a signature detail (a rule beside a headline, a badge, a numbered row, a glyph eyebrow, a struck figure, stacked rings) appears on at most two slides unless the deck's language names it — and then never on neighbours and on no more than a third of the deck. Habits (a framed clip, a light field, a divider, one headline entrance) stay under a quarter to a third of the deck. The design pass receives a ledger of what the deck has spent before every batch.
- **A bare slide is a headline over nothing** — none of those. A designed solid field (disc + lottie, a stat trio, tiles) is not bare; use two or three per deck for rhythm.
- **Voiceover pairing:** on-screen text ≈ one third of what is spoken. If the slide reads like a transcript, cut it.

### 9. Polish checklist (fail any → fix before moving on)
- Nothing below 28px · headline ≤ 3 lines · body ≤ 4 lines · no orphan last word.
- Every unit's parts read as ONE object: label inside its plate, eyebrow with its headline, detail on the tight gap under its title.
- Every wired item has its button; every button has its one icon; nothing readable within 86px of an edge.
- Every text element's role matches its actual length; a headline never runs past the safe area.
- Every list/menu/button item has a distinct, relevant icon.
- All text over footage sits on a tint/scrim or a solid panel and passes contrast.
- One primary action per interactive slide; question answers equal.
- No duplicate or soft footage; every graphic from real figures (a graphic may share a slide with a stat trio or a legend).
- The accent used where it means something; a status colour is a hex; ≤ 2 field colours deck-wide plus the brand-colour tint.
- Composition differs from the previous slide; family not already used 4×; no signature device past its two uses, no headline entrance on a quarter of the deck.

### 10. Building in myiDecide (img.ly CE.SDK v1.74.1) — what the builder does with a scene
- Page **1558 × 720**, one page per slide. The stage's solid field is the page colour itself; footage is placed through the platform's api (device-adaptive renditions), never as a raw fill.
- Every text is **one block per string**, auto-height, named by role (`headline`, `eyebrow`, `body`, `numeral`…); buttons are `btn:<target>` / `btnurl:<url>` / `btnfinish:<TITLE>|<url>` plates with `btn/icon` + `btn/label`; lists are `row/*`, `card/*`, `chip/*`, `step/*`; graphics `bar/*`, `dot/*`, `stat/*`, `chart`.
- Group marks (`idecide/group`, `idecide/groupRole`) ride in the slide's bytes so wiring, animation, edits and audits all see the same units.
- **Animation:** heroes rise on their baseline; buttons grow; panels and rows slide from the nearer edge (the edge rule); bars grow upward; everything else fades; slots 0.18s apart bottom-to-top, parts 0.08s apart; stage media fades in slowly; EaseOutQuint everywhere; every layer runs to the slide's end (narration + 0.5s tail).
- The page is stamped `idecide/layout: fixed`: later passes only tighten text boxes, keep buttons apart and keep everything on canvas — a designed composition is never re-packed.
- **Keep effects simple:** solid or single 2-stop gradient fills, at most one soft shadow per element, plain rect / rounded rect / ellipse. No blur, blend modes, masks or layered shadows.
- Personalization tokens are literal bracketed strings in their own text blocks; the sender block is one element.

### 11. Failure modes this document prevents
| Symptom | Root cause | Correct move |
|---|---|---|
| Paragraph crammed where a numeral belongs | Wrong role | Re-classify: a stat graphic + a headline that says what the number means |
| Three words floating in a body box | Role mismatch | Promote to hero |
| Headline running 5 lines | Box too narrow / role too big | Widen the box or drop to longline, or write shorter |
| Text unreadable over footage | Missing tint/scrim | Add a scrim under the copy or move the copy onto a panel |
| Seven items in a slide | Content past what a phone reads | Merge or cut to 3–5; a genuine set of N things becomes N panels or two beats |
| Two icons in a button | An arrow added beside a concept icon | One icon per button — the concept, else the arrow |
| A click that misses | Text button with no plate | The builder plates text buttons; keep the row 64px tall |
| Every slide looks the same | Families repeating | Change the family, the axis, the field, the density |
| Slide reads like the narration | Copy dumped from script | Keep the key phrase; let voiceover carry the rest |
| Numbers as prose | A figure buried in body text | A bar chart / dot grid / ring / stat graphic — the number IS the slide |

---
## PART II — WHAT THE TEST BUILDS TAUGHT (decks 302 and 303, 2026-09-15)

Twenty-eight demo slides across ten markets and a full 25-slide Patagonia deck were composed freehand against this canvas. What held up, and now is the rule:

- **Bespoke beats templated.** Every slide composed for its own content read better than the closest library template. The compositions that worked: full-bleed footage with a left scrim and a hero; split with a 42–46% media column; a centred statement on a darkened clip; a stat with a dot grid or an animated bar chart beside a short headline; three or four photo cards under a headline; numbered steps across the lower half; a quote centred with a fine attribution; a menu of pills over a tinted clip; a hamburger of text-style rows on a quiet field.
- **Data graphics sell.** The finance response's animated bar graph, the charity response's dot grid and the city-planning response's rings made the point faster than any sentence. Where the script gives a figure, draw it.
- **The edge rule.** An element on the right half enters from the right, on the left from the left; text rises; buttons grow; the stage never animates. Motion that agrees with position reads as intent.
- **One icon per button, always animated.** Concept icon when the label has one; otherwise an arrow; a check on finish/agree; a left arrow on Back. Doubling an arrow next to a concept icon read as clutter.
- **Text buttons need a plate.** A label with no background missed clicks until a transparent plate covered the row and carried the action.
- **Condensed faces run small.** Oswald and Barlow Condensed needed 15–20% more size than the band suggested.
- **Tints between 0.4 and 0.65 keep footage alive** under white type; 0.7+ turns it into texture (fine for dense slides).
- **Track choices by default.** Question answers, menu topics and final outcomes are recorded for the sender without being asked.
- **The real logo, found online, on every slide the design asks for** — uploaded once, placed as a Contain image, light or dark drawing by field.

---
## PART III — WHAT THE FIRST LIVE 2.0 BUILD TAUGHT (deck 304, Patagonia, 2026-09-16)

The renderer worked; the designs were thin. Thirty-seven slides came out as a headline over darkened footage, a headline beside a hard-cornered panel, three grey rows with empty wells — competent and generic — and the client said so. What changed, and is now binding:

- **A slide is layered — six to twelve elements.** Stage · eyebrow or accent rule · headline · ONE substance element (cards on stroked plates with icons in tinted wells, rows with wells, a dot grid, an animated bar chart, photo tiles with captions, a pull-quote and its attribution) · ONE detail (a badge, a divider, a soft disc behind a numeral, a legend). Two-element slides are for section intros and closing lines, at most four per deck. The design contract's worked scenes show how such a slide is written; compose to that standard in the deck's own language.
- **Every slide is designed.** The menus never reached the design pass (the outline had given them their items, so the template-era shortcut skipped them) and fell back to a flat row list. In 2.0 a slide without a scene is a slide that still needs designing, whatever copy it already has.
- **The band is binding and the builder sizes by length.** Headlines were written at 40 and 34, support at 26, a figure as body at 104. `size` is honoured only inside the role's band; a figure is a numeral; a two-to-four-word headline is one line (the box widens before the words break).
- **The corner mark is the builder's.** The real logo, small, top-left (or the first free corner) on every slide after the reveal — the way deck 303 carried it — unless the scene places a logo or says `logoMark:false`. The reveal draws the mark large and centred from its real proportions (`logoUris.aspect`), whatever box the scene gave.
- **Framed panels get 24px corners; columns never overlap.** A panel that touches no edge is rounded by default; a text box that runs into the column beside it narrows to a 28px gutter before anything is drawn (the overlap resolver had shoved a list's first row sideways instead).
- **An empty well is worse than no well.** Icons draw first; wells go in behind an icon that landed. The tones a scene names (an accent well icon) are prepared with the slide's icons, and a tone that still fails falls back to the field's own.
- **Glyphs persist as Contain.** The engine resets an icon's fill mode on load exactly as a photo's; the in-visit guard now re-asserts Contain on glyphs, so the byte-level verify stops counting them and no slide pays a corrective revisit for its animated icons.
- **The sender block keeps its lead line** ("For more information, contact") — a 110px box used to drop it; the default is 140.
- **The review edits by element.** The reviewer sees the element list (index · type · role · text fragment · box) and matches by role family or text; a redesign is a full scene under the same contract and anatomy, never three text blocks in a corner.

---
## PART IV — WHAT THE EXEMPLAR SLIDES ARE MADE OF (decks 302/303, dumped block by block, 2026-09-16) — THE CRAFT, NOT THE MOTIFS

Bren named thirteen slides as the standard. Every one was dumped from the engine and read; `updates-findings/anatomy/2026-09-16-deck-302-303-exemplars.json` is the record. What they share is CRAFT, and that is the rule; the motifs they happen to use (a vertical accent rule, numbered rows, a struck price, stacked rings, a two-tone wordmark, a colour key) belonged to those brands and are not a kit (2.1.1, below):

- **Loose parts, not composites.** 14–38 blocks a slide; nothing canned. A row, a card, a bar, a stat are each built from three to six parts on a plate (visible or not) and move as one. The `repeat` element stamps one designed cell per item; the contract's worked scenes show the mechanics.
- **Three weights on one slide.** Regular subheads, Medium row labels, Bold eyebrows and SemiBold captions — in the body face. The display face for headlines, figures and the wordmark.
- **28 is the working size** of every secondary line; 30–34 subheads and labels; 44–72 figures in units; 56–110 headlines with deliberate breaks at lh 0.82–0.95; spaced caps at ls 0.2–0.3.
- **The accent is everywhere it means something** — numerals, chevrons, rules, values, the winning bar, one word of a two-tone wordmark. Muted white is alpha 0.8–0.92.
- **Several fields**: footage under the field tint + a scrim; footage under the brand colour at 0.85+; a framed clip on a solid field; a clean solid field carried by its own unit. Three of the thirteen carry no footage at all.
- **Chosen motion, varied**: each slide's entrance picked for what it shows — wipes on rules and bars, grows on pills, tiles and discs, slides from the near edge on rows and panels, and a different text entrance on different slides.
- **Geometry on a grid**: columns of different widths on purpose, each filled top to bottom by its unit; a shift in the grid between two levels of content (two cells over three).
- **Two button voices**: a solid primary and a ghost secondary; an arrow may be typed into the label.
- **The rules that had to go**: accent-once, one-figure-per-slide, footage-on-every-slide with a ghost clip behind solid fields, "40–60% air", one-graphic-per-slide. They were written to stop templates misfiring; against the exemplars they read as the opposite of designed.

---
## PART V — WHAT DECKS 305 AND 306 TAUGHT (Patagonia, Opus and Fable, 2026-09-16/17)

The same outline built on two models came out "noticeably better" and nearly identical in its faults — so the faults were the builder's and the contract's, not the model's:

- **The exemplars became a kit.** A vertical accent rule on 9 and 12 of 45 slides, a headline swipe on 25 and 30, short accent dashes on 12 and 27, numbered rows, "// " eyebrows and stroked badges on slide after slide. The client: "each build should be choosing unique elements for each presentation's specific needs." Now the outline invents the deck's DESIGN LANGUAGE (`theme.language`), the design pass composes from it, a ledger of spent devices rides with every batch (signatures spent at two uses; habits rested at their share of the deck), and the reviewer checks device rhythm.
- **A framed clip's corners are its SHAPE's.** The radius was written on the graphic block, where the engine ignores it — square footage inside a rounded plate. The builder now writes it on the shape, and a plate drawn on the same box shares it.
- **Measured text moves its column.** Headlines that wrapped landed on the line under them; half a row of cells moved without the other half. Units (a stack, a repeat's cells, a plate and what sits on it) are now measured and pushed as wholes; a repeat's cells grow to hold a label that wrapped; a column that still runs long rises into its own top margin, then takes the difference from its display line — never by stacking type on type.
- **A line meant as one line gets its width.** A row label, an eyebrow or a name line with a token widens toward its column (never under the part beside it); running copy wraps where the designer's column says.
