> **Reference for the myiDecide Presentation Builder skill.** Verified platform
> behaviour and footguns — every entry was proven live before it was written
> down. Read the relevant section when something behaves unexpectedly.

# Verified platform facts (myiDecide / CE.SDK v1.74.1)



Every item below was proven live in the builder and re-verified after a full page
reload. These are the rules the extension's inject scripts must follow — several were
learned the hard way and silently corrupt a deck if ignored.

Last updated 2026-10-20.

> **NEVER call the img.ly export family — `engine.block.export`, `exportVideo`,
> `exportWithColorMask`, `exportAudio`, `cesdk.utils.export` (Bren, 2026-09-28).**
> img.ly METERS every one as a billable export against the myiDecide licence;
> the review snapshots were counted that way and pushed the account into a
> higher, costlier tier. Not once, not "to compare", not because the method is
> on the BlockAPI page — it is on the API and it is banned here. To LOOK at a
> slide use the engine's PREVIEW path (the calls the timeline itself uses):
> `engine.block.generateVideoThumbnailSequence(page, 296, t, t, 1, cb)` → one
> `ImageData` of the composed page at time `t`, or
> `engine.block.generateThumbnailAtTimeOffset(296, t)` → a PNG `Blob` of the
> current page (browser only). The extension wraps both as
> `IDP.pageThumbnail(pageId)`; `scripts/package.mjs` refuses to zip a tree that
> calls an export. Full record: verified-platform-facts.md § 2026-09-28.

> **Dual-environment rule (Bren, 2026-08-07).** This file and the project runbook
> (`WORKFLOW.md` / `AIAGENT_API.md` / `BUILD_NOTES.md` at the repo root) must BOTH be
> updated whenever a platform behaviour, standing preference, or build technique is
> learned or changed — and so must the code that enforces it (`composer.js` for wiring
> names and composition, `pipeline.js` for phase runners, `sh/*` for the helper toolkit).
> The extension and the agent runbook drive the same platform through different code
> paths, so a finding recorded in only one place is lost to the other. Prose alone is not
> enough.

---

## 2026-10-20 — the stock calls live: ranked search, the query ladder, Pexels as the last rung

Everything in the 2026-10-17 section below was VERIFIED LIVE on my.idecide.com on
2026-10-20 (deck 302 for read-only searches, test deck 319 "Pixabay Ranking Test
2026-10-20" for every import) — with these corrections and additions:

- **Both call families are live on the server.** `api.assets` carries `searchImages`,
  `searchVideos`, `importPexelVideoBatch`, `importPexelImageBatch` AND the stock family;
  `currentSlide.images` has `insertPexelImage`, `replaceWithPexelImage`, `insertStockImage`,
  `replaceWithStockImage`; `currentSlide.videos` has `insertPexelVideo`, `insertStockVideo`,
  `insertPexelBatchVideo`, `insertImportedVideo`, `replaceVideoWithPexelBatch`,
  `replaceVideoWithImported`. The Pexels searches still answer.
- **The provider-neutral import takes a Pexels ref.** `importStockVideoBatch([{provider:
  'pexels', id: 854110}])` → success in 4.0 s, 4 renditions, uri under
  `r2-dev…/stock/pexels/videos/`; `importStockImageBatch([{provider:'pexels', id:6769779}])`
  → 5 renditions under `/stock/pexels/images/`. So the extension's Pexels FALLBACK is only a
  Pexels SEARCH (`searchVideos` / `searchImages`) normalised into the stock shape; the
  import and the placement are the stock calls. A Pexels PHOTO's id lives only in its url
  (`/photos/<id>/`); a Pexels video result carries `pexelVideoId`.
- **The endpoint is `POST /api/builder/stock/{videos,images}/search` `{query, page
  (1-based), perPage}`.** The bridge sends `perPage` = `defaultPhotoPageSize` = 20 and
  converts to 0-based pages. The server honours `perPage` up to **80** (asked 200, got 80);
  `SH.stockSearchRaw` asks it directly for 80 and falls back to the bridge's 20. It IGNORES
  `provider`, `orientation`, `order`, `video_type`/`videoType`, `image_type`/`imageType`,
  `min_width`, `editors_choice`, `category`, `safesearch`, `lang` — identical ids back for
  every spelling. The request to pass them through is
  `docs/requests/2026-10-20-stock-search-filters-for-kevin.md`.
- **Pixabay's `type` is not surfaced** (film/animation; photo/illustration/vector), nor
  views/likes/downloads. An IMAGE's `pageUrl` says its kind (`/photos/<slug>-<id>/`;
  illustrations and vectors live under `/illustrations/`, `/vectors/`); a video's is
  `/videos/id-<id>/`, no kind. 3D renders and flat illustrations sit in video results with
  ordinary tags ("tooth, implant, dental"; "handshake, agreement, contract").
- **Tight vs loose pools — the tell is `totalResults`.** A query Pixabay can satisfy
  answers a real count (`handshake` 16, `dentist` 39, `mortgage` 5, `mortgage loan` 28 —
  every result carries the words). A multi-word query it cannot satisfy is silently
  BROADENED and reports `totalResults: 500` with mostly partial matches (`paperwork
  signing` 500 with 0 full matches; `patagonia zebra skyscraper` 500). "20 results" never
  means "20 matches"; the first landscape result of a loose pool can be unrelated
  ("handshake business deal office" → a keyboard-typing clip; "kids classroom learning
  spanish" → a book on a beach). Pixabay's default order is `popular`, so position is
  popularity, never relevance.
- **Processed output is no longer always 960x540.** A 3840x2160 source came back
  1080x608 (renditions 1080/720/640/340); a 1920x1080 source came back 960x540. The
  long-side floor (720) still holds; the old "4K takes longer" penalty is gone from the
  ranker because the bigger source is now the better output.
- **Tag spam is real:** a clip tagged with every coffee word ("coffee, powder, instant
  coffee … barista, espresso, shop") matched "barista espresso coffee shop" 100%. The
  ranker counts a hit among the author's FIRST FIVE tags (Pixabay's `label`) in full and a
  hit further down at 0.75. `ai generated` IS a tag on AI stills/clips.
- **What the extension does with all this (media.js r12, `SH.searchStock`):** every search
  answers RANKED — weighted coverage of the hint's terms by the result's tags (the first
  term is the subject; the outline prompt writes hints subject-first), a phrase bonus, a
  subject bonus, penalties for non-footage tags, the orientation / duration / resolution
  gates underneath; a loose first pool or a weak best match earns a parallel LADDER (the
  hint minus one term each, the first two, the subject alone) ranked against the full hint;
  Pexels only when Pixabay answered nothing. **The quality floor (the Patagonia build,
  same day):** a best match under 40%, or one whose subject is unmatched, widens to the
  slide's other query (`photo.query` ↔ `imageHint`, `altQuery`) and the deck's
  `theme.footage` (4-6 brand-safe generic searches the outline writes; rotated per hint),
  each ranked against ITSELF — a fallback candidate that answers its own query (subject
  matched, ≥50%) leads, the hint's pool trails. A niche subject ("hands patching outdoor
  jacket workbench") otherwise leaves a loose pool whose best matched one incidental word.
  The batch paths add an LLM re-rank (one Haiku call per chunk through the panel,
  `stock_rerank_system.md`, told the deck's brand line and which candidates are fallbacks;
  `IDP.stockRerank` is the page↔panel queue, 25 s timeout → mechanical order; a re-query's
  results are ranked before they lead). Every record carries `stock.query`, `stock.why`,
  `stock.llm`. Measured against the old first-landscape pick: handshake → a real office
  handshake; classroom → a real classroom; the renders/illustrations are the Haiku pass's
  job until the server passes `video_type=film`. A review re-hint also rewrites
  `photo.query` — a photo panel searches that, not `imageHint`.
- **Zoom auto-fit is an engine call; the timeline collapse is NOT ours to click.**
  `engine.scene.enableZoomAutoFit(page, 'Both')` is the zoom menu's "Auto-Fit Page" (the
  control reads "Auto"; `isZoomAutoFitEnabled` / `disableZoomAutoFit` / `setZoomLevel` /
  `getZoomLevel` exist) — view state, nothing dirty; the extension sets it after every
  changeSlide, inject and the final landing (`IDP.tidyView` / `TIDY_VIEW_FN`). The timeline
  toggle is `[data-cy="expand-timeline"]` inside the `#root-shadow` shadow root: its
  `aria-label` is the state ("Collapse Timeline" = expanded; `aria-pressed` never changes),
  `click()` toggles it, an in-app `changeSlide` keeps the state, a page load starts it
  expanded. By hand a click on sight held — but under a build CE.SDK 1.74's layout code
  fights it (forces the expanded height back, runtime errors; Bren, second Patagonia build,
  2026-10-20), so the collapse was removed the day it was added. Leave the timeline alone
  until the platform exposes a call for it.
- **The focus-tab bridge is TESTER-ONLY.** `package.mjs` strips `resolve.js` (and
  `web_accessible_resources`) from the Chrome Web Store build (the 0.9.182 hardening), so
  `window.postMessage({__idp:"focus-tab"})` is answered only by the tester build or an
  unpacked load — on the store build `data-idp-base` is never set and the message is
  ignored. A tab driven through the Claude-in-Chrome javascript bridge loads
  `window.aiagent` and the engine in the background anyway (measured: behind another tab,
  and with the Claude app covering Chrome) because the bridge attaches the debugger; the
  Extension's own connect pulls the tab forward itself (`bringBuilderForward`).

---

## 2026-10-17 — stock media goes provider-neutral (verified live 2026-10-20 — see above)

Sources: "AI agent bridge: stock media API migration" (2026-10-17) and the
builder code it describes (iDecideServer `28d155cda`: `react-app/src/api/stock.ts`,
`aiagent/aiAgentBridge.ts`, `diy/pixabay/*`, `diy/stock/StockBatchImportService`).
Written from the source before the release was deployed; every item was then proven on a
live page on 2026-10-20, and the section above records what differed.

The extension now sources every photo and clip through the provider-neutral calls
and **no longer imports or places through the Pexels family** (`importPexelImageBatch`,
`importPexelVideoBatch`, `insertPexelImage`, `insertPexelBatchVideo`, `replaceWith…Pexel…`)
**or the engine's `pexelVideos` / `pexelImages` asset sources** (`SH.applyPexelVideo`,
`preResolveVideos`, `resolveSlideClipLegacy` are gone). The Pexels SEARCHES
(`searchImages`, `searchVideos`) survive only as the last rung of `SH.searchStock`
(2026-10-20), and what they find is imported by the stock batch. The panel refuses to
connect to a builder page without the stock calls — error **TAB-06** (hard-reload the tab).

| Was | Now |
| --- | --- |
| `assets.searchImages(terms, page)` → `photos[{url, alt, …}]` | `assets.searchStockImages(terms, page)` → `{ provider, page, perPage, totalResults, images: StockImage[] }` |
| `assets.searchVideos(terms, page)` → `videos[{pexelVideoId, url, …}]` | `assets.searchStockVideos(terms, page)` → `{ …, videos: StockVideo[] }` |
| `assets.importPexelImageBatch([url or {url,name}])` | `assets.importStockImageBatch([StockImage or {provider, id, name?}])` |
| `assets.importPexelVideoBatch([{pexelVideoId}], cb)` | `assets.importStockVideoBatch([{provider, id, slideId?}], cb)` → results carry `provider` + `stockId` |
| `currentSlide.images.insertPexelImage(url, …)` | `currentSlide.images.insertStockImage(StockImage or ref, …)` |
| `currentSlide.videos.insertPexelBatchVideo(rec, …)` | `currentSlide.videos.insertImportedVideo(rec, …)` (name only) |

- **Import by provider + id, never by URL.** Every `StockImage` / `StockVideo` carries
  `provider` and `id`; pass the result itself. Pixabay URLs carry no id and expire.
- **Never place, fetch or store `previewUrl` / `thumbUrl`.** Temporary (Pixabay's last
  24h) and placing them hotlinks the provider. The Logo Reveal cover used to FETCH a
  Pexels url and re-upload the bytes; it now imports the still and uses the import's
  `meta.uri`.
- **Two ids on an imported video:** `id` is the upload's, `stockId` the provider's.
  Clip records in `IDP.videoUris` now carry `provider`, `stockId` (also `assetId`),
  `uploadId`, and `stock` (provider · id · label · tags · author); `pexelVideoId` is gone.
- **Repeat imports are instant** — the server reuses an asset any session imported.
  (§ "Batch video import" below says there is no server-side dedupe; that was the
  Pexels-era endpoint.) The run's `used` sets now hold `provider:id` keys.
- **Pages are 0-based, 20 per page** (the old still searches asked for page 1 — the
  SECOND page — and now ask for 0). A query answers at most 500 results.
- **Pixabay matches keywords.** `label` is the first five tags; `tags` is every distinct
  keyword (empty on Pexels); safe search is on; a blank query returns editor's choice;
  photos import at most 1280px wide. Hints are now written as 3-6 concrete keywords, and
  `SH.searchStock` retries an EMPTY answer with the query's content words, then its
  first three and first two (an error is not retried; q is capped at 100 chars).
- **Tags reach the LLMs that judge imagery:** the vision reviewer gets each slide's
  STOCK MEDIA (label + keywords) beside the snapshot and may re-hint footage that
  contradicts the slide; the revision chat gets the deck's STOCK MEDIA list once a turn
  (to choose an `addMedia` clip). Since 2026-10-20 the ranker is tag coverage first (see
  the section above) and a Haiku re-rank confirms the batch picks.
- **Attribution:** the extension never shows search results to a person, so the
  provider/author attribution the providers require for result lists does not arise;
  `stock.authorName` / `authorUrl` / `pageUrl` are kept on every record if it ever does.
- **`insertImportedVideo` takes our synthesized records** (read from the bridge):
  `insertPexelBatchVideo` is now a one-line alias of it, and it refuses only a record
  with `success === false` or no `meta.sourceSet` — so the `{id, label, meta:{uri,
  sourceSet}}` records the extension builds, and plain upload records, still place.
- **The bridge swallows search failures:** a failed `searchStock*` resolves to an empty
  page with `provider: ''` (a real answer always names its provider). `SH.searchStock`
  treats that as an error and does not retry it shorter.
- **Pixabay is asked for its default order (`popular`)** — the server sends no `order`
  — so the first results are the most popular matches, not the most relevant. The
  server also cuts `q` at 100 characters, mid-word if need be; `SH.searchStock` trims
  at a word boundary first.

---

## 2026-09-03 — tester-run findings

Three tester build records and Bren's own rebuild, read against the logs.
Each item names the code that now enforces it.

### Slide 1 has exactly ONE action element

§13 below already says it: **never set actions on slide 1 — any click there
starts the presentation.** What the records showed is the other half of that
fact: a *second* button on the cover is not harmless, it is a defect. The
planner wrote `copy.items = [{label:"Get Started", target:"Welcome"}]` on the
cover; archetype O drew the static "Click anywhere to Begin" pill, then the
"nav items are buttons, everywhere" pass built a wired `btn:Welcome` pill
beside it, the wire pass tried to give it an action the platform ignores,
and the design reviewer flagged a "missing Begin button". Now:

- `composer.js compose()` strips wired items from a `kind:"cover"` plan and
  skips the nav-button row (`cover: slide 1 advances on any click — dropped N
  wired item(s)…` in the log); archetype O no longer draws the accent chip.
- `pipeline.js wireSlideBlocks` skips the cover (first slide of the deck or
  `kind:"cover"`) and removes any action record already sitting on a
  `btn:`-named block there; `buildNav` never calls it for the cover.
- `applyEdits`: `setAction` / a targeted `addButton` on the cover are refused
  with "slide 1 advances on any click — the platform never carries button
  actions there; nothing to wire"; `delete` of a wired button on the cover is
  ALLOWED (it is the duplicate).
- `review_system.md` / `revise_system.md` / the plan prompts carry the rule.

### The editor does not finish booting in a BACKGROUND tab

After the panel appended `?aiagent=` and reloaded, the tab was behind another
one. The API trace stops after `GET builderSession` — no templates, no
uploads, no blockactions calls — and `window.aiagent` never appears. The
mount probe returned `{ok:false, err:"no engine"}` instantly, but the thrown
message was hard-coded to the "45s / no current page" wording, so every retry
"failed after 45s" in the same second. **Activate the tab
(`chrome.tabs.update({active:true})` + `chrome.windows.update({focused:true})`)
before waiting on the editor, and wait for `window.aiagent.engine`, not just
`window.aiagent`.** The same applies to every navigated pass — img.ly needs a
visible tab for `changeSlide` / snapshots (preview-path thumbnails since 2026-09-28 —
`block.export` is BANNED, see that section); backgrounded runs
lost slides to 30 s stalls in two of the three records.

### Colour changes go through `IDP.recolorDeck`, never a rebuild

A `theme` change in the revision chat rebuilt all 45 slides through the plan
with the palette registered at connect time (`SH.init` colours = the OLD
accent, and the composer's `C()` reads `IDP.job.theme.paletteRoles`, also
old) — so the rebuild painted the orange back on. Now:

- `IDP.recolorDeck(pairs, names, opts)` — detached like `buildNav`; walks each
  slide as it stands and repaints solid fills, gradient stops, strokes, text
  (whole block when uniform, per-run spans when mixed — `getTextColors(id,i,i+1)`
  scans), the page background (fill + `backgroundColor/color`) and icons whose
  recorded `idecide/iconTone` matches (re-placed via `uploadAndInsertImage`
  when `IDP.iconUris` has `<tone>:<concept>`, else reported in
  `missingIcons`). Per-channel tolerance 0.035; the layer's alpha is kept.
  Each touched slide gets the applyEdits commit pair (documented dirty +
  `exportAndPutSlide`) and the closing `commitEdits` navigation. Result in
  `IDP.recolorResult`, settle flag `status().recolorFinished`.
- `IDP.registerTheme(theme)` re-runs the colour half of `IDP.init` (the SH
  token map AND `IDP.job.theme`) without the font lookups. The panel calls it
  after any theme change. A theme change schedules NO rebuild.

### A silent marker is silence

The script model wrote the literal "(NO VOICEOVER)" into rows;
`restoreDroppedNarration` restored it "from the script (2 words)"; the voice
service spoke it (1.0–1.7 s clips) or failed with "audio missing after
generation". `IDP.isSilentNarration(text)` is the one test — empty,
punctuation-only, `(NO VOICEOVER)` / `(no voice over)` / `(no narration)` /
`[silent]` / `(none)` / `none` / `n/a` / `—`, with or without brackets and
periods, and a leading marker such as `(NO VOICE OVER) Get Started` — and
`buildNav`'s `wantText`, the Slide Notes writers and the wire pass all use
it: no TTS call, no audio-missing error, no notes, default timing. The
script contract now says a silent beat is an EMPTY Script cell.

### A CSV row with a blank Slide Name is a slide that never exists

Tester run 2 left the Slide Names cell blank on the answer rows.
`reconcilePlanNames` dropped "Answer 1 - Correct/Incorrect" (no CSV
identity), `normalizeWiring` kept the now-unresolvable `target`, the dead-
option fallback never ran, and the wire pass failed with `unknown target` on
every visit — a viewer who answered would have hit a dead end. The script
prompts now state, bluntly, that every row carries its Slide Name; the panel
fills blank names from the outline order, keeps plan slides that are
referenced as targets, and drops unresolvable targets so the fallback
re-points them.

### Anthropic API — RETIRED 2026-10-16 (the extension no longer calls it)

Until 2.2.6 the extension called `api.anthropic.com` from the browser with a
key the person pasted, and an identity-linked key made with no workspace
chosen needed an `anthropic-workspace-id` header on every call (second tester
run, 2026-09-03; `KEY-05` / `KEY-06`). Since 2.3.0 no call leaves the
extension for Anthropic — see the next section — so the header, the key UI,
the Test-key diagnostic and the `KEY-*` codes are gone. The finding stays here
for the record only.

---

## 2.3.0 — every AI call goes through the myiDecide backend (lead developer, 2026-10-15; verified on his walkthrough + Bren's charity: water build 2026-10-16)

The Builder's backend proxies the Anthropic Messages API for the extension at
`<builder>/api/extension` (`config.js`: `IDP_API_URL = IDP_BUILDER_URL +
"/api/extension"`; production `https://my.idecide.com`, dev
`http://localhost:8099`). It authenticates with the person's **myiDecide
sign-in cookie** — the panel is an extension page with `host_permissions` for
the origin, so `fetch(…, { credentials: "include" })` sends the cookie and CORS
does not apply — runs the call on iDecide's own Anthropic account, logs tokens
and real cost per user, and meters each user against a **daily allowance**.

- `GET /user` → `{ firstName, lastName, email, profileImageUrl }` for the
  signed-in person; `401` when there is no session. The panel calls it on
  open, on visibility, before a build, and every 4 s while the sign-in bar is
  open.
- `GET /usage` → `{ usedUsd, limitUsd, remainingUsd, percentUsed, resetsAt, … }`
  (and `tokensUsed`). The panel shows the percentage only (the footer meter
  and the Settings card); refreshed when every Messages stream ends, on
  sign-in, on visibility, when Settings opens, and just after `resetsAt`.
- `POST /v1/messages` — the Messages API, proxied unchanged, with these
  rules the proxy enforces (a violation is a `400`): `stream: true` on every
  call; only the four models in `API_MODELS`
  (`claude-fable-5-1`, `claude-opus-5-5`, `claude-sonnet-5-5`,
  `claude-haiku-5-5`); `max_tokens ≤ 32000`; tools are `custom` ones plus
  the two server tools **`web_search` and `web_fetch`** (the lead developer
  let them through on 2026-10-16, evening — his patch restores
  `RESEARCH_TOOLS` and drops the `anthropic-beta: web-fetch-2025-09-10`
  header, web fetch being GA; the proxy also accepts the `web_*_20260209`
  variants); only the server-side-fallback beta header. Web research is
  therefore ON again in the script stage — the planner reads the client's
  site and searches for the figures the script needs; `theme.scriptEvidence`
  records every one. (Earlier the same day the proxy refused every non-
  `custom` tool and the prompts said "no web access"; emptying
  `RESEARCH_TOOLS` turns that back on.)
- `GET /v1/models` — the model list, through the same proxy (the picker's
  catalogue when it is switched on).
- `401` on any call = signed out: the panel opens `<builder>/login` once per
  15 s burst, shows the sign-in bar, and re-checks `/user` when a myiDecide
  tab finishes loading a non-login page (`chrome.tabs.onUpdated`, no extra
  permission). `429` with `x-should-retry: false` = the daily allowance is
  spent: fatal for the run (no retry loop until midnight), `ACCT-02`.
- Every request carries `anthropic-version` and `x-client-version` (the
  manifest's version) so the backend can tell builds apart.


### The person never sees the AI vendor's name (Bren 2026-10-16)

The first evening on the proxy put "Your credit balance is too low to access
the Anthropic API. Please go to Plans & Billing to upgrade or purchase
credits." on a client's screen, verbatim, under `API-01`. Two rules since:
the refusal gets its own code (`ACCT-04`, iDecide's own account out of
prepaid credit — not the person's allowance), and every error that reaches
the UI goes through `IDE_ERRORS.forUser()` — an `API NNN: {json}` refusal
reduced to its human message and request id, vendor names / model ids /
vendor-side instructions removed; the four `ACCT-*` codes show no detail at
all. The log and the record keep the raw text. The KB and the help page name
no vendor; the finished screen names models only when the picker is on.

### The person chooses nothing (Bren 2026-10-16) — the selectors are switches

`FEATURES` in `config.js` (`modelPicker`, `effortPicker`, `advancedSettings`,
`buildModePicker`) are `true`/`false`; all four are **off**, and `LOCKED`
(`claude-opus-5-5` · `high` · `quick`) is what every call uses. A switch that
is off forces its value into the stored settings at boot (`lockSettings()`),
hides its pill, and the Settings note says what is locked. A deck opened for
REVISION still follows the mode it was built in (`MODE_PIN`), so a 2.2.x
Creative Build keeps composing scenes with the switch off. Build records
carry `settingsUsed.features` so a run says which switches it had.

### A fresh install opens the help page at its next step (2.3.0)

`chrome.runtime.onInstalled` with `reason === "install"` (never `update`)
opens `https://idecide.com/extension-help/#pin` in a new tab — the step after
installing (pin the icon, open the panel), with sign-in and the first build
below it. Every step on that page is a stable anchor (`#install`, `#pin`,
`#login`, `#open`, `#build`, `#usage`, `#history`, `#share`); the 2.2.x ids and
the retired `KEY-*` codes redirect.

### Review frames: both preview calls can time out — stop asking (charity: water, 2026-10-16)

On Bren's 58-slide Quick Build (deck built 16:16–16:43), **every** slide that
asked for a review frame got neither: `generateVideoThumbnailSequence` gave
no callback in 20 s and `generateThumbnailAtTimeOffset` no Blob in 20 s, on
seven slides in a row — 4 min 40 s of a 26-minute build waiting for frames
that never came, and "thumbnail primary failed …" painted on the build card
each time while the review carried on from layer dumps. (Session 222 on
2026-09-28 had the primary answering in ~30 ms; something between the Builder
and this Chrome changed, and it is not yet known what.) Since 2.3.0
`pageThumbnail` gives the primary 8 s and two strikes, then the direct call
12 s and two strikes, then switches frames OFF for the page session (callers
degrade to the layer dump at once; a reload starts clean), and none of it
reaches the card — `note()` writes the log and the console only. The record's
`metrics.thumb` says how the frames were drawn.

### The record's trace is a hook on the SHARED engine — an engine line is anyone's (lead developer, 2026-10-16)

Reading the same charity: water record, the lead developer asked what
`engine.block.setSourceSet(85, "fill/image/sourceSet", [])` followed by
`setString(85, "fill/image/imageFileURI", "https://r2-dev…/images/23fc11a5…")`
was, and whether the extension was setting fills through CE.SDK again. It
was not, on either count. **What the trace is:** the panel wraps the mutators
on `window.aiagent.engine.block` IN PLACE at injection; the myiDecide Builder's
own code calls the same object, so every mutation on the page is recorded —
the extension's and the platform's api inserters' internal work alike (reads
are skipped). Both quoted runs are the platform's inserters: block 85's owner
is `brand-mark` (the 128×40 corner mark at 86,36) placed by
`api.currentSlide.images.uploadAndInsertImage(blob…)` — the `POST
/api/builder/builderSession/225/uploads` two lines earlier is that call's own
upload, and `create → rect → image fill → setFill → setSourceSet([]) →
imageFileURI → Cover → appendChild → setSize → setPosition → resetCrop →
adjustCropToFillFrame` is what the platform's uploaded-image inserter writes
(an EMPTY sourceSet with the imageFileURI is ITS convention for an uploaded
file, exactly as an empty `fileURI` with a 4-entry sourceSet is its convention
for a clip — see 2026-09-28); the extension's own calls follow once the id
comes back (`name`, timing, Contain, `insertChild`, destroy the placeholder).
`setSourceSet(24117313, "fill/video/sourceSet", [{uri:…/videos/4861a903…}])` is
the full-slide `ghost-video` on Main Menu - Return (Pexels 33733737, imported
by `importPexelVideoBatch`) placed by `insertPexelBatchVideo(record, 0, 0,
1558, 720, 0, null)`: `create → rect → video fill → setFill → setMuted → Cover
→ appendChild → fileURI "" → sourceSet → setSize → setPosition → resetCrop`.
`setMuted`, `setSize`, `setPosition` and `resetCrop` are not called anywhere
in the extension — they are the inserter's fingerprint. **Across the whole
record:** 108 `setSourceSet` lines — 62 video, every one `fileURI ""` + the
batch's renditions (insertPexelBatchVideo); 46 image — 41 uploaded
(37 brand-mark + 4 brand-logo, uploadAndInsertImage) and 5 Pexels stills with
5-rendition sets (insertPexelImage); ZERO non-empty `fill/video/fileURI`
writes and ZERO `playback/looping|muted` setBools, the fingerprints of the old
`SH.applyVideoUri` / `applyPexelImage`. That matters because this record was
built on **v2.2.5** (the panel log's first line) — the developer's test build
is his zip, whose base commit is 2.2.5, so it still CARRIES those helpers;
none ran. 2.3.0 (the build on idecide.com/builds) is his four commits on top
of 2.2.6, where they throw and `package.mjs` refuses a tree with a fill write.

**Why the runs looked orphaned, and the fix.** `api.currentSlide` is a getter
on the api's prototype (docs/AIAGENT_SURFACE.md caveat 1); the tracer walked
`Object.keys`, never saw it, and so no `insertPexelBatchVideo` /
`uploadAndInsertImage` / `insertUploadedImage` line was ever written while
every engine call they made was. Since 2.3.0 the tracer shadows the getter on
the instance and wraps what it returns (`get` and `blockActions` excepted —
the wiring step hand-traces those with the button's name and target), so the
api line now precedes its engine work, and every trace opens with a legend
line (`TRACE_LEGEND`) saying what an engine line is. smoke-panel checks both.

### A profile page is not the brand's domain (test user, 2026-10-16 — the Instagram wordmark became the client's mark)

A test user with no site of her own gave `instagram.com/inkbeedle` as the
website. Brandfetch, asked about `instagram.com`, answered with Instagram's
own brand data; the planner recognised it ("Instagram the platform, not the
artist", `theme.logoCandidates: []`, palette from the stated orange and
cream) — but the build-time logo hunt is fed from the Brandfetch files
whatever the planner decided (22 candidates, `cdn.brandfetch.io/ido5G85nya/…`),
found the dark SVG, recoloured and uploaded both tones, and the deck carries
Instagram's wordmark as `brand-logo` / `brand-mark` on 41 slides and in the
Logo Reveal. Since 2.3.0 `profileHostOf(domain)` (the panel's research step,
`PROFILE_HOSTS`: the social, marketplace, link-in-bio, portfolio, code,
video, music and site-builder hosts, matched on the host and its subdomains)
makes the research step skip Brandfetch and the CDN candidate, log why, and
tell the planner the site is a profile page (`brandFactsDirective()`); the
hunt goes by name only. The script side was already right: the writer named
the brand "Inkbeedle" from the handle and never used the sentence the person
typed into the brand-name field.

### The footage wave's "fell back" never said why (same record)

Three chunks of ten searches "fell back" in half a second each, then 0/33 at
the review pass, and every one went to the slow per-slide path — and the
record could not say whether the search answered nothing, errored, or every
candidate was spent, because `r.failures[k].err` was tallied into
`IDP.clipFailures` and never printed. Since 2.3.0 each chunk's log line is
followed by a `note()` with the distinct reasons tallied, and the per-slide
"could not source it" line is a `note()` too (a recovery; it was painting the
card) and carries the wave's reason for that key. The cause of this build's
fallbacks is therefore still OPEN — the next record will say.

### Review frames: the primary answers nine times, then stops (two 2.3.0 records)

YETI (18:10) and the illustrator's deck (18:28), the first two 2.3.0 builds:
`metrics.thumb` = `primaryOk 9, primaryTimeouts 2, fallbackTimeouts 2, off`
on both — nine storyboard frames in well under a second each, then the tenth
and eleventh time out at 8 s, the direct preview times out at 12 s on the
same slides, and frames go off. The slides that time out share nothing
(an icon row, a photo trio, a bio, a logo reveal). The strike policy holds
the cost to ~40 s a build; why the engine's thumbnail path stops answering
after nine calls in a page session is for the lead developer (the 2.2.5
charity: water build had the fallback timing out from the first call).

### A cut script is finished, not planned from (three of the day's four 2.3.0 builds)

YETI 17:54 ("STOCK: gear lineup laid out across a tailg"), the sample deck
18:48 ("…weeks before it would have become a serious problem. … Pulse g"),
YETI 19:24 ("…when something does go wrong, it"): the script CSV stopped
mid-cell, the `===OUTLINE===` marker and a complete outline followed, and the
record could not say whether the model stopped writing or the proxy dropped a
chunk (the trace holds requests, not the stream). What happened next was the
real damage: `repairBlankSlideNames` gave the nameless fragment the NEXT
outline name, every later outline slide was "dropped (no CSV identity)" —
Finish Up 1–4 and the Hamburger Menu — both menus were rebuilt without them,
and the audit put "no Finish Up chapter — the viewer has no way to choose to
leave" on the card. The 47-row illustrator deck, the shortest of the four, was
whole. Since 2.3.0 (evening) the script stage (1) logs the reply's stop
reason, the model's own `output_tokens`, the characters received, the number
of text blocks and any unparseable `data:` lines — the evidence for next time
(tokens ≫ chars/4 = text lost in transit; `end_turn` with a fragment = the
model stopped); (2) `scriptCutDiagnosis()` recognises a cut — a last data row
with no Slide Name or fewer cells than the header, a `max_tokens` stop, or an
outline tail after the last named row that names a Finish Up / Hamburger
slide the CSV lacks; (3) `continueScriptCsv()` makes ONE more streaming call
through the same proxy — the same system prompt (a cache hit), the answers,
the script so far, the outline's entries for the missing slides — for the
missing rows only, drops a header or fence it may add, ignores rows whose
names the outline does not have, logs what is still missing, and appends the
rest before `refinePlan` runs. The per-row detection was proved on the three
real fragments; the continuation call is exercised by the next cut build.

**What the cut is (A.M.WakeUpCall, 21:44, the fourth cut record).** Every
cut record shares one more signature the whole ones lack: a 69–107 s AI call
at the top of the build before the first plan line — the OUTLINE FALLBACK in
`runBuild`, which runs only when the combined reply carried no parseable
outline. So in every case the whole reply was cut, CSV and outline together,
which is what the output ceiling does: `max_tokens` is 32,000 on the proxy
and extended thinking at effort high counts against it; a 60-slide script
plus its outline plus the thinking that precedes them does not always fit
(the 63-row charity: water script did, the 62-slide A.M.WakeUpCall one was
cut at row 32 and the fallback outline planned 62 slides from 31 rows — 30
dropped). The record could not show it because `runBuild` began with
`buildLog = []` and wiped everything the script stage had logged. Since
2.3.0 (late): generate()'s log survives into the record (`keepBuildLog`);
every call's usage rides in `usage.log` with its stop reason (`out` includes
thinking — an `out` at the ceiling with stop `max_tokens` is the diagnosis);
with no outline the continuation is OPEN-ENDED ("finish the script to the
Hamburger Menu"), up to three rounds while a reply keeps hitting the ceiling,
and the fallback outline then plans from a whole script. Open (owner: lead
developer): a higher `max_tokens` ceiling on the proxy (64,000) would make
the cut rare instead of handled.

**Proved on the Stripe record (22:37, the first built with the evidence).**
`usage.log[0] = {stage:'plan', out: 35802, stop:'max_tokens'}` and the
`script reply:` line: 26,198 characters received (CSV 20,007 + outline 6,191)
— the reply ran out of output budget with the thinking counted in, exactly the
mechanism above. This time the cut landed in the OUTLINE: the CSV was whole
(Finish Up 1–3 and the Hamburger Menu present), so the continuation had
nothing to add and the fallback outline planned from a whole script; the deck
was complete. 52 slides, 22.8 min, 19 calls.

### The "Importing Video" dialog is the per-slide fallback, not the wave (Stripe, 2026-10-16)

Bren: "the uploading video popup from img.ly came up a few times throughout
the build, which I thought our aiagent bulk video upload call avoided." The
trace says the bulk calls were used throughout: 9 `assets.importPexelVideoBatch`
(the wave, in chunks), 63 `currentSlide.videos.insertPexelBatchVideo`
placements, 20 `assets.importLottieBatch` → 66 `images.insertUploadedImage`,
39 `images.uploadAndInsertImage` (icons, the logo, stills). The twenty dialogs
were the twenty single `POST /pexels/import/video` requests of the PER-SLIDE
fallback — the path a slide takes when the wave could not source its key (here:
139 searches into the Pexels hourly limit, the searches answered empty, plus a
few "the server could not process pexels N"). That fallback resolved through
the ENGINE's asset source, `e.asset.apply('pexelVideos', …)`: the platform's
single import, which shows the dialog, creates a temp block, polls the CDN and
returns ONE rendition (960×540 — the table at the top of this file). Since
2.3.0 (late) `SH.resolveSlideClip` runs one key through `batchResolveClips`
(the wave's search), tries the engine's asset source as a second SEARCH when
that answers nothing, and imports through `importPexelVideoBatch` either way —
no dialog, four renditions; `asset.apply` survives as `resolveSlideClipLegacy`
for a Builder bundle without the batch surface. Not yet exercised live.

### Three geometry faults every Quick Build deck carried (Stripe, 2026-10-16; mock-replayed on the day's eight records)

Bren saw two on the Stripe deck — "extra back plates were added behind the
lotties in some slides, which caused some overflow" and "on a number of final
cta slides throughout builds, the logo appears to get placed below the contact
info lines" — and the mock engine, replaying the eight build records of the
day through `__COMPOSE`, found both on every deck plus a third: 24 card wells
outside their card, 21 card bodies packed BELOW their card, 26 logos on the
contact lines, across 379 Quick Build slides. The mock agreed with the engine
to the pixel where it could be checked (the Stripe record's own log line
`icons: 6 stacked icon(s) enlarged — card/icon/box ×1.76, card/icon ×2.00 …`;
the sample deck's record packing `card/bg+11+itemBody+2`; the screenshot's plate 10 px
left of the card). The three causes and fixes are in element-contract.md ("A
card holds its own rows; the well rides with its glyph") and the CHANGELOG;
the short form: the stacked-icon pass tested the well's NAME for `icon-well`
when the well is named `<glyph>/box` (the role is `icon-well`), so the well
was grown as a second icon; the packer's plated-row rule paired a card's body
off the card's bottom edge; the terminal CTA's `brand-logo` was pinned as
architecture while the sender lines packed onto it. After the fix the same
379 slides replay with 0 / 0 / 0, and the six text overlaps that remain
(LiftProtocol "Inside the Plan - 2", the sample deck's "Not Right Now") are the same
six as before — the stacked-buttons fault already on the open list.

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
result can be matched to its slide. `meta.uri` is a plain `.mp4`. ⚠ The 08-22 finding that
applying it to a block WE created "renders identically" is RETIRED: an engine
fill write is banned (Bren 2026-08-24, restated by the lead developer
2026-09-28) — hand the whole record to `insertPexelBatchVideo(record, …)`,
which sets the four-rendition sourceSet itself. See § 2026-09-28 (2.2.6).

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

## Rotation pivots at the block's TOP-LEFT (verified 2026-09-25, deck 311 / 44313)

`block.setRotation(id, radians)` turns the block about its `getPositionX/Y`
point — the untransformed top-left corner — not its centre. Read live on the
demo mortgage slider's value-bubble tip: a 20×20 rect at (501.2, 417.1)
rotated 45° reports a global bounding box of 487.1 → 515.4 (28.3 wide),
centred on x = 501.2 = its own position x = the bubble's centre x. So a tip
that must hang centred under a bubble is DRAWN with its x at the bubble's
centre; a shape that must spin about its centre is drawn offset by the
rotated corner. The scene contract's `rotate` (degrees) inherits this.

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
call times out and only a browser reload recovers. ⚠ The 07-30 answer that stood here
(create a video fill, write `fill/video/fileURI`, `setFill`) is **BANNED** since
2026-08-24 and restated by the lead developer on 2026-09-28: a fill WE write carries one
rendition — the large file — and crashes phones. For already-hosted media, get the
platform's record and let the api insert it:

```js
const rec = await SH.findUploadRecord(uri);                    // {id,label,meta:{uri,sourceSet[4],…}} from the library listing
const bid = await A.currentSlide.videos.insertPexelBatchVideo(rec, x, y, w, h, 0, null);
// then move geometry/timing/marks from the old block to bid and destroy the old one
// (composer.transferDress) — never the other way round
```
`api.currentSlide.videos.uploadAndInsertVideo(file, x, y, w, h, 0, null)` exists on the
live api (verified 2026-09-28) for a File you still hold.
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

## Duplicating a deck RESETS slide 1 to the stock template (verified 2026-08-28)

Duplicating a presentation from the myiDecides list copies every slide except
the opener. Slides 2..n arrive intact — text, media, narration, timing, wiring.
**Slide 1 comes back as the stock "Play Button" welcome**: `Welcome` at 90px,
the placeholder images and shapes, and the two 24px `[sender-name]` /
`[sender-email]` layers that sit under our 28px floor.

Observed on a duplicate of the Northbound demo: 13 blocks, all of them
template, none of them the cover that was there. The slide keeps the name
"Play Button" too, so a renamed opener loses its name as well.

**So a duplicate needs its cover rebuilt** — the same clear → sweep → verify →
compose pass as a from-scratch build. Anything that duplicates a deck for a
client, a template or a demo has to redo the opener or ship the stock one by
accident.

## Slide 1: clear the stock cover, then compose your own (verified 2026-08-28, deck 3)


**Standing rule (Bren, 2026-08-28): every from-scratch build clears the default
slide-1 graphics and composes a new cover.** The stock "Play Button" welcome is
a template, not a starting point — repurposing its text leaves a deck that
looks like the demo it came from.

**Slide delete is REST, not aiagent.** There is no delete anywhere on the
aiagent surface — not `api.slides` (`changeSlide`, `changeSlideByIndex`,
`create`, `createMultiple`, `get`, `getDimensions`, `setAutoAdvance`,
`setMenuSlide`, `setName`) and not `api.provider.getCtx()` (which offers only
`blocksRemove`). **But the platform deletes slides perfectly well over REST**,
and the extension has done so since well before this note:

```js
await fetch(`/api/builder/builderSession/${sid}/slides/${slideId}`,
            { method: 'DELETE', credentials: 'include' });
```

Retry once after ~600ms on a non-ok response; the builder occasionally refuses
the first attempt.

**Sweep only on a build, never on a revision.** A from-scratch build should
delete every live slide that is not in its plan — that is how a stray
"Play Button" gets removed rather than repurposed. An edit or revision pass
must never do this: a slide the user added by hand is not yours to destroy.
Make it an explicit opt-in flag rather than default behaviour.

**Prefer deleting.** Build your own slides, then purge what the platform
shipped. Clear-and-recompose is the FALLBACK for when the DELETE is refused —
and slide 1 is the one case where it is the only option, because a deck must
open on something and the extension explicitly refuses to delete the opener.

**`clearAllVisual()` does NOT clear everything.** On the stock cover it removed
6 of 13 blocks and left five text layers plus one decorative shape sitting on
the canvas — which then collide with whatever you compose. Sweep the
survivors:

```js
CS.blocks.clearAllVisual();
await new Promise(r => setTimeout(r, 900));
// template blocks ship with LOW sequential ids; anything you create gets a
// large one, so on a FRESH deck this is a safe discriminator
for (const b of await CS.blocks.getVisual())
  if (b.blockId < 1000) CS.blocks.removeBlock(b.blockId);
```

Then verify by count before composing. On deck 3 the survivors were ids
111/114/117/120/123/126 (five texts + a `gif`) and one `shape` graphic at
798,238 — a decorative bar that sat directly over the video panel.

**`getFrameHeight` returns the DECLARED box, not the rendered text.** A
two-line 60px headline in a box declared 210 tall reads back as 210, so it
cannot tell you a line wrapped. A 76px headline in a 620px column silently
wrapped to three lines and overran the block beneath it. Set explicit heights
from the type size and line count, and re-measure after any size change.

## Three shapes that bite on a from-scratch build (verified 2026-08-28, deck 3)

Found while building the Northbound demo end to end through the plugin. All
three read as "the call is broken" and are none of them.

**1. `assets.searchVideos(q)` does not return an array.** It returns
`{ page, perPage, totalResults, videos[] }`, and the items carry
**`pexelVideoId`**, not `id`. `(res||[]).find(...)` throws
*"find is not a function"*. Use `(res.videos || [])` and map to `pexelVideoId`
before handing the list to `importPexelVideoBatch`.

**2. `blocks.getVisual()` speaks a different coordinate space than the engine
setters.** Entries key on **`blockId`** (not `id` — `b.id` is `undefined` and
every engine read on it fails silently), and their `x/y/width/height` are
**viewport/preview** coordinates, not canvas: a block set to `y:230 h:104` on
the 1558×720 canvas reads back as `y:274 h:59` at a 113% editor zoom. Use
`getVisual()` to ENUMERATE, then `E.block.getPositionX/Y` and
`getFrameWidth/Height` for real geometry. Mixing the two silently misplaces
everything.

**3. Block ids do not survive a `changeSlide`.** The page re-hydrates and mints
new ids, so an id captured before a slide change throws *"Block <id> is
unknown."* on return. **Name every block you will come back to**
(`E.block.setName(id, 'menu-plate-1')`), then re-read and match by name — or by
its text via `E.block.getString(id,'text/text')`. This is the same class of
error as verifying media by filename after a save.

Also confirmed on this run: `E.block.findByKind('page')[0]` is the page id for
the rule-13 `insertChild(page, audioId, 0)` audio placement, and the platform's
default "Play Button" cover ships `[sender-name]` and `[sender-email]` at
**24px** — under our own 28px floor. A from-scratch build should raise them
(`setTextFontSize` then commit with `updateText(id, sameText)`).

## Creating a NEW presentation from the agent (verified 2026-08-25, deck 184)

> **Superseded 2026-09-04.** `POST /create/aiagent/new` creates the deck
> without a builder tab, skips the Presentation Info modal and answers an
> `editUrl` that already carries `?aiagent` — see "2026-09-04 — create
> endpoint, image batch, lottie batch" at the end of this file. Everything
> below still holds and is the FALLBACK path (a non-200 from the endpoint).

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

## 2026-09-04 — create endpoint, image batch, lottie batch (verified)

All four verified live on my.idecide.com (deck 200). The rest of the surface is
unchanged — `api.assets.searchImages/searchVideos/importPexelVideoBatch`,
`api.currentSlide.images.uploadAndInsertImage/insertPexelImage/
replaceWithPexelImage`, `api.currentSlide.videos.*`, the narration batch,
`slides.createMultiple` — and the full instructions text still lives on the
page as `window.aiagent.instructions`.

### A. A presentation can be created without a builder tab

`POST /create/aiagent/new` — same origin (`https://my.idecide.com`), JSON body
`{name: "Q4 Sales Deck", slideCount: 5}`, `Content-Type: application/json`;
the browser session cookie authenticates, so it works from any page on the
site. Response `200` JSON:

```json
{ "editUrl": "/builder/create/201?aiagent&slide=40717",
  "userSessionId": 201, "name": "Q4 Sales Deck",
  "introSlideId": 40716, "slideIds": [40717, 40718] }
```

- `slideCount` N creates **the intro slide PLUS N blank slides**;
  `introSlideId` is the platform's Slide 1, `slideIds` the blanks in order
  (the sample above is abbreviated).
- `editUrl` **already carries `?aiagent`** and points at the first blank
  slide. Navigating to it lands in the editor in aiagent mode; the
  Settings / "Presentation Info" modal is **skipped**. No `&aiagent=`
  re-navigation, no overlay to dismiss.
- A signed-out session gets no deck: the request is redirected to the login
  page (an HTML 200 with `response.redirected` set) or refused — test for that
  before reading the body.
- **There is no archive/delete call in the aiagent surface.** A deck minted is
  a deck kept: create exactly once, when a build is actually starting.

The build flow now owns this: it POSTs from the myiDecide tab it is on (the
dashboard is enough — it opens one if the active tab is elsewhere), names the
deck from the brand answer when it already has one and "myiDecide
presentation" otherwise, asks for **one** blank slide (the shells phase
creates the rest with `createMultiple` and clears whatever it did not make),
navigates to `editUrl`, waits for `window.aiagent` with the tab in front, and
carries on through the same connect path as before. The `/create/new`
navigation below is kept only as the fallback for a non-200 answer; both
paths log which one ran. The plugin's SKILL.md teaches the same endpoint.

### B. `api.assets.importPexelImageBatch(images)` — stills in one request

`images: Array<string | {url, name?}>` → `Promise<(UploadedFileResponse|null)[]>`.
One server request, **at most 50** entries, results in input order; a
non-Pexels url or a failed import is `null` at its index and does not take
the batch down. Makes **no blocks**; not page-bound. Internally `POST
/builder/{sid}/pexels/import/image/batch`. The image twin of the footage
wave: resolve the deck's stills in Phase 0.5 beside the video batch, place
them later with D.

### C. `api.assets.importLottieBatch(lotties)` — Lottie JSON in one request

`lotties: Array<string | {lottie: string, name?}>` → `Promise<(UploadedFileResponse|null)[]>`.
Each entry is the **raw Lottie JSON document as a STRING** — not a url, not a
parsed object. At most 50; makes no blocks. Returns `{id, label, meta: {uri,
thumbUri, width, height, …}}` per entry (`null` on failure). Internally `POST
/builder/builderSession/{sid}/uploads/lottie/batch`. Supply a `name`.

### D. `api.currentSlide.images.insertUploadedImage(upload, x, y, w, h, startTime|null, duration|null)`

→ `Promise<BlockId>` (**0 on failure**). Places a batch-imported image **or**
lottie (an `UploadedFileResponse` from B or C) on the current page — no server
call. For a lottie it calls `engine.asset.defaultApplyAsset({… mimeType:
'application/json+lottie', looping: true})` and then `setLooping(true)` — the
engine treats it as an `application/json+lottie` asset and **the platform
loops it**. For both it sets **Cover** fill mode + `adjustCropToFillFrame`,
then applies the size, position and timing given. Icons and lotties are
switched to **Contain** afterwards (and re-sized to their native aspect) so
they keep their whole frame; a documented `api.*` insert, so the slide is
dirty and the next `changeSlide` commits it.

**Lottie in the editor (phase 3, 2026-09-04).** The extension's animated-icon
path: catalogue match → recolour panel-side → `importLottieBatch` (≤50) →
`insertUploadedImage` → Contain + native aspect, looping, duration = the
slide's (the file is a 60 s seamless loop). Import + insert were verified live
on deck 200.

**PROVED 2026-09-05 (deck 202): the editor's Lottie renderer does NOT honour a
time-remapped loop — it plays the first cycle and freezes on the last frame;
N back-to-back precomp layers play and loop.** Method: place both builds of
the same icon (`arrows/arrow-1`, `ui/clock`, white tone) on "Find Your Sport -
3", then `engine.block.setPlaybackTime(page, t)` for t = 0.2 … 7.3 s and a
frame grab at each t, hashing the bytes (the 09-05 proof used `block.export`;
that call is BANNED since 2026-09-28 — grab frames with
`generateVideoThumbnailSequence(block, 128, t, t, 1, cb)` and hash `result.data`): the
`--mode tm` file's frames stopped changing after its ~1 s source cycle; the
`--mode layers` file's frames kept changing through every sample and repeated
its cycle. Bren confirmed both on the canvas ("your 2 test lottie uploads are
working"). Consequences: the bundled library is regenerated with `--mode
layers` (`catalog.json` → `rules.loopMode: "layers"`, gated by
`smoke-panel.mjs`), and a lottie placed from the old `tm` build on a deck
built before 2026-09-05 stays frozen until the slide is rebuilt (deck 202's
one arrow). Placed-block facts: the fill is `//ly.img.ubq/fill/video` with
`fill/video/totalDuration` 60; `playback/looping` is not readable on the
block (`getBool` throws) — read looping from the fill if it is ever needed;
the block's duration is the slide's, which trims the 60 s loop.

**PROVED 2026-09-05 (deck 205): `api.currentSlide.images.insertUploadedImage`
must be called ONE AT A TIME.** Three concurrent calls for three lottie
uploads (the materialiser's chunk of 4 in `Promise.all`) left only the LAST
promise resolving inside 12 s; the other two hung, and in the build they
eventually came back as graphic blocks with a **black colour fill**, `Contain`
set, the icon's name and metadata — the "black squares" on the Lordicon test
menu. The platform's `engine.asset.defaultApplyAsset` keeps one pending apply,
exactly like the pexel upload slot did (BUILD_NOTES "asset pre-resolution").
Sequential calls (`await` each) all resolve with `fill/video` blocks
(`getState` → `Ready`, `fill/video/totalDuration` 60.87 / 60 / 60.25 s for
the three test files). `SH.placeUploadedImage` therefore serialises every
placement through one promise chain (SH `2026-09-05-r9`); callers may still
fire placements in parallel. Corollary: an in-flight apply that never
resolves blocks every LATER apply on that page until the page is reloaded —
a harness that leaves hung inserts behind must reload before building again.
Also proved on the way: `engine.asset.defaultApplyAsset` with a `blob:` URL
returns a block with a colour fill (it does not load blob: sources) — test
lotties through `importLottieBatch` + `insertUploadedImage`, never through a
blob URL.

**Lordicon files (cdn.lordicon.com, 2026-09-05) come in three shapes, and the
engine's Lottie loader accepts all three once prepared** (`prepareLordicon` in
the library generator's `lottie-core.js` — every file is prepared before it is
hosted; the extension repo's `tools/LOTTIE-LIBRARY.md` has the rules): **A. marker
states** (`markers[]` name `in-reveal` / `default:hover-…` / `morph-…` /
`loop-…` with `tm` start + `dr` frames; each state a precomp layer limited to
its window, `st` = its start; no expressions); **B. expression state layers**
(root precomps `hover-N` / `loop-N` all spanning the timeline, opacity driven
by an expression reading a `State-…` effect slider on a `Color & Stroke
Change` null, plus a time remap; without expressions every state renders at
once); **C. one timeline** (root layers are the animation). B and C carry a
`watermark` shape layer (ip 2.5–25 frames) whose opacity only an expression
hides. Colours are two slots — primary `#121331`, secondary `#08a88a` — that
the library thresholds classify as "accent", so the recolourer is told their
roles (`colourRoles`). The 500 px canvas keeps ~15–20 % air around the
artwork: the materialiser draws a Lordicon animation 28 % larger than the
glyph box it replaces. Icons not yet on the public CDN ("latest", most of the
*system* family on 2026-09-06) are served to the site's own player from
`media.lordicon.com/icons/<family>/<style>/<index>-<name>.li` — base64 of the
same Lottie JSON XOR `0x2a` — and the harvest runner decodes those.

Toolkit parity: `sh/media.js` → `SH.preResolveImages(specs, opts)` (search
via `searchImages`, best landscape result, dedupe by url, ≤50 per request,
returns `{query → UploadedFileResponse}`), `SH.placeUploadedImage(upload, x,
y, w, h, {contain, nativeAspect, name, radius, start, duration})`, and
~~`SH.applyPexelImage(block, query, {upload | uploads})`~~ (refuses since
2.2.6 — it wrote `fill/image/imageFileURI`; use `SH.placeUploadedImage` /
`images.insertUploadedImage(record, …)`). The lottie build step itself (which slides get one) is
being implemented separately in the composer.

## 2026-09-15 — template-free (2.0.0): what the test decks proved, and what the renderer must do

> **2026-09-19:** this section describes what is now the **Creative Build**
> mode. It is no longer the only path — 2.2.0 reinstated the template library
> as **Quick Build**, which is the default; see "2026-09-19 — two build modes"
> at the end of this document. Every platform fact below is unchanged by that:
> they are facts about the engine and the save model, not about which renderer
> issues the calls.

Verified LIVE on decks 302 (28 demo slides across ten markets) and 303 (a
25-slide Patagonia deck built without any template, narrated, wired, tracked)
through `window.aiagent` — every finding below was reload-checked. The scene
renderer that automates them (`inject/scene.js`) is verified against the
mock-engine gate (`scripts/smoke-scene.mjs`), and has since been proven through
the panel on live decks 304–309.

- **Bespoke compositions persist through the aiagent save model exactly like
  archetype ones.** Raw engine writes (createTextBox + setFont, rects,
  gradients, opacity, corner radii, drop shadows, in-animations) persist when
  the slide is committed by `changeSlide` after any documented api.*
  mutation on it — the same rule as before; 302/303 added nothing new to the
  save discipline, only to what is drawn.
- **`api.slides.createMultiple(count, atIndex)` inserts at the exact index**,
  and every NEW slide defaults `autoAdvanceAfterNarration: true` — an
  interactive slide created that way must be set `autoAdvance(id, false,
  nextId)` or it walks past its buttons (all 27 slides of 302 needed it).
- **A transparent click plate carries the action for a backgroundless
  button.** A rect with fill `{r:0,g:0,b:0,a:0}` inserted as the BACKMOST
  child of the group, named for wiring (`btn:<target>`), works exactly like a
  visible plate: the label and icon ride on top, the click never misses, the
  editor renders nothing. Verified on the 303 Hamburger, the 302 city-planning
  rows and the text links. scene.js builds every `style:"text"` button this way.
- **Every button carries exactly ONE animated icon** (Bren, after a pass that
  doubled arrows next to concept icons): the label's concept when it has one,
  else `arrow-right`; a `check` on finish/agree labels; `arrow-left` on Back.
  The panel uploads those three with every slide's icons.
- **Native data graphics animate as parts.** Bars (rects growing upward from
  a baseline, value above, label below), horizontal bars (track + fill growing
  right), dot grids (ellipses, "on" in the accent), rings (charts.js SVG +
  the figure as real text), stats (numeral + label). The finance / charity /
  city-planning responses on 302 are the reference; the `graphic` element is
  the contract.
- **The edge rule reads as intent.** An element whose centre is on the right
  half enters from the right (slide direction π), on the left half from the
  left (0); text rises on its baseline; buttons grow; the stage never
  animates. A SLIDE's direction is a FLOAT (radians) — `setFloat(anim,
  'animation/slide/direction', θ)`; `setEnum` throws on it. `SH.applyInAnimation`
  now takes a numeric direction for that; scene.js records the intent per
  block on `idecide/anim` and `pipeline.animatePage` applies it after the
  own-images (icons, logo, rings) have landed as real blocks (`transferDress`
  carries the mark).
- **Rub Your Screen on the Logo Reveal** is built by `assembleLogoReveal`
  exactly as the hand-verified anatomy (VO1 0→D1, the full-canvas rect named
  "Logo Reveal" timed 0→D1 with `idecide/rub-your-screen` + `block_type` +
  `fallback-name` + `idecide/block-action-id`, the record pushed through
  `ctx.getBlockActions()` + `markCustomDataDirty()` + `updateTiming`, reveal
  layers shifted by D1, VO2 at D1, page = D1 + D2 + 0.5). The scene renderer
  draws only the reveal composition; the assembler owns the beat.
- **Track Choice by default.** Question answers `{store:true, valueType:'text',
  varName:'<trackAs> - <answer>', value}`, menus `Topic Viewed - <topic>`, and
  terminal CTAs the finish envelope — scene.js writes the `idecide/track`
  mark, `wireSlideBlocks` writes the record (unchanged).
- **Condensed display faces** (Oswald, Barlow Condensed) read 15–20% smaller
  than their band: the playbook says so; the scene may pass `size`.
- **Glyph coverage:** ★ renders in Nunito; ♡ does not in Fira Sans. Fold smart
  quotes; do not rely on symbols the face may lack.
- **The logo online (verified 2026-09-15, deck 303):** Wikimedia Commons
  direct file URLs (`upload.wikimedia.org/wikipedia/commons/<h0>/<h0h1>/<File>`,
  md5 of the file name) and the Commons API with `origin=*`,
  `cdn.worldvectorlogo.com/logos/<slug>.svg`, and `cdn.brandfetch.io/<domain>`
  are CORS-open to a my.idecide.com page; `api.brandfetch.io/v2/brands/<domain>`
  answers 401 without a key (so the panel calls it with the packaged key, host
  permission `https://api.brandfetch.io/*`); clearbit, Google favicons, the
  client's own site and vectorlogo.zone are not fetchable in-page. An SVG's
  fills recoloured in text (`#231f20` → `#FFFFFF`) upload through
  `uploadAndInsertImage` and place as Contain on every slide; the container's
  own egress is blocked for upload.wikimedia.org — the fetch must happen in the
  page. `IDP.huntLogo` is that recipe, in order: Brandfetch's files → the
  Brandfetch CDN → a Commons search → worldvectorlogo slugs → URLs the planner
  saw on the site; both tones (`IDP.logoUris.onDark / onLight`) upload and
  `logoImg` picks by field.
- **`idecide.com/lottie-library` sends no CORS header** — the page cannot fetch
  it (the panel can, and does); `cdn.lordicon.com/<code>.json` is CORS-open,
  `media.lordicon.com` is not. Unchanged since 2026-09-06; recorded because the
  test builds tripped on it.
- **A rejected tool call may already have run** (built-in browser, 45 s
  javascript_tool timeout): a long in-page script keeps running after the call
  is cut off. Audit the page before re-running a mutation — the "add arrows"
  pass doubled every icon that way. The same discipline applies to the
  extension's `exec()`: never re-issue a mutation on a timeout without reading
  the page first.

## 2026-09-16 — the engine resets a GLYPH's fill mode on load, exactly as a photo's (deck 304)

`verifyCover` (the byte-level check of the persisted slideData) counted
`block_content_fill_mode: 0` on every animated icon and SVG glyph the scene
renderer had placed with Contain — 3 on a question with three buttons, 5 on a
menu — and paid a corrective revisit on every such slide (4–7 s each, 37
slides). The in-visit cover guard had skipped glyphs on purpose ("they are
Contain, never Cover"), so nothing re-asserted them after the resource load
reset them to Crop. Now the guard and `recoverCover` re-assert Contain on
glyphs (`idecide/icon` / `idecide/lottie` marks, `icon-*`, `*/icon`,
`stat/ring`, `chart*`), and what persists is what was composed. For a
square Lottie canvas Crop and Contain draw the same picture, which is why
nobody saw it; the cost was time, not pixels.

Also verified on 304: a full-canvas frame of the page can be pulled from the
aiagent page and shown as an `<img>` overlay for a screenshot — the cheap way
to look at a slide from a browser pane that cannot zoom. ⚠ The 09-16 proof
used `block.export`, BANNED since 2026-09-28 (metered by img.ly): use
`engine.block.generateThumbnailAtTimeOffset(296, t)` (PNG Blob, browser only)
or `generateVideoThumbnailSequence(page, 296, t, t, 1, cb)` — both take the
time as an argument, so no `setPlaybackTime` first. `logoUris.aspect` (w/h) now travels with the logo uploads:
`IDP.setLogo` reads an SVG's viewBox / width+height or decodes a raster,
and `composer.logoImg` / `scene.placeLogo` size every placement from it.

## 2026-09-16 — the engine's grow direction is Horizontal / Vertical / All; the text animations exist; a page keeps its block id across changeSlide

- `//ly.img.ubq/animation/grow` takes `animation/grow/direction` ∈
  Horizontal | Vertical | All. `Up` / `Down` / `Left` / `Right` throw and the
  animation keeps its default — every scene grow before 2.1.0 landed there.
  `wipe` and `baseline` take Up / Down / Left / Right; `block_swipe_text`
  takes Left / Right / Up / Down; `spread_text` and `typewriter_text` take
  no direction; `slide` takes a float angle in radians (0 = travels right,
  π = travels left). Read off the exemplar slides' own animation blocks
  (decks 302/303, `window.__dump()`) and CONFIRMED LIVE on deck 304's page
  the same day: `setEnum(grow, 'animation/grow/direction', 'Up')` throws
  "Invalid enum value Up"; Horizontal / Vertical / All are accepted; wipe
  Down and block_swipe_text Right are accepted; spread_text and
  typewriter_text animations create and attach.
- `spread_text`, `block_swipe_text`, `typewriter_text`, `pan` and
  `ken_burns` are real animation types (SH.ANIM_TYPES) and the exemplar
  headlines use the first three; only text blocks accept the text kinds.
- `api.slides.changeSlide(id)` on the aiagent route REUSES the page block id
  (always 3 on decks 302/303): a wait for "the page id changed" never
  returns. Wait on content instead — the child blocks' name+x signature —
  then settle ~2s before reading. The first pass of the exemplar dump
  recorded 42669 as a copy of 42660 because of this.
- `engine.block.setTextColor(id, color, from, to)` colours a character range
  (the two-tone wordmarks); `engine.block.getTextColors(id)` returns the
  ranges' colours — confirmed live on deck 304's page (two ranges back).
  The typeface object from `engine.asset.findAssets('ly.img.typeface', …)`
  (what `SH.availableFont` wraps) keeps every variant in `fonts[]` by
  `subFamily` (Source Sans Pro: Black, Bold, ExtraLight, Light, Regular,
  SemiBold + italics), and `setFont(id, variant.uri, typeface)` switches the
  file — confirmed live: `text/fontFileUri` reads
  `SourceSansPro-SemiBold.ttf` and `getTextFontWeights` reports `semiBold`.
  `text/lineHeight` and `text/letterSpacing` take floats; a "\n" in
  `text/text` is a line (`getTextVisibleLineCount` = 2). One footgun:
  `findAssets` called in the first seconds after the page loads threw a
  WASM BindingError ("parameter 1 has unknown type … FindAssetsResult") —
  transient; the same call succeeded a few seconds later.

## 2026-09-17 — a graphic's corner radius lives on its SHAPE; a page render waits for a painted window

- `shape/rect/cornerRadiusTL` / `TR` / `BL` / `BR` are properties of the
  block's **shape** (`engine.block.getShape(id)` → a `//ly.img.ubq/shape/rect`
  that lists the four keys in `findAllProperties`). On the graphic block
  itself `getFloat(id, key)` throws *"Component ubq/designblocks/RectShape is
  not set on entity 16"* and `setFloat(id, key, v)` throws *"Entity ID 16 of
  type //ly.img.ubq/graphic doesn't have component
  ubq/designblocks/RectShape"* — so a write wrapped in `try { } catch { }`
  does nothing and says nothing. CONFIRMED LIVE on deck 306 ("Why We Repair -
  1", 2026-09-17): the api-placed `photo-panel` (video fill) had shape radius
  0 while the `scrim` and `photo-panel/stroke` drawn over it by `rect()` read
  28 — the "square video inside a rounded frame" Bren reported on decks
  305/306. Writing the shape of the panel took; the value was restored and
  the page reloaded without a commit (the saved slide still reads 0/28/28).
  2.1.1 writes every radius on the shape (`composer.shapeRadius`,
  `pipeline.setRadius`, `transferDress`, `SH.placeUploadedImage`), and
  inspect reads it from the shape.
- A page render needs the editor to be drawing (proven with `block.export`,
  BANNED since 2026-09-28; the preview-path `generateVideoThumbnailSequence` /
  `generateThumbnailAtTimeOffset` run in the same engine tick, so expect the
  same — not separately re-proven): with the Claude desktop window MINIMIZED the
  promise never settles (a 45s check timed out; the same call returned a
  frame on deck 304 with the window up). A page script that changes a value
  and then awaits an export can therefore time out with the value still
  changed — after any timed-out script, read the state back, restore it,
  and reload to discard.

## 2026-09-17 — the `?aiagent` param is not proof of anything; a dead slide id HANGS; the tab's real state is readable

Evidence: three build records from testers on the live store build (1.2.6 /
1.2.7), read on 2026-09-17. These are field records, not a fresh live probe —
where that matters it is said below.

- **The Builder rewrites its own address.** Slide navigation writes
  `?slide=<id>` and the `aiagent` key does not always survive it. In two of
  the three records the panel's own probe reported `window.aiagent` LIVE
  while the URL no longer carried the param — and `injectEngine`, which keyed
  off the param alone, reloaded the tab for nothing (a 20-90s wait, and any
  uncommitted slide discarded). **The live object is the test, the query
  string is not.** 2.1.2: no reload when `state.agent` is true, whatever the
  URL says. Adding the param is still right when the tools are genuinely
  absent.
- **Driving a slide id the deck no longer has does not throw — it hangs.**
  A record shows `slides.changeSlide` on an id from a previous session's map
  sitting until the panel's own 3-minute ceiling, with eleven slides already
  built and the client told nothing had landed. So a stale id costs a
  timeout, not an error: `api.slides.get()` before any structural work is the
  only defence, and a carried id map must be REPLACED by it, never merged
  into it (a merge keeps the dead entry, which is exactly how this happened).
- **The panel can read why a tab is not being drawn**, without the page's
  help: `chrome.tabs.get(id)` gives `active` and `status`,
  `chrome.windows.get(tab.windowId)` gives `focused` and
  `state === "minimized"`, and `chrome.scripting.executeScript` in the MAIN
  world returns `document.visibilityState` **even while the page is not being
  painted** — the script runs, the renderer does not. That is the whole
  vocabulary needed to say *minimised* / *another tab in front* / *behind
  another app* instead of a paragraph about background tabs, and to stop
  waiting once the answer is known (2.1.2 gives up ~9s after the reason is
  known, where it used to wait 90s).
- **An uploaded file belongs to the presentation, not to us.** POST
  `/api/builder/builderSession/{sid}/uploads?source=user_upload` (the call
  behind `SH.uploadAsset`) takes the client's own picture or clip and the
  platform keeps it in that presentation's library, so a file the client
  attaches in the panel needs no storage of ours anywhere. Place the returned
  uri the sanctioned way — `uploadAndInsertImage(blob, x, y, w, h, 0, null)`
  for a still, `insertPexelBatchVideo({id, label, meta:{uri, sourceSet:[…]}},
  …)` for a clip. A raw r2 URL in a plain fill still hard-freezes the
  renderer (§ 9); that has not changed.

## 2026-09-17 — reading a built deck live: what the engine says vs what the mock says (deck 307)

A read-only pass over all 42 slides of deck 307 through `window.aiagent`
(`slides.get` → `changeSlide` → walk `block.getChildren(page)` for name, type,
x/y/w/h, `getFrameHeight`, `text/fontSize`, `getTextVisibleLineCount`), run
from a second tab while the deck was open. Verified facts from it:

- **The mock's text model is ~12-15% wide.** `replay-scenes.mjs` predicted 30
  wrapped short labels in this deck; the engine had **2**
  ("THE TURNING POINT" at 28px in 280px, "BUILT TO BE REPAIRED" at 28px in
  340px). Its per-character estimate (0.56 sans / 0.60 display) and its 0.9
  usable-width rule are deliberately conservative, which is right for
  PREDICTING but wrong for CORRECTING: a fix driven by the estimate shrinks
  type that fits. Correct from `getFrameHeight` against a one-line frame
  (`setWidthMode('Auto')`), never from the estimate.
- **`setWidthMode(id,'Auto')` makes the frame hug its text** — one line unless
  the string carries "\n". The sandbox modelled it as still wrapping at the
  old box width, which made every measured-correction pass a no-op in the
  gate; fixed 2.1.3.
- **A screenshot taken mid-timeline lies.** At 7.5s of a 7.1s slide the
  headline's baseline animation is still clipping its second line, which reads
  exactly like an overlap with the body below. Scrub with
  `engine.block.setPlaybackTime(page, duration - 0.3)` before judging a
  layout, and read `engine.block.getDuration(page)` for the number.
- **A stroke does not imply a hollow fill.** A `//ly.img.ubq/graphic` created
  by the kit carries a solid fill; `setStrokeEnabled` + `setStrokeColor` draws
  the outline ON TOP of it. To get a ring, clear the fill first
  (`getFill(id)` → `setColor(f, 'fill/color/value', {r:0,g:0,b:0,a:0})`).
  Confirmed live on deck 307: three concentric "rings" rendered as one cream
  disc with an orange ring inside it.
- **The editor does not initialise a tab it is not drawing** — the 2.1.2 fact
  again, from the other side: a second tab opened on the deck sat at the
  loading spinner with `document.visibilityState === "hidden"` and
  `engine.scene.getCurrentPage()` returning null, for as long as its Chrome
  window stayed behind another app. `chrome.scripting.executeScript` still
  runs in it (that is how the state is readable), but nothing renders and
  `changeSlide` never settles. Bren, twice: "you need to pull focus to the
  window for it to initialize". Bring the window forward, then read.
- **AND AN AGENT CANNOT RAISE A TAB IT OPENS (2026-09-18).** Proven with a
  stopwatch, not inferred: a tab created with the Chrome extension's
  `tabs_create_mcp` and then pointed at a deck sat at
  `document.visibilityState === "hidden"` with
  `engine.scene.getCurrentPage()` returning null for 17 seconds, while
  `window.aiagent` AND `window.aiagent.engine` were both present the whole
  time. That is the trap — the objects exist, so a readiness check that looks
  for them passes while nothing renders. Check `getCurrentPage()`.
  Three ways out were tried and none works: `navigate` loads the URL but does
  not activate such a tab; `window.focus()` and a synthetic click are refused
  for a background tab; `resize_window` resizes it and leaves it hidden. The
  desktop bridge cannot do it either — Chrome is grantable there only in READ
  mode (see the screen, no interaction), by design, because the extension is
  meant to be the interaction path, and the extension's own
  `chrome.windows.update({focused:true})` runs in its panel, not in the page.
  **So an agent reading a deck uses ONE tab and navigates it in place** — the
  first tab of a group is active; keep it and change its URL.
- **…EXCEPT THROUGH THE EXTENSION, WHICH NOW LETS THE PAGE ASK (2.1.5).**
  Bren, an hour after the above was written: "Can we possibly use the extension
  as a pipeline to open and focus tabs when checking our work?" It can, and the
  permission was already there — `chrome.tabs.update({active:true})` and
  `chrome.windows.update({focused:true})` need no user gesture, and the panel's
  `bringBuilderForward` has called them since 2.1.2. What was missing was a way
  in from the page. `resolve.js` (isolated world) now relays
  `window.postMessage({__idp:"focus-tab"})` to `background.js`, which raises
  **the sender's own tab** — un-minimise, activate, focus the window — and the
  answer lands on `document.documentElement.dataset.idpFocus`:
  `ok` · `denied:not-an-agent-url` · `denied:throttled` · `error:<msg>`.
  Proven live on deck 308, extension reloaded: a tab that had been `hidden`
  with `getCurrentPage()` null went `visible` with page 3 **within 1.2s of one
  postMessage**, nobody touching anything. A second post inside 3s answered
  `denied:throttled`; the same deck URL without `&aiagent=` answered
  `denied:not-an-agent-url`; an unknown message shape was ignored.
  Two gates, because this ships: the content script only runs on the myiDecide
  hosts in the manifest, so no other site can post it at all, and the URL must
  carry the agent flag, so a viewer's ordinary session can never pull focus.
  The one-tab habit above is still the cheaper path — ask for focus when a
  second tab is genuinely needed, not as a substitute for reusing the first.
- **Reading a deck from a second tab is safe.** No write, no
  `markCustomDataDirty`, no save: 42 `changeSlide` calls and a full block walk
  left the deck untouched (the panel's own session kept the lock).

## 2026-09-18 — what the animated library actually contains, and how a match goes wrong

From reading deck 307's `lottieMeta` back against the library's own
`catalog.json` (3,687 items) and re-running the panel's matcher over the
builder's whole 433-concept vocabulary:

- **Every item carries a `class`**: `multicolour` (3,337 — the 2-tone WIRED
  drawings, ink plus an accent) or `black-only` (350 — the 1-tone SYSTEM
  drawings). A `black-only` item has no accent to paint, so it renders
  visibly lighter than its neighbours in a row of 2-tone icons. The class is
  the thing to match on for a set; `variants` (flat / system-outline /
  system-solid) is about style, not family.
- **An item's `aliases` are the builder's vocabulary attached at ingest, and
  they lost the wishlist's ORDER.** `lordicon-wishlist.json` lists accepted
  Lordicon names per concept in preference order ("flag": flag, milestone,
  goal-flag), but the ingest hangs each accepted name on whichever file it
  harvested, so a third-choice drawing ends up carrying the concept as an
  alias. Any scorer that ranks an alias above a file name therefore inverts
  the wishlist. Verified: "flag" → sport-and-fitness/goal-sign, "globe" →
  nature-and-weather/planet in the shipped deck.
- **The vocabulary is not fully covered**: 74 of 433 concepts match nothing at
  all (move, trending-up, activity, clipboard-list, quote, banknote, …) and
  fall back to the static SVG glyph. That is the visible mismatch when five
  animated icons sit beside one flat one.
- **A concept can be answered by a specialisation**: nothing is named "chart",
  but `finance-and-stats/pie-chart`, `bar-chart-vertical-grow` and
  `line-chart-grow` all are. Accepting a name whose HEAD is the asked noun
  (with only shape qualifiers in front) is safe; accepting any compound is
  not ("hand truck" is a trolley, "paper plane" is a paper dart).
- **The matcher can be tested offline.** `lottieFor` / `lottieForWords` /
  `lottieNames` are pure functions of (concept, catalogue): lift them out of
  the panel source, load the real catalog.json, and run the wishlist's
  concepts through both the old and the new version. That diff — 17 changes,
  17 improvements — is how this change was verified, not by eye.

## 2026-09-18 — the gate's ruler disagreed with itself (a simulated six-slide run)

A deck was written by hand the way the design pass writes one — a cover, two
content slides, a question, a menu and a CTA for an invented credit union —
and put through `replay-scenes.mjs` and the new `paint-scenes.mjs` (which
draws the sandbox's blocks as HTML and shoots a PNG, so a slide can be LOOKED
at without a browser session). Nothing here is a platform behaviour; all of it
is about the model the gate grades with, which had been trusted without being
checked against itself.

- **`Auto` frame width and the wrap test used different rulers.** The mock
  measured an Auto frame at `chars x cw` and wrapped an Absolute one at
  `(word + 1) x cw` against **90%** of the box. So a frame set to its own
  measured ink "wrapped" in the model. Every pass that widens-to-ink and then
  steps the size down when that fails — scene step 0c — therefore shrank type
  the platform holds: a solo pill's label came out **29px** in the replay
  where the engine gives 33. One ruler now, 98% usable, and an Auto frame
  reports the width at which its longest HARD line just holds. Consistent by
  construction: measure, set that width, and the line holds.
- **The Auto width of a multi-line string was the whole string.** `"The first
  key\nis the hardest."` measured as one 28-character line — 1.9x its real
  ink. Every `match:"ink"` binding and every trim off a hard-broken headline
  was wrong by that much. It is the longest hard line now.
- **Capitals were not modelled at all.** composer's own estimator has
  multiplied by 1.1 for upper case since 2.0; the mock did not. The one
  treatment that is ALWAYS capitalised is the eyebrow, so every spaced-caps
  line measured ~10% narrow on top of the letter-spacing the mock ignored
  until 2.1.4 — "A FEW MINUTES ABOUT YOUR FIRST HOME" was graded a clean
  one-liner in an 858px box and wrapped at 900. Same 1.1 factor as the
  estimator now.

What that cost, measured on deck 307's record: **four text blocks under the
phone band** that the replay had signed off, and a class of correction passes
that were quietly fighting each other. After the fix, 0.

- **A replay that ignores the renderer's own findings grades a slide clean
  that the build would send back.** `IDP.designFaults` — where the designed-
  overlap reports (2.1.4) and now the bottom-margin ones land, and what the
  pipeline hands the slide reviewer — was not read by `replay-scenes.mjs` at
  all. The simulated cover shipped its sender line 43px into the bottom margin
  under a pill it overlapped, and the tool said "0 defects". It reads them now.

## 2026-09-19 — two build modes (2.2.0), and the Track Choice the template path never wrote

The extension now offers TWO layout paths, picked per build by the operator
with the **Build mode** selector beside the model pill. **Quick Build** (the
default) draws a layout from the restored 375-variation library through
`composer.js`'s archetype renderers; **Creative Build** designs a scene per
slide and draws it with `inject/scene.js`. `DESIGN_MODE` is gone — the panel
now carries `BUILD_MODES` / `DEFAULT_BUILD_MODE` / `BUILD_MODE` / `MODE_PIN`,
and `sceneMode()` reads the setting instead of a constant. Nothing about the platform
changed; what changed is which renderer issues the calls, and every platform
fact in this document holds on both paths.

- **The composer never wrote `idecide/track`, in ANY version — so a
  template-path deck recorded no viewer choices at all.** The platform records
  a viewer's answer in the same `clickActionData` envelope as the finish
  payload (§13); `blockActions.set` cannot carry it, so `wireSlideBlocks()` in
  `pipeline.js` reads the mark off the clickable plate and writes the record by
  hand. `scene.js` has written that mark since 2.0. `composer.js` never did —
  not in 1.2.8, not before. **Nothing errors**: the reader simply finds no
  metadata and writes nothing, so the deck ships, plays, and silently records
  nothing on a product whose whole point is recording what the viewer chose.
  It went unnoticed because the template path was retired four days before
  anyone looked. Fixed 2026-09-19: `button()` marks the plate with the same
  rule as `scene.js` (question answers `"<trackAs> - <label>"`, menu topics
  `"Topic Viewed - <label>"`, navigation labels excluded), guarded on the scene
  flag so exactly one owner marks a given plate — `scene.js` calls that same
  constructor and marks it itself. **The lesson is the shape of the failure,
  not the fix: a metadata contract with no gate on the writer fails silently.**
- **`cornerMark()` was in the same position** — the brand mark in the first
  free corner ran only on the scene path. It now runs in `__COMPOSE`'s tail,
  same corner order, same 36px pad, same aspect-aware height, same "skipped —
  every corner is taken", guarded the same way.
- **A deck's mode is a property of the deck, not of the panel.** Opening one
  for revision pins the mode to what that deck actually is: its record's
  `settingsUsed.buildMode`, else inferred from whether its slides carry `scene`
  or `skel`+`spec`. The revision path branches on `sceneMode()` in five places
  (load the template library? attach the element contract? bind a new slide to
  a variation or give it a scene? how is an updated slide merged?), and a
  Quick Build deck handed a scene-path slide gets a slide with no binding.
  Choosing a mode in the popup clears the pin as an explicit override, logged.
- **Known difference, recorded not fixed (no owner action):** `scene.js`'s 64px
  minimum button height is NOT ported to the template path. Template button
  heights are computed against template geometry (`rh = Math.max(56, rh)` in
  one renderer), and a blanket floor risks overflowing tight rows.
- **A device in a deck's design language governs FORM, never WORDS.** Deck 309
  shipped five eyebrows reading `sk_test_menu`, `sk_test_billing`,
  `sk_test_subs`, `sk_test_ready`, `sk_live_global`. Nothing malfunctioned: the
  deck's own design language minted a device described as *"key chip — a small
  pill holding a mono-styled tag (sk_test_ style)"*, the design pass obeyed the
  description literally, and the review pass repaired the chip's geometry
  without ever questioning the words inside it. The guard is three-part — the
  rule in `deck_outline_system.md`, a copy rule in both design and both review
  contracts, and a mechanical `placeholderFault()` detector in
  `scripts/replay-scenes.mjs` (all five caught, zero false positives across 51
  other slides). **The carve-out is the ROLE:** `sk_test_51H…` as the VALUE in
  a labelled cell is a legitimate specimen; the same shape in an `eyebrow` is a
  defect.

## 2026-09-28 — `engine.block.export` is METERED by img.ly; it is BANNED; snapshots come from the preview path (extension → 2.2.4)

**What happened.** img.ly contacted the lead developer: the extension's
`engine.block.export(page, 'image/jpeg', …)` calls register as exports of the
licensed system, the account's average exports rose sharply, and the licence
is being moved to a higher, more expensive tier. The calls were the
design-review snapshots — one per composed slide in the build loop
(`snapAudit`), one per inspected slide in the revision chat (`exportSnap`),
one after every `applyEdits`. A 30-slide build with a review pass and a few
revisions is on the order of a hundred exports; that is what img.ly saw.

**The rule.** `engine.block.export`, `engine.block.exportVideo`,
`engine.block.exportWithColorMask`, `engine.block.exportAudio` and
`cesdk.utils.export` are never called — from the extension, the SH runtime,
the Claude plugin's page scripts, or any runbook recipe. Not to compare, not
to verify an animation frame by frame, not because the BlockAPI page lists
them. Every older recipe in this file that said `block.export` has been
amended to point here. `scripts/package.mjs` walks every staged `.js` /
`.mjs` / `.html` and refuses to zip — STORE and TESTER alike — if the export
family is called (a comment naming the method passes; a call does not).

**The replacement — verified live 2026-09-28 (session 222, 1558×720 page,
15 s, engine 1.74.1):**

- `engine.block.generateVideoThumbnailSequence(page, 296, t, t, 1, cb)` →
  `cb(0, ImageData)` of the composed page at time `t`: 641×296 in 20–35 ms
  (`block.export` took seconds). Overlaid on the canvas it was the same
  composition, every layer. `t` is honoured independently of the playhead
  (playhead parked at 7.48 s; frames requested at 0.3 s and 14.92 s differed
  correctly, later-timed blocks absent from the early one) — so no
  `setPlaybackTime` / `SH.seek` before and no rewind after. img.ly's own
  words: passing a page "re-renders the composed scene instead of decoding a
  file"; pages, graphics, text, groups and tracks are accepted; blocks render
  in their RESTING state; video fills inside the composition come from cached
  frames (approximate — fine for layout review). One request per block at a
  time; the returned function cancels. The callback runs inside the engine's
  update loop — copy the frame out and leave; do the canvas work outside.
- `engine.block.generateThumbnailAtTimeOffset(296, t)` → PNG `Blob` of the
  CURRENT page at `t`, 41–51 ms; browser only. The fallback.
- Both cap at 512 px high; 296 → 641 wide keeps the frame the reviewer always
  received. Both exist on 1.74.1 (so do the deprecated
  `getPageThumbnailAtlas` / `getVideoFillThumbnail*` — do not use those).
  img.ly documents both under "Thumbnail Previews", apart from the export
  section, as the API the timeline's filmstrip and page storyboard use.
- **CONFIRMED NOT METERED (Bren, 2026-09-28, from the lead developer / img.ly):**
  `generateVideoThumbnailSequence` and `generateThumbnailAtTimeOffset` do not
  count as exports. (Six back-to-back preview calls had also produced no
  network request from the page.) The preview path is the only render call we
  make, and it is free.
- Like export, the render needs the engine ticking: a background tab or a
  minimized window stalls it. Bring the tab forward; `pageThumbnail`'s 20 s
  ceiling cancels the request instead of hanging the pass.
- `IDP.pageThumbnail(pageId, {time, height, quality})` in `pipeline.js` wraps
  both and returns the same JPEG data URL as before (`IDP.getSnapshot` reads
  the MIME from the prefix); `snapAudit`, `inspectSnap` (was `exportSnap`) and
  `applyEdits` all go through it. The 1150 ms fill-settle wait stays; the
  seek-to-end / rewind-to-0 pair is gone because the time is an argument.

**Recipe amendments made in this file.** 2026-09-05's lottie loop proof
("export at each t, hashing the bytes") → grab frames with
`generateVideoThumbnailSequence(block, 128, t, t, 1, cb)` and hash
`result.data`. 2026-09-16's "the cheap way to look at a slide from a browser
pane" → `generateThumbnailAtTimeOffset(296, t)` then an `<img>` overlay.
2026-09-17's "export waits for a painted window" → expected of the preview
path too (same tick), not separately re-proven.

**Addendum, same day (Patagonia proof build → 2.2.5): an ACTIVE tab in a
covered window is undrawn too.** 49 clean snapshots in the main pass; then, in
the rebuild pass, three thumbnails timed out (20 s primary + 20 s fallback;
one recovered, two skipped) and a `changeSlide` stalled, all inside the three
minutes the Chrome window sat behind Terminal. `chrome.tabs.get().active` was
true throughout, so the panel's background-tab nudge never fired; the page
itself knew — `document.visibilityState === "hidden"` — and Chrome throttles
its timers too (an 800 ms timeout fired at 1.2 s in a background MCP tab).
Rule: **the render clock starts only when the document is visible.**
`pipeline.js whenVisible()` waits up to 45 s for `visibilitychange`; the
panel's probe reports `hidden` and the 20 s nudge focuses the window for it
(`bringBuilderForward` already did `windows.update({focused:true})` and
un-minimised — only the trigger was missing). Export had the identical
weakness; this is the first fix for it.

## 2026-09-28 (later) — every media block is api-inserted with the PLATFORM's record; the paperclip video was not (extension → 2.2.6)

The lead developer asked whether the extension adds videos with the aiagent
`uploadAndInsertVideo` / `insertPexelVideo` / `insertPexelBatchVideo` calls or
writes `fill/video/fileURI` itself — the api calls upload device renditions and
a fill we write plays the full file on a phone and crashes the browser.

**Audit of the shipped 2.2.5 tree.** Video: 10 call sites, all
`videos.insertPexelBatchVideo(record, …)` (pipeline ×7, composer ×3); zero
`setString(…'fill/video/fileURI'…)` on any live path. Images: `insertPexelImage`,
`uploadAndInsertImage` (icons, charts, logo), `insertUploadedImage` (lotties and
library files). The engine-fill helpers the developer remembered —
`SH.applyVideoUri`, `applyPexelImage`, `addImageBlock`, `addIcon`, `addIconRow` —
still existed in `sh/media.js`, exported, with **no caller anywhere in the
extension** (grep-proven); since 2.2.6 they THROW (`SH_MEDIA_BAN`) and their
fill writes are deleted. `package.mjs` and `smoke-panel` refuse a tree that
writes `fill/video/fileURI`, `fill/image/imageFileURI` or `setSourceSet`.

**The one real gap — the 2.1.2 paperclip's VIDEO (and image) attachment.**
`SH.uploadAsset` POSTed the file to `/uploads?source=user_upload` and kept only
`meta.uri`; the `addUpload` op then inserted through `insertPexelBatchVideo`
with a SYNTHESIZED record `{meta:{uri, sourceSet:[{uri}]}}` — api-inserted, but
one rendition, the large file. Verified on session 222: the POST answers the
full `{id, label, meta:{uri, thumbUri, sourceSet, posterUri, lowLesUri, width,
height, duration, mimeType}}`, the server makes four video renditions
(large 1080 / medium 720 / small 640 / smaller 340 wide) and four to five image
renditions, and the library listing (`GET …/uploads?source=user_upload`) carries
`sourceSet`, `s3Key`, `s3PosterKey`, `width`, `height`, `duration`,
`contentType` per entry. 2.2.6: `SH.uploadAssetRecord(file)` returns the whole
record and the upload record keeps it as `rec.upload`; `addUpload` hands
`rec.upload` to `insertPexelBatchVideo` / `insertUploadedImage`; a record
without one (pre-2.2.6, or rehydrated bare) is recovered by
`SH.findUploadRecord(uri)` (tail-match against `s3Key` / the listing's
sourceSet); only when both fail does the single-rendition record remain, and
the log says "phones will load the large file". Proved live: from a large uri
alone → 4 renditions recovered → `insertPexelBatchVideo` → the block's fill
reads a 4-entry `fill/video/sourceSet` with an EMPTY `fileURI` (the platform's
convention); the test block was discarded by reload.

Also verified: `api.currentSlide.videos.uploadAndInsertVideo(videoData: Blob |
string | File, x, y, w, h, startTime, duration)` IS on the live api — AIAGENT_API
Recipe H ("there is no `videos.upload*`") was stale and is corrected.

## 2026-10-21 — a button-heavy slide, proven in the mock (deck 322, the YETI build)

- `scripts/replay-template.mjs <record> [--slide "…"] [--log] [--blocks]`
  replays a record's Quick Build slides through `__COMPOSE` in the mock
  engine and reproduces the live `canvas fit` numbers to the pixel. Run it on
  every new record before reading the deck by eye; it reports off-canvas,
  text-over, torn units, piles and under-floor plates.
- `fitToCanvas` used to clamp every off-canvas unit to the bottom margin —
  the same y — so a stack that did not fit became a PILE, and the verify
  pass (overlap COUNT only) accepted the pile because it has fewer measured
  pairs than the spread. Both are closed: the pile guard chains a unit under
  what it would land on; `IDP.unitFaults` (a button part outside its plate,
  two plates sharing a third) reverts a repair that tears a unit.
- The packer's row gate (`rows.length <= 12`) switched the whole pass — and
  every rung in it — off on the slides with the most rows. It is 24 now; a
  set (two abreast, a column of rows) is one unit whatever its row count.
- The group-scale rung scaled button plates with no floor (deck 322's menus
  went to 40px rows). Plates stop at 56; a circle/square target stops at its
  content (icon + label frame + gap + pads).
- A sub-beat named "<Chapter> - <Word>" is part of `<Chapter>` when a slide
  with that stem exists (`parentStemOf`); only ` - <number>` was stripped
  before, and each such beat became a chapter the main menus offered.
- `text({ maxH })` walks a line down its band until its block fits; the
  one-line grow re-fit discarded it until it was re-applied after the grow.
- `api.currentSlide.images.insertUploadedImage` / `uploadAndInsertImage` can
  fail to answer at all (deck 324: one of five inserts on a page that had
  already timed out its thumbnail calls; 6½ minutes of silence until the tab
  was reloaded). Every insert in `materializeOwnImages` has a 45 s ceiling
  (`T_INSERT`); a call that does not answer is a failed image, logged by name.
- A re-hint from the review used to re-source the clip ("a new hint is a new
  clip", 2.2.2) — retired 2026-10-21: the first pick stands; only the brand-
  safety block list re-sources. The review's `media: "replace"` is honoured for
  safety only (the reviewer's own sentence must name the reason, or the
  mechanical audit raised it).
- The build card paints `P.msg` — `log()` cuts it at 180 characters and the
  `footage picks:` line is `note()` (record-only); the panel clips the line and
  offers "more".
