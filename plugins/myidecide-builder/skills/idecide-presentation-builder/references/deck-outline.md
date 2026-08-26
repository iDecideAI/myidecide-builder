> **Reference for the iDecide Presentation Builder skill.** The per-slide plan
> contract — the fields each slide object carries and what they mean. Written
> as instructions to a planner; apply it to your own planning.

<!--
  DECK OUTLINE CONTRACT (modular template flow, tier 1 of 2)
  ────────────────────────────────────────────────────
  A FAST deck-level pass: theme + one CATEGORY assignment per slide — no copy,
  no narration, no photo queries (those come per slide in tier 2). The
  CONCRETE VISUAL VARIATION within each category (25 per category, 375 total)
  is assigned downstream by a seeded randomizer — your job is the right
  category, honest photo intent, and clean structure.
-->

You are the deck designer doing the STRUCTURAL pass. The composition playbook
above is binding. Read the script CSV + answers and assign each slide its
CATEGORY from the catalog provided — chosen by the beat's content SHAPE
(counts, numbers, items, choice vs statement vs story) — honoring deck rhythm
(§8): alternate density, section intros consistent, and the imagery floor
below.

## THE IMAGERY FLOOR — at most FOUR bare slides in the whole deck

These play full-screen on a phone. A slide with no moving background and no
framed image or video reads as a slide that failed to load. So **at most four
slides in the entire deck** may have `photoZone: "none"`. Everything else gets
either a full-bleed background video or a framed image/video zone, plus a
unique `imageHint`.

Spend your four bare slides deliberately — on the beats carrying the most
on-screen text, where a background would compete. Every other slide, including
menus, questions and CTAs, gets imagery behind it.

Output ONE JSON object (no fences, no commentary):

{
  "theme": {
    // DERIVATION PRIORITY: (1) colors/fonts the client stated → use verbatim;
    // (2) otherwise MEASURE their real branding — fetch their website and read
    // hex codes + font families from the raw HTML/CSS, or search
    // "<brand> brand colors"; (3) only when research finds nothing, design a
    // tasteful industry-appropriate palette. Never default to a stock palette
    // when the client's true branding is discoverable.
    "paletteRoles": { "ink": "#hex", "surfaceLight": "#hex", "surfaceTint": "#hex",
      "primary": "#hex", "primaryDeep": "#hex", "accent": "#hex",
      "onDark": "#FFFFFF", "onDarkMuted": "#hex" },
    "display": {"family": "...", "variant": "..."},   // the HEADLINE face — a real
                                                      // per-brand choice, not a default.
                                                      // family MUST come from the
                                                      // AVAILABLE FONTS list; variant =
                                                      // the closest WEIGHT to the brand's
                                                      // (Light/Regular/Medium/SemiBold/
                                                      // Bold where the family has them)
    "sans":    {"family": "...", "variant": "..."},   // the body/UI face — same rule:
                                                      // catalog family + closest weight
    "voiceName": "...",          // EXACT name from the voice catalog provided
    "casting": "...",            // one line: who appears in photos, deck-wide
    "brandEvidence": "..."       // REQUIRED when a website was researched: what
                                 // you actually READ off the page, as selector +
                                 // declaration pairs, e.g.
                                 // 'h1.hero__title {font-family:Poppins;700} ·
                                 //  .btn--cta {background:#FF5A1F} ·
                                 //  body {color:#1B1F23} · fonts.googleapis:
                                 //  Poppins,Inter'. Name anything you INFERRED
                                 // rather than saw. "" if no site was given.
  },
  "slides": [
    {
      "name": "...",             // the CSV Slide Name, VERBATIM (identity + wiring)
      "kind": "cover"|"content"|"question"|"answer"|"menu"|"cta"|"hamburger",
      "autoAdvance": true|false, // false: cover, menus, questions, CTA, hamburger
      "advanceTo": "...",        // OPTIONAL. Where this slide advances to, by
                                 // NAME, when that is NOT the next slide in
                                 // deck order. Set it on the LAST beat of every
                                 // topic chapter: "Main Menu - Return".
      "sectionIntro": true,      // ONLY topic-chapter openers
      "category": 1-15,          // from the CATEGORY CATALOG, by content shape
      "field": "light"|"dark"|"primary",
      "photoZone": "full"|"left"|"right"|"top"|"bottom"|"none",   // photo INTENT
      "imageHint": "...",        // 4-8 word Pexels VIDEO phrase (motion bg), "" if none.
                                 // UNIQUE per slide — vary subjects/settings.
      "grid": {"cols":1-6,"rows":1-4}   // OPTIONAL. A beat that IS a set of N things
                                        // shown side by side — four services, three
                                        // steps, a 2×2 of moments. Say the number and
                                        // the builder draws that grid, whatever the
                                        // bound template's own zones declare. Every
                                        // panel of a multi-panel grid gets its OWN
                                        // footage by default (add "videoPanels": false
                                        // for stills). This is the only way to ask for
                                        // more than one video on a slide, so ask.
    }, ...
  ]
}

## THE MENU LOOP — how a viewer actually moves through this deck

```
Get Started → Welcome → hook + intro questions → Logo Reveal
   ↓
Main Menu - First     "explore in any order — what would you like to see first?"
   ↓ viewer picks a topic
<Topic> - 1 … <Topic> - n     last beat advanceTo → "Main Menu - Return"
   ↓
Main Menu - Return    "pick another topic, or Finish Up"    ← same options as First
   ↓ …until the viewer picks Finish Up
Finish Up - 1 … Finish Up - n    last beat = THE OUTCOME QUESTION (kind "question",
   ↓ viewer picks an outcome           autoAdvance false, one button per outcome)
<Outcome> - 1 … <Outcome> - n    its own chapter, ending on a TERMINAL CTA
   ↓
(nothing — the CTA is the last thing the viewer sees)
```

Binding consequences:
- **Every topic chapter's last beat** gets `autoAdvance: true` and
  `advanceTo: "Main Menu - Return"`. Without it the chapter spills into the
  next chapter and the viewer never chooses again. That last beat plays like
  a REGULAR content slide — no navigation copy, no buttons (the return menu
  owns the choose-next prompt).
- **Subfork chapters** (testimonials, product lines — content browsed
  piecemeal): the chapter's FIRST beat is its own menu — kind "menu",
  `autoAdvance: false`, one item per sub-beat plus a "Move Ahead" item
  targeting "Main Menu - Return". Each sub-beat carries its content plus a
  single "Back" item targeting the chapter's first beat, and
  `autoAdvance: false` (a viewer leaves a sub-beat only via Back).
- **"Finish Up" is always a topic**, offered in BOTH menus, even when the
  client gave no closing content — it is how a viewer chooses to leave.
- **Every outcome the Finish Up question offers gets its own chapter**, ending
  on a `kind: "cta"`, category-15 terminal. If the client named no call to
  action at all, that terminal is a Thank You screen — still category 15, still
  the last thing on screen.
- Nothing follows a terminal CTA. It never auto-advances.

SUBFORK CLARIFICATION (the case builds keep getting wrong): the LAST
sub-beat of a subfork chapter is a sub-beat like its siblings — it points back
to the chapter's own menu and NEVER to "Main Menu - Return". The chapter-close
rule does not apply inside a subfork; only the subfork menu's "Move Ahead"
leads out. The builder now enforces this mechanically, but plan it correctly
so the narration matches the flow.


Structural rules:
- cover = category 1. Menus/hamburger = category 2. Intro questions = category 4.
- **CALL TO ACTIONS (category 15) are the TERMINAL slides**: every closing fork
  points at one — the last thing a viewer sees. They are ACTION-NAMED slides
  ("Schedule A Call", "Get Started as Customer", "Visit Our Website",
  "Not Right Now"), kind:"cta", one per closing outcome, targeted BY NAME from
  the Finish Up fork. Nearly every deck has "Schedule A Call"; every deck has a
  polite decline terminal. Never end a path on a content slide.
- Section intros = category 3, ALL of them (consistency = orientation).
- Logo Reveal sits immediately before Main Menu - First, and it is an
  INTERACTIVE slide with two beats the builder assembles mechanically:
  * `rubText` — one short spoken line (≤ 10 words, NO [tokens]) inviting the
    viewer to rub the screen: "Rub your screen to meet Patagonia." While it
    plays, a full-screen Rub-Your-Screen interaction covers the slide and the
    timeline pauses until the viewer finishes rubbing.
  * `narration` — the reveal: the brand/product introduction that plays as
    the logo and headline appear underneath. Standard content copy applies.
  Give the slide BOTH fields, `autoAdvance: true`, and design its copy as the
  reveal moment (logo + brand line). Do not put buttons on it.
  Optional `rubCoverHint` — a query for the DARK teaser image under the rub
  layer (default: a dark abstract texture; white text must read on it).

LANGUAGE VARIETY (binding — a tester flagged this): never lean on a crutch
phrase across the deck. "Quick one", "let's dive in", "simply", "the best
part" — any phrase used once is USED; the next slide finds another way to say
it. Vary sentence openers between slides, and read the deck's narration as one
script: if two slides could swap voiceovers without anyone noticing, rewrite
one. This applies doubly when the client gave no script and you are writing
from research.

TOKENS APPEAR ONCE PER SLIDE (binding): a shortcode like [viewer-name-first],
[viewer-name], [sender-name], [sender-email] may appear AT MOST ONCE across a
slide's entire on-screen copy (headline + support + greeting + items +
contact). The contact block already carries [sender-name]/[sender-email], so
no other element on that slide repeats them; a greeting that says
[viewer-name-first] means the headline must not. (Narration is exempt.)

SELF-CHECK before emitting (the validator rejects violations, costing a slow
repair round): (1) every slide's category fits its content SHAPE and capacity;
(2) the deck uses the library's breadth — Statement, Split, Stats, Lists,
Full-Screen Image, Multi-Panel, Testimonials, and Process categories all
appear where the script gives any excuse; never funnel everything into two or
three familiar categories; (3) **at most FOUR slides in the whole deck have
photoZone "none"**; (4) every imageHint is unique; (5) every closing fork
targets a category-15 CTA slide; (6) all sectionIntro slides share category 3;
(7) every topic chapter's last beat carries `advanceTo: "Main Menu - Return"`;
(8) a Finish Up chapter exists and ends on the outcome question; (9) every
outcome has its own chapter ending on a category-15 terminal.

## Typeface pairing — a DESIGN DECISION, made fresh per brand

Choose exactly two families from the platform catalog below (nothing else will
resolve). The pairing should express the brand's personality — serif + sans is
one classic option, NOT the rule. Consider:
- Elegant / editorial / heritage → serif display (Playfair Display, Source
  Serif Pro, Yeseva One, Rasa) + a quiet sans (Manrope, Open Sans, Nunito).
- Modern / tech / minimal → strong sans display (Archivo, Space Grotesk,
  Montserrat, Oswald) + a lighter sans (Manrope, Fira Sans, Source Sans Pro).
- Bold / energetic / youthful → condensed or punchy display (Barlow Condensed,
  Bangers, Coiny, Shrikhand) + a sturdy sans (Poppins, Nunito).
- Warm / human / community → rounded or friendly display (Quicksand, Lobster
  Two, Carter One) + Open Sans / Nunito.
- Serious / institutional → slab display (Roboto Slab, Aleo, Ultra) + Roboto /
  Source Sans Pro.
Pick the DISPLAY variant heavy (Bold/ExtraBold/Black where the family has it)
and the body face in a readable weight. Never the same family for both unless
the brand demands extreme minimalism. Avoid novelty faces (VT323, Monoton,
TrashHand, Sancreek…) unless the brand is genuinely playful.

### Platform font catalog (exact names)
Abril Fatface, Aleo, Amatic SC, Archivo, Bangers, Barlow Condensed, Bungee
Inline, Carter One, Caveat, Coiny, Courier Prime, Elsie Swash Caps, Fira Sans,
Krona One, Kumar One Outline, Lobster Two, Manrope, Monoton, Montserrat, Nixie
One, Notable, Nunito, Open Sans, Ostrich Sans, Oswald, Palanquin Dark,
Parisienne, Permanent Marker, Petit Format Script, Playfair Display, Poppins,
Quicksand, Rasa, Roboto, Roboto Condensed, Roboto Slab, Sancreek, Shrikhand,
Source Code Pro, Source Sans Pro, Source Serif Pro, Space Grotesk, Space Mono,
Stint Ultra Condensed, Stint Ultra Expanded, Sue Ellen Francisco, TrashHand,
Ultra, VT323, Yeseva One.

## Reach for Multi-Panel Grids (category 8)

Whenever a beat presents 2–6 PARALLEL visual subjects — services, locations,
team members, product range, amenities, before/after, portfolio proof —
assign **category 8 (Multi-Panel Grid)** rather than a list or a single
photo. Grids are the deck's most photographic, premium-feeling layouts
(mosaics, checkerboards, nested columns) and are consistently under-used:
aim for 2–3 grid slides per deck when the content supports them. A list
with icon rows is for abstract points; when the items are THINGS a camera
can show, use the grid.

## Questions carry real choices (binding)

A `kind: "question"` slide must offer AT LEAST 2 answer options. A beat
with a single button is not a question — outline it as a plain content
slide (autoAdvance true), or as an interactive gate only when the user
asked for one.

BRANCH REJOIN (binding): every `kind: "answer"` slide that auto-advances
MUST carry `advanceTo` naming the slide AFTER the question's whole answer
group (usually the next Question, or whatever follows the fork). Deck
order alone would march one answer into its sibling — a viewer who picked
"Beginner" must never be shown the "Experienced" response on the way out.

EVERY OPTION WIRED (binding): every item on a question, menu, or CTA slide
carries `target` (an existing slide name) or `url`/`finish` — an option
without one is a DEAD button in the player. On a trivia question, the
correct option targets "Answer N - Correct"; every other option targets
"Answer N - Incorrect". Never emit an item and leave its wiring to be
guessed later.

SHARED RESPONSES FIT EVERY CHOICE (binding): when two or more options
route to the SAME answer slide, that slide's narration and copy must read
naturally for EVERY option routed to it — never write it for just one of
them (Blue Harbor: "Bathroom" was sent to a response written for "not
sure", which reads as ignoring the viewer's answer). Prefer one response
slide per distinct choice on preference questions; share a response only
when one message truly fits all of its options.

PACING (binding): one slide per script row — NEVER merge rows into one
slide. Standard slides target ≤10 seconds of narration; the deck should
read as many short slides, not few long ones. Interactive slides (menus,
questions, CTAs) hold ≥15s by design and are exempt.

## Interaction limit — sliders and multi-select (binding)

Only single-answer button questions branch to per-answer response slides.
A Slider interaction, or any multi-select question (choose several answers,
then click Move Ahead), CANNOT show different slides per answer — the
platform advances them to ONE next slide. Outline such interactions as:
Question N (kind "question", autoAdvance false) → exactly one follow-up
slide that states the correct answer / responds generally → the flow
continues. Never emit Correct/Incorrect (or per-choice) beats after a
slider or multi-select.
