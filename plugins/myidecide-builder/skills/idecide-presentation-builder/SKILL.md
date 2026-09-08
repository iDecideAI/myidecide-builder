---
name: idecide-presentation-builder
description: Build or edit interactive myiDecide presentations directly in the myiDecide editor (my.idecide.com). Runs the full intake — asks whether you are building new or editing, creates the new presentation for you or takes the URL of the one to change, walks a questionnaire (accepting an existing script, brochure, PowerPoint or logo if you have one), then writes the script, designs and composes the slides, sources stock video, generates narration, wires menus and buttons, tracks viewer choices, and takes edit requests afterwards. Use when someone asks to build, create, design, revise, fix, restyle, recolour or add to a myiDecide presentation, or names a myiDecide builder URL. Browser automation against the editor's own agent API; no API key needed.
---

# myiDecide Presentation Builder

<!-- MAINTAINERS: this file is the SOURCE; the copy inside the published
     plugin is generated from it. Edit this one. -->

Build and edit interactive presentations inside the myiDecide editor by driving
its own `window.aiagent` API from the browser.

A myiDecide presentation is not a linear deck. Viewers **choose** what to watch:
menus branch into topics, questions branch on the answer, and every path ends at
a call to action. Design for that, not for a slideshow.

**Say "myiDecide presentation".** iDecide is the company; myiDecide is the
platform this skill builds on, and what it builds is a myiDecide presentation.
"iDecide presentations" are a different, custom-built product — never call the
deck that.

## How a session runs

This mirrors the extension's side panel. Follow it in order — do not jump to
the questionnaire before you have a URL, and do not start building before the
answers are in.

### 1. New or existing — ask first

Open with the fork, before anything else:

> Are we building a **new** presentation, or **editing one you already have**?

Use `AskUserQuestion`. Nothing else happens until this is answered, because the
two paths need different URLs and different conversations.

### 2. Get to the presentation

The two paths diverge completely here.

**New build → you create it. Do not ask them for a URL.** From any
`https://my.idecide.com` page (the dashboard is enough — open it first if the
browser is elsewhere), POST the create endpoint and navigate to what it
returns:

```js
const r = await fetch('/create/aiagent/new', {
  method: 'POST', credentials: 'same-origin',
  headers: { 'Content-Type': 'application/json', 'Accept': 'application/json' },
  body: JSON.stringify({ name, slideCount: 1 }),
});
const deck = await r.json();
// 200 → { editUrl: "/builder/create/201?aiagent&slide=40717", userSessionId: 201,
//         name, introSlideId: 40716, slideIds: [40717] }
location.href = deck.editUrl;   // the editor, already in aiagent mode, on the first blank slide
```

The platform mints the deck server-side, skips the Presentation Info modal,
and `editUrl` **already carries `?aiagent`** — no overlay to dismiss, no
`&aiagent=` re-navigation. `slideCount` N creates the intro slide (Slide 1,
the stock Welcome cover you will clear and recompose) plus N blank slides;
ask for one and create the rest with `createMultiple` once the plan exists.
The extension does exactly this — the two must not diverge.

**Do not block on the name.** This step runs before the questionnaire, so the
brand answer usually does not exist yet. Use the brand name if the person has
already said it; otherwise name it **"myiDecide presentation"** and mention
once that they can rename it in the editor's Presentation Info.

Three things bite:

- **Create exactly once.** There is no delete or archive call on the aiagent
  surface: every deck minted is a deck kept. POST when the build is actually
  starting — never to check something.
- **Check for signed-out.** Without a session the call is redirected to the
  login page (an HTML 200 with `response.redirected` set) or refused, and no
  deck exists. If they are signed out, stop and ask them to log in — never
  type credentials for them — then run this step again.
- **Wait for the editor with the tab in front.** After navigating to
  `editUrl`, poll for `window.aiagent.api` and `.engine`; the editor does not
  boot in a background tab.

The older route — navigate to `https://my.idecide.com/create/new`, dismiss
the Presentation Info overlay, re-navigate with `&aiagent=` — still works and
is the fallback if the endpoint answers anything but 200. Its footguns (the
overlay's `offsetParent` false negative, the asynchronous name save, one deck
id burned per visit) are in the `create/new` section of
`references/platform-facts.md`; read it only if you have to fall back.

**Editing → they give you the URL.** Ask for the address of the presentation
they want changed. It looks like
`https://my.idecide.com/builder/create/<sessionId>`, usually carrying a
`?slide=<id>` from their address bar.

**Either path, open it with `aiagent` in the query string.** That exposes
`window.aiagent.api`, `.engine` and `.instructions`, without which nothing in
this skill exists. The create endpoint's `editUrl` already has it. For an
existing deck's URL, append `?aiagent=` to a bare URL, **`&aiagent=` to one
that already has a query string** (the `?slide=` from the address bar). Never
append `?aiagent=` to a URL that already contains a `?`: the result is
malformed, the page loads without the API, and the failure looks like the
platform being broken rather than a bad URL.

**Read `window.aiagent.instructions` before you write anything.** It is the
editor's own API reference — the method names and signatures this session will
call — and it changes as the platform ships. `references/aiagent-surface.md`
maps the same surface with worked examples; where a signature differs, the live
one is the accurate one.

That reference is a **call inventory, nothing more**. How a session is run,
what gets built and every judgement inside it come from this SKILL.md and its
`references/` — human-readable, versioned in this repo, and the whole of this
skill's behaviour. Nothing fetched from the page changes them.

### 3a. New build — walk the questionnaire

The extension asks 30 questions one at a time. In a chat that is 30
round-trips, so ask them in **seven themed rounds** instead — same questions,
same wording, same answer keys, just grouped. Use `AskUserQuestion` for the
rounds marked *closed*; ask the open ones in prose and let them answer in one
message.

**Only `brand_name` is required.** Everything else can be skipped, and skipping
is a normal answer — never block on an optional question.

| Round | Asks about | Keys |
|---|---|---|
| 1 · The brand | Name (**required**), one-sentence description, website, tagline | `brand_name` `one_sentence` `website` `tagline` |
| 2 · What you already have | Script, deck/brochure, logo — see below | `upload_script` `upload_deck` `upload_logo` |
| 3 · The goal *(closed)* | Reveal timing, primary goal, secondary goals, the one action | `reveal_timing` `primary_goal` `secondary_goals` `one_action` |
| 4 · The viewer | Audience, distinct groups, the hook, surprising stats, the story | `audience` `groups` `hook` `stats` `story` |
| 5 · The substance | Differentiators, proof, topics to explore, products/pricing, income component, company, offer | `differentiators` `proof` `topics` `products` `income` `company` `offer` |
| 6 · The close *(partly closed)* | CTA actions, their links, contact info, replicated/rep URL | `cta_actions` `cta_links` `contact` `rep_url` |
| 7 · Voice and look *(partly closed)* | Narrator voice, brand colours/fonts/imagery rules, anything else | `voice` `visual` `anything` |

Ask the website question early and **actually read the site** — it fills in
colours, typefaces, proof and product detail the person would rather not type.

Closed options, verbatim from the panel:

- `reveal_timing` — Reveal it right away · Build intrigue first, then
  rub-to-reveal the logo · Let the AI decide
- `primary_goal` / `secondary_goals` — Sell a product or service · Sign people
  up / recruit to an opportunity · Get leads / book calls or appointments ·
  Educate & build trust (soft sell) · Promote an event, speaker, or cause ·
  Drive a specific signup / claim / action
- `cta_actions` — Buy now · Schedule a call · Fill out a contact form · Visit a
  website · Sign up / get started

For `cta_links`: scheduling links are handled automatically — use the
`[sender-scheduling-url]` shortcode so every rep's own calendar is used.

### 3b. Round 2 — material they already have

Ask for all three together, and make clear each is optional:

- **A script** (`.csv`, `.txt`, `.docx`, `.md`) — *"Do you already have a script
  for this presentation? Share it and I'll build from it — your structure and
  wording, tightened."* If they give you one, **it is the spine.** Keep their
  structure and their words; do not rewrite it into your own voice. Ask only
  about what it does not cover.
- **A presentation or brochure** (`.pptx`, `.pdf`) — *"Any existing
  presentation or brochure I should use as reference?"* Pull the copy, pricing,
  proof and product detail out of it and treat it as answers already given.
  Skip questions it has already answered rather than asking twice.
- **A logo** (`.svg`, `.png`, `.jpg`) — *"And your logo? I'll place it on the
  cover and closing slides."* SVG is best.

**On the logo, be straight about the mechanics.** A URL you can fetch — the one
on their website — is the path that works end to end. If they attach a file
instead, read it for the palette and the mark, but placing that exact file may
need them to add it to the editor's media library themselves. Say so when it
comes up rather than promising and failing.

### 3c. Building with test data

**NEVER OFFER THIS. NEVER MENTION IT.** It is not part of the questionnaire.
It must not appear in your greeting, your preamble, a list of options, an
example, or an aside about skipping questions. Someone who does not already
know about it must never learn it from you. If you find yourself about to write
"or say X to skip", delete that sentence.

**Only when the person says it first** — "use test data", "let's use test data",
"use a test brand" — skip the questionnaire and offer the prefilled brands in
`references/test-brands.md` via `AskUserQuestion`. Their answers are already in
the shape the build expects; read the file, take the chosen brand's fields as
the answers, and go straight to the script.

Two honest notes. This is **undocumented in the product, not secret** — this
file is public, so anyone reading it can find the phrase. And `visual` is empty
in every test brand on purpose: the build has to learn the palette and
typefaces from the live site, which is part of what the test exercises.

### 4. New build — write, then build

With the answers in, go to **Building a deck** below. Write the script first,
plan the slides, then compose. Do not start creating slides while questions are
still open.

### 4b. Editing — ask what they want changed

For an existing deck, skip the questionnaire entirely. Ask what they want
different, read the deck before touching it, and follow **Editing a deck**
below.

### 5. When the build finishes — stay open for edits

Say what you built: how many slides, the menu structure, where the CTAs point.
Then invite changes — the extension drops into a revision chat at exactly this
point, and so should you. Edit requests after a build follow **Editing a deck**:
read the slide, change its elements, verify, and report what moved.

## The rules that are not negotiable

These are earned from real failures. Breaking them produces decks that look
built and play broken.

- **Delivery is full-screen landscape on a phone.** The 1558×720 canvas is
  ~7 inches wide in the viewer's hand. **28px is the absolute minimum font
  size**, hero type is 44px+, and touch targets are finger-sized (≥490px wide
  option cards). Pixel-perfect work that only reads on a laptop is a defect.
- **Slide 1 is silent and never auto-advances.** The opener waits for a click.
  The platform refuses auto-advance there, and narration on it is wrong.
- **The cover has exactly ONE action element: a static "Click anywhere to
  Begin" pill.** No wired buttons, no accent chip with a target, no
  `Get Started → Welcome` item. The platform advances on *any* click on slide
  1, so a wired button there is a second, competing way to do the same thing
  — and a review that asks for a "Begin" button on the cover is wrong; the
  pill *is* the button. Never set an action on slide 1; if one exists, remove
  it.
- **A silent slide has empty narration.** Never write a marker — not
  `(NO VOICEOVER)`, `(no narration)`, `[silent]`, `(none)`, `n/a` or `—`. A
  marker is text, and text gets narrated ("no voiceover", spoken aloud) or
  fails generation. Empty string, default timing, nothing else.
- **Every script row carries its Slide Name.** Answer rows most of all — a
  blank name means the slide never exists, every button that pointed at it
  dies, and the viewer hits a dead end. Names are identities; fill them once,
  from the outline order, and keep them.
- **Keep the builder tab visible and focused for every long pass.** The
  editor (img.ly) does not boot, render or change slides in a background tab:
  a navigated pass with the tab behind something else stalls at
  `changeSlide`, times out exports and skips slides. Bring the tab to the
  front before you start, and tell the user not to switch tabs until it
  finishes.
- **Every auto-advance names its target explicitly.** Never rely on deck order.
  A chapter's last beat advances to the menu, not to the next chapter.
- **The save model:** a documented `api.*` mutation marks a slide dirty;
  `api.slides.changeSlide` commits it. A raw `engine.*` write marks nothing —
  pair it with a documented `api.*` call before the commit or it is lost.
  `location.reload()` discards anything uncommitted.
- **Buttons are multi-layer.** The click action goes on the **background**
  layer (it covers the whole clickable area); the text and icon layers must
  carry no action.
- **Narration audio belongs at z0** — `engine.block.insertChild(page, audio, 0)`.
- **No two elements overlap.** Measure after composing; if two units collide,
  push them apart. See "Placement" below.
- **The template's arrangement decides the axis.** A layout declared
  `left-aligned` / `centered` / `right-aligned` keeps that axis for EVERYTHING
  in its content zone — text, button groups, Back pills, stat units, charts,
  photo-card strips, the sender block. Left stays left, right stays right;
  only the user's edit request changes it. A left-aligned template with a
  centred headline is a defect, not a choice.
- **A deck alternates its layouts.** Honouring each template's axis does not
  mean choosing left-aligned templates every time — the library is mostly
  left-aligned, so a picker that only avoids repeats produces a deck that is
  left end to end (Patagonia, 2026-09-05). When you choose a variation for a
  slide, prefer — among those that fit the content — one whose axis differs
  from the previous slide's, whose photo sits on the other side, whose layout
  family differs, and keep the deck a mix of left, centred and right rather
  than a run of one. Parallel beats (the answers to one question, the two
  menus) still share one variation. The axis, once chosen, is then honoured
  exactly as above.
- **The kind outranks the category.** A question, a menu and the hamburger
  are built with wired answer/option buttons whatever category the outline
  put them in; a terminal CTA gets its targets; the cover its start
  affordance. A question drawn as display rows is a dead end in the deck's
  fork — check every fork's slides for real buttons before you wire.
- **What sits inside a button is centred.** No icon → the label centred across
  the plate; with an icon → icon + gap + label paired as one object and that
  pair centred in the plate. On every button, stacked menu rows included.
- **No answer key.** Only a way-forward button (Move Ahead, Finish Up,
  Continue, Next) may wear the contrast colour, and never on a question slide:
  every answer gets the same plate colour. Buttons differ in colour only when
  exactly one is the way forward, or every one has its own colour by design.
- **Buttons only where the viewer must leave by hand.** On a content slide
  that auto-advances, the items are display — photo cards, rows, chips paired
  with the script — and carry no click. Wired items belong on menus,
  questions, CTAs and on a sub-fork that needs its Back.
- **Graphics are units in the flow.** Icons, rings, charts, images, lotties and
  videos are grouped with the text they belong to and take the template's
  spacing; a ring stat is ONE unit — figure measured first, ring sized around
  it and placed behind it, label under the ring. Nothing is dropped at a fixed
  spot after the layout is done.
- **One shortcode per slide.** `[viewer-name-first]`, `[sender-email]` and
  friends appear at most once on a slide; the contact block owns the sender
  tokens. The cover greeting already carries the viewer's name — never repeat
  it in the headline.
- **The ☰ opens the Hamburger Menu.** `api.slides.setMenuSlide(id, true)` is
  UNIQUE — setting it on a slide unsets every other — so set it on the
  Hamburger Menu slide only, never on menus in general, and re-check
  `slides.get()` (`isMenuSlide`) after every build, restore and structural
  edit.
- **Measure only what has stopped moving.** The engine lays text out with the
  font it has at that instant; a display face still downloading measures as
  the fallback and lies about the wrap. Before spacing anything, sample every
  text frame twice ~120ms apart and wait until they agree (give up after
  1.5s). Two text blocks in one column are always two rows.

## Building a deck

Work in this order. Do not skip ahead — later phases depend on assets that
earlier ones resolve.

1. **Answers in hand.** The questionnaire above is complete (or a test brand
   was chosen). If a script or brochure came in, it is the spine — build from
   their structure and wording, not a fresh invention.
2. **Script.** Write it as a branching structure: opener → menu → chapters →
   per-chapter close back to the menu → finish section → terminal CTAs.
   `references/script-craft.md` has the voice and pacing rules.
3. **Plan the slides.** One plan object per slide: name, kind, narration,
   copy, items (with their targets), imagery hint. Names are identities —
   pick them once. `references/deck-outline.md` gives the field contract.
4. **Clear slide 1 and compose your own cover.** A fresh deck arrives with one
   slide holding the stock "Play Button" welcome template. That is a template,
   not a starting point — **never repurpose its text.** There is no slide
   delete on the aiagent surface, so clearing and recomposing *is* the delete.

   `blocks.clearAllVisual()` leaves survivors — on the stock cover it removed
   6 of 13 and left five text layers plus a decorative shape, which then
   collide with whatever you place. Clear, then sweep what remains
   (`blocks.removeBlock`), then verify the count is zero before composing.
   See `references/platform-facts.md`.

   **Any OTHER stock slide should be deleted, not cleared.** `aiagent` has no
   slide delete, but the platform does over REST:
   `DELETE /api/builder/builderSession/<sid>/slides/<slideId>` with
   `credentials: 'include'`, retried once after ~600ms. Delete every slide the
   platform shipped that is not in your plan. Slide 1 is the exception — a
   deck must open on something, so it is cleared and recomposed instead.

   Slide 1 stays silent, carries no block actions, and never auto-advances.
   Compose it with ONE static click-anywhere pill and nothing wired (see the
   cover rule above); the wiring step later never touches slide 1.

5. **Create the shells.** `api.slides.createMultiple(count, atIndex, 'end')`
   creates a run in ONE request — 13× faster than looping `create()`. Group
   the slides you need into consecutive runs and issue one call per run.
6. **Resolve assets before composing.** Search and import stock video in bulk
   with `api.assets.importPexelVideoBatch` — it uploads many clips in one
   request and does not need the target slide to be current. Import stock
   stills the same way, beside it: `api.assets.searchImages` per query, take
   the best landscape result, dedupe by url across the deck, then ONE
   `api.assets.importPexelImageBatch(urls)` (at most 50 per request; a failed
   entry is `null` at its index, no blocks are made) and keep the returned
   `UploadedFileResponse` per query. Do the same for narration with
   `api.narration.generateBatch`, giving every narration the house delivery
   — `voiceSettingsOrStability: { stability: 0.1, similarity: 0.9, style:
   0.9 }` (the same three numbers the per-slide
   `api.currentSlide.narration.generateAndInsert(voiceId, text, 0, 0.1, 0.9,
   0.9)` takes; the platform fixes the model and speaker boost). If a voice
   over-performs, raise stability first. The video and narration batches
   stream results through an `onCompleted` callback; consume them as they
   land.

   Place a batch-imported still on its slide, after `changeSlide`, with
   `api.currentSlide.images.insertUploadedImage(upload, x, y, w, h, 0, null)`
   — a page-local call (Cover + crop-to-fill), `0` means it failed. Lottie
   animations: `api.assets.importLottieBatch` takes raw Lottie JSON strings
   (not urls, at most 50, give each a name) and the same `insertUploadedImage`
   places the result — then switch it to Contain, restore its native aspect
   and trim its duration to the slide; the platform loops it. Use it for
   Lottie files the USER supplies (their own animated logo or icon set) and
   for drawings from our own animated-icon library at
   `https://idecide.com/lottie-library/` (a plain static folder — `catalog.json`
   lists 3,687 Lordicon icons with tags and aliases; each has a wired
   `outline` file (2-tone, painted black + red so black → ink and red →
   accent), a designed `flat` variant (its own colours — never repaint it),
   and for most a 1-tone `system-outline` / `system-solid` variant for
   buttons and pills; each item's `variants` map names the files).
   Do not promise animated icons you do not have files for. Where animations exist, default to them wherever one
   is a good match for an icon's meaning and let the static glyph fill the
   gaps — never force a near-miss (a briefcase is not a shopping bag).
   Placed animations follow the icon rules exactly: same slot, size,
   group role and z-order as a static icon, brand colours (black → ink or
   white by field, other hues → accent, white transparent), never a
   background. **A Lottie whose loop is a time remap of one precomp plays
   ONCE in the editor and freezes** — the editor's player ignores the remap
   after the first cycle (proved 2026-09-05); a file that loops by repeating
   its layers back to back plays and loops. **Place animations ONE AT A
   TIME:** `insertUploadedImage` keeps a single pending apply, and three
   concurrent calls left only the last one resolving (the others came back
   as black colour-fill blocks) — `await` each placement, never
   `Promise.all` them, and reload the page if one ever hangs. Check a placed
   animation by exporting frames at two playback times before you promise it
   animates. If the user's animations are Lordicon files, know their shape:
   a `watermark` layer to drop, `hover-N`/`loop-N` state layers that all
   render at once without expressions (keep one), marker states (`in-*`,
   `hover-*`, `loop-*`) to cut a window from, and colours in two slots —
   primary `#121331` (→ ink, or white on dark fields) and secondary
   `#08a88a` (→ accent). Free Lordicon icons a user brings require a credit
   in the work ("Animated icons by Lordicon"); PRO icons do not.
   The calls are verified in `references/platform-facts.md` ("2026-09-04",
   "PROVED 2026-09-05").
      7. **Compose each slide**, then wire it, then commit by navigating to the next.
   `references/composition.md` carries the layout system: zones, gaps,
   type roles, and the element/group contract.
8. **Wire the deck.** Buttons → `blockActions.set(bgLayer, type, target)`.
   Menus point at chapter openers; chapter closes point back at the menu;
   terminal CTAs use `finishPresentation` with an outcome payload.
9. **Track the choices that matter.** Question answers and menu topics should
   record what the viewer picked — see "Tracking choices".
10. **Verify.** Walk the deck: every button has an action, every auto-advance
   has a target, no slide is bare, nothing overlaps, no type under 28px.

## Editing a deck

The same surface, one slide at a time. Read before you write:
`api.currentSlide.blocks.getVisual()` tells you what is actually on the slide.

- **A slide you did not design is not yours to redraw.** Change its elements —
  text, colour, position, timing, images, video, buttons. Never clear the page
  and recompose from a plan that never described it.
- **A colour change is an in-place recolour, never a rebuild.** "Make the
  orange green" means: visit each slide and replace that colour wherever it
  occurs — solid fills, gradient stops, strokes, text runs (whole block when
  uniform, per run when mixed), the page background, and icons of that tone —
  then commit the slide and move on. It does not mean re-planning or
  recomposing the deck: a rebuild changes layouts the user didn't ask about,
  and if the old palette is still registered it paints the old colour straight
  back. Match with a small tolerance (≈0.035 per channel), leave locked blocks
  alone, and report what you replaced slide by slide. An animated icon
  follows both of its colours — its tone (ink) and its accent — keeping its
  drawing; a designed (wired-flat) animation is left alone.
- **An icon can become another icon, another drawing, or another colour —
  in place (2026-09-08).** "Change the arrow to a list icon", "make the
  solid icon an outline one", "change the blue in that icon to red" are all
  edits of ONE block, never a rebuild. Read the block's marks first: every
  icon the builder placed carries `idecide/icon` (the concept) and
  `idecide/iconTone`; an animated one also `idecide/lottie` (the library
  id), `idecide/iconStyle` (`outline` · `flat` · `system-outline` ·
  `system-solid`), `idecide/iconAccent` (`brand`, a hex, or `none` for a
  designed drawing), `idecide/iconSwaps` and `idecide/iconColors` (the
  hexes it shows) — `engine.block.getMetadata(id, key)`. Then:
  - *another icon* → the new concept's drawing in the same family (a
    `btn/…` icon stays a 1-tone system drawing; a standalone icon stays
    wired), same tone and style;
  - *another drawing* → the same library item's other file: "outline" on a
    system icon is `system-outline`, on a wired icon the 2-tone wired
    outline; "solid" is `system-solid`; "flat" the designed wired drawing;
    a static glyph becomes the animation when the library has one;
  - *another colour* → repaint the file: the tone (black → the colour), the
    accent (red → the colour), or swap one specific colour for another
    wherever it appears (take it from `idecide/iconColors`; the nearest
    colour stands in for a named one). A designed flat drawing is never
    repainted by a tone or accent — only an explicit colour swap changes
    it, because the user asked; say so and offer the swap or the outline
    drawing, which takes any tone.
  Import the new file (`importLottieBatch`), place it with
  `insertUploadedImage` in the old block's box (grow the box 28 % when a
  static glyph becomes a Lordicon animation, shrink it back the other way —
  the drawing keeps a margin in its canvas), copy the old block's name,
  group marks, timing and stack position, write the new marks, destroy the
  old block, commit. Never say an animated icon cannot be changed, restyled
  or recoloured.
- **Say what you changed, with the layer names**, not "done".
- **Verify before replying.** Re-read the slide (or re-export a snapshot) and
  confirm the change landed. Never promise to check something after your reply
  — your reply is the last thing that happens.
- Structural changes (add / remove / re-order a slide) mean the deck's
  auto-advance chain and menu flags must be re-asserted afterwards.

## Tracking choices

The platform records viewer selections and shows them to the sender. It lives
in the button's blockAction record: `clickActionData` with
`{store: true, varName, value, valueType: 'text'}`.

`blockActions.set()` cannot carry it, so set the action first, then write the
payload into the live record via `api.provider.getCtx().getBlockActions()`,
call `markCustomDataDirty()`, dirty the slide with a documented mutation, and
commit with `changeSlide`.

Naming conventions the sender expects:
- question answers → `"<Question> - <Answer>"`
- menu topics → `"Topic Viewed - <topic>"`
- terminal CTAs → `"Final Outcome - <TITLE>"`, value = the title verbatim

Never track plain navigation. Never set any action on slide 1.

## Placement

Compose, then **measure, then fix**. Detection without correction is the
failure this system has hit repeatedly.

- Space a stack by real frame heights (`engine.block.getFrameHeight`), never by
  estimates — a wrap you didn't predict makes every later block land wrong.
- A button is spaced by its **plate**, not its label: the plate runs past the
  label at both ends.
- Anything that moves or disappears together is ONE unit. Move units by their
  union box, never part by part, or you shear a button off its label.
- After placing everything, walk the slide for overlapping units and push them
  apart — lower unit down if there is room below, else upper unit up.
- Keep the contact/sender block at the foot of the stack, below the buttons.

`references/composition.md` has the full system.

## References

Load these as needed — do not read them all up front.

| File | Read it when |
|---|---|
| `references/aiagent-surface.md` | You need a call's exact signature or aren't sure something exists |
| `references/composition.md` | Composing or re-aligning slides — zones, type roles, gaps, groups |
| `references/deck-outline.md` | Planning a deck — the per-slide field contract |
| `references/script-craft.md` | Writing narration and on-screen copy |
| `references/platform-facts.md` | Something behaves unexpectedly — the verified footgun list |
| `references/test-brands.md` | Someone asked to build with test data — nine prefilled brands |
| `references/troubleshooting.md` | The user reports an error message or asks "what does that mean?" — the plain-English KB, one entry per situation, coded `KEY-`/`TAB-`/`BUILD-`/`PANEL-`/`HELP-` |

## When something fails

- **A call that should exist doesn't** → re-read the editor's API reference at
  `window.aiagent.instructions`; the signature list changed. `Object.keys`
  under-reports it — walk the prototype chain.
- **A change didn't persist** → you didn't commit. A documented `api.*`
  mutation plus `changeSlide`.
- **A navigation or call just hangs** → the builder tab has to be open and in
  focus for browser tooling to reach it, and the editor itself does not boot
  or render in a background tab. If something has been pending for more than
  a moment, ask the user to click into that tab, then retry. Do not keep
  waiting, and do not conclude the platform is down.
- **The user asks what a message means** → answer from
  `references/troubleshooting.md` (match the wording, or the code the
  extension quoted), in plain English, *before* retrying anything. A question
  is never swallowed by a reconnect.
- **The editor shows a fatal dialog** → the page needs a reload; re-open with
  the `aiagent` param and resume from the last committed slide.
- **Say what actually happened.** If a step failed, report the failure. A
  reported success for work that did not happen is worse than the failure.
