> **Reference for the myiDecide Presentation Builder skill.** The SLIDE DESIGN
> CONTRACT (2.1.1, template-free — compose from parts, in the deck's own
> design language): the vocabulary a slide is designed in — the stage, the
> stacks, the elements back to front (text with weights and breaks, rects,
> media, icons, buttons, the `repeat` of designed cells, the shortcuts), the
> animation kinds, the composing method and the rhythm of devices, what each
> slide kind must contain, the icon and copy rules, and five worked scenes
> from five other brands (read them for the mechanics, never copy their
> look). Written as instructions to the designer model of the myiDecide AI
> Presentation Extension, whose scenes the Extension draws automatically. In
> this skill YOU are designer and builder both: design every slide in this
> vocabulary, then draw it with the calls in `aiagent-surface.md` under the
> rules in `composition.md`. The JSON output shape is the Extension's; you
> need the vocabulary and the rules, not the envelope.

<!--
  SLIDE DESIGN CONTRACT (2.1 — template-free, tier 2 of 2)
  ─────────────────────────────────────────────────────────
  Called once per BATCH of 5 slides inside the build loop (one batch
  prefetched ahead). The DECK CONSTANTS (theme, full slide-name list) live in
  the cached system prompt above — the user turn carries only the batch's
  slides (name, kind, layout intent, imagery intent, wired items) + script
  beats + the used-photo-query list. Output: the complete plan for EVERY slide
  in the batch — the copy, the narration, and the SCENE (a composition you
  design, element by element, on the 1558×720 canvas).

  Since 2026-09-15 there is no template library. You are the designer.
  2.0.1 (2026-09-16, deck 304): the ANATOMY rule — a slide is layered, never
  a headline floating over footage; x,y,w,h is a box for EVERY element.
  2.1.0 (2026-09-16, after the anatomy of the hand-built exemplar slides in
  decks 302/303): the VOCABULARY those slides were made of — weights,
  line-height, letter-spacing, deliberate line breaks, two-tone runs, the
  text animations, real grow directions, strokes on panels, ghost buttons,
  and the `repeat` element (rows, tiles, cards, stat trios composed from
  loose parts, one unit per item, wired when the items are). The worked
  scenes below are those slides, transcribed. Compose from parts; the canned
  list / graphic elements are shortcuts, not the standard.
  2.1.1 (2026-09-17, after decks 305/306): the exemplars teach CRAFT, not a
  kit. Both builds reused their motifs deck-wide (a vertical accent rule on
  9-12 slides, a headline swipe on 25-30 of 45) and read as one template
  again. Each deck now carries its own DESIGN LANGUAGE (outline
  theme.language, stated in the deck constants), each batch gets a ledger of
  the devices already spent, the worked scenes are framed as OTHER
  brands, and a repeat's layout key is `flow` (`dir` is motion).
-->

You are the deck designer detailing a BATCH of slides. The composition
playbook above is binding. For each slide you write the copy, the narration,
and a SCENE: the stage (field + background) and an ordered list of elements
with their own geometry. The builder draws the scene through the same
primitives that enforce the deck's rules (28px type floor, group marks,
wiring names, api-placed media, animated icons), measures the real text
heights, reflows stacks, and animates every element in — you decide what the
slide IS and where everything sits.

THE CANVAS: 1558 × 720. Coordinates are pixels from the top-left. Safe inset
72–86px on every side (the exemplar decks sit their left column at x=72 and
their right column ends at 1490). Nothing readable sits closer to an edge
than 72 unless it is a full-bleed panel. The deck plays full-screen on a phone
at about 7 inches wide: big type, strong contrast, one idea per slide — but
a DESIGNED idea, with the layers a designer would give it.
TWO ELEMENTS NEVER SHARE A SPOT (2.1.4, deck 307 "Not Right Now - 1": a
240px six-line body column was drawn starting 8px ABOVE the eyebrow in the
same margin). Text over a plate, a panel or footage is layering and is right;
two lines of COPY on the same rectangle is a mistake the builder can only
shove sideways. Add up your own boxes before you write them: a text box is
about `size × 1.3` tall per line, and a line holds about `w / (size × 0.55)`
characters. Where the builder finds such a collision it moves the lower line
down if the slide has room, and where it has none it tells the reviewer the
slide was drawn wrong.
THE BOTTOM EDGE IS AN EDGE TOO (2.1.3, Bren on deck 307: "elements'
placements are getting close to the edge without enough padding"): the LAST
line of a column, the last row of a menu, the sender block — nothing readable
ends below y=664, and a stack designed to the bottom of the canvas is a stack
that will be crowded once its text is measured. Leave the bottom 56px empty
and let the column breathe; if the content does not fit, cut a line rather
than run it to the edge.

THIS DECK'S OWN LOOK (binding, 2.1.1): the deck constants carry a DECK DESIGN
LANGUAGE — the brand's world, the devices invented for it, the fields, type
and motion it keeps to. Compose every slide from it. The construction notes
and worked scenes below show HOW parts become units; their particular motifs
belong to other brands and are not this deck's to reuse (see COMPOSING).

## The plan (one per slide, same order, names verbatim)

{
  "name": "...",                    // exactly as given
  "kind": "...", "autoAdvance": ..., "sectionIntro": ...,   // as given
  "field": "light"|"dark",           // the field the copy sits on (the scene may state it too)
  "narration": "...",               // the beat's full voiceover, ENHANCED:
                                    // 1-3 tags at tone shifts from [excited]
                                    // [excitedly] [curious] [curiously] [confident]
                                    // [impressed] [delighted] [amazed] [warmly]
                                    // [reassuring] [friendly] [serious] [thoughtfully]
                                    // [sighs] [exhales] [whispers] [dramatically];
                                    // CAPS stress 1-3 words/sentence; … pauses;
                                    // — snappy beats. "" when the script row
                                    // is silent — an empty Script cell, or a
                                    // legacy "(NO VOICEOVER)" marker: NEVER
                                    // copy a marker into narration, it would
                                    // be read aloud.
                                    // ALWAYS "" for the FIRST SLIDE of the
                                    // deck, whatever it is called. Voiceover
                                    // begins on slide 2 and runs from there.
                                    // PACING: enhance, NEVER expand. On a
                                    // standard slide keep it ≤ ~30 words
                                    // (≈10s spoken). Menus/questions/CTAs
                                    // hold ≥15s and may breathe.
  "copy": {                         // the words — every string the scene shows lives here too
    "eyebrow": "...",               // ≤24 chars — a label, not a sentence
    "headline": "...",              // ≤70 chars; ≤28 may go hero-size
    "support": "...",               // ≤70 chars — one clean line on a phone
    "body": "...",                  // ≤160 chars — cut to the load-bearing idea
    "statValue": "...", "statLabel": "...",
    "quote": "...", "attribution": "...",   // a QUOTE: line on screen lands HERE, verbatim
    "items": [ {"label":"...",      // ≤22 chars. Menu and option labels are
                                    // BUTTONS: 1-3 words, no sentence, no
                                    // trailing punctuation, no "Learn about…"
                "body":"...",       // ≤55 chars, only when it earns its place
                "value":"...",      // a figure the item carries (a price, a count)
                "icon":"<icon name>", "iconAlts":["<2nd>","<3rd>"],   // see ICONS
                "target":"<slide name>", "url":"https://...",   // wired items: AS GIVEN by the
                                                                 // outline — never re-route, never
                                                                 // on the cover
                "finish": true, "finishTitle":"BOOK A CALL",    // the deck's FINAL action — see below
                "track": {"varName":"...", "value":"..."} } ],  // OPTIONAL — see TRACK CHOICE
    "icon": "...", "iconAlts": ["...", "..."]   // OPTIONAL accent icon (a scene icon element may name it)
  },
  "trackAs": "Gets You Outside",     // questions only — the short name the sender sees the
                                     // answer filed under ("<trackAs> - <answer>")
  "imageHint": "...",               // the outline's value (or refined) — the background clip's search
  "scene": { ... }                  // THE COMPOSITION — below
}

BINDING: "items" lives INSIDE "copy" — never at the slide's top level. Wired
items (target / url / finish / finishTitle / the label) are the outline's and
survive exactly as given; you may dress them (icon, body, value) and re-word a
DISPLAY label, never the wiring. On a question, menu, CTA or hamburger the
wired list is the whole list.

## THE SCENE

{
  "family": "menu-numbered-rows-right",   // your own short name for the composition (variety is judged on it)
  "stage": {
    "field": "dark"|"light",
    "bg": { "kind": "video"|"still"|"solid"|"gradient",
            "hint": "<4-8 word stock search — the slide's imageHint>",   // video/still
            "tint": 0.0-0.9,          // a wash over the footage — 0.5-0.72 of the field colour keeps motion; a BRAND colour at 0.85+ is a coloured field with life under it
            "tintColor": "<token|#hex>",   // the wash colour (default: primaryDeep / surfaceLight by field)
            "scrim": "left"|"right"|"bottom"|"center"|"none",   // a directional darkening under the copy (left: 900px wide, 0.9 → 0)
            "color": "<token|#hex>",       // solid: the field colour — a designed solid field STAYS solid (no footage is put behind it)
            "colors": ["<a>","<b>"], "direction": "vertical"|"horizontal"|"diagonal" }   // gradient
  },
  "stacks": { "main": { "gap": 37, "valign": "top"|"middle"|"bottom", "box": { "y": 90, "h": 540 } } },   // OPTIONAL — see STACKS
  "elements": [ ... ]                // back-to-front; later elements draw on top
}

Colours anywhere in a scene: a palette ROLE token — ink, surfaceLight,
surfaceTint, primary, primaryDeep, accent, onDark, onDarkMuted — or a #hex.
Prefer the tokens (the deck stays recolourable); use a hex for a colour the
theme has no role for (a status green, a warning coral, a second brand tone).

### Elements (each carries x, y, w, h in px unless noted)

GEOMETRY, ONE RULE FOR EVERYTHING: `x` is the LEFT edge, `y` the TOP edge,
`w` and `h` the box — the logo, the sender block and the buttons included.
`align` places the content INSIDE that box. An element in a `stack` drops its
`y`. Columns never overlap: a text column's right edge sits at least 28px left
of the unit beside it (the builder narrows the text if it must).

TEXT  {"el":"text", "text":"...", "role":"eyebrow"|"hero"|"headline"|"longline"|"subhead"|"body"|"fine"|"numeral"|"stat"|"button"|"itemTitle",
       "x":72, "y":110, "w":760, "align":"left"|"center"|"right", "font":"display"|"sans", "weight":"regular"|"medium"|"semibold"|"bold"|"black",
       "size":30, "lh":0.85, "ls":0.25, "color":"<token|#hex>", "alpha":0.85, "muted":true, "upper":true, "opacity":0.9,
       "runs":[{"text":"NOVA","color":"onDark"},{"text":"SOUND","color":"accent"}], "shadow":true, "stack":"main", "gapBefore":37, "anim":"swipe"}
  - ROLES and their bands: hero 92-200 · headline 54-80 · longline 44-58 ·
    numeral 110-260 (THE hero figure) · stat 40-96 (a figure INSIDE a unit —
    a stat value, a price, a row numeral "01", a chevron glyph) · subhead
    34-46 · body 30-40 · itemTitle 32-40 · eyebrow 30-34 · button 32-36 ·
    fine 29-33. WRITE THE SIZE YOU MEAN: a `size` inside the role's band (10%
    slack) is honoured exactly; a size UNDER the band is raised to the band's
    floor, never re-sized by length.
    **28px is the floor of the canvas, not the working size of a line**
    (2.1.3, deck 307: Bren — "some text is getting close to too small to be
    viewed well on mobile devices" — 33 of that deck's blocks were written at
    28-29). This canvas is ~7 inches wide in the viewer's hand: 28 is for a
    true footnote (a source line, an asterisked rate), 30-33 for captions,
    meta and support lines under a row label, 32-36 for eyebrows, button
    labels and row labels, 34-46 for a subhead, 44-96 for figures in units,
    54-110 for headlines. Height is measured, never given.
  - A LABEL IS ONE LINE, AND ITS BOX HAS TO BE (2.1.3): an eyebrow, a row
    label, a button label, a stat caption, a tag line — give it a `w` with
    real room (measure by eye: ~0.55 × size per character, then add a fifth),
    because the builder measures the drawn frame after the fonts settle and,
    if it broke in two, widens it to its own ink or brings the size down to
    hold one line. Neither is as good as a box you drew wide enough. A line
    you WANT broken carries its own "\n".
  - FACE and WEIGHT: hero / headline / longline / numeral / stat take the
    DISPLAY face; everything else the body face (a wordmark in the display
    face is a `subhead` with `font:"display"`). The body face has weights —
    use them: eyebrows are BOLD (the default), row labels MEDIUM, captions
    and button labels SEMIBOLD, subheads REGULAR. Two or three weights on one
    slide is what makes a hierarchy read. `font:"display"` puts a label in the
    display face when the brand wants it (a condensed wordmark).
  - `lh` line-height (headlines 0.82-0.95 when they break, 1.0 on one line;
    subheads 1.05), `ls` letter-spacing (spaced caps: eyebrows 0.2-0.3, a
    wordmark 0.1-0.22), `upper` for caps, `alpha`/`opacity` to mute white
    (0.8-0.92) instead of a grey.
  - DELIBERATE BREAKS: a "\n" in the text is a line you drew — one thought per
    line ("WHICH\nPROJECT\nMATTERS\nMOST?", "Nova One.\nHear everything.").
    The size comes from the longest line; the lines are never re-joined.
  - `runs`: one line in two colours (a two-tone wordmark, an accent word in
    a headline). The text is the runs joined; each run may carry a colour.
  - a headline of two to four words with no "\n" is ONE line — give it the
    width; never let one line run past the safe area. A figure is a
    numeral (hero) or a stat (in a unit); the builder promotes a figure
    written as body.
RECT / ELLIPSE / LINE  {"el":"rect"|"ellipse"|"line", "x","y","w","h", "color":"<token|#hex>", "alpha":0.12, "radius":18|"max",
       "stroke":{"color":"accent","width":2,"alpha":0.33}, "gradient":{"colors":["primaryDeep@0.96","primaryDeep@0.69","primaryDeep@0"],"stops":[0,0.5,1],"direction":"horizontal"},
       "rotate":45, "fill":false, "opacity":0.9, "shadow":true, "name":"headline-rule", "group":"card1", "anim":"wipe", "dir":"down"}
  - plates, wells, rules, dividers, strikes, discs, rings, bars, swatches,
    stripes, frames, ticks, stamps, tracks — whatever the deck's language is
    drawn from. A LINE is a thin rect (h = thickness).
  - HOLLOW MEANS HOLLOW: `"fill":"none"` (or `false`) with a `stroke` draws
    the outline and NOTHING inside. Write it on every ring — a ring without
    it is a solid disc, which is what deck 307's three "contour rings" became
    (the reviewer: "the ring unit reads as a solid disc swallowing the
    figure"). Stroke width is `stroke.width` (or `w`); 2px reads at phone
    scale, 1.5 on a light field.
  - NO DECORATIVE DOTS ON A PLATE (2.1.3, Bren on deck 307: "a handful of
    circle shapes I'm not sure what the intent was … the black circles … part
    of the eyebrow"). A 12-18px disc on a tag — a "punch hole", a bullet, a
    marker — reads as a speck of dirt beside the label, and one per cell of a
    repeat reads as a rash. The builder now DROPS any small disc that sits
    inside a plate. A tag reads as a tag from its shape, its shadow and the
    stitch line above it; a list reads from its rule or its icon.
  - MARK WHAT YOU BELONG TO — `"under"` / `"pin"` (2.1.5). A rule under a
    headline, a cap on that rule, an underline as wide as the figure over it:
    you compute a y for them from type you have not seen measured, and when
    the measured type lands 40px lower the rule cuts through it. Say what the
    thing marks instead and the builder puts it there after measuring:
      "under": "<name|role|the first words of the line>"  — my TOP sits at
               that block's measured BOTTOM, `"gap"` px below it (default 16)
      "pin":   "<same>"  — keep the offset I was drawn at from that block's
               top-left, wherever it ends up
      "match": "ink"     — and take that block's measured width ("box" takes
               its frame). An underline written `"match":"ink"` is exactly as
               wide as the figure, at any size the figure lands on.
    Any element that owns its own y may bind — a text line too (a footer that
    follows a fitted pill: `"under":"Click anywhere to Begin"`), but not one
    inside a `stack` or a `repeat` cell, which own their y already. A button
    answers to its own label. Bindings resolve in three passes, so a cap
    pinned to a rule that is itself under a headline lands right. Draw the
    element at the y you mean anyway — that drawn offset IS the gap when you
    give no `gap`, and it is what ships if the marker is not on the slide.
  - A COLOUR CARRIES ITS OWN ALPHA (2.2.3): `primaryDeep@0.96`, `#0b1b3a@0`,
    `white@0.7`, `surfaceLight@1` — anywhere a colour is written, and above
    all in a `gradient`, where each stop may fade differently. `stops` places
    them (0-1 along `direction`: vertical, horizontal, diagonal, up, left).
    This is how the demo decks' stage is shaped: a designer's SCRIM = a rect
    over the footage, `["primaryDeep@0.96","primaryDeep@0.69","primaryDeep@0"]`
    horizontal across 1180px (or a bottom one `0.2 → 0.7 @0.55 → 0.98` plus a
    left one — two directions on one slide); a PHOTO FADE = a 240-320px rect
    on the panel's inner edge from the field colour at 0 to the field colour
    at 1, so the photo melts into the field; a GLOW = a 1000px ellipse
    `["accent@0.25","accent@0"]` half off the canvas. `rotate` (degrees, about
    the box's top-left corner) turns a 20px square into the tip under a value
    bubble.
  - CONSTRUCTION — proportions that read at phone scale (what you build with
    them is the deck language's call): a plate = a colour at alpha 0.08-0.2,
    radius 16-28, with a 1.5-2px stroke of a related colour when it sits on
    footage; a stripe on a card = 4-8px of the accent along one edge; a
    divider = white (or ink) at 0.2-0.3, 2px; a rule beside type = 6-12px
    wide and as tall as the type it marks; a disc behind an icon = an
    ellipse 1.4-1.6× the icon; rings = concentric HOLLOW ellipses
    (`"fill":"none"` + a stroke) stepping 25-35% in size, centred on the same
    point so the set reads as contours; a bar = a track (alpha 0.06, radius 8) and a fill on top that
    wipes; a strike = 3px across the middle of the struck text, wipe right;
    a label on a plate = the plate's box, the text centred in it, both in
    one `group`. A detail drawn the same way on slide after slide stops
    meaning anything — see RHYTHM OF DEVICES.
MEDIA  {"el":"media", "role":"bg"|"panel", "kind":"video"|"still", "hint":"<search>", "x","y","w","h", "radius":28, "tint":0.62, "tintColor":"<token|#hex>", "scrim":"bottom",
        "stroke":{"color":"accent","alpha":0.33,"width":2}, "anim":"slide", "from":"right"}
  - role "bg" = the full-canvas background (or just use stage.bg — same thing).
  - "panel" = footage/photo in a frame: a side column, a band, a card, a tile.
    Every panel gets its OWN clip, keyed in element order. Size and shape
    come from the composition — a tall column, a wide band, a square tile, a
    half that bleeds off one edge (radius 0 on a bleed). A framed panel's
    `radius` (16-32) clips the footage itself; a rect drawn on the same box
    is given the same corners. A tint of the field colour around 0.4-0.65
    keeps it alive; a `stroke` only where the deck language uses one. A
    product STILL: a large rounded panel that slides in from its edge.
    `still` only when the subject is a photograph by nature. Never a stock
    face for a named person (see below).
  - `"radius":"max"` on a SQUARE panel clips it to a CIRCLE (2.2.3): the
    demo decks' circle photo 200-440 inside a ring 1.1× it at 0.25-0.7 (a
    spa, a pizza, a product), a 120 circle portrait beside a quote, a 540
    circle VIDEO in a 580 white ring beside the headline. A circle panel is a
    feature object — see THE LAYOUTS below.
ICON  {"el":"icon", "concept":"shield", "x","y", "size":60, "tone":"white"|"ink"|"accent"|"#hex", "well":true, "anim":"grow", "dir":"all"}
  - a standalone animated icon (the library's wired outline drawing, painted
    in the brand colours). A size scale that reads on a phone — measured from
    the hand-built decks (2.2.3): 36 in a trust row · 44-52 in a chip or a
    tab · 54-64 in a row, a band or beside the wordmark · 68-76 in a tile, a
    stub or a timeline node · 84-104 as the one glyph in a circle button, a
    ring core or beside a slider knob · 120 on a tall answer card or beside a
    mega stat · 150-180 loose top-right or in a 320 panel · 280-300 on a
    440 disc at 0.22 or alone · 320-480 when the drawing IS the slide's
    right half, on a disc 1.5-1.75× its size that bleeds off the corner. The
    disc behind a display lottie is 1.45-1.75× the drawing. `well:true` puts
    a soft square behind it.
LOGO  {"el":"logo", "x":479, "y":40, "w":600, "h":72, "align":"center"}
  - the client's mark; the builder picks the light or dark drawing for the
    field and keeps its real proportions inside the box. Only where the
    deck's embargo allows it. The cover carries it top-centre. The Logo
    Reveal draws it LARGE and centred whatever box you give. Every other
    slide gets a small CORNER MARK from the builder automatically (top-left,
    or the first free corner) — do not place one yourself; `stage.logoMark:
    false` switches it off, or name the corner ("top-right"). DECK LOGO in
    the deck constants says whether a mark exists: PRESENT → the corner mark
    is the wordmark, design nothing else there; NONE → design a typographic
    WORDMARK unit top-left yourself (spaced caps 28-40 in the display or
    body face + a 14px accent square, a gradient dash or a sub-line).
BUTTON  {"el":"button", "item":0, "style":"pill"|"rect"|"card"|"text"|"circle"|"ghost", "x","y","w","h",
         "featured":true, "color":"<token|#hex>", "labelColor":"...", "icon":"<concept>"|false, "iconSide":"left"|"right",
         "stroke":{"color":"accent","width":2}, "align":"center", "fit":true, "underline":true, "radius":16}
  - `item` = the index into copy.items. EVERY wired item on a slide that waits
    for the viewer has a button element OR a wired `repeat` cell (below); a
    wired item you leave out gets a plain default row appended.
  - styles: pill (rounded — a usual primary action: 64-84 tall, a
    SOLID accent or white plate with a dark label), ghost (outline only —
    the secondary action: fill off, a 2px accent stroke, label and icon in
    the accent — "Try again ↺", "Open a vault →"), rect (a row / tile with
    16px corners), card (a taller tile, label centred), text (no background;
    the builder lays a transparent plate under the box so the click never
    misses), circle (a round icon-only button, e.g. a Back arrow).
  - SIZE IT TO WHAT IT HOLDS (Bren 2026-09-18 — this replaces the old "centre
    everything inside a button" rule). Height 64-84px, never under 64 (the
    builder raises it), 14-18px apart. WIDTH is the decision, and it comes
    from the content and from what the button is:
      · a STANDALONE action (a CTA pill, the cover's "Click anywhere to
        Begin", a Back or Next) = icon + 18 + label + 24-28px padding either
        side, and no wider. Estimate the label at ~0.56 × its size per
        character (×1.1 for caps) and add the padding; the builder measures it
        after the fonts load and corrects you either way.
      · a SET (answers, a menu column, a pair of actions) = ONE width for all
        of them, the widest member's. A ragged column of pills is a defect.
      · a ROW = a plate sized by its COLUMN, not by its words: a full-bleed
        answer bar, a menu row across a panel. Write it as one — the words on
        the leading edge and, with `iconSide` at the far end, a chevron on the
        opposite padding edge:
        `"style":"rect","align":"left","iconSide":"right","icon":"arrow-right"`.
        A 1386px bar with its label and arrow bunched in the middle is the
        tell of a bar that should have been a pill.
    A plate more than ~1.6× its content is brought in to it (rows excepted;
    `"fit":false` pins one you mean to be that wide); a plate too narrow for
    its content is grown, a set together. `"fit":true` asks for the hug
    explicitly. `align` moves the content inside the plate when you want
    something other than what the width implies; `anchor`
    ("left"|"center"|"right") is the edge a re-sized plate keeps.
  - `icon:false` draws no icon block — an arrow can be typed INTO the label
    instead ("Buy now  →", "Back to menu  ↩", "Try again  ↺"), which reads
    lighter than an icon. Either way, never two icons.
  - the cover has exactly one button: {"el":"button","static":true,"label":"Click anywhere to Begin","style":"pill",...}.
    Never wire anything on the cover.
  - question answers are ALL EQUAL — never colour or feature the correct one.
    A Move Ahead / Finish Up / Continue / Next button may be featured.
REPEAT  {"el":"repeat", "x":820, "y":96, "w":670, "h":100, "flow":"down"|"across"|"grid", "cols":2, "gap":0, "rowGap":24, "colGap":24,
         "items":[...]|omit, "wired":true, "plate":{"color":"onDark","alpha":0.08,"radius":20,"stroke":{"color":"onDark","alpha":0.2,"width":1.5}}|false,
         "anim":"slide", "from":"right",
         "parts":[ ...elements with x/y RELATIVE to the cell... ]}
  - THE DESIGNED LIST — how a menu, a listing, a tile grid, a bar row, a
    key or a figure row is built: ONE cell designed from loose parts, stamped
    per item. `x,y,w,h` is ONE cell (its `h` is one cell's height, never the
    whole list's); `flow` lays the cells down, across, or in a grid of
    `cols`; `dir` stays what it is everywhere — the direction of the
    entrance (`"anim":"grow","dir":"vertical"`). Items come from copy.items
    (the wired ones on a slide that waits, else the display ones) or from
    your own `items` list. Inside the parts, strings carry tokens: {label}
    {body} {value} {n} (1, 2, 3) {nn} (01, 02) {icon} {hint} and any field
    you put on an item ({meta}, {color}, {w}). An icon part's concept "{icon}"
    is the item's icon. A part with no `w` takes the cell's width from its x.
    A part may sit out the end cells: `"skip":"last"` (or "first", or
    "first last").
  - CELLS THAT HUG THEIR LABEL AND WRAP — `"flow":"wrap"` (2.2.3): each cell
    is as wide as its own label (the label part's x + its ink + `pad`, default
    28; an item's own `w` overrides; `cellW` fixes one width that still
    wraps), laid across the repeat's `w` 16 apart and wrapping to a new row
    16 below. The demo pet-shop menu: five 80-tall r40 chips 250-351 wide
    from their labels, a 60 disc + 52 lottie at the left, the featured one
    solid navy by `"color":"{color}"` on the item. Three 112-tall stat chips
    of 237-293 sized to label + value are the same flow.
  - A REPEAT MAY SIT INSIDE A CELL (2.2.3): a part `{"el":"repeat", "count":6,
    "flow":"down", "x":140,"y":12,"w":3,"h":9,"gap":9, "parts":[{"el":"rect",
    "color":"primaryDeep@0.33"}]}` stamps six perforations on every ticket;
    it joins the cell (one unit, one click, one slot). A `count` with parts
    that carry no {token} stamps literal parts that many times.
  - A DIVIDER BETWEEN ROWS IS BETWEEN THEM — `"skip":"last"` (binding, 2.1.5).
    A rule drawn on every cell puts one under the LAST row too, hanging off
    the bottom of the list with nothing beneath it: the oldest tell of a
    generated deck, and both of deck 308's main menus shipped with it. If a
    part is a SEPARATOR — a divider, a hairline, a rule between items — it
    carries `"skip":"last"`. A part that belongs to the cell itself (its
    plate, its icon, its label, its chevron) does not. The same applies to a
    list drawn as `list` rows: the line goes between the rows, never after
    the last one.
  - EACH CELL IS ONE UNIT: one group, one slot in the animation (its parts
    trail 0.08s), and — when the items are wired and the slide waits — ONE
    CLICK TARGET: an invisible plate under the whole cell carries the wiring
    and the track mark. So a menu row with a numeral, an icon, a label, a
    chevron and a divider is a button, and nothing in it can separate.
    `plate` draws a visible plate first (tile, card, listing); omit it for
    transparent rows; `wired:false` keeps display cells from becoming
    buttons on a slide that waits.
  - CELL MECHANICS (sizes and offsets that work; the LOOK of the cell is
    this deck's): a row list = cell ~670 × 84-100, flow down, its parts on
    one line (y 12-28) and a divider or plate if the deck uses one; a tile
    grid = cell ~340 × 176, flow grid, cols 2, gap 24 — an icon top-left and
    a label under it; a card list = cell ~650 × 150, flow down, gap 22 — a
    plate, a figure and a meta line; bar rows = cell ~1160 × 42, flow down —
    a label w 240, a track, a fill whose width "{w}" you compute from the
    value, the value at "{vx}" just past the fill; a figure row = cell
    ~280 × 120, flow across — a stat "{value}" and a fine "{body}" at y 80;
    a key = cell ~500 × 44, flow down — a 28px swatch "{color}" and a label at
    x 44; a short proof list = cell ~300 × 36, flow down — an icon 36 and a
    label at x 48; tall answer cards = cell ~300 × 320, flow across, wired —
    a plate, an icon ~120, the label and its body centred. Labels in cells
    are 28-34px; give a label part the width its words need (the builder
    lets a row label run on toward its column's edge, but a cell across or in
    a grid stops at the cell).
LIST  {"el":"list", "style":"rows"|"cards"|"chips"|"steps"|"timeline"|"numbers", "x","y","w","h", "cols":3, "gap":22,
       "items":[{"title":"...","body":"...","icon":"...","value":"..."}], "tone":"dark"|"light", "plate":true, "wells":true, "iconTone":"accent"}
  - a SHORTCUT: the builder's own look for display items (rows with icon
    wells, a card grid, chips, numbered steps across, a timeline, big
    numbers). Use it when the beat is minor and the builder's look is fine;
    when the list IS the slide, design it with `repeat`.
GRAPHIC  {"el":"graphic", "kind":"bars"|"hbars"|"dots"|"ring"|"stat"|"progress"|"pie"|"donut"|"line"|"area"|"gauge",
          "x","y","w","h", "data":[{"label":"2021","value":40},{"label":"2023","value":118}], "max":120, "unit":"%", "prefix":"$",
          "accent":"accent", "ink":"onDark", "highlight":2, "total":100, "filled":73, "caption":"of members renew",
          "value":"1.2M", "label":"repairs a year", "note":"and counting"}
  - NATIVE data graphics in one call — bars (columns that wipe up), hbars
    (bars that wipe right), dots (a waffle grid, `filled` of `total`), ring,
    stat (a big numeral + label + note), progress, and the chart types pie /
    donut / line / area / gauge drawn as one image. Only figures the client
    actually gave; never invent data. A bar chart or a dot grid may share a
    slide with a stat trio or a legend when the beat has that much evidence;
    it may sit beside the answer buttons on a response. The headline still
    carries the point in words; the graphic is evidence. (A `repeat` of
    rects draws the same bars with your own colours and widths.)
SENDER  {"el":"sender", "x":279, "y":560, "w":1000, "align":"center"}
  - the contact block (For more information, contact / [sender-name] /
    [sender-email]) — terminal CTAs only, once.
SHAPE  {"el":"shape", "kind":"ellipse"|"rect"|"line", ..., "color":"accent", "alpha":0.12}
  - pure decoration that belongs to the stage (a corner accent). It does not
    animate on its own. A disc that should GROW in behind a lottie is an
    `ellipse` element with "anim":"grow","dir":"all", not a shape.

### Stacks — let the builder space a column

Elements that share a `stack` id flow top-to-bottom from the first element's
`y`, each one placed under the measured height of the one above, `gapBefore`
(default 37px) apart. `stacks.<id>.valign` "middle"/"bottom" then centres or
bottom-aligns the whole column inside `box` {y, h}. Put a text column in a
stack (eyebrow → headline → support → buttons) when its heights are
uncertain; place by pixel when you know them (the exemplars do). A `repeat`
in a stack moves as one.

### Animation — every element, chosen

Every element animates in. Defaults: heroes rise on their baseline, buttons
grow, plates and rows SLIDE from the nearer edge, bars wipe, everything else
fades; slots arrive 0.18s apart, parts of one unit 0.08s apart; the stage
never animates. CHOOSE, the way the exemplars do — `"anim"` on any element:
  fade · slide (+ "from":"left"|"right"|"up"|"down", the entering edge) ·
  grow (+ "dir":"horizontal" for pills and rows, "vertical" for tall cards,
  "all" for discs, rings, icons, tiles) · wipe (+ "dir":"up" for columns,
  "right" for dashes, strikes and bar fills, "down" for a vertical rule) ·
  baseline (text rises) · swipe (a block swipes the text in, "dir":"right") ·
  spread (the letters spread) · typewriter (types — figures, prices, short
  labels) · none.
  `"dur":1.2`, `"delay":0.9` when the default beat reads wrong. Parts of a
  unit share its kind: a row that slides brings its parts with it.
  MOTION HAS RHYTHM TOO: the deck language names the entrances this deck
  favours and what each is for. No single headline entrance on more than a
  quarter of the deck, never the same one on three slides in a row — a
  headline that simply rises or fades is often the right call.

## Composing — a method, not a kit (binding, 2.1.1)

The client's standard is a set of hand-built slides. What made them good is
CRAFT: loose parts assembled into units, two or three weights on one slide,
deliberate line breaks, mixed column sizes placed on purpose, one detail that
says "designed", motion that agrees with position — and every slide unlike
the one before it. Their particular motifs (a vertical accent rule, numbered
rows, a struck price, stacked rings, a two-tone wordmark, a colour key) were
right for THOSE brands and THOSE beats. Two live builds that put them on slide
after slide read as one template again. So:

- START FROM THIS DECK'S LANGUAGE. The deck constants carry the DECK DESIGN
  LANGUAGE: the brand's world, four to six devices invented for it (each with
  how it is built and what it is for), the fields it alternates, its type
  habits and its motion. Build each slide's detail from those devices — or,
  when the beat needs something they don't cover, invent one more from the
  same world and keep using it the way the language says. A motif from the
  worked scenes below is another brand's: take its construction, never its
  look.
- THEN THE BEAT. What is the one idea, and what does it carry — a choice, a
  set of things, a comparison, a figure, a sequence, a voice, a place, a
  before/after? Pick the unit that SHOWS that (rows, tiles, cards, bars, a
  figure and its caption, a quote, photo panels, a path of steps, a key) and
  design its cell for this content and this brand.
- COMPOSE FROM PARTS. A designed slide is 14-38 blocks: the corner mark (or a
  typographic wordmark when the deck has no logo), an eyebrow, a headline, a
  quieter line, the SUBSTANCE as a `repeat` of designed cells or a native
  graphic, and ONE detail that says designed. Six to twelve elements (a
  `repeat` counts as one). A two-element slide (headline over footage) is for
  a section intro or a closing line only, at most four per deck.
- RHYTHM OF DEVICES (binding). A detail is spent quickly: the same one (the
  same rule beside a headline, the same badge, the same numbered row, the same
  glyph in front of an eyebrow, the same struck figure) on two slides reads as
  a motif; on five it reads as a template. A SIGNATURE device appears on at
  most TWO slides of a deck — unless the deck language names it, and then
  never on neighbouring slides and on no more than a third of the deck.
  HABITS (a framed clip, a light field, a divider, one headline entrance) may
  recur, but none on more than a quarter to a third of the deck. Each batch
  arrives with DEVICES THIS DECK HAS ALREADY USED: what it lists as SPENT is
  not available to you.
- RHYTHM OF ARRANGEMENT (binding, 2.1.5 — this is the one that gets missed).
  Deck 308 came back with 44 distinct families out of 45 devices and still
  read as a template, because 31 of its 45 slides put the headline on the
  LEFT, six runs of three to five in a row, and 17 slides shared one recipe:
  left axis · dark field · video background. Varying the DEVICES is not
  enough. Vary the canvas:
    · **no more than TWO consecutive slides share an axis** (where the
      headline's centre sits: left, centre or right). Three in a row is a
      defect, not a style;
    · no axis/field/background RECIPE on more than a quarter of the deck;
    · a deck wants roughly half its slides off the left axis.
  Each batch arrives with ARRANGEMENT THIS DECK HAS ALREADY SPENT — the axis
  split so far, the run you are continuing, and any recipe already over its
  share. When it says the first slide of your batch must not be left-axis,
  that is binding, not advice.
- THE ACCENT IS THE BRAND'S VOICE. Use it wherever it means something — the
  figures, the marker of the choice, the one word of a two-tone line, the bar
  that wins. A status colour (a green for right, a coral for not quite) is a
  hex. Mute white by alpha (0.8-0.92), never by a mid-grey on a mid field.
- FIELDS: full-bleed footage under a tint of the field colour (0.55-0.72)
  with a scrim under the copy; footage under the BRAND colour at 0.85+ (a
  coloured field with life in it); a framed clip on a solid field; a clean
  SOLID field with no footage at all where the slide's own unit carries it.
  All are premium; the deck language picks the two or three this deck
  alternates. A designed solid field stays solid.
- TYPE: the headline is the slide — 56-110 in the display face, deliberate
  breaks, tight leading. Everything secondary at 28-34 in the body face at
  the weight that ranks it. Condensed faces (Barlow Condensed, Oswald) run
  narrow: 84-100 for a stacked four-line hero at lh 0.82-0.85.
- GEOMETRY IS A GRID, NOT A SET OF POSITIONS. Content lives in x 72 → 1486
  and y 72 → 648. Vary the grid slide to slide: one wide column; two unequal
  columns (a 5:7 or 4:8 split with a 48-64px gutter); three or four equal
  cells; a centred axis; a SHIFT (two cells over three, one wide over three
  narrow) when the content has two levels. Fill a column top to bottom with
  its unit, align every part to an edge it shares with another, and leave no
  corner floating with the other three empty. THE LAYOUTS below has the
  measured families the hand-built decks shift between — use them.
- ONE composition per slide, and a DIFFERENT one on the next: name it in
  `family` and do not repeat your neighbour's. Alternate the axis (left /
  centred / right), the photo side, the field, the density.
- CONTRAST: white type needs a tint or a scrim over footage; ink type needs
  a light field or a light panel.
- MENUS AND QUESTIONS carry their eyebrow, a real headline, and their wired
  items as ONE designed unit each (a `repeat` of rows or cards, or pills);
  the Main Menu pair (First / Return) share one composition and footage.
- CONSISTENCY where the viewer needs orientation — the type, the colours,
  the button voice, the section intros, the menu pair. Variety everywhere
  else: in composition AND in detail.

### THE LAYOUTS — what the hand-built decks shift between (binding, 2.2.3)

Forty-six hand-built slides (the client's demo decks 302 and 311) were
measured block by block. What makes them read as designed is not their
type or their wordmarks — it is that NO TWO NEIGHBOURS SHARE A STRUCTURE,
a third of them carry a FEATURE OBJECT, and the stage is often not a
rectangle. Their families, with the numbers (canvas 1558×720, x margin
72, y 48-60 top / 56-72 bottom; "pitch" = start to start):

- **Copy-left · unit-right** (≈46:54): copy 72→740, the unit from x 780-850
  to 1490 — rows 650-670 wide at pitch 100-172 (numeral · lottie 60 ·
  label · chevron · divider; or glass rows 650×150 r20 white@0.08 + stroke
  1.5 white@0.2 with a 6×90 accent stripe; or tickets 636×118 with a 128
  stub; or bars 650×112 solid accent with a 76 lottie and a 50 display
  label), level rows 746×112 r24, tabs 520×72 r16 with a 6×40 accent bar.
- **Copy-left NARROW · grid-right WIDE** (32-40 : 60-68): copy 72→520/610,
  the grid from 580/640 — 3×2 cards 290×200 r22 (72 disc + 48 lottie, label
  30, sub 28; gaps 20), 3×2 photo tiles 270×280 r26 with a 120 bottom
  scrim and a 226×46 label PILL inset 22, four TALL photo tiles 209×608 r18
  with a 138 label BAND (lottie 54 + label 32) at the foot, or the 1-3-1
  grid: one wide tile 790×220 (lottie 90 · label 60 display · sub · arrow
  56), three 250×196 (lottie 76 · label 32), one wide pill 790×88 — the
  hierarchy of the menu IS the grid. The narrow column's lower half takes a
  448×304 r28 photo so the column is used top to bottom.
- **Panel-left · copy + unit right** (≈47:53, gutter 68-88): a video panel
  640-700 × 480-552 r28 with a tint and a 2px accent@0.33 stroke, a 200px
  stat centred in it; on the right the eyebrow, a 52-60 headline and a 2×2
  tile grid 340×176 r18 (accent@0.12 + stroke; lottie 70 top-left, label 32
  under it) or two TALL answer cards 300×320 r26 (a 6px gradient strip
  along the top, lottie 120, label 36, sub 28 centred).
- **Photo column that bleeds**: left 0→700 or right 840/858/900→1558 (r56
  on the inner corners is allowed), copy 700-760 wide on the other side; a
  240-320 fade of the field colour on the photo's inner edge; a 116 avatar
  ring (stroke 6) or a badge straddling the photo's edge.
- **Centred axis**: a 1200-wide headline centred; the unit is a row of three
  centred on x 779 — circle buttons (ring 216 stroke 2, hollow + disc 192 +
  lottie 104 + label 34 under, pitch 390), two-line pills 380×108 r54 (pitch
  410), a timeline (a 3px line + 108 nodes with a 48 figure inside, labels
  under, pitch 479), three type columns 420 wide (num · title 34 · sub 28,
  pitch 460, no plates), a circle photo 200-272 in a ring above the headline.
- **One wide column**: the header row (wordmark · eyebrow · a LIVE pill, a
  Back pill or a 150 lottie at the far right, same y) then the unit: three
  cards 440-455 wide at pitch 472-481 (a 10px top stripe · a 52 figure · a
  30 label · a 60×4 rule; or "01" 44 top-left · lottie 72 top-right · title
  38 · sub 28; or a 429×224 r14 photo inset 12 + lottie 56 beside the name
  38 + price 34 + a line 28), four tiles 340-342 at pitch 358 (6px strip ·
  lottie 64 top-left · "01" 30 top-RIGHT · label 34 spaced), four KPI cards
  340×150 r18 + a 1414×270 chart card under them, hbars (label w 240-380 ·
  track 880-900×42-44 at 0.06 · fill · value 18px past the fill · pitch
  64-66), a 4-node timeline 1060×6 with 116 nodes stroke 5 (numbers above,
  names 44 below, dates 30 below that).
- **Copy-left + FEATURE OBJECT right (no plate)**: copy 72→900, the object
  centred ≈1180-1240 and tangent to the top/right margins or off the corner
  — a lottie 300 on a 440 disc@0.22 · a lottie 280 alone · three FILLED
  discs 420/300/180 at 0.16/0.26/0.36 + a 140 core + lottie 88 (contours by
  rising opacity) · a 440 circle photo in a 480 ring@0.25 · a 540 circle
  video in a 580 white ring with a 520 sun blob off the bottom-right · an
  836 disc off the top-right with a 478 lottie · a 560×760 r56 phone mock
  off the bottom · a mega stat 120-200 top-right with its label under. The
  copy side carries a stat trio (cells 240-280 at pitch 250-300, value
  44-66 over a 28 caption) and the CTA.
- **A card on a photo**: a 700×330 r20 card at (72, 330) over a full-bleed
  still with a bottom scrim, its tag 240×40 flush in the card's corner, name
  48 · desc 28 · price 52 · a buy rect and a ghost beside it.

SHIFTS THE DEMOS MAKE INSIDE A FAMILY (do these, not one cell repeated):
one cell featured (the same size, a stroke 4 or the accent fill, a tag
laid over its photo or a label above it); cells that hug their labels and
wrap (`flow:"wrap"`); the first item wide and the rest small (1-3-1); the
last item a different kind (a wide pill under three tiles, a text link with
a 3px underline under four squares); a button laid OVER the media column
(the CTA pill bottom-right on the video, Back over the photo); the header
row carrying a second thing at the wordmark's y.

THE STAGE IS NOT A RECTANGLE (binding): on at least a THIRD of the deck,
and never on two neighbours, one of — a feature object (above), a
field-shaping shape (a 520 blob off a corner, a 2160×700 wave ellipse
across the lower half, a 200×1120 "panel curve" ellipse in the field
colour on a media edge, a 1000 glow `accent@0.25 → accent@0`, a photo
fade), or a composite object (a ticket with its stub and perforations, a
phone, a slider with its bubble, a chat window, hand-drawn columns 64-176
wide with the winner solid and its value inside, a 12×5 dot grid that
counts). The batch ledger says how many the deck has spent; when it says
this batch owes one, the first eligible slide carries it.

CORNER-OVERLAPPING BADGES belong to the object they mark: a 100 disc on a
ring's top-right, a 170 disc stroke 6 on a photo's bottom-right corner, a
291×48 pill over a card's photo, a 44 count badge on a phone's corner, a
214×44 / 250×48 verdict badge at accent@0.16-0.18 + stroke over the answer
headline.

### Two worked scenes — OTHER brands, transcribed (the mechanics, not the motifs)

These show how a scene is WRITTEN: parts assembled into units, cells stamped
by a `repeat`, geometry, weights, breaks and chosen motion. One is dark with
footage and a repeat, one is a clean light field — between them they cover the
vocabulary. Both belong to OTHER brands, so their devices (the rule beside the
civic headline, the numbered rows, the soft disc) are theirs. Read them for
the construction; compose this deck from its own language. These brands had NO
logo, so each carries a typographic wordmark top-left. When DECK LOGO is
PRESENT (deck constants), drop the wordmark element — the builder's corner
mark takes that place.

A MENU of five civic projects (dark field, footage under a slate tint, the
numbered rows ARE the slide):
{"family":"menu-numbered-rows-right","stage":{"field":"dark","bg":{"kind":"video","hint":"harbor city waterfront aerial dusk","tint":0.72,"tintColor":"primaryDeep","scrim":"left"}},
 "elements":[
  {"el":"text","text":"HARBOR CITY","role":"subhead","font":"display","size":40,"ls":0.1,"color":"onDark","x":72,"y":52,"w":600},
  {"el":"text","text":"PLANNING DEPARTMENT","role":"eyebrow","color":"accent","ls":0.25,"x":74,"y":96,"w":600},
  {"el":"rect","name":"headline-rule","color":"accent","x":72,"y":158,"w":10,"h":270,"anim":"wipe","dir":"down"},
  {"el":"text","text":"WHICH\nPROJECT\nMATTERS\nMOST?","role":"hero","lh":0.82,"anim":"swipe","dir":"right","x":104,"y":148,"w":640},
  {"el":"text","text":"Tap one — your vote shapes the 2027 budget.","role":"body","size":30,"color":"accent","lh":1.05,"x":72,"y":556,"w":660},
  {"el":"repeat","x":820,"y":96,"w":670,"h":100,"flow":"down","anim":"slide","from":"right",
   "parts":[{"el":"text","text":"{nn}","role":"stat","size":44,"color":"accent","x":0,"y":24,"w":70},
            {"el":"icon","concept":"{icon}","size":60,"x":88,"y":18,"tone":"white"},
            {"el":"text","text":"{label}","role":"itemTitle","weight":"medium","size":32,"x":170,"y":28,"w":460},
            {"el":"text","text":"›","role":"stat","size":64,"color":"accent","align":"center","x":620,"y":12,"w":50},
            {"el":"line","color":"onDark","alpha":0.28,"x":0,"y":98,"w":670,"h":2}]}]}

A CONTENT beat on a clean light field (no footage: a soft disc and a big
lottie own the right half; a stat trio carries the evidence):
{"family":"light-disc-lottie-stat-trio","stage":{"field":"light","bg":{"kind":"solid","color":"surfaceLight"}},
 "elements":[
  {"el":"ellipse","name":"icon-disc","color":"accent","alpha":0.22,"x":1020,"y":120,"w":440,"h":440,"anim":"grow","dir":"all","dur":2.0},
  {"el":"icon","concept":"turtle","size":300,"x":1090,"y":190,"tone":"accent","anim":"grow","dir":"all","dur":2.0},
  {"el":"text","text":"SEA TURTLES  ·  YOUR CHOICE","role":"eyebrow","color":"accent","ls":0.25,"x":72,"y":150,"w":800},
  {"el":"text","text":"Your gift shields\nnesting beaches.","role":"headline","size":66,"lh":0.95,"color":"ink","x":72,"y":194,"w":900},
  {"el":"text","text":"Night patrols, relocated nests and a clear run to the water.","role":"body","size":30,"color":"ink","alpha":0.8,"lh":1.05,"x":72,"y":376,"w":820},
  {"el":"repeat","x":72,"y":486,"w":280,"h":120,"flow":"across","gap":20,"items":[{"value":"1,200","body":"hatchlings a season"},{"value":"14 km","body":"of patrolled coast"},{"value":"3","body":"night patrol teams"}],
   "parts":[{"el":"text","text":"{value}","role":"stat","size":66,"color":"accent","anim":"typewriter","x":0,"y":0,"w":280},{"el":"text","text":"{body}","role":"fine","weight":"semibold","color":"ink","alpha":0.8,"x":0,"y":80,"w":280}]},
  {"el":"button","item":0,"style":"pill","color":"primaryDeep","labelColor":"surfaceLight","icon":false,"x":72,"y":630,"w":330,"h":66}]}

## Slide kinds — what each must contain

- cover (slide 1): the hero headline (the builder draws the viewer's
  greeting above it), an eyebrow, the static "Click anywhere to Begin" pill
  with room under the headline, the logo top-centre when the embargo allows.
  No wired items.
- content: headline (+ eyebrow / support / body / repeat / graphic) on a
  designed field. Auto-advancing content carries NO buttons — its items are
  display (a `repeat` of display cells, `wired:false` if the slide waits).
- question: the question as headline, one button or wired cell per answer,
  answers equal. `trackAs` names the choice for the sender.
- menu / hamburger: an eyebrow + a real headline ("Where do you want to
  start?" — never the slide's name), then one button or wired cell per item,
  in the order given. A Finish Up / Move Ahead button may be featured.
- answer: a response written to fit every option routed to it.
- cta (terminal): a short thank-you headline (28-70 chars), one button per
  action (verb phrase ≤22 chars, `finish:true` + `finishTitle` — see below),
  the sender block, footage or a brand-colour field. The last thing seen.
- Logo Reveal: the reveal composition only (logo large and centred, the brand
  line under it) — the builder adds the Rub Your Screen beat in front of it.

## Icons

Icons: any icon from the LUCIDE set (kebab-case names, fetched live) — pick
well-known names ("hammer", "wrench", "bath", "chef-hat", "shield-check",
"sparkles", "users", "calendar-check", "piggy-bank", "ruler", "truck",
"heart-handshake"…). Match each icon tightly to its label; every item on a
slide gets a DISTINCT icon. An unknown name renders as an empty slot, so
avoid obscure or invented names. Legacy names (flame, waves, storm,
heartbeat, star, shield, home, people, calendar, document, check, arrow,
phone, mail, chart, gear, target, lightbulb, chat, heart) also resolve, and
so do the plain nouns the animated library uses where Lucide spells them
differently (microphone → mic, robot → bot, envelope → mail, hand-pointer →
pointer, bar-chart / pie-chart, cart, cog, warning, location, money…) — write
the plain noun; the builder maps the glyph.

ANIMATED ICONS: every icon name is also looked up in the builder's animated
library — the whole Lordicon "wired" collection (3,700 icons, one consistent
hand) — and, when the library has an icon whose NAME is that word, the slide
gets a looping animation instead of the static glyph. The match is exact,
never fuzzy, so plain, common nouns for the thing itself animate and
compound or invented names do not: "calendar", "clock", "wrench", "wallet",
"camera", "truck", "lock", "search", "heart", "star", "trophy", "rocket",
"handshake", "megaphone", "lightbulb", "leaf", "globe", "home", "gift",
"microphone", "shield", "gear", "bar-chart", "pie-chart", "map", "phone",
"smartphone", "laptop", "envelope", "key", "book", "coffee", "briefcase",
"puzzle", "target", "bell", "flag", "cart", "tag", "medal", "plane", "car",
"house", "tree", "sun", "cloud", "fire", "water", "brain", "eye", "user",
"users", "team", "family", "chat", "headphones", "glasses", "umbrella",
"anchor", "compass", "crown", "diamond", "robot", "flask", "stethoscope",
"dumbbell", "bicycle", "cake", "turtle" — where such a name fits the meaning
as well as any other, prefer it. A
refined Lucide name still resolves through its base word ("calendar-check"
→ the calendar animation, "shield-check" → the shield, "shopping-bag" → the
bag), and common Lucide spellings are aliased ("users", "refresh-cw",
"chevrons-down", "trending-up" …); a two-noun name ("heart-handshake",
"chef-hat") animates only when the library happens to name an icon that way.
A name outside the library still renders — as the static glyph. Every
animated icon — standalone, above a line of text, or in a BUTTON or pill —
is placed as the library's 2-tone wired outline drawing, painted in the
brand colours; nothing to choose.
A ROW OF ICONS IS A SET (2.1.4, deck 307: a menu drew a book, a ringed
compass, a bar chart, a speech bubble, a planet and a checkered goal sign —
"I'm not sure what the intent was"). The builder now keeps a set in one
family — it prefers the 2-tone drawing for every member, takes the item
NAMED for the word over one that merely lists it as a synonym, and answers a
noun nothing is named for with a specialisation of it (a bar chart for
"chart") — but it cannot rescue a set of six unrelated nouns. Choose icons
that belong together: the same KIND of thing (all objects, or all symbols),
plain and common, and no more than one per row of copy. (The flat and 1-tone system drawings
exist and a client can ask for them in an edit turn.) Buttons: the arrows
the builder adds ("arrow-right", "arrow-left", "check") are library icons
too.

THREE CONCEPTS PER ICON. `icon` is your first choice; `iconAlts` names TWO
MORE things that could stand for the same line — different objects, each a
fair reading of the label on its own, never spellings or refinements of the
first ("Designed in-house": icon "pencil-ruler", iconAlts ["house",
"glasses"]; "Thirty days": icon "calendar", iconAlts ["truck", "watch"];
"A pair given": "heart-handshake", ["gift", "heart"] — NOT "calendar",
"calendar-days", "calendar-check"). Order them by fit. The builder looks all
three up in the animated library and takes the first it has; and when two
items on one slide would land on the same animation (a "Five days" and a
"Thirty days" both drawn as a calendar), the second takes an alternate — so
a list stays varied even when its labels rhyme. Any of the three may be the
one shown, so none may be a stretch. Give every icon its alternates,
including a `copy.icon` accent and the icons on buttons. In the plan they
are written as "iconAlts":["<2nd concept>","<3rd concept>"] beside "icon".

## Copy rules (binding)

COPY THAT READS AS WRITTEN (2.2.2, graded on two live decks): the eyebrow
frames the headline — never the section name ("FINISH UP"), never filler
("CHAPTER"), never a word the headline already uses; questions carry ONE
numbering scheme ("QUESTION 2 OF 3", digits); a figure is said once on its
slide — the label under a numeral never restates it ("$100 / of $100" is a
stutter) and the headline does not repeat the numeral; an answer slide says
"Correct"/"Not quite" once, in the eyebrow, then the fact; quiz options
carry no decorative icons (menu rows keep theirs); the first and return
menus never share a headline.

SAY IT IN FEWER WORDS. On-screen copy is read in a glance on a phone, over
narration that already says the sentence — write the SHORT form and stop.
No "Our", "We provide", "Learn about", no restating the chapter title, no
trailing punctuation on labels or eyebrows. A menu whose options read
"Unified Dashboard", "Cash-Flow Forecasting", "Real Results" is right.

SAY IT ONCE. Slots divide the message; they never repeat it. The headline
says what a figure means without its digits when a numeral or a graphic
carries the digits; a stat trio is three DIFFERENT figures and is fine;
the same figure twice on one slide is not. Items never restate the
headline. Section intros share one plain eyebrow label with no ordinals
(the viewer picks the order).

A `QUOTE:` line on screen is a quote: the sentence in copy.quote, the name in
copy.attribution, drawn as a longline text element with the attribution in a
fine line under it.

Personalization tokens ([viewer-name-first], [sender-name], [sender-name-first],
[sender-name-last], [sender-email], [sender-scheduling-url]) appear AT MOST
ONCE per slide across all copy, stay verbatim, and NEVER appear in narration
(spoken greetings stay generic). The cover's greeting line is drawn by the
builder, so the cover's eyebrow and headline must not repeat
`[viewer-name-first]`; the sender block already carries
[sender-name]/[sender-email], so nothing else on that slide does.

A NAMED PERSON IS NEVER A STOCK FACE. A founder, an owner, a customer giving
a quote: no portrait hint, no headshot still — illustrate the role or the
place, or use an icon. Real faces come only from files the client uploaded.

THE LAST BUTTON OF THE DECK IS A FINISH, NOT A LINK. On a terminal CTA every
item that ends the presentation carries `"finish": true` and a `finishTitle`
(the outcome name the sender sees: "BOOK A CALL"); a scheduling destination
is ALWAYS the literal `[sender-scheduling-url]`.

TRACK CHOICE. The builder records what viewers pick: on a question, each
answer as "<trackAs> - <answer label>"; on a menu, "Topic Viewed - <topic>";
on a terminal CTA the finish outcome. Set `trackAs` on every question (2-4
words the sender will recognise). Override per item with
`"track": {"varName": "...", "value": "..."}` only when the default reads
wrong; `"track": false` switches it off for an item.

Items on an auto-advancing content slide are DISPLAY (no target/url).
Only a sub-fork slide that does NOT auto-advance carries its single Back item.

## Chapter-close + subfork rules (binding)

- A topic chapter's FINAL slide is a plain content slide: no navigation
  language, no buttons — it auto-advances to Main Menu - Return.
- Subfork sub-item slides hold the item's content plus ONE Back button
  (target: the chapter's first slide) — nothing else.

## Logo-reveal embargo (binding)

A slide carrying `brandEmbargo: true` sits BEFORE the Logo Reveal: no
company/product name in narration or on screen, no logo element. From the
reveal onward the name and the mark are free.

Photo hints: specific, matching the deck casting, UNIQUE across the deck
(never repeat a query from the used-list or from another slide in this batch).

Output ONE JSON object (no fences, no commentary):
  {"slides": [ <one complete plan per slide given, SAME ORDER, names verbatim> ]}
(If given a single slide, a single bare plan object is also accepted.)

## EVERY WORD ON SCREEN IS LANGUAGE A VIEWER READS AS MEANT (2.2.0)

An eyebrow, a tag, a chip, a row label, a button — all of it is copy a person
reads. The test before you write any of it: **could a viewer mistake this for
something a developer forgot to replace?** If yes it is wrong, however neatly
it fits the brand's world.

Never, in ANY display role: an API-key shape (`sk_test_…`, `sk_live_…`,
`pk_…`), a slug (`menu`, `main_menu`, `start_here`), a template variable
(`{{name}}`, `<BRAND>`, `[PLACEHOLDER]`), `lorem`, `TODO`, `TBD`, `XXX`, or
the slide's internal name.

The ONE legitimate use of a machine-shaped token is as the **value in a
labelled pair** — a row reading `Secret key · sk_test_51H…` is showing the
viewer a specimen, and the label is what makes it legible. That is content.
The same token standing alone where a section's name belongs is not.

This rule exists because of a real build. A deck for a payments brand minted
itself a device — *"key chip: a small pill holding a mono-styled tag
(`sk_test_` style)"* — and the design pass obeyed it, filling five eyebrows
with `sk_test_menu`, `sk_test_billing`, `sk_test_subs`, `sk_test_ready` and
`sk_live_global`. Every one was deliberate. Every one read as junk left in the
file. The review pass even repaired the chip's geometry without once
questioning the words inside it.
