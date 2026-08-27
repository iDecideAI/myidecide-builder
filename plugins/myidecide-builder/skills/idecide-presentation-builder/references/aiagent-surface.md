> **Reference for the iDecide Presentation Builder skill.** The complete
> `window.aiagent` call surface with worked examples. The LIVE
> `window.aiagent.instructions` is the editor's live signature list — read it
> each session and diff against this file; where a signature differs, the live
> one is the accurate one. It describes calls, not conduct: the skill's
> behaviour lives in SKILL.md and these references.

# window.aiagent — complete call surface, with examples

Enumerated live from `https://my.idecide.com/builder/create/179?aiagent=` on
**2026-08-23**, CE.SDK v1.74.1. Signatures are taken verbatim from
`window.aiagent.instructions`; every example is written against those signatures.

This is the *map*: what exists, where, and how to call it. For the surrounding
discipline — save model, template rules, footguns, the SH/REST fallback — see
**AIAGENT_API.md**, which stays the working reference.

**Totals**

| Surface | Calls |
|---|---|
| `aiagent.api` | **48** |
| `aiagent.engine` | **547** |

Two caveats that matter if you enumerate this yourself:

1. **`Object.keys` is not enough — it undercounts badly.** `api` reports four keys
   and has **48** methods; `api.currentSlide` is a getter, and `helloWorld` and
   `resolveUserSessionId` sit on the prototype. `engine.block` shows 327 own
   properties and has **372** methods. `engine.asset` shows *three* own keys and has
   **27** — including `apply`, which the whole single-clip upload path depends on.
   Walk the prototype chain with `getOwnPropertyNames` or you will conclude a call
   does not exist when it does.
2. **`api.currentSlide` is a live getter.** Read it fresh after any navigation
   rather than holding the object across a `changeSlide`.

**The save model, in one line:** a documented `api.*` mutation marks the slide dirty;
`changeSlide` commits it; `location.reload()` discards anything uncommitted. A raw
`engine.*` write marks nothing — pair it with a documented `api.*` call before the
commit or it is lost.

---

# Part 1 — `window.aiagent.api` (48 calls)

## 1.0 Top level — 3

### `api.helloWorld(): string`

A liveness probe. Not on `Object.keys(api)` — it is a prototype method.

```js
if (typeof window.aiagent?.api?.helloWorld === 'function') {
  console.log(window.aiagent.api.helloWorld());   // "Hello from myiDecide builder AI Agent API"
}
```

### `api.resolveUserSessionId(): Promise<string>`

Returns the session identifier. Relevant because the REST route requires an
`X-Session-UUID` header on every mutating call — without it you get a 400 or 409.

```js
const sid = await window.aiagent.api.resolveUserSessionId();
await fetch(url, { method: 'PUT', headers: { 'X-Session-UUID': sid, 'Content-Type': 'application/json' }, body });
```

### `api.provider.getCtx(): object`

**Undocumented** — the editor's own React context. Handy for spelunking (it exposes
`engine`, `markCustomDataDirty`, `createSlides`, `blocksAddText`, … — the internals
each `api.*` method wraps), but it is not part of the contract. Read it to understand
behaviour; do not ship against it.

```js
const ctx = window.aiagent.api.provider.getCtx();
console.log(Object.keys(ctx));   // engine, getSlides, markCustomDataDirty, createSlides, …
```

---

## 1.1 `api.slides` — 9 calls (deck level)

### `slides.get(): SlideSummary[]`

Every slide, in deck order.

```js
const slides = window.aiagent.api.slides.get();
console.log(slides.map(s => `${s.index}. ${s.name} (#${s.id})`).join('\n'));
const byName = Object.fromEntries(slides.map(s => [s.name, s.id]));
```

### `slides.getDimensions(): { width, height }`

```js
const { width, height } = window.aiagent.api.slides.getDimensions();  // 1558 x 720
const M = Math.round(width * 0.055);   // canvas margin, container-relative
```

### `slides.create(atIndex, insertPosition, changeSlide): Promise<{id, index} | null>`

- `atIndex` — 0-based; **takes precedence** over `insertPosition` when provided.
- `insertPosition` — `'afterCurrent'` (default) | `'end'` | `'afterSpecified'` | `'start'`.
- `changeSlide` — navigate to it immediately (default `false`).

```js
const A = window.aiagent.api;
const made = await A.slides.create(null, 'end', false);          // append, stay put
const at5  = await A.slides.create(5, 'afterSpecified', false);  // insert at index 5
console.log(made.id, made.index);
```

**Use `createMultiple` for more than one.** Sequential `create()` measured ~587ms each.

### `slides.createMultiple(count, atIndex, insertPosition): Promise<{id,index}[] | null>`

All slides in the run are written to the server in **one request**.

- `count` — documented as `> 1`. `count: 1` does work, but do not rely on it.
- `atIndex` again takes precedence over `insertPosition`.
- Returns one entry per slide **in insertion order**, so `made[k]` is the k-th.
- **Never changes the current slide.**

Measured: **223ms for 5 slides vs 2935ms for 5 × `create()` — 13.2×.**

```js
const A = window.aiagent.api;
const made = await A.slides.createMultiple(12, null, 'end');
if (!made || made.length !== 12) throw new Error('short return — the id mapping is unknowable');
plan.slides.forEach((s, k) => { ids[s.name] = made[k].id; });
```

Batch *consecutive runs* so a resume still lands its slides at the right index:

```js
const runs = [];
for (const i of missingIndices) {
  const last = runs[runs.length - 1];
  if (last && i === last[last.length - 1] + 1) last.push(i); else runs.push([i]);
}
for (const run of runs) {
  const made = run.length > 1
    ? await A.slides.createMultiple(run.length, run[0], 'end')
    : [await A.slides.create(run[0], 'afterSpecified', false)];
  run.forEach((planIdx, k) => { ids[plan.slides[planIdx].name] = made[k].id; });
}
```

### `slides.changeSlideByIndex(index): Promise<boolean>`

**Do not call until every outstanding promise has resolved** — the instructions say so
explicitly, and a navigation mid-flight loses the work in flight.

```js
await Promise.all(pending);
await window.aiagent.api.slides.changeSlideByIndex(0);
```

### `slides.changeSlide(id): Promise<boolean>`

Navigate by id — **and this is the commit.** A dirty slide is written when you leave
it, so a final `changeSlide` is what saves the last slide you touched.

```js
const A = window.aiagent.api;
for (const s of plan.slides) {
  await A.slides.changeSlide(ids[s.name]);
  await buildSlide(s);                 // documented api.* mutations mark it dirty
}
await A.slides.changeSlide(ids[plan.slides[0].name]);   // commits the last one
```

### `slides.setName(id, name): Promise<boolean>`

```js
await window.aiagent.api.slides.setName(38042, '03 — What it costs');
```

### `slides.setMenuSlide(id, isMenuSlide): Promise<boolean>`

```js
await window.aiagent.api.slides.setMenuSlide(menuId, true);
```

### `slides.setAutoAdvance(id, autoAdvanceAfterNarration, autoAdvanceSlideId): Promise<boolean>`

Always name the target explicitly — the deck contract forbids implicit next-slide
advance.

```js
const A = window.aiagent.api;
await A.slides.setAutoAdvance(ids['02 intro'], true,  ids['03 menu']);  // advance to a named slide
await A.slides.setAutoAdvance(ids['03 menu'], false, 0);                // menu waits for a click
```

---

## 1.2 `api.assets` — 3 calls (deck level, no current-slide requirement)

### `assets.searchImages(terms, page): Promise<{page, perPage, totalResults, photos[]}>`

`photos[]` carries `width`, `height`, `url`, `alt`, `preview`.

```js
const r = await window.aiagent.api.assets.searchImages(['warehouse', 'morning'], 1);
const landscape = r.photos.filter(p => p.width > p.height);
```

### `assets.searchVideos(terms, page): Promise<{…, videos[]}>`

Same shape. **Rank on the real `width`/`height`** — the query does not guarantee
orientation, and a portrait clip in a landscape frame is a defect. Each result also
carries a `pexelVideoId` derived from its url (`0` when it could not be determined)
that feeds straight into `importPexelVideoBatch`.

```js
const r = await window.aiagent.api.assets.searchVideos('busy kitchen', 1);
const ranked = r.videos
  .filter(v => Math.max(v.width, v.height) >= 720)   // long-side floor
  .sort((a, b) => (b.width > b.height) - (a.width > a.height));  // landscape first
```

### `assets.importPexelVideoBatch(videos, onCompleted): Promise<ProcessedVideoResponse[]>`

Uploads many clips in **one server request**, from any slide. Entries are a bare
Pexels id, or `{ pexelVideoId, slideId }` — the optional `slideId` is echoed back so
you can match a result to its slide. **Creates no blocks**; place the result with
`videos.insertPexelBatchVideo`. `onCompleted` fires per clip as soon as the server has
downloaded, converted and uploaded it; the promise resolves when the whole batch is
done, one result per input, in order.

Measured: 8 clips / 19.1s, first ready at 4.8s; 10 clips / 22.3s. Output is capped at
**960×540** regardless of source — on this path *and* on `engine.asset.apply`.

```js
const ready = {};
await window.aiagent.api.assets.importPexelVideoBatch(
  picks.map(p => ({ pexelVideoId: p.pexelVideoId, slideId: p.slideId })),
  (r) => { ready[r.slideId] = r; console.log('landed', r.slideId); }   // stream, don't wait
);
```

The extension runs this as a **continuous uploader**: one queue for the whole deck,
ten clips a request, two requests in flight, and each slide takes its clip as it lands.

---

## 1.3 `api.narration` — 2 calls (deck level)

### `narration.getVoices(): Promise<VoiceInfo[]>`

```js
const voices = await window.aiagent.api.narration.getVoices();
const voiceId = voices.find(v => /rachel/i.test(v.name))?.voice_id;
```

### `narration.generateBatch(narrations, onCompleted): Promise<CompletedNarration[]>`

One server request for many narrations; the server generates and uploads a handful in
parallel. **Creates no audio blocks** — change to the slide and insert the result with
`blocks.addAudio`.

- `Narration`: `{ voiceId, slideId, narration, startTime?, voiceSettingsOrStability?, similarity?, style? }`
- `CompletedNarration`: `{ success, slideId, uri, duration }`
- Narrations for an unknown slide, for the **first (intro) slide**, or missing a
  script or voice are never sent and come back `success: false`.

```js
const A = window.aiagent.api;
const done = {};
const all = await A.narration.generateBatch(
  plan.slides.filter(s => s.vo).map(s => ({ voiceId, slideId: ids[s.name], narration: s.vo })),
  (r) => { if (r.success) done[r.slideId] = r; }
);

for (const r of all.filter(x => x.success)) {
  await A.slides.changeSlide(r.slideId);
  const audio = await A.currentSlide.blocks.addAudio(r);   // uses its uri + duration
  window.aiagent.engine.block.insertChild(pageId, audio, 0);  // narration lives at z0
}
```

---

## 1.4 `api.currentSlide` — 31 calls

### The slide itself — 3

```js
const CS = window.aiagent.api.currentSlide;

const slide = CS.get();            // SlideSummary | null
const dur   = CS.getDuration();    // seconds
CS.setBackgroundColor('#0e1e33');  // hex string -> boolean
```

### `currentSlide.blockActions` — 3

`blockActions.set(blockId, actionType, actionTarget)` where `actionType` is
`'noAction' | 'goToSlide' | 'openLink' | 'toggleSelection' | 'finishPresentation'`.

**Universal rule 12:** put the action on the **background** layer of a multi-layer
button — it covers the whole clickable area — and clear the text/icon layers.

```js
const CS = window.aiagent.api.currentSlide;

CS.blockActions.set(btnBgId, 'goToSlide', ids['07 pricing']);
CS.blockActions.set(ctaBgId, 'openLink', 'https://example.com/book');
CS.blockActions.set(endBgId, 'finishPresentation', 'Thanks for watching');

CS.blockActions.remove(btnLabelId);          // the label must not also be clickable
CS.blockActions.remove(btnIconId);

console.table(CS.blockActions.getAll());     // audit what is actually wired
```

### `currentSlide.blocks` — 19

#### `blocks.addText(text, x, y, w, h): Promise<BlockId | null>`

The quick path. `createTextBox` is the one to use when you care about styling.

```js
const id = await window.aiagent.api.currentSlide.blocks.addText('Built for the rush.', 86, 180, 700, 120);
```

#### `blocks.createTextBox(params): BlockId`

```js
const id = window.aiagent.api.currentSlide.blocks.createTextBox({
  text: 'Built for the rush.',
  layout: { x: 86, y: 180, width: 700, height: 140 },
  timing: { startTime: 0, duration: 6.2 },
  style:  { font: 'Avenir Next', size: 64, color: '#ffffff', alignment: 'Left', lineHeight: 1.1 },
  shadow: { enabled: true, blur: 18, offsetX: 0, offsetY: 4, color: '#00000055' },
  overflow: { clip: false }
});
```

Platform defaults worth knowing: **`style.size` is floored at 28** (which is also our
own rule); `lineHeight` defaults to **0.7** when omitted — always pass one; `overflow.clip`
maps to `text/clipLinesOutsideOfFrame`. Returns `0` on failure, not `null`.

#### `blocks.addAudio(audio, durationOrStartTime?, startTime?): Promise<BlockId>`

Overloaded: pass a `CompletedNarration` (its `uri` and `duration` are used), or a url
plus an explicit duration.

```js
const B = window.aiagent.api.currentSlide.blocks;
await B.addAudio(completedNarration);         // uri + duration from the record
await B.addAudio(completedNarration, 0.4);    // ...starting 0.4s into the slide
await B.addAudio(url, 8.3);                   // url + duration in seconds
await B.addAudio(url, 8.3, 0.4);              // url + duration + startTime
```

Duration ≤ 0 or missing is read from the file. Then move it to z0 (rule 13).

#### `blocks.updateText(blockId, text): boolean`

**Flattens two-tone headlines** — a range-coloured run comes back one colour. Re-apply
the ranges after.

```js
const CS = window.aiagent.api.currentSlide, E = window.aiagent.engine;
CS.blocks.updateText(id, 'Built for the rush.');
E.block.setTextColor(id, { r: .04, g: .66, b: .81, a: 1 }, 11, 15);   // restore the accent run
```

It is also the **commit** for a raw engine text write:

```js
E.block.setTextFontSize(id, 44);                 // run-level resize (PPTX imports)
CS.blocks.updateText(id, E.block.getString(id, 'text/text'));   // same text, marks dirty
```

#### `blocks.updateTextBoxStyling(blockId, layout, timing, style?, shadow?, overflow?): boolean`

Same shapes as `createTextBox`, positionally. Same 28 floor and 0.7 lineHeight default.

```js
window.aiagent.api.currentSlide.blocks.updateTextBoxStyling(
  id,
  { x: 86, y: 210, width: 700, height: 140 },
  { startTime: 0, duration: 6.2 },
  { size: 56, color: '#ffffff', alignment: 'Left', lineHeight: 1.15 }
);
```

**It does not move imported PPTX text size** — displayed size lives in the runs; use
`engine.block.setTextFontSize` and commit with `updateText(sameText)`.

#### `blocks.updateTiming(blockId, startTime, duration): boolean`

`null` leaves that field alone. Also the commit for a raw engine resize or reposition.

```js
const B = window.aiagent.api.currentSlide.blocks;
B.updateTiming(id, 0.8, 5.4);
B.updateTiming(id, null, audioEnd + 0.3);   // 0.3s graphic tail past narration
```

#### `blocks.setCornerRadius(blockId): Promise<boolean>`

```js
await window.aiagent.api.currentSlide.blocks.setCornerRadius(plateId);
```

#### `blocks.moveBlocks(blockId | blockId[], 'front' | 'back'): boolean`

```js
const B = window.aiagent.api.currentSlide.blocks;
B.moveBlocks(bgVideoId, 'back');
B.moveBlocks([labelId, iconId], 'front');
```

#### `blocks.removeBlock(blockId): Promise<boolean>`

```js
await window.aiagent.api.currentSlide.blocks.removeBlock(strayId);
```

#### `blocks.get / getVisual / getAudio`

```js
const B = window.aiagent.api.currentSlide.blocks;
const one     = B.get(blockId);     // VisualBlockDescription | null
const visuals = B.getVisual();      // VisualBlockDescription[]
const audio   = B.getAudio();       // AudioBlockDescription[]
const hasVideo = visuals.some(v => v.type === 'video' || /video/i.test(v.fill || ''));
```

`getVisual()` is the honest way to ask "does this slide already have media?" — the
ghost-background check that stopped slides 32 and 35 being covered twice.

#### `blocks.clearAllVisual() / clearAllAudio(): Promise<number>`

Both return how many they removed. This is how Phase 1 empties the platform's default
Slide 1 after Phase 0.5 has pre-resolved its assets.

```js
const B = window.aiagent.api.currentSlide.blocks;
console.log('cleared', await B.clearAllVisual(), 'visual +', await B.clearAllAudio(), 'audio');
```

#### `blocks.ANIM_TYPES`

Short keys mapped to engine animation paths:

```js
Object.keys(window.aiagent.api.currentSlide.blocks.ANIM_TYPES);
// fade, grow, slide, baseline, wipe, spread_text, block_swipe_text, typewriter_text, pan
```

#### `blocks.setAnimation(blockId, opts): BlockId`

`opts`: `{ type, mode: 'in'|'out', duration, easing, direction, writing }`.

**Direction is travel direction** for `slide` (radians — `0` = right, `π` = left) and
**sweep direction** for the enum types. Enter from the nearest edge.

```js
window.aiagent.api.currentSlide.blocks.setAnimation(id, {
  type: 'slide', mode: 'in', duration: 1.2, easing: 'EaseOut', direction: Math.PI
});
```

#### `blocks.setInAnimation(blockId, kind, opts) / setOutAnimation(...): BlockId`

The shorthands — `type` and `mode` are implied.

```js
const B = window.aiagent.api.currentSlide.blocks;
B.setInAnimation(heroId,  'grow', { duration: 2.4, easing: 'EaseOut' });   // bg/hero media 2–4s
B.setInAnimation(labelId, 'fade', { duration: 1.1, easing: 'EaseOut' });   // everything else 1–1.5s
B.setOutAnimation(labelId, 'fade', { duration: 0.8, easing: 'EaseOut' });
```

#### `blocks.animateSequence(blockIds, opts): void`

`opts`: `{ start = 0, stagger = 0.3, end, inAnimation, outAnimation, minDuration = 0.5 }`.
The staggered entry the deck contract expects, in one call.

```js
window.aiagent.api.currentSlide.blocks.animateSequence(
  [headingId, subId, card1Id, card2Id, card3Id],
  { start: 0.4, stagger: 0.28, end: audioEnd + 0.3,
    inAnimation:  { type: 'slide', duration: 1.1, easing: 'EaseOut', direction: Math.PI / 2 },
    outAnimation: { type: 'fade',  duration: 0.6, easing: 'EaseOut' },
    minDuration: 1.2 }
);
```

### `currentSlide.narration` — 1

#### `narration.generateAndInsert(voiceId, narration, startTime?, stability?, similarity?, style?): Promise<BlockId>`

Single-slide generate **and** insert. Slow — the extension caps it at 3 minutes. For a
whole deck use `api.narration.generateBatch` instead.

```js
const id = await window.aiagent.api.currentSlide.narration.generateAndInsert(
  voiceId, 'Here is what happens next.', 0, 0.45, 0.85, 0.2
);
```

### `currentSlide.images` — 3

#### `images.insertPexelImage(url, x, y, width, height, startTime, duration): Promise<BlockId>`

```js
const id = await window.aiagent.api.currentSlide.images.insertPexelImage(
  photo.url, 86, 64, 686, 592, 0, null      // null duration = the whole slide
);
```

#### `images.replaceWithPexelImage(url, blockId): Promise<BlockId>`

Swaps a photo in place — geometry, timing and animation survive.

```js
await window.aiagent.api.currentSlide.images.replaceWithPexelImage(photo.url, existingId);
```

#### `images.uploadAndInsertImage(imageData, x, y, width, height, startTime, duration): Promise<BlockId>`

`imageData` is a `File` (preferred), `Blob`, or string. This is the path for a
recoloured Iconify SVG or anything else you generated.

```js
const svg  = await fetchIconSvg('shield-check', '#0989cf');
const file = new File([svg], 'icon.svg', { type: 'image/svg+xml' });
const id   = await window.aiagent.api.currentSlide.images.uploadAndInsertImage(file, 640, 300, 52, 52, 0, null);
```

### `currentSlide.videos` — 3

#### `videos.insertPexelVideo(url, x, y, width, height, startTime, duration): Promise<BlockId>`

**Only ever with a genuine Pexels payload.** A non-Pexels URL here **freezes the
renderer** — swap the block's fill to a video fill instead.

```js
const id = await window.aiagent.api.currentSlide.videos.insertPexelVideo(
  clip.url, 0, 0, 1558, 720, 0, null
);
```

#### `videos.insertPexelBatchVideo(video, x, y, width, height, startTime, duration): Promise<BlockId>`

Places a `ProcessedVideoResponse` that `importPexelVideoBatch` already uploaded. This
is the fast path: the upload happened in the background, so placement is immediate.

```js
const A = window.aiagent.api;
await A.slides.changeSlide(rec.slideId);
await A.currentSlide.videos.insertPexelBatchVideo(rec, 779, 0, 779, 720, 0, null);
```

#### `videos.uploadAndInsertVideo(videoData, x, y, width, height, startTime, duration): Promise<BlockId>`

`File` / `Blob` / string. Omitted or `null` duration uses the video's own length.

```js
await window.aiagent.api.currentSlide.videos.uploadAndInsertVideo(file, 0, 0, 1558, 720, 0, null);
```

---

# Part 2 — `window.aiagent.engine` (547 methods)

The raw CE.SDK engine. Powerful, undocumented by the platform, and **it marks nothing
dirty** — every engine write must be followed by a documented `api.*` call before the
`changeSlide` commit, or it is silently discarded on reload.

| Namespace | Methods | What it is |
|---|---|---|
| `engine.block` | **372** | Every block: create, place, style, time, animate, query |
| `engine.editor` | **106** | Editor state — history, settings, roles, scopes, selection |
| `engine.scene` | **37** | The scene/document — load, save, pages, zoom, playback |
| `engine.asset` | **27** | Asset sources, search, and `apply` |
| `engine.variable` | 4 | `findAll`, `setString`, `getString`, `remove` |
| `engine.event` | 1 | `subscribe` |
| `engine.reactor` | 1 | `_resolveNextReaction` — internal |

## 2.1 `engine.block` — 372, by function

| Group | ~Count | Representative calls |
|---|---|---|
| Query / predicates | 55 | `findAllSelected`, `findByType`, `findByKind`, `isValid`, `getType`, `getKind`, `getName`, `getUUID`, `supportsFill`, `supportsStroke` |
| Geometry | 41 | `getPositionX/Y`, `setPositionX/Y`, `getWidth/Height`, `setWidth/Height`, `getFrameX/Y/Width/Height`, `getGlobalBoundingBoxXYWH`, `setRotation`, `setCrop*`, `setFlip*` |
| Fill & colour | 41 | `getFill`, `setFill`, `createFill`, `destroyFill`, `setFillEnabled`, `setColor`, gradient-stop accessors |
| Timing & media | 30 | `setDuration`, `setTimeOffset`, `setTrimOffset`, `setTrimLength`, `setLooping`, `setVolume`, `setMuted`, `getNativeWidth/Height`, `forceLoadAVResource` |
| Text | 28 | `replaceText`, `setTextColor`, `setTextFontSize`, `setTextFontWeight`, `setTextCase`, `getTextVisibleLineCount`, `getTextVisibleLineGlobalBoundingBoxXYWH`, `setTextCursorRange` |
| Size mode / visibility | 23 | `setContentFillMode`, `setSizeMode`, `setPlaceholderEnabled`, `setClipped`, `setVisible`, `setOpacity`, `setBlendMode` |
| Lifecycle | 20 | `create`, `destroy`, `duplicate`, `saveToString`, `loadFromString`, `export`, `replace` |
| Shadow | 17 | `setDropShadowEnabled`, `setDropShadowColor`, `setDropShadowOffsetX/Y`, `setDropShadowBlurRadiusX/Y` |
| Effects | 17 | `createEffect`, `appendEffect`, `getEffects`, `createBlur`, `setBlur`, `setBlurEnabled` |
| Stroke | 14 | `setStrokeEnabled`, `setStrokeColor`, `setStrokeWidth`, `setStrokeStyle`, `setStrokePosition` |
| Hierarchy | 10 | `getChildren`, `getParent`, `insertChild`, `appendChild`, `group`, `ungroup`, `sortChildren` |
| Properties / metadata | 9 | `getProperties`, `setString`, `getString`, `setMetadata`, `getMetadata`, `findAllMetadata` |

### The ones that carry project rules

**`saveToString([pageId])` — the only correct serializer for the REST route.**
`engine.scene.saveToString` produces bytes the platform cannot load.

```js
const E = window.aiagent.engine;
const page  = E.scene.getCurrentPage();
const bytes = await E.block.saveToString([page]);
await SH.putSlideRecord(Object.assign({}, rec, { slideData: _b64enc(bytes) }));
```

**`insertChild(page, audioBlock, 0)` — narration goes to z0 (rule 13).**

```js
const E = window.aiagent.engine;
E.block.insertChild(E.scene.getCurrentPage(), audioId, 0);
window.aiagent.api.currentSlide.blocks.updateTiming(audioId, 0, null);   // commit
```

**`setTextFontSize` — the run-level resize PPTX imports need**, where
`updateTextBoxStyling` and the `text/fontSize` property are both ignored.

```js
const E = window.aiagent.engine, CS = window.aiagent.api.currentSlide;
E.block.setTextFontSize(id, 44);                                  // whole block
E.block.setTextFontSize(id, 64, 0, 12);                           // a run
CS.blocks.updateText(id, E.block.getString(id, 'text/text'));     // commit
```

**Element/slot metadata — the group contract.** Read the marks, never assume them.

```js
const E = window.aiagent.engine;
E.block.setMetadata(id, 'idecide/group', 'button:2');
E.block.setMetadata(id, 'idecide/groupRole', 'background');
const role = E.block.findAllMetadata(id).includes('idecide/groupRole')
  ? E.block.getMetadata(id, 'idecide/groupRole') : null;
```

**Measuring instead of estimating** — how the cover gap got fixed:

```js
const E = window.aiagent.engine;
const h = E.block.getFrameHeight(id);                    // real laid-out height
const [x, y, w, hh] = E.block.getGlobalBoundingBoxXYWH(id);
const lines = E.block.getTextVisibleLineCount(id);
```

**Native gradient fills** (rule: convert PPTX image scrims to real gradients):

```js
const E = window.aiagent.engine;
const fill = E.block.createFill('gradient/linear');
E.block.setGradientColorStops(fill, [
  { color: { r: 0, g: 0, b: 0, a: .72 }, stop: 0 },
  { color: { r: 0, g: 0, b: 0, a: 0   }, stop: 1 }
]);
E.block.setFill(plateId, fill);
```

## 2.2 `engine.editor` — 106

History (`undo`, `redo`, `canUndo`, `addUndoStep`, transactions), settings
(`getSettingBool`, `setSettingColor`, …), roles and scopes (`setRole`,
`setGlobalScope`, `setScopeEnabled`), selection and cursor, and change subscriptions.
Mostly UI-facing; the build path barely touches it.

```js
const E = window.aiagent.engine;
E.editor.addUndoStep();                       // one undo entry for a whole batch
console.log(E.editor.getSettingBool('features/pageCarouselEnabled'));
```

## 2.3 `engine.scene` — 37

Own properties: `onZoomLevelChanged`, `onActiveChanged`, `loadFromString`,
`loadFromURL`, `loadFromArchiveURL`, `create`, `createVideo`, `createFromImage`,
`createFromVideo`, `get`, `applyTemplateFromString`, `applyTemplateFromURL`, `getMode`,
`setMode`, `setDesignUnit`, `getDesignUnit`, `getLayout`, `setLayout`, `getPages`,
`getCurrentPage`, `findNearestToViewPortCenterByType`,
`findNearestToViewPortCenterByKind`, `setZoomLevel`, `getZoomLevel`, `zoomToBlock`,
`enableZoomAutoFit`, `disableZoomAutoFit`, `isZoomAutoFitEnabled`, `setPlaying`
(+ 8 prototype methods, including `saveToString` — **do not use it**).

```js
const E = window.aiagent.engine;
const page = E.scene.getCurrentPage();      // the id every saveToString([pageId]) needs
E.scene.setPlaying(false);                  // stop playback before a bounds audit
E.scene.zoomToBlock(page);
```

## 2.4 `engine.asset` — 27

`onAssetSourceAdded`, `onAssetSourceRemoved`, `onAssetSourceUpdated`,
`registerApplyMiddleware`, `registerApplyToBlockMiddleware`, `addSource`,
`addLocalSource`, `addLocalAssetSourceFromJSONString`,
`addLocalAssetSourceFromJSONURI`, `removeSource`, `findAllSources`, `findAssets`,
`fetchAsset`, `getGroups`, `getSupportedMimeTypes`, `getCredits`, `getLicense`,
`canManageAssets`, `addAssetToSource`, `removeAssetFromSource`, **`apply`**,
`applyToBlock`, `applyProperty`, `defaultApplyAsset`, `defaultApplyAssetToBlock`,
`assetSourceContentsChanged`, `dispose`.

`asset.apply` is the single-clip upload path (rule 7): it uploads to r2-dev and creates
a temp block whose fill you then steal. Same 960×540 output ceiling as the batch import.

```js
const E = window.aiagent.engine;
const before = new Set(E.block.getChildren(E.scene.getCurrentPage()));
await E.asset.apply('ly.img.video.upload', { id: clipId, meta: { uri: clip.url, mimeType: 'video/mp4' } });
const tmp  = E.block.getChildren(E.scene.getCurrentPage()).find(id => !before.has(id));
const fill = E.block.getFill(tmp);                 // steal it
E.block.setFill(targetId, fill);
E.block.destroy(tmp);
```

## 2.5 `engine.variable` — 4, `engine.event` — 1

```js
const E = window.aiagent.engine;
E.variable.setString('viewer-name-first', 'Bren');
console.log(E.variable.findAll(), E.variable.getString('viewer-name-first'));
E.variable.remove('viewer-name-first');

const off = E.event.subscribe([blockId], (events) => console.log(events));
off();
```

---

# Re-running the enumeration

Paste into the builder console with `?aiagent=` in the URL. Note the prototype walk —
a plain `Object.keys` misses 32 of the 48 `api` calls.

```js
const keysOf = (o) => { const s = new Set(); let c = o;
  while (c && c !== Object.prototype) {
    Object.getOwnPropertyNames(c).forEach(k => { if (k !== 'constructor') s.add(k); });
    c = Object.getPrototypeOf(c);
  } return [...s]; };

const walk = (o, path, depth, out) => {
  if (!o || depth > 3) return out;
  for (const k of keysOf(o)) {
    let v; try { v = o[k]; } catch (_) { continue; }
    const p = path ? path + '.' + k : k;
    if (typeof v === 'function') out.push(p + '(' + v.length + ')');
    else if (v && typeof v === 'object' && !Array.isArray(v)) walk(v, p, depth + 1, out);
  }
  return out;
};

const a = window.aiagent;
console.log('api:', walk(a.api, '', 0, []));
console.log('engine:', Object.fromEntries(
  ['block','editor','scene','asset','variable','event','reactor']
    .map(ns => [ns, keysOf(a.engine[ns]).filter(k => typeof a.engine[ns][k] === 'function').length])));
console.log(a.instructions);
```

Arities printed by that script come from `Function.length`, which stops counting at the
first default or rest parameter — read them as "required arguments", not "arguments".
Diff the result against this file, and read `window.aiagent.instructions` for anything
new: that text is the editor's own signature list, and this document is a map of it.
