---
name: idecide-presentation-builder
description: Build or edit interactive iDecide presentations directly in the myiDecide editor (my.idecide.com) — write the script, design and compose slides, source stock video, generate narration, wire menus and buttons, track viewer choices. Use when someone asks to build, create, design, revise, fix, restyle or add to an iDecide/myiDecide presentation, or names a myiDecide builder URL. Runs through browser automation against the editor's own agent API; no API key needed.
---

# iDecide Presentation Builder

Build and edit interactive presentations inside the myiDecide editor by driving
its own `window.aiagent` API from the browser.

An iDecide presentation is not a linear deck. Viewers **choose** what to watch:
menus branch into topics, questions branch on the answer, and every path ends at
a call to action. Design for that, not for a slideshow.

## Before anything else

1. **Get the builder URL.** It looks like
   `https://my.idecide.com/builder/create/<sessionId>`. If the person hasn't
   given you one, ask for it — you cannot work without it.
2. **Append `?aiagent=`** to the URL and open it. This exposes
   `window.aiagent.api`, `.engine` and `.instructions`. Without it, nothing
   below exists.
3. **Read `window.aiagent.instructions` first, every session.** It is the
   platform's own contract and it changes. `references/aiagent-surface.md` in
   this skill maps the whole surface with examples, but the live instructions
   win where they disagree.
4. **Establish which job you are on:**
   - *Build from scratch* → follow "Building a deck" below.
   - *Edit an existing deck* → follow "Editing a deck". Never redraw a slide
     you did not design; change its elements instead.

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

1. **Interview.** Ask about the business, the audience, the goal, the tone,
   what the viewer should be able to choose between, and where they should end
   up. Ask for a website you can read for brand colours and typefaces.
   Use `AskUserQuestion` where the options are genuinely closed.
2. **Script.** Write it as a branching structure: opener → menu → chapters →
   per-chapter close back to the menu → finish section → terminal CTAs.
   `references/script-craft.md` has the voice and pacing rules.
3. **Plan the slides.** One plan object per slide: name, kind, narration,
   copy, items (with their targets), imagery hint. Names are identities —
   pick them once. `references/deck-outline.md` gives the field contract.
4. **Create the shells.** `api.slides.createMultiple(count, atIndex, 'end')`
   creates a run in ONE request — 13× faster than looping `create()`. Group
   the slides you need into consecutive runs and issue one call per run.
5. **Resolve assets before composing.** Search and import stock video in bulk
   with `api.assets.importPexelVideoBatch` — it uploads many clips in one
   request and does not need the target slide to be current. Do the same for
   narration with `api.narration.generateBatch`. Both stream results through
   an `onCompleted` callback; consume them as they land.
6. **Compose each slide**, then wire it, then commit by navigating to the next.
   `references/composition.md` carries the layout system: zones, gaps,
   type roles, and the element/group contract.
7. **Wire the deck.** Buttons → `blockActions.set(bgLayer, type, target)`.
   Menus point at chapter openers; chapter closes point back at the menu;
   terminal CTAs use `finishPresentation` with an outcome payload.
8. **Track the choices that matter.** Question answers and menu topics should
   record what the viewer picked — see "Tracking choices".
9. **Verify.** Walk the deck: every button has an action, every auto-advance
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

## When something fails

- **A call that should exist doesn't** → re-read `window.aiagent.instructions`;
  the surface changed. `Object.keys` under-reports it — walk the prototype
  chain.
- **A change didn't persist** → you didn't commit. A documented `api.*`
  mutation plus `changeSlide`.
- **The editor shows a fatal dialog** → the page needs a reload; re-open with
  `?aiagent=` and resume from the last committed slide.
- **Say what actually happened.** If a step failed, report the failure. A
  reported success for work that did not happen is worse than the failure.
