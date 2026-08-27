> **Reference for the iDecide Presentation Builder skill.** Verified platform
> behaviour and footguns — every entry was proven live before it was written
> down. Read the relevant section when something behaves unexpectedly.

# Verified platform facts (myiDecide / CE.SDK v1.74.1)



Every item below was proven live in the builder and re-verified after a full page
reload. These are the rules the extension's inject scripts must follow — several were
learned the hard way and silently corrupt a deck if ignored.

Last updated 2026-08-21.

> **Dual-environment rule (Bren, 2026-08-07).** This file and the project runbook
> (`WORKFLOW.md` / `AIAGENT_API.md` / `BUILD_NOTES.md` at the repo root) must BOTH be
> updated whenever a platform behaviour, standing preference, or build technique is
> learned or changed — and so must the code that enforces it (`composer.js` for wiring
> names and composition, `pipeline.js` for phase runners, `sh/*` for the helper toolkit).
> The extension and the agent runbook drive the same platform through different code
> paths, so a finding recorded in only one place is lost to the other. Prose alone is not
> enough.

---

## Batch video import — the footage wave (measured 2026-08-22, deck 179)

`api.assets.importPexelVideoBatch(videos, onCompleted)` and
`api.currentSlide.videos.insertPexelBatchVideo(...)`, plus
`api.assets.searchVideos(query, page)`.

**Why it changes the build shape.** The batch is a SERVER-side job: it is not
page-bound, creates no blocks, and works whatever slide is open. So the whole
deck's footage can be in flight while slides compose, narrate and wire — the
same thing the narration wave already does. Footage was the last stage still
shaped like the old per-slide narration, and it was the slowest thing in a build.

| measurement | result |
|---|---|
| 8 searches, in parallel | **0.69s** |
| 8-clip batch: first clip usable | **4.8s** |
| 8-clip batch: all done | **19.1s** |
| 10-clip batch (6 slides + a 4-panel grid) | **22.3s**, 10/10, all unique |
| a bad Pexels id | fails in **~200ms**, `success:false`, no `meta`, batch unaffected |

The old path resolves ONE clip in about that time, sequentially, holding the
page and the engine's single upload slot while it does it.

**`searchVideos` replaces two calls.** It returns
`{ page, perPage, totalResults, videos:[{ pexelVideoId, url, width, height,
duration, preview }] }` — width/height/duration included, so it stands in for
both `e.asset.findAssets('pexelVideos')` AND the separate
`/api/builder/pexels/videos/search` proxy hit we used to make purely to rank.

**`ProcessedVideoResponse`** = `{ id, label, success, pexelVideoId, slideId,
meta: { uri, thumbUri, posterUri, sourceSet[{uri,width,height}], width, height,
duration, mimeType } }`. `slideId` is echoed straight back from the request so a
result can be matched to its slide. `meta.uri` is a plain `.mp4` — verified live
that applying it to a block WE created renders identically to
`insertPexelBatchVideo`, so the composer keeps its own slots, groups and marks
and only the sourcing engine changed. `setSourceSet(fill,'fill/video/sourceSet',
meta.sourceSet)` works on our blocks too.

### The resolution ceiling — 960×540, on BOTH paths

Measured by loading the **delivered mp4** into a `<video>` element and reading
`videoWidth`/`videoHeight` — not by trusting any API's own claim:

| path | delivered file |
|---|---|
| `api.assets.importPexelVideoBatch` (new) | **960×540** |
| `e.asset.apply('pexelVideos', …)` (what we shipped before) | **960×540** |
| eight different 3840×2160 sources, batch | all **960×540** |
| one 1920×1080 source, batch | **426×240** (two renditions, not four) |

So the batch is **not** a quality regression — the two paths are identical — but
960×540 is the platform's ceiling either way. **Every clip in every deck built
so far has been 960×540**, upscaled ~1.62× onto the 1558×720 stage. Nothing had
ever measured the delivered file, so it was invisible.

Consequences:

- **A 720px SHORT side is unreachable.** The short axis is always 540.
  Bren's rule (2026-08-22) is *"no video's width or height smaller than 720px"*;
  the code states it that way, reports the shortfall once with the real numbers,
  and enforces the achievable half — the long side — because rejecting on the
  short side would ship decks with no footage at all. The moment processed
  output reaches 1280×720, the rule starts biting correctly with no code change.
- **Source resolution barely matters; ORIENTATION does.** A portrait clip
  becomes 540×960 and looks dreadful cropped to a landscape stage, so the
  ranker weights orientation above everything else. (The old path's `minDim:
  720` filter ran against SOURCE dimensions and bought nothing, since
  everything is normalised afterwards regardless.)

> **For the platform team.** Raising the processing ceiling to 1280×720 or above
> is the single change that would sharpen full-bleed footage on every deck.
> Separately, a 1920×1080 source processing to 426×240 with only two renditions
> looks like a source-file selection bug rather than intended behaviour.

**2. Output quality cannot be predicted from the search metadata.** A
**1920×1080 source processed to 426×240** with only two renditions, while eight
4K sources processed to 960×540 with four. The server picks whichever source
file it picks. So the floor is checked AFTER processing and anything under
720px on the long side is re-requested with the next candidate (bounded to one
retry; a soft clip still beats no clip). Verified: two 426×240 results were
detected and re-sourced to 960×540, retry included, in 10.9s.

**3. There is no server-side dedupe.** Re-importing the same Pexels id produces
a NEW upload every time. Our own uri map is the only thing stopping a rebuild
paying twice.

## The Iconify API — measured, not assumed (2026-08-22, api v3)

Endpoint `/{prefix}/{name}.svg`. Three of its failure modes return **HTTP 200
with a plausible SVG**, so a status check proves nothing.

| Query | Result |
|---|---|
| `?color=%230989CF` | `stroke="#0989CF"` — substituted verbatim |
| `?color=nope` | **200**, `stroke="nope"` — no validation whatsoever |
| `?color=%23GGG` | **200**, `stroke="#GGG"` — invalid paint, renders black or not at all |
| `?color=#0989CF` (unescaped `#`) | **200**, `stroke="currentColor"` **and** `width="1em"` — the `#` starts a URL fragment and discards the whole rest of the query |
| `?color=` on a palette icon set | **200**, ignored entirely — `fluent-emoji-flat` keeps its six hard-coded fills |
| no `color` | `stroke="currentColor"`, `width="1em"` |
| bad icon name | clean **404**, body `Not found` |
| `?box=1` | adds `<rect x="0" y="0" width="24" height="24" fill="rgba(255,255,255,0)"/>` |

**Consequences for this build:**

1. **Percent-encode the `#`.** `encodeURIComponent` does it. The failure is
   silent and takes the sizing with it.
2. **Refuse a colour you cannot parse — never substitute a default.** The API
   will happily bake `stroke="nope"` into a file we then upload and cache under
   the colour the client asked for.
3. **Assert the colour is in the returned file.** A 200 is not evidence.
4. **`color` needs a MONOTONE set.** Lucide draws with `currentColor`, so it
   works. Changing `ICON_SET` to a palette set would silently produce
   stock-coloured glyphs stored under colour keys.
5. **Send `box=1`.** Lucide's glyphs do not fill their 24×24 viewBox, and they
   do not fill it by *different amounts*:

   | glyph | ink | fills |
   |---|---|---|
   | `paw-print`, `phone`, `star`, `heart` | 20×20 | 83% |
   | `calendar` | 18×19 | 79% |
   | `check` | 16×11 | 67% |
   | `arrow-right` | 14×14 | 58% |

   A rasterizer that crops to ink bounds — which is what the Iconify docs say
   design tools do, and what the "glyph sits 12px above and 4px below its well"
   report looks like — scales each glyph up to fill its block. `arrow-right`
   then renders **44% larger than `paw-print`** in the same-sized block. The
   transparent box makes every glyph draw at its true proportion, so a row of
   icons is finally one visual weight.

   Expect icons to look ~17% smaller on average after this, and the
   worst-cropped ones (`arrow-right`, `check`) noticeably more. That is the
   inconsistency being removed, not a regression — but if the new size reads
   small, the fix is to raise the icon sizes ~1.2×, not to drop `box=1`.

## Mutating REST calls set their own `X-Session-UUID`

The header is required (wrong or absent → 400/409). It used to be supplied ONLY
by capture.js's network hook — which is absent on any page that was open when
the extension was last updated, i.e. **every page, every time a new build is
installed**. `sh/rest.js` `_sessionHeaders()` and `media.js` `uploadAsset` now
set it themselves from `IDP._sessionRec.sessionUuid` / `window.__IDECIDE_SESSION_UUID`,
both published by `IDP.init`. The hook remains, for the trace only.

## `slideData` (UBQ1) is UTF-8 — `btoa` is not

`btoa(JSON.stringify(...))` throws `InvalidCharacterError` on any character
above U+00FF, and the house style recommends em dashes and ellipses; `atob` on
the way back gives latin-1 mojibake for the same text. Use `_b64enc` / `_b64dec`
in `sh/rest.js`, which round-trip through `TextEncoder`/`TextDecoder`.

## Long operations expose their own "settled" flag

`IDP.status()` reports `buildFinished`, `removeFinished`, `dupFinished`,
`retimeFinished`, `realignFinished`, `verifyFinished`. Polling the shared phase
counters (`phase === 'shells' && done >= total`) is wrong for anything but the
build: those counters are already satisfied when the op starts, so the poll
passes on its first tick and the caller reads a result that has not been written
— which is how "REMOVED nothing" was reported over slides that were gone.

## 1. Final CTA buttons — "Finish Presentation" outcomes

**Standing rule (Bren).** A terminal call-to-action button is ALWAYS a Finish
Presentation action, never a plain `openLink`. Payload:

```json
{ "clickActionType": "finishPresentation", "clickActionTarget": "",
  "clickActionData": {
    "store": false, "value": "", "varName": "", "valueType": "text",
    "finalOutcomeTitle": "<CHOICE IN ALL CAPS>",
    "finalOutcomeNotification": "Hey [viewer-name-first], [sender-name] just finished watching your presentation and chose to <CHOICE IN ALL CAPS>.",
    "finalAction": { "type": "openLink", "url": "<destination>", "openInNewTab": true }
  }}
```

* Final action ALWAYS opens in a new tab.
* **Every "Schedule a Call" destination is the shortcode `[sender-scheduling-url]`** —
  never a hard-coded booking link, and never left unwired waiting for a URL.

Panel field → JSON: Track Choice → `store`; Final Outcome Title → `finalOutcomeTitle`;
Final Outcome Notification → `finalOutcomeNotification`; Final Action + its URL and
"Open in new tab?" → the nested `finalAction`.

### Writing it
`blockActions.set(id, type, target)` takes only 3 args and CANNOT carry the data. Write
the payload into the live model instead:

```js
await A.currentSlide.blockActions.set(btn, 'finishPresentation', '');   // type + block-action-id
const ctx = A.provider.getCtx();
const rec = (ctx.getBlockActions() || []).find(r => r.slideId === sid);
rec.clickActionType = 'finishPresentation';
rec.clickActionTarget = '';
rec.clickActionData = { /* payload above */ };
ctx.markCustomDataDirty();
A.currentSlide.blocks.updateTiming(btn, st, du);   // ALSO dirty the slide
// …changeSlide away to commit
```

**`markCustomDataDirty()` alone is not enough** — on the first attempt the FIRST of
three slides silently dropped its outcome while the other two saved. Dirty the slide too.

In the extension this is the `btnfinish:<TITLE>|<url>` wiring name (composer emits it,
`pipeline.js → IDP.wire` consumes it).

### Verification footguns (these produce FALSE NEGATIVES)
* REST `GET /api/builder/builderSession/{sid}` → `blockActions` **does not return
  `clickActionData`** even when saved correctly. Never verify outcomes over REST.
* `ctx.getBlockActions()` only hydrates `clickActionData` for slides **loaded in the
  current session**. An unvisited slide reads `{}` — that is not data loss. Navigate to
  the slide first, then read.
* A session `PUT` with a modified `blockActions` array returns `200 "true"` and is
  silently ignored. Real dead end; use the live model.

---

## 2. Two-colour text runs — `updateText` destroys them

PPTX headlines with a highlighted phrase are one text block with multiple colour runs.

* `E.block.setTextFontSize(id, px)` **preserves** runs.
* `A.currentSlide.blocks.updateText(id, sameText)` **FLATTENS** every run to the first
  colour. This is the long-standing cause of two-tone headlines going mono after a font
  pass.

**Commit multi-run text with `updateTiming`, never `updateText`.** Range forms
`getTextColors(id, from, to)` / `setTextColor(id, colour, from, to)` DO work on imported
multi-run blocks (they only appear to ignore ranges on a block freshly made with
`createTextBox`, which has a single run). Pass: read distinct colours/sizes → per-char
scan only if mixed → resize per size-run → re-apply each colour run → commit with
`updateTiming`.

---

## 3. Animation easing — `EaseOutQuint`, never bare `EaseOut`

The engine accepts 16 easing values but the builder UI only has labels for some. Setting
`EaseOut` renders the raw i18n key `property.animationEasing.EaseOut` in the selector and
reads as broken. Use **`EaseOutQuint`** for "smooth decelerate" (`EaseInOutQuint` for a
symmetrical ease). Default on a freshly-set animation is `Linear` — always set it.
`typewriter_text`, `spread_text` and `block_swipe_text` have NO `animationEasing`
property; catch and skip.

## 4. Ken Burns is its own type, not `pan`

Not in `ANIM_TYPES`. Build via the engine:

```js
const an = E.block.createAnimation('//ly.img.ubq/animation/ken_burns');
E.block.setEnum (an, 'animation/ken_burns/direction', 'Right');   // Up|Right|Down|Left
E.block.setBool (an, 'animation/ken_burns/fade', true);
E.block.setFloat(an, 'animation/ken_burns/travelDistanceRatio', 1);
E.block.setFloat(an, 'animation/ken_burns/zoomIntensity', 0.5);
E.block.setEnum (an, 'animationEasing', 'EaseOutQuint');
E.block.setDuration(an, 2.4);
E.block.setInAnimation(target, an);
```

## 5. Narration audio is inserted TRIMMED

`generateAndInsert` leaves the block ~50–80 ms short of the real footage, with a matching
`trimLength`. Always untrim:

```js
const full = E.block.getAVResourceTotalDuration(a);   // throws until the audio loads — poll it
E.block.setDouble(a, 'playback/trimOffset', 0);
E.block.setDouble(a, 'playback/trimLength', full);    // setDouble, NOT setFloat
E.block.setMetadata(a, 'idecide/playback/trimLength', String(full));
A.currentSlide.blocks.updateTiming(a, 0, full);
```
Then extend the page to `max(pageDur, full + 0.3)` and push every visual layer to the new
last frame. Audio always sits at z0.

## 6. Blocks can nest inside `/track` blocks

Once a slide is touched in the timeline UI its layers may live inside
`//ly.img.ubq/track` children of the page, so a flat `getChildren(page)` returns only
tracks and the slide looks empty. Structure varies slide to slide within one deck —
always recurse:

```js
const allBlocks = root => { const out = []; const walk = id => {
  for (const c of E.block.getChildren(id))
    (E.block.getType(c) || '').includes('/track') ? walk(c) : out.push(c);
}; walk(root); return out; };
```

## 7. Video: `insertPexelVideo` is Pexels-URL-only

Passing a non-Pexels URL (e.g. an r2 upload) **hard-freezes the renderer** — every later
call times out and only a browser reload recovers. For already-hosted media, swap the
image block's FILL instead (also a cleaner 1:1 swap: geometry, z-order, timing and
animation are preserved because the block never changes):

```js
const f = E.block.createFill('video');
E.block.setString(f, 'fill/video/fileURI', uri);
E.block.setFill(target, f);
E.block.setContentFillMode(target, 'Cover');
A.currentSlide.blocks.updateTiming(target, st, du);   // commit
```
`api.assets.searchVideos(query, page)` returns `duration`; `engine.asset.findAssets`
does not. Pick clips longer than the slide.

## 8. Heavy decks load slowly

A 33 MB PPTX import leaves `api.currentSlide.get()` null for a long time and
`changeSlide` can take tens of seconds. Long awaits inside one injected call hit the 45 s
CDP timeout and eventually wedge the renderer. Run multi-slide passes as detached
promises writing progress to a `window.__X` global and poll with short calls. Slides with
4+ media inserts can wedge the FOLLOWING `changeSlide` — pause between inserts.

**Platform-outage signature** (not a deck problem): all REST endpoints return 200 but the
editor never creates a scene, `currentSlide.get()` stays null, no canvas element, and the
only console line is `[UBQ] No scene found after editor creation`. Confirm by loading a
known-good session; if it stalls too, halt and wait.

## 9. Verifying media swaps across a save boundary

Never verify by filename. On save, the platform **renames uploaded video URIs to
canonical GUID filenames** — the URI returned by the uploads POST will not match the
fill's `fill/video/fileURI` after a reload. Additionally, `GET .../builderSession/{sid}`
slide content is encoded bytes (not greppable) and `GET .../uploads` returns **no video
entries at all** (images only). The only reliable verification: reload the editor
(committed work survives — that IS the test), navigate to the slide, and read the fill
live with `engine.block.getString(fill, 'fill/video/fileURI')`, or visually confirm the
canvas. A reload also wipes all `window.__*` progress globals — persist any apply log
(target → uploaded file) before reloading.


## 10. Video fills can degrade to color fills after upload-processing failure

A committed video fill can come back as a plain COLOR fill on the next editor load if
the uploaded file failed server-side processing (silent; no error). After any batch of
video swaps: reload, then count video fills per touched slide against the expected
number. Repair by re-uploading (a smaller variant succeeds where a larger one failed)
and re-running the fill swap on the degraded block, then verify with another reload.

## 11. Animation direction — semantics verified against img.ly v1.79 docs + live scrub

**`slide` stores TRAVEL direction as float radians, entering from the opposite side:**
0 = slides RIGHT (enters from left), π/2 = down, π = slides LEFT (enters from right),
3π/2 = up. (The img.ly types/edit pages describe these values as entry edges — that is
wrong; the create/base page and live behavior agree on travel-direction semantics.)
`getEnum` throws on it and a getter cascade silently misreports it via `getBool → false`.

**Enum direction types** (`wipe`, `block_swipe_text`, `baseline`, `ken_burns`):
values name the travel/sweep direction ([Up, Right, Down, Left]); 'Left' sweeps
right→left. Never read enum directions via getFloat (returns the index, unreliably).

Read pattern: match the property with `/\/direction$/`, try `getEnum`, fall back to
`getFloat`. Change IN PLACE (`setEnum`/`setFloat` on the EXISTING animation) + commit via
`updateTiming` + changeSlide. Never re-apply a preset or call `setAnimation` — both
destroy/recreate the animation, reset its settings, and can change its TYPE.

Discovery: `findAllProperties(anim)`; `getEnumValues('animationEasing')` exposes this
build's extended easing set (incl. EaseOutQuint). "No animation" sentinel on this build
is 4294967295 (docs say 0) — treat both as none.

Verify visually: pause + `setPlaybackTime(page, start + dur*0.15)` and screenshot two
nearby frames; never verify from property names or a single doc page.

**Standing design rule (Bren): animate from the closest edge** — left-half elements
travel rightward (slide 0 rad / enum 'Right'), right-half elements travel leftward
(slide π rad / enum 'Left'); vertical baseline entries exempt.


## 12. Complete animation type matrix (24 types, empirically enumerated)

`createAnimation()` accepts exactly 24 keys; the ANIM_TYPES map lists only 9 — pass key
strings directly. Slots are strict (In/Out types rejected as loops and vice versa).

- **In/Out on any block (13):** slide (direction float radians = travel dir, fade bool),
  fade, blur (fade, intensity), grow (direction enum Horizontal/Vertical/All/4 corners,
  scaleFactor float), zoom (fade), pop (no easing), wipe (direction enum U/R/D/L),
  pan (direction float radians, distance float, fade), baseline (direction U/R/D/L),
  spin (direction Clockwise/CounterClockwise, fade, intensity), ken_burns (direction
  U/R/D/L, fade, travelDistanceRatio float, zoomIntensity float), crop_zoom (fade,
  scale float), typewriter_text (writingStyle Character/Word only).
- **In/Out TEXT-ONLY (3):** block_swipe_text (direction U/R/D/L, blockColor, useTextColor;
  no easing), spread_text (fade, intensity), merge_text (direction Right/Left, intensity).
- **Loop slot only (8):** breathing_loop / pulsating_loop / blur_loop / sway_loop
  (intensity float), spin_loop (Clockwise/CounterClockwise), jump_loop (direction U/R/D/L
  + intensity), fade_loop, squeeze_loop. No easing or text props on loops.
- **Shared:** animationEasing enum — 16 options on this build (Linear, EaseIn/Out/InOut
  + Quart/Quint/Back/Spring trios); absent on pop, the *_text swipe/typewriter types and
  loops. textAnimationWritingStyle (Block/Line/Character/Word) + textAnimationOverlap
  (0–1, default 0.35) on most In/Out types, text blocks only.
- **Typing footgun:** getBool coerces on float props (grow/scaleFactor, pan/distance,
  ken_burns/zoomIntensity, crop_zoom/scale are floats). Confirm property types with a
  setFloat/setEnum round-trip on a throwaway instance; enum options via
  getEnumValues(propertyPath).

## 13. Track Choice = the same clickActionData envelope

Track Choice settings live in the button's blockAction record: `store` (toggle),
`varName` (Choice Name), `valueType` ("text"), `value` (Choice Value) inside
`clickActionData`, alongside `clickActionType: "goToSlide"` + `clickActionTarget`.
Off = empty `{}`. Same envelope as finishPresentation outcomes; same write path
(mutate ctx.getBlockActions() record + markCustomDataDirty + dirty slide + changeSlide)
and same verification footguns (REST GET omits it; unvisited slides read `{}`).
Convention: varName = "<Question label> - <Answer text>", value = verbatim button label.

Track Choice conventions: question buttons varName "<Question> - <Answer>"; menu topics
"Topic Viewed - <topic>"; final CTAs keep their finishPresentation envelope with
store:true + varName "Final Outcome - <TITLE>" / valueType "text" / value = title verbatim; never track plain navigation; NEVER set actions on slide 1 (any click there
starts the presentation). Match records to labels by y-order, never rect containment
(record coords normalize to a different space than canvas px).

Watch: blockAction records can vanish wholesale (both menus lost their 5 topic-tile
actions once) — after any customData commit, re-verify record COUNTS per interactive
slide. Menu-tile actions belong ON the tile block; its idecide/block-action-id metadata
links the engine block to its record GUID. The two Finish Up tile videos (22/23) have
degraded video→color twice — recount 6/6 fills there after every session.

---

## 14. Auto-advance takes an explicit TARGET — use it

`api.slides.setAutoAdvance(id, enabled, targetSlideId)` writes
`autoAdvanceAfterNarration` **and** `autoAdvanceSlideId` (stored as a string).
The third argument is the whole point of the call: without it the platform
falls through to deck order.

**Why it matters.** In a menu-driven deck, wiring the chain positionally
(`slides[i] → slides[i+1]`) is correct inside a chapter and wrong at every
chapter boundary — a viewer who picks a topic from the menu is carried into
the *next* chapter instead of back to `Main Menu - Return`, and never gets to
choose again. The deck plays as a linear video with a menu bolted on the front.

**The rule.** A chapter's LAST beat targets `Main Menu - Return` by id.
Everything else keeps its positional successor. Interactive beats — cover,
menus, questions, terminal CTAs, hamburger — don't auto-advance at all.

```js
// chapter close
await api.slides.setAutoAdvance(ids['Coverage - 4'], true, ids['Main Menu - Return']);
// inside a chapter
await api.slides.setAutoAdvance(ids['Coverage - 3'], true, ids['Coverage - 4']);
```

**Survives re-PUTs.** `SH.putSlideField` does a GET → merge → full PUT, and the
patch it sends (`name`, `autoAdvance`, `autoAdvanceAfterNarration`) doesn't
include `autoAdvanceSlideId`, so a later batch re-PUT preserves the target.
Order still matters: set the targets AFTER the shells exist, and never send a
hand-built record that omits the field.

**Verifying.** The REST slide list carries `autoAdvanceSlideId` — read it back
and confirm every auto-advancing slide points at an id that exists in the deck.
`SH.gradeDeck()` does this as rule R15.

**Footgun.** `setAutoAdvance` rejects the first slide ("Cannot add auto-advance
to the first slide"). That's fine — slide 1 is the cover and waits for a tap.

## Serialized slideData: fills are sibling entities (verified 2026-08-17)
In decoded slideData (`UBQ1` + base64 JSON), a graphic element's `fill`
property is null. Fills exist as their OWN entries in `designElements`
(`//ly.img.ubq/fill/image`, `//ly.img.ubq/fill/video`,
`//ly.img.ubq/fill/color`, `//ly.img.ubq/fill/gradient/linear`). Any
static analysis of imagery must scan element ids, never `graphic.fill`.

## contentFillMode: the engine RESETS it to 'Crop' on video load (verified 2026-08-18)

Setting `'Cover'` on a graphic whose video resource has NOT finished loading
succeeds (never throws, reads back Cover) — but when the AV resource load
completes, the engine RE-INITIALIZES the block's contentFillMode to `'Crop'`
with identity crop values (= visual STRETCH on a 654×720 panel holding 16:9
footage). Proof (Nova Trail, 44 slides): every slide whose only visit
composed against a just-uploaded clip persisted mode 0/'Crop'; every slide
that got a SECOND (refine) visit — resource already cached, no load event —
persisted 1/'Cover'. The old Phase-0.5 pre-resolution flow never hit this
because every URI was already loaded before any compose.

**Rule: re-assert `setContentFillMode(id,'Cover')` LATE in the visit (after
the fill has been rendering for seconds), immediately before the commit.**
buildNav's cover-guard does this deterministically (two passes, 1s apart)
and logs `cover-guard (<slide>): re-asserted Cover on …` when it corrects.

**Bonus footgun:** `adjustCropToFillFrame(id)` REQUIRES a `minScaleRatio`
argument — the one-arg form always throws ("Expected a number, but received:
undefined"). Call `adjustCropToFillFrame(id, 1)`.

### UPDATE 2026-08-19: the getter LIES in-visit — verify the serialized bytes

The v0.9.2 in-visit cover-guard read back 'Cover' on every slide (zero
corrections logged) yet 28/45 slides persisted mode 0/'Crop'. So the
load-reset is NOT observable through `getContentFillMode` during the first
visit — the engine echoes the set value while the serializer writes 'Crop'.
The ONLY reliable check is the persisted slideData AFTER the commit
(`"block_content_fill_mode": 0` = Crop). Enforcement that works:
1. PRE-WARM: `forceLoadAVResource` on a throwaway video fill with the clip's
   URI BEFORE compose applies it (~0.7s on a CDN mp4; race a timeout). Cover
   set on a cached resource latches. (The one observed forceLoad freeze was
   a malformed cache-busted URI — valid URIs return promptly.)
2. POST-COMMIT VERIFY: read the slide's serialized bytes; any mode-0 fill →
   ONE corrective revisit (resource now cached → Cover latches; proved on
   2×44-slide decks by the refine-visit correlation).

## The CE.SDK fatal "Unknown Error" dialog lives in a SHADOW ROOT (2026-08-19)

The entire editor UI — including the blocking "Unknown Error / Please try to
reload the page" alertdialog — renders inside an OPEN shadow root on
`div#root-shadow`. `document.querySelector('[role="alertdialog"]')` returns
NULL while the dialog is up; query `document.querySelector('#root-shadow')
.shadowRoot` instead. CSS-module class hashes are unstable — match only
role="alertdialog" + aria-label/h4 text ("Unknown Error"). Also:
position:fixed modals have offsetParent === null, so test visibility with
getClientRects().length, never offsetParent. When the dialog is up the
engine is DEAD and every aiagent/engine promise hangs forever — the only
recovery is a page reload, then re-inject + restore + retry (the panel's
pageProbeFn + withRecovery path; probe cadence ~2.5s). Full signature +
snippets: assets/docs/builder-error-dialog.md.

## Canvas backgrounds: the page fill, and it CAN be a gradient (2026-08-19)

The slide's canvas background is the PAGE block's fill — an ordinary
engine fill, so everything that works on a block's fill works here.

**Read** (verified live on slide 35638 while a gradient was applied):
```js
const pg = engine.scene.getCurrentPage();
const f  = engine.block.getFill(pg);
engine.block.getType(f);   // //ly.img.ubq/fill/color  |  /fill/gradient/linear|radial|conical
engine.block.getGradientColorStops(f, 'fill/gradient/colors');
//   → [{color:{r,g,b,a}, stop:0}, …]   stop is 0–1
engine.block.getFloat(f, 'fill/gradient/linear/startPointX');  // + startPointY/endPointX/endPointY
//   radial: centerPointX/centerPointY/radius · conical: centerPointX/centerPointY
```
`engine.block.findAllProperties(fill)` enumerates the exact property set
for whichever gradient kind is applied.

**Write:**
```js
const f = engine.block.createFill('//ly.img.ubq/fill/gradient/linear');  // radial | conical also valid
engine.block.setGradientColorStops(f, 'fill/gradient/colors', [
  { color: {r:1,g:1,b:1,a:1}, stop: 0 }, { color: {r:.09,g:.21,b:.16,a:1}, stop: 1 }]);
engine.block.setFloat(f, 'fill/gradient/linear/startPointX', 0.5); // …Y 0 → …endPointY 1 = top→bottom
engine.block.setFill(pg, f);
```
Points are NORMALIZED (0–1) canvas coordinates, not pixels.

**Footguns.**
- `api.currentSlide.setBackgroundColor(hex)` is the documented setter and
  takes a FLAT hex only — it cannot express a gradient. Use it for solids
  (it marks the slide dirty for the app's own save); use the engine fill
  for gradients, then pair with a documented mutation before the commit.
- The page carries BOTH `backgroundColor`/`backgroundEnabled` properties
  and `block_fill`. The FILL wins visually (verified: page backgroundColor
  was opaque white while the visible canvas was the fill's #16352A).
- Page gradients DO survive the platform's own save: a live gradient
  serialized into slideData as `//ly.img.ubq/fill/gradient/linear` whose
  `block_render_connections` point at the page entity.
- Reading the canvas right at page-mount can return the pre-restore solid;
  read after the slide settles (gotoSlide already polls).

## Data graphics are generated locally — no chart API (2026-08-19)

charts.js draws ring / gauge / pie / donut / bars / hbars / line as plain
SVG strings and uploads them through `SH.uploadSvg` (the same verified path
Iconify icons use). Deliberately NOT a hosted chart service (QuickChart,
Image-Charts): each would add a host permission + a Chrome Web Store
privacy disclosure, send the client's figures to a third party, and add a
rate-limitable network dependency mid-build. Local generation costs nothing,
works offline, and takes the deck's palette exactly.

Surfaces: `window.__CHART.svg(spec)` / `.key(spec)`; `IDP.chartUri(spec)`
(uploads + caches per run in `IDP.chartUris`); composer places `plan.chart`;
revision op `addChart`. Chart SVGs are placed as image fills with
contentFillMode 'Contain' so they never distort.

## Batched narration: api.narration.generateBatch + blocks.addAudio (verified 2026-08-20)

`api.narration.generateBatch(narrations, onCompleted?) → Promise<CompletedNarration[]>`
generates a whole set of narrations in ONE server request over SSE —
ElevenLabs generation and R2 upload run together server-side, the pool is
kept saturated across concurrent users, and `onCompleted` fires per file.
Measured on alpha deck 87: 3 short clips in 4.1s total; 6 realistic
~30-word clips (72s of audio) in 6.1s total, arrivals 5.1-6.1s — fully
parallel. Compare generateAndInsert: ~8-20s per slide, serialized.

- Narration fields: `voiceId`, `slideId`, `narration`,
  `voiceSettingsOrStability: {stability, similarity, style} | number`
  (+ flat `similarity`/`style`), `startTime` (unused, parity only).
- CompletedNarration: `{success, slideId, uri, duration}`.
- Requests for an unknown slide, the intro slide, or empty script/voice
  are filtered CLIENT-side and return `success:false` (batch still resolves).
- generateBatch creates NO blocks. Insert with
  `api.currentSlide.blocks.addAudio(narration | url, durationOrStart?, start?)`
  → block id, or 0 on failure. Verified: block lands at start 0 and the
  page duration syncs to the audio duration; the z0 move + retime rules
  still apply afterwards.

FOOTGUNS: slides.get() returns [] until the scene hydrates (poll it, don't
trust the first read). A freshly-deployed bundle can expose the methods on
one load and not the next — feature-detect on every run, never cache
availability across reloads.

## Slide names: who writes where (verified on my.idecide.com, 2026-08-20)

Tested end-to-end on PROD (deck 172, scratch slide created and deleted):

- `api.slides.create(index, null, false)` creates the slide with an EMPTY
  name — the name must be written afterwards. (Passing a name as the first
  argument does nothing; that's how unnamed slides appear at all.)
- `api.slides.setName(id, name)` WORKS on prod: the app store AND the REST
  record both show the new name. It is the preferred writer.
- `PUT /api/builder/builderSession/{sid}/slides/{id}` with a full record
  updates the DATABASE only — the in-memory app store keeps the OLD name
  until a reload (observed: REST "prod-probe-REST" vs app store
  "prod-probe-A"). So a REST-only metadata write can be clobbered by a
  later app-store commit. ALWAYS pair a REST metadata write with the
  matching api.* setter.
- `DELETE /api/builder/builderSession/{sid}/slides/{id}` returns 200 and
  the slide disappears from the REST list immediately.
- An empty shell serializes to slideData of length 0, so a size threshold
  cleanly separates "empty duplicate shell" from "slide with real content"
  before any automated delete.


---

## Block metadata: `idecide/group` + `idecide/groupRole` (verified 2026-08-21, deck 175)

The composer writes a group mark on every part of a composite —
`idecide/group` (the unit's id, e.g. `button:1`) and `idecide/groupRole` (its
part, e.g. `background` / `label` / `icon`). Three facts, all verified live:

1. **It survives save, reload and revision.** A slide built hours earlier, after
   a full page reload, still reported `button:1` on its background, label and
   icon and `contact:2` on both sender lines.
2. **In serialized slideData it is an ARRAY, not a property.**
   `metadata: [{key:'idecide/group', value:'button:1'}, …]`. Reading
   `el['idecide/group']` returns `undefined` — an audit rule written that way is
   silently inert, which is exactly what happened to the first version of the
   slot-aware animation rules.
3. **`saveToString` output is base64.** Grepping the returned string for
   `idecide/` finds nothing even though the marks are there; decode first
   (`JSON.parse(atob(s.substring(4)))`).

The page itself additionally carries `idecide/layout` (`fixed` | `flow`) and
`idecide/balancedBy` (the composer revision that packed it) — a slide already
balanced by the current build is left alone, and a `fixed` layout is never
flow-packed.

**Why it matters:** grouping is DECLARED, not inferred. Any pass that moves,
resizes, retimes, animates or deletes a block must resolve its group first and
act on every sibling — that is the difference between moving a button and
moving a label off its button.

## api.slides.createMultiple — blank shells in one request

`createMultiple(count, atIndex, insertPosition)` -> `Promise<Array<{id,index}>|null>`

Verified on deck 179, 2026-08-23:

- `count` is documented as "must be > 1"; `count: 1` in fact succeeds, but the code
  still routes single slides to `create()` so nothing depends on undocumented behaviour.
- `atIndex` takes precedence over `insertPosition`. Pass `atIndex` for a positioned
  insert and `'end'` as the position when appending.
- The returned array has one entry per created slide, **in insertion order**, so
  `made[k]` maps to the k-th slide of the run. A short return makes the mapping
  unknowable - throw rather than guess.
- **The current slide is never changed.** No `changeSlide` commit is needed for the
  creation itself, and no navigation cost is incurred.

Measured: **223ms for 5 slides, versus 2935ms for 5 sequential `create()` calls - 13.2x.**
`putSlideField` costs 385ms/slide because it re-reads the whole slide list each time
(GET-all 222ms + PUT 163ms); with one snapshot hoisted out of the loop and
`putSlideRecord` writing the full record, that becomes a single GET plus 163ms/slide.

Modelled over a 49-slide build the shells phase goes from **47.6s to 8.4s**.

## Block actions: a deck-level store with its own REST endpoint (verified 2026-08-24, deck 179)

`api.provider.getCtx().getBlockActions()` returns the WHOLE deck's action
records: `{slideId, blockId, timeOffset, duration, x, y, width, height,
clickActionType, clickActionTarget, clickActionData, interactionData}`.

Facts, each verified live across a real reload:

- **The record is self-contained.** Geometry is normalized 0-1 canvas
  fractions and the PLAYER draws its clickable hotspot from the record's own
  box — not from the engine block. `blockId` is a row key, NOT an engine
  reference: the api's own `blockActions.set` writes a uuid that matches no
  block in the scene, and the server RE-MINTS it on save.
- **Persistence is per slide**, via
  `PUT /api/builder/builderSession/{sid}/slides/{slideId}/blockactions`
  with the record ARRAY as the body (X-Session-UUID required; the uuid comes
  from `GET /api/builder/builderSession/{sid}` → `sessionUuid`). A hand-pushed
  record in `getBlockActions()` + `markCustomDataDirty()` does NOT persist
  unless that slide itself is dirtied and committed — the REST PUT needs
  neither.
- **Consequence:** wiring can be written for ANY slide with zero navigation.
  Synthesize records from engine geometry (`getGlobalBoundingBoxXYWH / W,H`)
  and PUT the array. This is what the nav-free build does.

## Nav-free build (SHIPPED 0.9.184-185, REVERTED 0.9.186 — see the autopsy in BUILD_NOTES): the workbench-slide clobber rule

In nav-free mode every slide is assembled on the ONE loaded page and persisted
by `engine.block.saveToString([pageId])` → record PUT. The single hazard: any
app commit (ANY `changeSlide` away from the workbench) writes the workbench's
scratch content into the workbench SLIDE's record. Rules:

1. Never call `changeSlide` between the first visit and the end-of-build
   reload.
2. Re-PUT the workbench slide's authoritative record as the LAST act (the
   heal) — it also repairs any accidental mid-build commit.
3. The panel's reload after the build discards the scratch and hydrates every
   PUT slide.

Crash model: each slide's record PUT and blockactions PUT are durable the
moment they return — a crash loses at most the slide in progress, and the
resume ledger (builtSlides) skips everything already written.

### Why the nav-free build was reverted (YETI build, 2026-08-24)

The mechanisms above all hold — the INTEGRATION failed on two fronts:

1. **The app store is a second writer.** After the group-4 reload the app
   hydrates every slide; any REST PUT made after that point (the review-fix
   rebuilds) updates the SERVER while the app still holds the pre-fix bytes —
   and the verify pass's per-slide navigation then commits that stale state
   straight over the fresh PUTs. Every navigated pass (verify, edits,
   recoverCover, wire) is a potential clobber of every REST-only write. The
   navigated build never has this problem because the app store IS the truth
   the whole way through.
2. **Navigation was not the bottleneck.** Per-slide TTS (generateAndInsert,
   5-8s), footage waits, snapshot exports and verifyCover's REST reads
   dominate the ~10s per-slide pipeline; the changeSlide accounted for ~1-2s.
   Measured: 850s for the YETI build — no better than navigated.

Rule going forward: REST-only writes are safe ONLY when no navigated pass can
run after them before a reload, or when a navigated setter re-asserts them
(the shells pattern). The blockactions endpoint remains verified and useful
for one-shot wiring repairs.

## insertPexelBatchVideo — VERIFIED to persist, including synthesized records (2026-08-24, deck 179)

Bren asked for a live end-to-end api test after seeing engine fill-swaps in a
trace. Result, each step across a real reload:

- `api.assets.importPexelVideoBatch([{pexelVideoId, slideId}])` → 7.3s for one
  clip, record carries `meta.uri` + a 4-rendition `meta.sourceSet`
  (960/720/640/340).
- `api.currentSlide.videos.insertPexelBatchVideo(rec, x, y, w, h, 0, null)` →
  **8ms**, returns the blockId, builds a graphic with a video fill whose
  `fill/video/fileURI` is EMPTY BY DESIGN — playback rides the sourceSet.
- **The block persists**: changeSlide commit + `location.reload()` → the block
  is back at its exact geometry with all 4 renditions.
- **Synthesized records work**: `{id, label, meta: {uri, sourceSet}}` built
  from OUR stored fields (no Pexels provenance) inserts and fills identically —
  so clip REUSE can go through the api, not just fresh imports.

Correction to the earlier note: the renderer-freeze footgun belongs to
`insertPexelVideo` (the raw-URL variant) only. `insertPexelBatchVideo` is the
sanctioned placement call for uploaded records and is now the FIRST-CHOICE
path wherever a NEW video block is created (edit ops `sourceVideo` and
`addMedia`-reuse). What remains engine-only, correctly: RETARGETING an
EXISTING block's fill (the compose placeholder swap) — no api call does that;
the missing platform call would be `videos.replaceWithBatchVideo(blockId, rec)`.

## Rub Your Screen — the custom interaction block (verified 2026-08-24, deck 179)

Reverse-engineered from Bren's hand-built reference (slide 38541), replicated
from scratch, survived reload with the EDITOR RENDERING IT NATIVELY (white
hand + text + cover image + timeline clip). Anatomy:

- A plain rect **graphic**, full canvas, timed **exactly** to the rub-prompt
  narration — the ONE element with no tail. Name "Logo Reveal".
- Four metadata keys: `idecide/rub-your-screen` (the config JSON),
  `block_type` = `idecide/rub-your-screen`, `fallback-name` =
  "Rub Your Screen", `idecide/block-action-id` = a uuid.
- A **blockAction record** whose `blockId` is that uuid, empty
  clickActionType, and `interactionData` = the SAME config JSON. Created by
  pushing into `ctx.getBlockActions()` + `markCustomDataDirty()` + a
  documented dirty + changeSlide commit. The server RE-MINTS the uuid on save
  — on both sides consistently, so the link survives.
- Config: `{id: "logo-reveal-<rand>", elementType, autoAdvanceSlideId: null,
  pauseWhileActive, fadeIn, text ("Rub Your Screen" = Line 1; a second line is
  optional), style: {handColor}, images: {coverImageUrl}}`.
- `coverImageUrl` must be an UPLOAD-library r2 url. Under api-only media, get
  one by fetching a dark Pexels still → `uploadAndInsertImage(Blob)` → read
  the fill's uri → remove the block. The upload stays in the library.
- `pauseWhileActive: true` = "Pause When Finished": the timeline pauses at
  the block's end until the rub completes, then resumes automatically.
- Slide structure: VO1 (rub prompt) 0→D1 · rub block 0→D1 exact · every
  reveal layer shifted to start at D1 · VO2 (the reveal) D1→D1+D2 · page =
  D1+D2+tail · slide auto-advances.

## uploadAndInsertImage wants the FULL signature (verified 2026-08-25, deck 180)

`api.currentSlide.images.uploadAndInsertImage(file, x, y, w, h, 0, null)` —
all seven arguments. The short call `uploadAndInsertImage(blob)` returns
**block 0** even with a valid `File`/`Blob` of the right MIME type; nothing
is inserted and no error is thrown. (URL strings return 0 the same way — a
separate, previously verified fact.) Wrap raw bytes in a named `File`
(`new File([blob], 'name.svg', {type: 'image/svg+xml'})`) and pass the
placement rect. The uploaded asset lands in the library and the block's fill
carries a sourceSet (fileURI stays empty, as with all api-created media).

## E.block.duplicate copies group metadata wholesale (verified 2026-08-25, deck 180)

Duplicating a marked part (`idecide/group` / `idecide/groupRole` /
`idecide/block-action-id`) copies those marks verbatim — the duplicate is a
PHANTOM member of the source's group until re-marked. Rules when cloning a
composite: (1) immediately set a fresh group id + role on every duplicate;
(2) mint a NEW `idecide/block-action-id` on a cloned plate (never reuse the
source's uuid); (3) cleanup sweeps must match on group mark AND expected
membership, not the mark alone — a half-configured duplicate still wearing
the source's mark escapes a mark-only sweep. Duplicates insert adjacent to
their source, so cloning plate → label in source order preserves
plate-backmost for the new group; a fresh api-uploaded icon is then
`insertChild`ed just above its plate.

## Creating a NEW presentation from the agent (verified 2026-08-25, deck 184)

`https://my.idecide.com/create/new` **auto-creates a deck server-side** and
redirects to `/builder/create/{newId}` with the Presentation Info overlay
(`#presentationOverlay`) open. Two things to know:

1. **The redirect DROPS the query string.** `create/new?aiagent` lands on
   `/builder/create/184` with no `?aiagent`, so `window.aiagent` is NOT
   exposed. After the overlay is dismissed, re-navigate to
   `/builder/create/{id}?slide={firstSlideId}&aiagent=` and wait for
   `window.aiagent.engine`.
2. **The form inputs are React-controlled.** `el.value = x` is reverted on the
   next render — write through the native setter and fire the events:

```js
const setNative = (el, val) => {
  const d = Object.getOwnPropertyDescriptor(Object.getPrototypeOf(el), 'value');
  d.set.call(el, val);
  el.dispatchEvent(new Event('input',  { bubbles: true }));
  el.dispatchEvent(new Event('change', { bubbles: true }));
};
setNative(document.getElementById('presentationTitle'), name);
setNative(document.getElementById('companyName'), company);   // optional
document.getElementById('getStartedBtn').click();             // closes the overlay
```

Overlay fields: `#presentationTitle` (required, defaults "Untitled
Presentation"), `#companyName`, `#website`, plus toggles for template /
replicated URLs / analytics. Submit is `#getStartedBtn` (`form=presentationForm`).
Poll until `#presentationOverlay` is gone or hidden.

**Verify by record, not by DOM**: `GET /api/builder/builderSession/{sid}`
returns `presentationName` and `companyName` — read them back to confirm the
name stuck. The fresh deck arrives with exactly one slide named
**"Play Button"** (the Welcome / "Click anywhere to Begin" default), matching
the from-scratch build assumption.

**For the extension**: the Build flow should own this — create the deck, name
it from the questionnaire's brand answer, dismiss the overlay, re-navigate
with `?aiagent`, then run Phase 0.5. The Edit flow keeps the opposite
contract: it expects to already be on the presentation the user wants edited.

**Re-verified end-to-end through the browser on 2026-08-27 (deck 301)**, and
the SKILL now owns this flow rather than only documenting it.

 Three details
the 2026-08-25 write-up did not have:

1. **The re-navigation is always `&aiagent=`, never `?aiagent=`.** By the time
   the overlay is dismissed the URL already reads
   `/builder/create/{id}?slide={firstSlideId}`. Appending `?aiagent=` to that
   yields two `?` and the API never appears.
2. **Do not test the overlay with `offsetParent`.** `#presentationOverlay` is
   `position: fixed`, so `offsetParent` is `null` while it is plainly on
   screen — a false negative that reads as "already dismissed". Poll
   `display` / `visibility` instead, as the snippet above does.
3. **The builder tab must be open and in focus** for browser tooling to reach
   the page. A navigation or call that hangs for more than a moment is usually
   a backgrounded tab, not a slow platform — ask the operator to click into
   the tab and retry rather than continuing to wait.

### Logged-out: `/create/new` redirects to `/login` (verified 2026-08-25)

Visiting `https://my.idecide.com/create/new` without a session lands on:

```
/login?targetUrl=%2Fcreate%2Fnew
```

Observed signature — no deck is created, nothing is lost:

| Signal | Logged out | Logged in |
|---|---|---|
| `location.pathname` | `/login` | `/builder/create/{newId}` |
| `#presentationOverlay` / `#presentationTitle` | absent | present |
| `input[type="password"]` | present | absent |
| `window.aiagent` | undefined | undefined until re-nav with `?aiagent` |
| `document.title` | **"myiDecide Builder"** | "myiDecide Builder" |

`document.title` is identical in both states — **do not** use it to detect
auth. Test the path (and the password field as a second signal):

```js
const loggedOut = /^\/login/.test(location.pathname)
  || (!document.getElementById('presentationTitle') && !!document.querySelector('input[type="password"]'));
```

**Extension behaviour:** the Build flow must check this immediately after
navigating to `/create/new`, and if logged out, STOP and tell the user to log
in — never type credentials on their behalf. The `targetUrl` param means the
platform returns them to `/create/new` automatically once they sign in, so
the retry is simply "tell me when you're logged in" and re-run the same step.
Suggested copy: *"You're signed out of myiDecide. Log in in this tab — it'll
bring you right back to the new-presentation screen — then tell me to
continue."*

### After login, the resume lands in the SAME state (verified 2026-08-25, deck 185)

Signing in from `/login?targetUrl=%2Fcreate%2Fnew` resumes the intent: the
platform creates a fresh deck and lands on `/builder/create/{id}?slide={id}`
with the Presentation Info overlay open — identical to a logged-in visit. So
the Build flow needs no separate post-login branch: after the user says they
are signed in, re-run the same "name it, dismiss, re-navigate with `?aiagent`"
step. Confirmed: no password field, overlay open, title back to "Untitled
Presentation", `window.aiagent` still undefined until the re-navigation.

Two hard-won details:

1. **Every visit to `/create/new` burns a deck id.** The deck is created
   server-side *before* the overlay is shown, so a speculative navigation
   leaves an orphan "Untitled Presentation" behind. Navigate there exactly
   once, when the build is actually starting. (This run left deck 184 orphaned
   for precisely this reason.)
2. **The name save is ASYNCHRONOUS.** Reading
   `GET /api/builder/builderSession/{sid}` a few hundred ms after clicking
   Get Started still returns `presentationName: "Untitled Presentation"`;
   the same read 3 s later returns the real name. Do not treat the immediate
   read as a failure and do not re-submit — poll:

```js
async function confirmName(sid, want, tries = 10) {
  for (let i = 0; i < tries; i++) {
    const r = await fetch('/api/builder/builderSession/' + sid, { credentials: 'include' }).then(x => x.json());
    if (r.presentationName === want) return true;
    await new Promise(res => setTimeout(res, 1000));
  }
  return false;
}
```

The fresh deck's single slide is named **"Play Button"** and carries the
12-block Welcome default, which Phase 0.5 pre-resolves onto before Phase 1
clears it.

## Agent-bridge transport corrupts long string literals (verified 2026-08-25)

Not a platform fact — a constraint on *agent-driven* work, recorded so the
next session does not lose an hour to it.

Pushing a large base64 payload through the javascript bridge fails in **two**
distinct ways:

1. **Silent truncation** above roughly 6 KB per call — the paste does not
   error, it simply never executes.
2. **Silent corruption** below that threshold — a 2,776-character base64
   string arrived with the correct **length**, correct first 24 and last 16
   characters, and a **different SHA-1**. Characters in the middle were
   altered in place.

The second one is the dangerous one: any payload without an integrity check
(raw JSON, a scene blob, a CSV) would be accepted as valid and written to the
platform. Gzip is what saved this run — `DecompressionStream` refused the
stream with *"The compressed data was not valid: incorrect data check"*
because the CRC-32 trailer disagreed.

**Rules when an agent must move bytes into the page:**

- Always wrap in gzip (or carry an explicit SHA-256) so corruption is loud.
- Verify per chunk in-page before use:
  ```js
  const h = await crypto.subtle.digest('SHA-1', new TextEncoder().encode(str));
  ```
- Keep chunks well under the truncation ceiling (~4 KB of payload).
- Note that a failed inflate surfaces in Chrome as `TypeError: Failed to
  fetch` when consumed via `new Response(stream).text()` — misleading, and
  indistinguishable from a network error unless you read the stream manually
  with a reader, which reports the real cause.

**None of this applies to the extension.** It ships the library in its own
bundle and reads it with `fetch(chrome.runtime.getURL(...))` — same-origin,
no serialization, no size ceiling. Programmatic upload was proven end to end
in this session (`duo:bolt`, `duo:tap` uploaded with zero user involvement);
the bulk transfer is slow only because of the agent bridge in the middle.

## Loading the inject bundle into an ALREADY-OPEN page (2026-08-25)

The side panel injects `sh/*`, `charts`, `composer` and `pipeline` with
`chrome.scripting.executeScript`. That covers a BUILD, but there was no way to
get the same code into a page that is already open — repairing, retrofitting or
re-balancing an existing deck meant pushing ~320KB through an agent bridge in
4KB chunks, and that bridge corrupts oversized payloads while preserving
length, so only a checksum catches it.

Two manifest changes replace all of that with one `fetch`:

```jsonc
"web_accessible_resources": [{
  "resources": ["assets/js/inject/*.js", "assets/js/inject/sh/*.js"],
  "matches": ["https://my.idecide.com/*"]        // never <all_urls>
}]
```

plus `assets/js/inject/resolve.js`, a second content script that must run in
the **ISOLATED** world — `capture.js` is MAIN world, where `chrome.*` does not
exist, so the page cannot otherwise learn the extension's own id. The id also
differs between an unpacked load and a Store build, so a hard-coded literal
works locally and breaks on publish.

```js
const base = document.documentElement.dataset.idpBase;   // set by resolve.js
(0, eval)(await fetch(base + 'assets/js/inject/composer.js').then(r => r.text()));
```

Verified: 320,722 bytes in one call, `window.__GEOM` / `__BALANCE` / `__COMPOSE`
all live afterwards. **A manifest change needs an extension reload AND a reload
of the target tab** — content scripts only inject at page load, so an extension
reload alone leaves `data-idp-base` absent on tabs that are already open.

**Dev and TESTER only — the STORE build strips all three parts.** `package.mjs`
deletes `web_accessible_resources` from the staged manifest, filters the
resolve.js entry out of `content_scripts`, and removes
`assets/js/inject/resolve.js` from the stage. `assets/` is copied wholesale, so
removing the file matters as much as unwiring it — otherwise it ships as inert
dead code. 0.9.182's store-review sweep removed WAR deliberately: it publishes
the extension id to the host pages, a fingerprinting surface. That reason still
stands. Its other reason — "nothing loads the inject files by URL" — is what
resolve.js makes obsolete, and only for the agent path: a client building a deck
never needs it, because `executeScript` covers a build.

**Do not reach for `"use_dynamic_url": true`** to keep WAR in the store build.
`chrome.runtime.getURL()` returns the static URL while dynamic-URL resources are
only reachable through the rotating token, so the handshake may not survive that
flag unchanged. That needs a live test in Chrome, not a code read.
