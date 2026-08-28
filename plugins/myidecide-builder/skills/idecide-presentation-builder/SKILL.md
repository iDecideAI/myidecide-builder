---
name: idecide-presentation-builder
description: Build or edit interactive iDecide presentations directly in the myiDecide editor (my.idecide.com). Runs the full intake — asks whether you are building new or editing, creates the new presentation for you or takes the URL of the one to change, walks a questionnaire (accepting an existing script, brochure, PowerPoint or logo if you have one), then writes the script, designs and composes the slides, sources stock video, generates narration, wires menus and buttons, tracks viewer choices, and takes edit requests afterwards. Use when someone asks to build, create, design, revise, fix, restyle or add to an iDecide/myiDecide presentation, or names a myiDecide builder URL. Browser automation against the editor's own agent API; no API key needed.
---

# iDecide Presentation Builder

<!-- MAINTAINERS: this file is the SOURCE; the copy inside the published
     plugin is generated from it. Edit this one. -->

Build and edit interactive presentations inside the myiDecide editor by driving
its own `window.aiagent` API from the browser.

An iDecide presentation is not a linear deck. Viewers **choose** what to watch:
menus branch into topics, questions branch on the answer, and every path ends at
a call to action. Design for that, not for a slideshow.

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

**New build → you create it. Do not ask them for a URL.** Navigate to
`https://my.idecide.com/create/new`. The platform mints the deck server-side
and lands on `/builder/create/<id>?slide=<firstSlideId>` with the Presentation
Info overlay open. Dismiss the overlay, then re-navigate with `&aiagent=`.

**Do not block on the name.** This step runs before the questionnaire, so the
brand answer does not exist yet. Set the title only if you already have a name
from something the user has said; otherwise take the platform default
("Untitled Presentation") and mention once that they can rename it in
Presentation Info. The extension behaves identically — the two must not
diverge.

Read the `create/new` section of `references/platform-facts.md` before you run
this. Four things bite, and three of them look like something else:

- **Go there exactly once.** The deck is created *before* the overlay appears,
  so every visit burns an id and strands an orphan "Untitled Presentation".
  Navigate when the build is actually starting — never to check something.
- **Check for signed-out first.** `/create/new` without a session lands on
  `/login`, and `document.title` is identical either way. Test the path. If
  they are signed out, stop and ask them to log in — never type credentials
  for them. The platform returns them to the same screen afterwards, so the
  retry is just re-running this step.
- **The name save is asynchronous.** Poll the session record until
  `presentationName` matches. An immediate read still says "Untitled
  Presentation"; that is not a failure, and re-submitting makes it worse.
- **Do not test the overlay with `offsetParent`.** It is `position: fixed`, so
  `offsetParent` is `null` while the overlay is plainly on screen. Poll
  `display` / `visibility`.

**Editing → they give you the URL.** Ask for the address of the presentation
they want changed. It looks like
`https://my.idecide.com/builder/create/<sessionId>`, usually carrying a
`?slide=<id>` from their address bar.

**Either path, open it with `aiagent=` in the query string.** That exposes
`window.aiagent.api`, `.engine` and `.instructions`, without which nothing in
this skill exists. Append `?aiagent=` to a bare URL, **`&aiagent=` to one that
already has a query string** — and after `create/new` the URL always carries
`?slide=`, so the new-build path is always `&aiagent=`. Never append
`?aiagent=` to a URL that already contains a `?`: the result is malformed, the
page loads without the API, and the failure looks like the platform being
broken rather than a bad URL.

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

   Slide 1 stays silent, carries no block actions, and never auto-advances.

5. **Create the shells.** `api.slides.createMultiple(count, atIndex, 'end')`
   creates a run in ONE request — 13× faster than looping `create()`. Group
   the slides you need into consecutive runs and issue one call per run.
6. **Resolve assets before composing.** Search and import stock video in bulk
   with `api.assets.importPexelVideoBatch` — it uploads many clips in one
   request and does not need the target slide to be current. Do the same for
   narration with `api.narration.generateBatch`. Both stream results through
   an `onCompleted` callback; consume them as they land.
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

## When something fails

- **A call that should exist doesn't** → re-read the editor's API reference at
  `window.aiagent.instructions`; the signature list changed. `Object.keys`
  under-reports it — walk the prototype chain.
- **A change didn't persist** → you didn't commit. A documented `api.*`
  mutation plus `changeSlide`.
- **A navigation or call just hangs** → the builder tab has to be open and in
  focus for browser tooling to reach it. If something has been pending for
  more than a moment, ask the user to click into that tab, then retry. Do not
  keep waiting, and do not conclude the platform is down.
- **The editor shows a fatal dialog** → the page needs a reload; re-open with
  the `aiagent` param and resume from the last committed slide.
- **Say what actually happened.** If a step failed, report the failure. A
  reported success for work that did not happen is worse than the failure.
