---
name: idecide-presentation-builder
description: Build or edit interactive myiDecide presentations directly in the myiDecide Builder (my.idecide.com). Runs the full intake — asks whether you are building new or editing, creates the new presentation for you or takes the URL of the one to change, walks a questionnaire (accepting an existing script, brochure, PowerPoint or logo if you have one), researches the brand online (site, palette, the real logo), then writes the script, designs every slide for its own content — no templates — sources stock video, generates narration, wires menus and buttons, tracks viewer choices, and takes edit requests afterwards. Use when someone asks to build, create, design, revise, fix, restyle, recolour or add to a myiDecide presentation, or names a myiDecide Builder URL. Browser automation against the Builder's own agent API; no API key needed.
---

# myiDecide Presentation Builder

<!-- MAINTAINERS: this file is the SOURCE; the copy inside the published
     plugin is generated from it. Edit this one. -->

Build and edit interactive presentations inside the myiDecide Builder by
driving its own `window.aiagent` API from the browser.

A myiDecide presentation is not a linear deck. Viewers **choose** what to watch:
menus branch into topics, questions branch on the answer, and every path ends at
a call to action. Design for that, not for a slideshow.

**Names, binding.** What you build is a **myiDecide presentation**. The app you
build it in — the platform's authoring surface at my.idecide.com — is the
**myiDecide Builder**; call it "the Builder", never "the editor". The Chrome
extension that does the same job with its own model calls is the **myiDecide
AI Presentation Extension** ("the Extension"). iDecide is the company;
"iDecide presentations" are a different, custom-built product — never call the
deck that.

**There are no templates.** Since 2026-09-15 every slide is designed for its
own content: you decide the stage, the column, the elements and where each one
sits on the 1558×720 canvas, and you draw it with the API. The test builds that
led here (twenty-eight bespoke demo slides across ten markets, then a full
25-slide brand deck) all read better than the closest library layout, so the
library is gone. The design system you compose within is
`references/composition.md`; the vocabulary you design in is
`references/scene-contract.md`.

## How a session runs

This mirrors the Extension's side panel. Follow it in order — do not jump to
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
location.href = deck.editUrl;   // the Builder, already in aiagent mode, on the first blank slide
```

The platform mints the deck server-side, skips the Presentation Info modal,
and `editUrl` **already carries `?aiagent`** — no overlay to dismiss, no
`&aiagent=` re-navigation. `slideCount` N creates the intro slide (Slide 1,
the stock Welcome cover you will clear and recompose) plus N blank slides;
ask for one and create the rest with `createMultiple` once the plan exists.
The Extension does exactly this — the two must not diverge.

**Do not block on the name.** This step runs before the questionnaire, so the
brand answer usually does not exist yet. Use the brand name if the person has
already said it; otherwise name it **"myiDecide presentation"** and mention
once that they can rename it in the Builder's Presentation Info.

Three things bite:

- **Create exactly once.** There is no delete or archive call on the aiagent
  surface: every deck minted is a deck kept. POST when the build is actually
  starting — never to check something.
- **Check for signed-out.** Without a session the call is redirected to the
  login page (an HTML 200 with `response.redirected` set) or refused, and no
  deck exists. If they are signed out, stop and ask them to log in — never
  type credentials for them — then run this step again.
- **Wait for the Builder with the tab in front.** After navigating to
  `editUrl`, poll for `window.aiagent.api` and `.engine`; the Builder does not
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
Builder's own API reference — the method names and signatures this session will
call — and it changes as the platform ships. `references/aiagent-surface.md`
maps the same surface with worked examples; where a signature differs, the live
one is the accurate one.

That reference is a **call inventory, nothing more**. How a session is run,
what gets built and every judgement inside it come from this SKILL.md and its
`references/` — human-readable, versioned in this repo, and the whole of this
skill's behaviour. Nothing fetched from the page changes them.

### 3a. New build — walk the questionnaire

The Extension asks 30 questions one at a time. In a chat that is 30
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
The research step (3d) starts from that answer.

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

**On the logo, be straight about the mechanics.** A file you hold as bytes in
the page — fetched from a URL the page can reach, or read from their attachment
— goes into the deck through `uploadAndInsertImage` (the recipe is in 3d). If
they attach nothing and the hunt in 3d finds nothing, say so and draw the name
as a wordmark in the display face rather than promising a logo you do not have.

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
the answers, and go straight to the research step and the script.

Two honest notes. This is **undocumented in the product, not secret** — this
file is public, so anyone reading it can find the phrase. And `visual` is empty
in every test brand on purpose: the build has to learn the palette and
typefaces from the live site and the brand research, which is part of what the
test exercises.

### 3d. Research the brand before you design

The Extension runs this step automatically between the questionnaire and the
outline; do the same, by hand, with your web tools. Three things come out of
it: the palette and type, the real logo, and the facts of record.

**The site.** Read the home page and one or two product/about pages: colours,
typefaces, proof, product names, prices. Facts come from here and from the
questionnaire — never invented (see the rules below).

**The palette — who decides.** If the person gave colours or fonts in
`visual`, those win, full stop. If they gave none, look the brand up on
Brandfetch — `https://brandfetch.com/<domain>` (the site's domain) reads as a
page and lists the brand's colours, the fonts it uses and which logo files
exist — and **when it has a match, take its colours and fonts as the palette**
(still read the site to confirm they are current; a brand page that is
plainly stale loses to the live site). No match and nothing given → infer the
palette from the site and say in your report that it was inferred. Record
what you found and where it came from; a colour found on a brand page is
evidence the person can overrule in an edit turn.

**The logo — find the real mark.** When no logo file was attached, hunt for a
vector or a large PNG in this order, and stop at the first hit:

1. the client's own site — a `/brand`, `/press`, `/media-kit` or
   `/about/brand-guidelines` page, or the header mark (an inline SVG is often
   not returned by a summarising fetch; look for a linked `.svg`);
2. Brandfetch — the brand page above says which formats exist;
   `https://cdn.brandfetch.io/<domain>` (and `/w/512/h/512`) answers the brand
   icon with no key;
3. Wikimedia Commons — search "<brand> logo svg wikimedia commons"; the FILE
   page is cache-only for a fetch, so compute the original's URL instead:
   `https://upload.wikimedia.org/wikipedia/commons/<h0>/<h0h1>/<File_Name>`
   where `h` is the md5 of the file name with spaces as underscores (parentheses
   raw, then percent-encode the URL);
4. worldvectorlogo — `https://cdn.worldvectorlogo.com/logos/<slug>.svg`
   (try `<slug>-1`, `<slug>-logo`, `<slug>-2`).

Prefer SVG; accept a PNG at least 1000 px wide; reject anything with a baked-in
plate unless it is the brand's own lock-up. Fetch the bytes **from the Builder
page** (a `fetch` in the page, then a `File`) — upload.wikimedia.org,
cdn.worldvectorlogo.com and cdn.brandfetch.io answer the page with CORS;
the brand's own site usually does not, and neither do logo.clearbit.com or
Google's favicon service. Small clients will often miss; check anyway, because
a hit replaces a typed wordmark with the real mark on every slide.

**The upload / reuse recipe (verified).** Recolour the SVG *text* before
upload — the mark's fill to `#ffffff` for dark fields and to the brand ink for
light fields — two uploads, no image editing. Then, once per drawing:

```js
const f = new File([svgText], 'brand-onDark.svg', { type: 'image/svg+xml' });
const id = await api.currentSlide.images.uploadAndInsertImage(f, 100, 100, 400, 80, 0, null); // all seven args; 0 = failed
const fill = engine.block.getFill(id);
const uri = engine.block.getString(fill, 'fill/image/imageFileURI')            // an r2 /images/<uuid>.svg url (SVG)
         || engine.block.getSourceSet(fill, 'fill/image/sourceSet')[0]?.uri;  // raster uploads carry a sourceSet instead
engine.block.destroy(id);
const rec = { id: uri, label: 'brand-onDark', meta: { uri, width: W, height: H, sourceSet: [] } };
// place it anywhere, any slide, after changeSlide:
await api.currentSlide.images.insertUploadedImage(rec, x, y, w, h, 0, null);
```

A response-shaped object carrying the r2 uri places everywhere (26 placements
on the brand test deck, all Ready after a reload); a bare URL string returns 0.
Contain fill mode; size from the SVG's aspect; pick the light or dark drawing
by the slide's field. `references/platform-facts.md` has the full entry.

**Where the logo goes** is the design's call — small at a consistent corner
on content slides, top-centre on the cover, large and centred on the Logo
Reveal — and the **embargo** always holds: on every slide before the Logo
Reveal (when the person chose to build intrigue first) there is no logo and no
brand name, on screen or in narration.

**Facts of record.** Only a household-name brand with an unambiguous public
record may carry well-established facts — founders, launch year, headquarters,
flagship products — where the story invites them. Everything else comes from
the client. Never user counts, revenue or results for anyone.

### 4. New build — write, then build

With the answers and the research in, go to **Building a deck** below. Write
the script first, plan the slides, then compose. Do not start creating slides
while questions are still open.

### 4b. Editing — ask what they want changed

For an existing deck, skip the questionnaire entirely. Ask what they want
different, read the deck before touching it, and follow **Editing a deck**
below.

### 5. When the build finishes — stay open for edits

Say what you built: how many slides, the menu structure, where the CTAs point,
what the research found (palette source, logo source). Then invite changes —
the Extension drops into a revision chat at exactly this point, and so should
you. Edit requests after a build follow **Editing a deck**: read the slide,
change its elements, verify, and report what moved.

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
- **Facts come from the client.** Names, dates, places, products, prices,
  numbers and claims come from the questionnaire and the client's own site.
  Only a household-name brand with an unambiguous public record (Duolingo,
  Patagonia, IKEA) may carry well-established facts of record — founders,
  launch year, headquarters, flagship products — where the story invites
  them; never user counts, revenue or results for anyone, never a fact you
  are unsure of. In doubt, stay impressionistic ("two computer scientists").
  A data graphic draws only figures the client actually gave.
- **Every non-menu slide gets designed copy; its wiring never changes.**
  A question, a CTA or a sub-fork slide with only its buttons still needs a
  headline from the script beat — never its internal name ("Finish Up - 3")
  as a title. Designing it may dress its buttons (icon, body) but never
  change a label, target, url or finish the outline gave.
- **A named person is never a stock face.** An item or slide naming a real
  person (founder, owner, staff, a quoted customer) gets no portrait or
  headshot from stock — illustrate the role, the place or the product, or
  use an icon only. Real faces come only from the client's own files.
- **Production prefixes are script notation, never voice.** `(LABEL:
  slide16_p1_first)`, `IF "OPTION"` / `IF CORRECT` / `IF INCORRECT`, `STORE
  ANSWER IN NOTIFICATIONS` sit in the Script cell ahead of the words; strip
  them before anything is narrated or written to a slide's notes, and take
  EVERY line of a multi-line cell (a menu once narrated "LABEL slide sixteen
  p one first" for four seconds and lost its real sentence).
- **Every script row carries its Slide Name.** Answer rows most of all — a
  blank name means the slide never exists, every button that pointed at it
  dies, and the viewer hits a dead end. Names are identities; fill them once,
  from the outline order, and keep them.
- **Keep the Builder tab visible and focused for every long pass.** The
  Builder (img.ly under the hood) does not boot, render or change slides in a
  background tab: a navigated pass with the tab behind something else stalls
  at `changeSlide`, times out exports and skips slides. Bring the tab to the
  front before you start, and tell the user not to switch tabs until it
  finishes.
- **Every auto-advance names its target explicitly, on every slide.** Never
  rely on deck order. A chapter's last beat advances to the menu, not to the
  next chapter. And every slide `createMultiple` makes arrives with
  `autoAdvanceAfterNarration: true` pointing at "next-slide" — on a menu,
  question or CTA that is a 10-second auto-advance off a slide meant to wait,
  so the shell step sets `setAutoAdvance(id, false, next)` on every waiting
  slide and `setAutoAdvance(id, true, targetId)` on every content beat.
- **The save model:** a documented `api.*` mutation marks a slide dirty;
  `api.slides.changeSlide` commits it. A raw `engine.*` write marks nothing —
  pair it with a documented `api.*` call before the commit or it is lost.
  `location.reload()` discards anything uncommitted. A whole hand-composed
  slide (engine-created shapes, api media, api text) commits with one
  documented call + `changeSlide` — verified across a 21-slide build and a
  reload.
- **Buttons are multi-layer.** The click action goes on the **background**
  layer (it covers the whole clickable area); the text and icon layers must
  carry no action. **A text-style button — a label with no background — gets
  a transparent plate**: an alpha-0 rect (`{r:0,g:0,b:0,a:0}`) the size of the
  whole row, inserted as the BACKMOST layer of the button's group, timed with
  the label, carrying the action. A bare label misses clicks.
- **Every button carries exactly ONE animated icon.** The label's concept
  drawing when it has one; otherwise an arrow — right for a way forward, left
  for Back, a check on finish / yes / agree labels. Never an arrow beside a
  concept icon, never a typographic glyph (→ ← ✓ ›) standing in for one.
- **Narration audio belongs at z0** — `engine.block.insertChild(page, audio, 0)`.
- **No two elements overlap.** Measure after composing; if two units collide,
  push them apart. See "Placement" below.
- **The column you chose decides the axis.** A slide composed left-aligned /
  centred / right-aligned keeps that axis for EVERYTHING in its content zone —
  text, button groups, Back pills, stat units, graphics, photo-card strips,
  the sender block. Left stays left, right stays right; only the user's edit
  request changes it. A left column with a centred headline is a defect, not
  a choice.
- **A deck alternates its compositions.** Never the same composition family
  twice in a row (a mirrored split is still a split); no family more than four
  times in a deck; alternate the axis, the photo side, the field and the
  density from one slide to the next, so the deck is a mix of left, centred
  and right rather than a run of one. Parallel beats stay identical: one
  section-intro composition, one menu composition (First and Return share
  footage and layout), one CTA family per deck.
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
- **Where the beat carries a number, draw it.** A share, a growth, a
  comparison, a count: an animated bar chart, a dot grid ("73 of 100"), a
  ring, a big stat — one data graphic per slide, from the client's figures
  only, never on a menu, question or terminal CTA. The headline still says
  what the number means; the graphic is the evidence. The test builds' bar
  graph, dot grid and rings made their points faster than any sentence.
- **Graphics are units in the flow.** Icons, rings, charts, images, lotties and
  videos are grouped with the text they belong to and take the composition's
  spacing; a ring stat is ONE unit — figure measured first, ring sized around
  it and placed behind it, label under the ring. Nothing is dropped at a fixed
  spot after the layout is done.
- **One shortcode per slide.** `[viewer-name-first]`, `[sender-email]` and
  friends appear at most once on a slide; the contact block owns the sender
  tokens. The cover's greeting line ("Hi [viewer-name-first]," or the like)
  is its own text block and the only place the name token appears — never
  repeat it in the headline. Tokens never appear in narration.
- **Say it once.** One FIGURE per slide: on a stat composition the numeral or
  the graphic carries the number and the headline/eyebrow say what it means
  without the digits ("ABOUT 5 MINUTES / 5 minutes survives a bad day. / 5
  min" is one fact three times and reads as a mistake). Items never restate
  the headline word for word. Chapter openers share one eyebrow style with no
  ordinals — the viewer chooses the order, so "CHAPTER SIX" is wrong for
  whoever taps it first. Two answers to one question may share a headline (a
  viewer sees only one); any other two slides may not.
- **The 2-tone wired outline is the default icon drawing, everywhere.**
  Every animated icon — standalone, above a line of text, in a button or
  pill — is placed as the library's wired outline, painted in the brand
  colours (ink on light fields, white on dark). "These match the style of
  presentations a little better and feel more intentionally branded." The
  designed wired flat and the 1-tone system outline/solid drawings stay
  available: the user can ask for any of them by name (an edit of one
  icon, or a row), and they are the fallbacks when an icon has no outline —
  a button: outline → system outline (light) or solid (dark) → the other
  system style → flat; a display icon: outline → flat → the system pair.
- **Three concepts per icon.** Every icon slot names its first choice plus
  two alternates — different THINGS that could draw the same line, not
  respellings ("Designed in-house": a pencil-ruler, a house, glasses;
  "Thirty days": a calendar, a delivery truck, a watch). Take the first one
  the animated library actually has, and when two items on a slide would
  land on the SAME drawing, the later one takes an alternate — a list of
  rhyming labels ("Five days" / "Thirty days") must not draw one calendar
  twice. None in the library → the first choice stays and that slot uses the
  static glyph.
- **One icon style per group.** Buttons on a slide share one icon style
  (the 2-tone wired outline by default, or system solid / system outline);
  a row of icons above lines of text shares one (wired outline by default,
  or wired flat). When one member has no drawing in the default style but
  every member has the next style on the ladder above, the whole group
  takes that style; when no style is shared, the default stands and the odd
  icon falls back alone — down the ladder, then the static SVG as the last
  resort. An icon the library does not know stays SVG and never pulls the
  group off the default.
- **A detail line hangs off its title.** Pair a subordinate line with the
  text directly above it in its own column — never with a tall numeral or
  icon beside that text; tight inside the pair (16 px), wider between pairs.
- **A `STOCK:` line is direction, never copy.** Shot lists in the On Screen
  cell — `STOCK:`, `4 PANELS:`, `3 tiles:`, `clips:` — describe footage; a
  grid whose copy is its panel labels needs no headline.
- **The ☰ opens the Hamburger Menu.** `api.slides.setMenuSlide(id, true)` is
  UNIQUE — setting it on a slide unsets every other — so set it on the
  Hamburger Menu slide only, never on menus in general, and re-check
  `slides.get()` (`isMenuSlide`) after every build, restore and structural
  edit.
- **Measure only what has stopped moving.** The engine lays text out with the
  font it has at that instant; a display face still downloading measures as
  the fallback and lies about the wrap. Before spacing anything, sample every
  text frame twice ~120ms apart and wait until they agree (give up after
  1.5s). Two text blocks in one column are always two rows. Condensed display
  faces (Oswald, Barlow Condensed) render taller than the box maths says —
  give them 15–20% less size and check visually.
- **After any interrupted or timed-out page script, audit before you do
  anything else.** A script that has started keeps running in the page even
  when the tool call is rejected or times out; never assume it did not run.

## Designing a slide — the scene

You are the designer. For every slide, before you draw anything, decide the
**scene** in the vocabulary of `references/scene-contract.md` — the same
contract the Extension's designer model writes and the Extension draws:

- **the stage** — the field (light or dark), the background (a tinted and
  scrimmed stock clip, a still, a solid, a gradient), the tint that keeps
  footage alive under white type (0.4–0.65; 0.7+ turns it into texture);
- **the column** — where the copy sits (a 45–60% column on a split, 60–70%
  centred), stacked eyebrow → headline → support → list → buttons, spaced by
  measured heights;
- **the elements, back to front** — text by role (hero · headline · longline ·
  subhead · body · eyebrow · fine · numeral · button), rects and panels, media
  panels with their own clip, icons, the logo, buttons (pill · rect · card ·
  text · circle), lists (rows · cards · chips · steps · timeline · numbers), a
  data graphic (bars · hbars · dots · ring · stat · progress, or a chart
  drawn as one image), the sender block on a terminal CTA;
- **a family name** for the composition — variety is judged on it.

Then draw it with the API: page colour and footage for the stage, one text
block per string, rects and ellipses through `engine.block.create` +
`appendChild`, media through the api, icons and the logo through
`insertUploadedImage`, every part marked into its group. `references/composition.md`
is the binding design system (type roles sized by content length, colour
roles, footage rules, the interactive units, the rhythm rules across a long
deck, the polish checklist) and the element contract the drawing must obey.

The compositions that proved themselves on the test builds, and that a deck
should show a dozen of: full-bleed footage with a left scrim and a hero; a
split with a 42–46% media column; a centred statement on a darkened clip; a
stat with a dot grid or an animated bar chart beside a short headline; three
or four photo cards under a headline; numbered steps across the lower half; a
quote centred with a fine attribution; a menu of pills over a tinted clip; a
hamburger of text-style rows on a quiet field; a bento of tiles for a menu
with one featured topic.

**Animation follows position (the edge rule).** Heroes rise on their
baseline; buttons grow; panels and rows slide in from the nearer edge — an
element on the right half enters from the right, on the left half from the
left; bars grow upward; everything else fades; slots arrive 0.18s apart
bottom-to-top, parts of one object 0.08s apart; the background never
animates. Every layer runs to the slide's end (narration + 0.5s tail).

**Drawing a data graphic.** Bars: one rect per value on a shared baseline,
equal widths, heights in proportion to the largest, the value in a fine line
above each and its label below, entering with a grow-up animation staggered
left to right; a highlighted bar takes the accent. Dot grid: a 10×10 field
of small discs, `filled` of `total` in the accent and the rest in the muted
ink at low alpha, a caption under it. Ring: a stroked ellipse sized around
the measured numeral, the figure inside, the label under. A pie, donut, line
or gauge is an SVG you write and upload as an image the same way as the logo.
One graphic per slide; the figures are the client's.

**Rub Your Screen on the Logo Reveal** (when the person chose to build
intrigue first): a plain full-canvas rect named "Logo Reveal" on TOP, timed
exactly to the rub-prompt narration (0→D1, the one element with no tail),
carrying the four metadata keys and a blockAction record whose
`interactionData` is the same config; every reveal layer starts at D1; the
reveal narration plays at D1; the page runs D1 + D2 + 0.5s; `coverImageUrl` is
an upload-library r2 url (import a dark still with `importPexelImageBatch` and
read its uri). The Builder renders the rub natively after a reload — the hand
and "Rub Your Screen" over the dark cover. The exact anatomy is in
`references/platform-facts.md` ("Rub Your Screen").

## Building a deck

Work in this order. Do not skip ahead — later phases depend on assets that
earlier ones resolve.

1. **Answers and research in hand.** The questionnaire above is complete (or
   a test brand was chosen) and 3d has run. If a script or brochure came in,
   it is the spine — build from their structure and wording, not a fresh
   invention.
2. **Script.** Write it as a branching structure: opener → menu → chapters →
   per-chapter close back to the menu → finish section → terminal CTAs.
   `references/script-craft.md` has the voice and pacing rules.
3. **Plan the slides.** One plan object per slide: name, kind, narration,
   copy, items (with their targets), the layout intent, the imagery hint.
   Names are identities — pick them once. `references/deck-outline.md` gives
   the field contract, including the layout intents that keep neighbouring
   slides different.
4. **Clear slide 1 and compose your own cover.** A fresh deck arrives with one
   slide holding the stock "Play Button" welcome layout. That is stock, not a
   starting point — **never repurpose its text.** There is no slide delete on
   the aiagent surface, so clearing and recomposing *is* the delete.

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
   creates a run in ONE request — 13× faster than looping `create()` — and
   inserts at an exact 0-based index. Group the slides you need into
   consecutive runs and issue one call per run. Then set every slide's
   auto-advance explicitly (the rule above): `false` on every slide that
   waits for a click, `true` with its named target on every content beat.
6. **Resolve assets before composing.** Search and import stock video in bulk
   with `api.assets.importPexelVideoBatch` — it uploads many clips in one
   request and does not need the target slide to be current. Import stock
   stills the same way, beside it: `api.assets.searchImages` per query, take
   the best landscape result, dedupe by url across the deck, then ONE
   `api.assets.importPexelImageBatch(urls)` (at most 50 per request; a failed
   entry is `null` at its index, no blocks are made) and keep the returned
   `UploadedFileResponse` per query. Every media panel in a scene gets its
   own clip, so resolve one hint per background AND one per panel. Upload the
   logo drawings (3d) once here. Do the same for narration with
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
   accent — THE DEFAULT for every icon, buttons included), most a designed
   `flat` variant (its own colours — never repaint it), and a few hundred a
   1-tone `system-outline` / `system-solid` variant; each item's `variants`
   map names the files. The other styles are for the user to ask for, or
   the fallback when an icon has no outline).
   Do not promise animated icons you do not have files for. Where animations exist, default to them wherever one
   is a good match for an icon's meaning and let the static glyph fill the
   gaps — never force a near-miss (a briefcase is not a shopping bag).
   Placed animations follow the icon rules exactly: same slot, size,
   group role and z-order as a static icon, brand colours (black → ink or
   white by field, other hues → accent, white transparent), never a
   background. **A Lottie whose loop is a time remap of one precomp plays
   ONCE in the Builder and freezes** — the Builder's player ignores the remap
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
      7. **Design and draw each slide** (the section above), then wire it, then
   commit by navigating to the next. Never repeat the previous slide's
   family; keep the parallel beats identical.
8. **Wire the deck.** Buttons → `blockActions.set(bgLayer, type, target)` on
   the plate (the transparent one on a text-style button). Menus point at
   chapter openers; chapter closes point back at the menu; terminal CTAs use
   `finishPresentation` with an outcome payload.
9. **Track the choices that matter — by default.** Question answers, menu
   topics and final outcomes are recorded for the sender without being asked
   — see "Tracking choices".
10. **Verify after a full reload.** Walk the deck: every slide's block count,
   one audio block per narrated slide at z0, every button has an action (and
   its tracking payload), every auto-advance has its target, the menu flag
   sits on the Hamburger Menu only, no slide is bare, nothing overlaps, no
   type under 28px, one icon per button. A `changeSlide` fired against a dead
   session never commits — the reload count check is the only thing that
   catches it.

## Editing a deck

The same surface, one slide at a time. Read before you write:
`api.currentSlide.blocks.getVisual()` tells you what is actually on the slide.

- **A slide you did not design is not yours to redraw.** Change its elements —
  text, colour, position, timing, images, video, buttons. Never clear the page
  and recompose from a plan that never described it. A slide you DID design
  in this session may be redesigned as a whole — a new scene — when the person
  asks for a different layout; a request for a different word, colour or
  picture is an in-place edit of that element and nothing else.
- **Any slide can carry video, whatever its layout.** Never tell the person a
  slide cannot have footage because of how it is composed — a solid field
  becomes a tinted clip, a panel gets its own clip, in place.
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
  `system-solid`), `idecide/iconStroke` (the weight, when not regular),
  `idecide/iconAccent` (`brand`, a hex, or `none` for a
  designed drawing), `idecide/iconSwaps` and `idecide/iconColors` (the
  hexes it shows) — `engine.block.getMetadata(id, key)`. Then:
  - *another icon* → the new concept's drawing in the same style the old
    one wore (the 2-tone outline unless it had been switched to flat or a
    system drawing), same tone;
  - *another weight* → light / regular / bold, the three weights Lordicon
    draws the wired icons at. Ours are built at REGULAR; "make that icon
    bolder" / "thinner lines" scales every stroke width in the file (×1.5 /
    ×0.5 — what Lordicon's own stroke expression computes for menus 3 and
    1). It is a wired-drawing setting: the 1-tone system drawings are filled
    shapes with no strokes, so say that rather than pretending;
  - *another drawing* → the same library item's other file — this is how
    the user gets one of the other styles instead of the default: "outline"
    on a system icon is `system-outline`, on a wired icon the 2-tone wired
    outline; "solid" is `system-solid`; "flat" / "designed" the wired flat
    drawing; "system" / "1-tone" the system drawing the field picks
    (outline on light, solid on dark); "wired" / "2-tone" the default;
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
- **The logo can be swapped deck-wide.** A new file (theirs, or a better
  hit from the hunt in 3d) is uploaded once and every placement re-pointed
  through `insertUploadedImage` in the old block's box, at the old z-index,
  with the old timing — light and dark drawings by field.
- **Say what you changed, with the layer names**, not "done".
- **Verify before replying.** Re-read the slide (or re-export a snapshot) and
  confirm the change landed. Never promise to check something after your reply
  — your reply is the last thing that happens.
- Structural changes (add / remove / re-order a slide) mean the deck's
  auto-advance chain and menu flags must be re-asserted afterwards.

## Tracking choices

The platform records viewer selections and shows them to the sender. It lives
in the button's blockAction record: `clickActionData` with
`{store: true, varName, value, valueType: 'text'}`. **It is on by default**:
every question answer, every menu topic and every final outcome is recorded
unless the person says otherwise.

`blockActions.set()` cannot carry it, so set the action first, then write the
payload into the live record via `api.provider.getCtx().getBlockActions()`
(a record is matched to its block by
`engine.block.getMetadata(id, 'idecide/block-action-id')` — the record's
`blockId` IS that GUID; mutate `rec.clickActionData` in place), call
`markCustomDataDirty()`, dirty the slide with a documented mutation, and
commit with `changeSlide`.

Naming conventions the sender expects:
- question answers → `"<Question short name> - <Answer>"` (2–4 words the
  sender will recognise, e.g. "Gets You Outside - Hiking")
- menu topics → `"Topic Viewed - <topic>"` (not for Finish Up / Move Ahead /
  Back)
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
- `engine.scene.setZoomLevel(0.95)` shows the whole slide above the timeline
  for a screenshot; `engine.block.setPlaybackTime(page, s)` scrubs the
  preview to the frame at `s`. Check margins by numbers, not by eye.

`references/composition.md` has the full system.

## References

Load these as needed — do not read them all up front.

| File | Read it when |
|---|---|
| `references/aiagent-surface.md` | You need a call's exact signature or aren't sure something exists |
| `references/scene-contract.md` | Designing a slide — the stage / stacks / elements / animation vocabulary, and what each slide kind must contain |
| `references/composition.md` | Composing or re-aligning slides — the binding element contract and the design playbook (type roles, colour roles, footage, rhythm, polish) |
| `references/deck-outline.md` | Planning a deck — the per-slide field contract and the layout intents |
| `references/script-craft.md` | Writing narration and on-screen copy |
| `references/platform-facts.md` | Something behaves unexpectedly — the verified footgun list; also the Rub Your Screen anatomy and the upload recipes |
| `references/test-brands.md` | Someone asked to build with test data — nine prefilled brands |
| `references/troubleshooting.md` | The user reports an error message or asks "what does that mean?" — the plain-English KB, one entry per situation, coded `KEY-`/`TAB-`/`BUILD-`/`PANEL-`/`HELP-` |

## When something fails

- **A call that should exist doesn't** → re-read the Builder's API reference at
  `window.aiagent.instructions`; the signature list changed. `Object.keys`
  under-reports it — walk the prototype chain.
- **A change didn't persist** → you didn't commit. A documented `api.*`
  mutation plus `changeSlide`.
- **A navigation or call just hangs** → the Builder tab has to be open and in
  focus for browser tooling to reach it, and the Builder itself does not boot
  or render in a background tab. If something has been pending for more than
  a moment, ask the user to click into that tab, then retry. Do not keep
  waiting, and do not conclude the platform is down.
- **The user asks what a message means** → answer from
  `references/troubleshooting.md` (match the wording, or the code the
  Extension quoted), in plain English, *before* retrying anything. A question
  is never swallowed by a reconnect.
- **The Builder shows a fatal dialog** → the page needs a reload; re-open with
  the `aiagent` param and resume from the last committed slide.
- **Say what actually happened.** If a step failed, report the failure. A
  reported success for work that did not happen is worse than the failure.
