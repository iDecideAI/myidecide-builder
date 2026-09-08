> **Reference for the myiDecide Presentation Builder skill.** The plain-English
> troubleshooting KB shared with the Chrome extension: one entry per situation
> under a stable code (`KEY-01`, `TAB-04`, …) — what the user sees, what it
> means, what to do. Read it when the user reports an error or asks what a
> message means; answer from it before retrying anything. The `KEY-*` entries
> concern the extension's API key and do not apply to this skill.

# Troubleshooting — myiDecide Presentation Builder

Plain-English help for the messages the builder can show while it writes,
builds or edits a myiDecide presentation. Each entry has a short code
(`KEY-01`, `TAB-02`, …) that the side panel quotes next to an error, so you can
jump straight to the matching section. Nothing here needs technical knowledge;
where a step does, it says so.

The `KEY-*` entries are about the Anthropic API key the **Chrome extension**
uses. The Claude plugin (the "myiDecide Builder" plugin for Cowork) needs no
key and never shows those — every other entry applies to both.

## Codes at a glance

| Code | What you'll see (short) | In one line |
|---|---|---|
| `KEY-01` | "The Claude API rejected this key" · `401 authentication_error` | The key is mistyped, copied from the masked list, revoked or expired |
| `KEY-02` | "credit balance is too low" | The Anthropic account has no prepaid credit yet |
| `KEY-03` | `429 rate_limit_error` · "enforced_spend_limit_reached" | A monthly usage cap or rate limit was hit — wait, or raise the limit |
| `KEY-04` | `404 not_found_error` "model: …" · `403 permission_error` | The chosen model isn't available to this key — pick another model |
| `KEY-05` | "anthropic-workspace-id is required when authenticating with an identity-linked API key" | The key was created with no workspace chosen — paste its Workspace ID in Settings, or make a key with a workspace |
| `KEY-06` | "anthropic-workspace-id header must be a valid workspace ID" | The Workspace ID in Settings is mistyped or belongs to another organisation |
| `TAB-01` | "Open your presentation in the builder first" · "I'm not seeing a myiDecide tab in front" | The panel can't find the builder tab it should work in |
| `TAB-02` | "The builder tab hasn't finished loading its editor" · "The builder reloaded but its tools never came up" | The editor never finished booting, usually because the tab was in the background |
| `TAB-03` | "builder error dialog detected" · "The editor crashed N times … mid-update" | The myiDecide editor showed its Unknown Error dialog |
| `TAB-04` | "slides.changeSlide did not answer in 30s" · "could not open <slide> — skipped" · "export timed out after 20s" | The tab lost focus or the computer slept during a long pass |
| `TAB-05` | "This presentation is open in another tab" · `409 SESSION_MISMATCH` | The same presentation is open somewhere else and holds the lock |
| `BUILD-01` | "A build is running in your builder tab right now" | Edits wait until the running build finishes |
| `BUILD-02` | "I couldn't read any text out of <file>" | The uploaded script or brochure has no readable text |
| `BUILD-03` | "wire: unknown target" | A button pointed at a slide that doesn't exist — repaired automatically since 1.1.0 |
| `BUILD-04` | "(NO VOICEOVER)" is spoken aloud | A silent-slide marker was narrated — fixed in 1.1.0; older decks can be cleared |
| `BUILD-05` | "iconify unreachable" · "display font unresolved" | An icon or font service couldn't be reached; built-in fallbacks were used |
| `PANEL-01` | The side panel was closed while a build was running | Reopen it — History → Resume carries on where it stopped |
| `HELP-01` | You need to send something to support | Where the build record is and what to send to hi@idecide.com |

---

## API key (Chrome extension only)

### KEY-01 — The Claude API rejected this key

**What you'll see**

- On the setup screen: *"The Claude API rejected this key. Double-check it …
  and paste it again."*
- Before a resume: *"The Claude API rejected your key — check it in Settings
  (⚙), then press Resume again."*
- In the build log: `API 401: … "type":"authentication_error" …`
- If the text you pasted isn't even the right shape: *"That doesn't look like
  an Anthropic key"*.

**What it means**

Anthropic's servers looked at the key and didn't recognise it. One of four
things is true:

1. **It was mistyped or cut short.** A real key is long (over 100 characters)
   and is one unbroken run of letters, numbers and dashes. A missing character
   at either end is enough.
2. **It was copied from the masked key list.** After a key is created, the
   Console's *API keys* page only ever shows a shortened preview — a few
   characters, three dots, then the last four (something like
   `…api03-4Kq…m2AA`). That preview is a label, not the key. The full key is
   shown **once**, at the moment you create it.
3. **It was revoked** (deleted or disabled in the Console), by you or by
   someone else in the organisation.
4. **It expired.** When you create a key the Console asks for an expiry —
   3 hours, 1 day, 7 days, 30 days, a custom length, or *Never*. A 30-day key
   simply stops working on day 31, and expired keys can't be reactivated.

**What to do**

1. Sign in at **platform.claude.com** and, on the dashboard, press
   **Get API Key**.
2. Name it (e.g. "myiDecide builder"), choose an expiry you'll remember —
   *Never* if you're happy to rotate it yourself — and press **Create key**.
   Made from the dashboard, the key is tied to your workspace and works in
   the extension as-is. (A key made some other way, with no workspace, needs
   its Workspace ID pasted in Settings — see `KEY-05`.)
3. **Copy the full key from the confirmation dialog straight away** and paste
   it into the extension's ⚙ Settings. You can't come back for it later —
   only the masked preview remains.
4. Press **Test key**. It runs the real calls the builder makes and tells you
   in plain English what passed and what didn't.
5. Save. If a build had failed, open 🕘 History and press **Resume build** —
   progress is kept.

**If it keeps happening**

- Make sure nothing was added around the key when you pasted (a space, a
  quote mark, a line break).
- Ask whoever manages your Anthropic organisation whether the key was
  disabled or whether keys there are limited to a maximum lifetime.
- Still stuck? See `HELP-01`.

### KEY-02 — "credit balance is too low"

**What you'll see**

- On the setup screen: *"Key accepted — but the account has no credit yet."*
- During a build: a red status line quoting *"Your credit balance is too low
  to access the Anthropic API"*, and the build stops.
- Before a resume: *"Your Anthropic account still shows no available
  credit — top up … then press Resume again. Your progress is saved."*
- In the build log: `API 400: … credit balance is too low …`

**What it means**

The key is fine. The Anthropic Console account it belongs to is **prepaid**,
and it has no credit on it. A brand-new organisation starts at zero, so this is
the normal first hurdle rather than a fault.

**What to do**

1. Go to **platform.claude.com → Billing** (under Settings) and add credit.
   A small amount goes a long way: a typical build costs about **$1–2 with
   Sonnet**, **$5–10 with Opus or Fable**, and under **$1 with Haiku**. $10
   covers several builds.
2. Back in the extension, press **Test key** in ⚙ Settings — the credit check
   should now pass.
3. If a build had stopped, open 🕘 History → **Resume build**. Nothing that
   was already built is redone.

**If it keeps happening**

- Credit can take a minute to register after payment. Wait, then Test key
  again.
- If your organisation has a monthly **spend limit** set, reaching it looks
  similar — see `KEY-03`.

### KEY-03 — 429 rate_limit_error / "enforced_spend_limit_reached"

**What you'll see**

- In the build log: `API 429: … "type":"rate_limit_error" …`, sometimes with
  *"your organization has crossed its monthly API usage threshold"* and
  `"error_code":"enforced_spend_limit_reached"`.
- The build slows to a crawl, retries, or stops with that line on screen.

**What it means**

Anthropic is refusing calls for one of three reasons, all on the account
side:

- **The monthly spend cap of your usage tier.** Every organisation sits on a
  tier with a monthly ceiling; once it's reached, usage pauses until 00:00
  UTC on the first day of the next month unless a higher limit is granted
  sooner. The message says when access returns.
- **A spend limit you (or an administrator) set** on the organisation or
  workspace.
- **A short-term rate limit** — too many requests in a short window. These
  clear by themselves within a minute or so.

**What to do**

1. Wait a minute or two, then open 🕘 History → **Resume build**. If it was a
   short-term limit, it simply continues.
2. If it fails again straight away, sign in at **platform.claude.com** and
   open **Settings → Billing**: raise the spend limit you set, or — if the
   message names a date — request a higher tier from Anthropic. Tiers move up
   automatically over time with usage history and account standing.
3. Resume once the limit is raised or the date has passed.

**If it keeps happening**

- Pick a cheaper model for the rest of the build (⚙ Settings → model). Sonnet
  or Haiku use a fraction of the budget Opus does.
- If the cap is organisation-wide, someone else's usage counts too — ask the
  administrator.

### KEY-04 — 404 not_found_error "model: …" / 403 permission_error

**What you'll see**

- In the build log: `API 404: … "type":"not_found_error","message":"model:
  claude-…"`
- Or: `API 403: … "type":"permission_error" …`
- Test key reports that the chosen model failed while the key itself passed.

**What it means**

The key works, but the **model you picked isn't available to it**. Some
models are limited by organisation, workspace or region, and some are retired
over time. A 403 means the key has been scoped (for example to one workspace)
and that workspace can't use the model.

**What to do**

1. Open ⚙ Settings and choose a different model from the picker — **Opus 5**
   is the recommended default; **Sonnet 5** is the economical choice.
2. Press **Test key** — it checks the model you selected, not just the key.
3. Resume the build from 🕘 History if one had stopped.

**If it keeps happening**

- Sign in at platform.claude.com and check the workspace the key is scoped
  to; a key created for a restricted workspace inherits its limits. Creating
  a new key with the default workspace usually clears a 403.
- If you pasted a **Workspace ID** in Settings, a 403 can also mean that
  workspace isn't one your account may use — try a different one, or clear
  the box and use a key made from the dashboard's **Get API Key** button.

### KEY-05 — "anthropic-workspace-id is required when authenticating with an identity-linked API key"

**What you'll see**

- In **Test key**: the *Anthropic accepts the key* row (or a model row) turns
  red with *"This key was created with a Linked account but no workspace
  chosen …"*, and a **Workspace ID** box appears under the key.
- During a build or an edit: a card titled *"This key needs a Workspace ID
  (KEY-05)"* and the run stops before anything else is tried.
- In the build log: `API 400: … "anthropic-workspace-id is required when
  authenticating with an identity-linked API key; send the id of the workspace
  this request acts in." …`

**What it means**

The key itself is fine. A key linked to your account can be tied to **one
workspace** or left to work in **all** the workspaces you can use. Keys made
with the dashboard's **Get API Key** button are the first kind; a key created
from the Console's key-management pages without a workspace chosen is the
second, and Anthropic then needs to be told, on every call, which workspace
the call acts in. The extension does that automatically once you give it the
workspace's ID — until then the API refuses everything, even the one-token
test call.

(That is why the same steps worked for one person and not another: one key
carried its workspace inside it, the other did not.)

**What to do**

Either of these works; the first keeps the key you have.

1. In the extension open ⚙ **Settings** and press **"Key needs a Workspace
   ID?"** under the key (after a failed test the box is already showing).
2. In the Claude Console open **Settings → Workspaces** and copy the ID from
   the **ID** column of the workspace you want the builder to use. It starts
   with `wrkspc_`. Paste it into the **Workspace ID** box.
3. Press **Test key** — every row should turn green and the first row names
   the workspace — then **Save**.

Or:

1. On the dashboard at **platform.claude.com** press **Get API Key** — a
   key made there is tied to your workspace.
2. Copy the new key from the confirmation box, paste it into ⚙ Settings,
   leave the Workspace ID box empty, press **Test key**, then **Save**.

If a build had stopped, open 🕘 History → **Resume build** — progress is
kept.

**If it keeps happening**

- The Workspace ID must be pasted on its own — nothing before `wrkspc_` and
  nothing after the last character.
- A workspace ID from a different organisation than the key's is refused —
  see `KEY-06`.
- The extension cannot look the ID up for you: Anthropic's workspace list is
  an administrator-only call. If you can't see **Settings → Workspaces**,
  ask whoever manages your organisation for the ID, or make the key from
  the dashboard (second route above).

### KEY-06 — "anthropic-workspace-id header must be a valid workspace ID"

**What you'll see**

- In **Test key**: the *Workspace ID* row is red — either *"A workspace ID
  looks like wrkspc_ followed by letters and digits — this one doesn't"* or
  *"Anthropic doesn't accept that Workspace ID"*.
- During a build or an edit: a card titled *"The Workspace ID isn't valid
  (KEY-06)"*.
- In the build log: `API 400: … "anthropic-workspace-id header must be a
  valid workspace ID." …`

**What it means**

The extension sent the Workspace ID saved in Settings, and Anthropic didn't
recognise it for this key: it was mistyped, pasted with extra characters, or
it belongs to a different organisation than the key.

**What to do**

1. Open ⚙ **Settings**. Copy the ID again from the **ID** column of
   **Settings → Workspaces** in the Claude Console and paste it on its own.
2. Press **Test key**, then **Save**.
3. Or clear the box and use a key made with the dashboard's **Get API
   Key** button — then no ID is needed (see `KEY-05`, second route).

---

## The builder tab

### TAB-01 — "Open your presentation in the builder first" / "I'm not seeing a myiDecide tab in front"

**What you'll see**

- While connecting: *"I'm not seeing a myiDecide tab in front. Switch to the
  tab with your presentation open in the builder, then tell me again."*
- *"That's myiDecide, but not the builder. Open the presentation itself (the
  editing view)…"*
- When resuming or editing: *"Open your presentation in the builder first
  (the tab I built it in), then try again."*
- Older versions: *"Builder tab not found"*.

**What it means**

The panel reads whichever tab is **active** when you confirm, and it needs
that tab to be the myiDecide **editor** — the address looks like
`my.idecide.com/builder/create/<number>`. The dashboard, the presentation
list or the player don't count, and neither does a tab in another window.

**What to do**

1. In the same Chrome window as the side panel, click the tab with your
   presentation open in the builder (you should see the slides).
2. If you haven't opened one yet: sign in at my.idecide.com and open (or
   create) the presentation so you're looking at the slide editor. When you
   start a new build the panel can also make a blank one for you — choose
   **Make me a new one**.
3. Back in the panel, press **Try again**.

**If it keeps happening**

- Make sure you're signed in — a signed-out tab lands on the login page,
  which is "myiDecide, but not the builder".
- Chrome side panels belong to a window. If the presentation is open in a
  different window, either move the tab into this window or open the panel
  from that window.

### TAB-02 — "The builder tab hasn't finished loading its editor" / "The builder reloaded but its tools never came up"

**What you'll see**

- *"The builder tab hasn't finished loading its editor."* (versions before
  1.1.0 said *"builder page never mounted (engine present but no current
  page after 45s)"*.)
- *"The builder reloaded but its tools never came up. Give the tab a moment,
  make sure the presentation has finished loading and that its tab is in
  front, then tell me again."*

**What it means**

To work in your presentation the panel adds a small switch (`aiagent`) to the
tab's address and reloads it. The editor then has to finish starting up — and
the myiDecide editor **does not finish starting while its tab is in the
background**. If you switched to another tab, another window or another app
while it was reloading, it sat half-loaded and the panel gave up waiting.

Since 1.1.0 the panel brings the tab to the front itself before it waits, and
the message tells you which step it was waiting on.

**What to do**

1. Click the builder tab so it is the one you can see.
2. Wait until the slides appear and the editor stops showing its loading
   state.
3. In the panel, press **Try again** (or **Resume build** in 🕘 History).

**If it keeps happening**

- Reload the builder tab yourself (⌘R / F5), wait for the slides, then try
  again.
- Check that the address still ends with `aiagent=`; if you navigated
  elsewhere and back, the switch is gone and the panel will add it again on
  the next attempt.
- A very slow connection can take longer than the panel's ceiling — wait for
  the editor to be fully idle before pressing Try again.

### TAB-03 — "builder error dialog detected" / "The editor crashed N times … mid-update"

**What you'll see**

- In the build log: *"builder error dialog detected — refreshing the page"*,
  or a step name followed by *"builder error dialog detected"*.
- In the builder tab: a grey box titled **Unknown Error** — *"The application
  has encountered an unknown error. Please try to reload the page"* — with a
  single **Reload Page** button.
- If it happens repeatedly: *"The editor crashed 3 times in the last 8
  minutes and isn't recovering. This usually means the myiDecide platform is
  mid-update. Wait a minute, then hit Resume on the History screen — the
  build continues from where it stopped."*

**What it means**

The myiDecide editor itself crashed. A single crash is normal on a long build
— the panel notices, reloads the page and repeats the step it was on, so you
don't have to do anything. Several crashes in a row almost always mean the
platform is being updated underneath you (the page's files changed while the
tab was open); building through that fills the deck with empty slides, so the
panel stops on purpose and keeps everything it had.

**What to do**

1. For a single crash: nothing. Watch the log; the step repeats on its own.
2. If the build halted: wait a minute or two, then open 🕘 History and press
   **Resume build**. Finished slides are never lost.
3. If the dialog is sitting in the tab and nothing is running, click **Reload
   Page**, wait for the slides, then Resume.

**If it keeps happening**

- Try again in ten minutes — platform updates finish quickly.
- If it crashes on the **same slide** every time, ask for changes on that
  slide (a simpler layout, or a different background video) so the build can
  move past it, and tell support which slide it was (`HELP-01`).

### TAB-04 — "slides.changeSlide did not answer in 30s" / "could not open <slide> — skipped" / "export timed out after 20s"

**What you'll see**

In the build log, one or more of:

- `STALLED: slides.changeSlide did not answer in 30s`
- `build: could not open "<slide>" — skipped (its wiring and timing are
  skipped too)`
- `wire: could not open "<slide>" — its buttons are unwired`
- `export timed out after 20s`
- `inspect: could not open — "<slide>" …` (during edits)

**What it means**

The editor stopped answering for a while. The usual cause is that its tab
**lost focus**: the editor only renders — and only moves between slides —
while its tab is visible, so switching to another tab, minimising Chrome,
locking the screen or the computer going to sleep freezes it mid-step. The
panel waits a fixed time, marks the slide as skipped and carries on, so one
stall can cost a slide its design, its wiring or its thumbnail.

**What to do**

1. **Keep the builder tab visible for the whole run.** Don't switch tabs in
   that window; use a different window or a different device for other work.
   Keep the build overlay on — it's there to stop stray clicks.
2. Stop the computer from sleeping during a build (the extension asks Chrome
   to keep the display awake, but a manual lock or a closed laptop lid
   overrides that).
3. When the build finishes, look at the log for `skipped` lines. Each named
   slide can be redone from the finished screen: **Ask for changes** →
   *"rebuild <slide name>"*. The wiring pass on a rebuilt slide runs again
   automatically.

**If it keeps happening**

- A long deck on a slow machine can stall without any tab switching — close
  other heavy tabs and apps, then Resume.
- If every slide stalls, the editor is not really loaded: see `TAB-02`.

### TAB-05 — "This presentation is open in another tab" / 409 SESSION_MISMATCH

**What you'll see**

- A myiDecide message saying the presentation is open in another tab, or
  asking you to reload.
- In the build log: a save or upload failing with `409` (sometimes with
  `SESSION_MISMATCH` in the text), or repeated `400`s on every slide.

**What it means**

myiDecide lets **one tab at a time** write to a presentation. Every editor tab
gets its own session ticket, and the newest tab holds the lock. If the same
presentation is open in a second tab, a second window, another browser or
another device — or you duplicated the tab — the builder's tab is now the
stale one and the platform rejects its saves.

**What to do**

1. Close every other tab, window and device that has this presentation open.
2. Reload the builder tab (⌘R / F5) so it takes the lock back, and wait for
   the slides to show.
3. In the panel, open 🕘 History → **Resume build** (or Try again). Slides
   that failed to save are rebuilt; the rest are kept.

**If it keeps happening**

- Someone else on your team may have the presentation open. Ask them to close
  it while the build runs.
- Check the player too: a preview tab of the same presentation can count.

---

## During a build or an edit

### BUILD-01 — "A build is running in your builder tab right now"

**What you'll see**

*"A build is running in your builder tab right now — I can make changes as
soon as it finishes. Your message is safe to resend then."*

**What it means**

You asked for a change while the build (or a rebuild started by an earlier
change) was still working in the tab. Only one thing can drive the editor at
a time, so the panel parks the request rather than interrupting the build.

**What to do**

1. Let the build finish — the timer at the bottom keeps counting and
   *"Your presentation is ready!"* appears when it's done.
2. Send the request again from the finished screen (or 🕘 History → **Ask for
   changes**). Since 1.1.0 messages typed during a run are queued and answered
   when it ends, and a **Stop** button is available if you'd rather halt the
   run first.

**If it keeps happening**

- If nothing is visibly building and the message still appears, close and
  reopen the side panel, then open 🕘 History: if the run shows as running
  with no progress, press **Resume build** once, let it finish, then edit.

### BUILD-02 — "I couldn't read any text out of <file>"

**What you'll see**

*"Before I write anything — I couldn't read any text out of "<file name>",
and building without it would mean inventing a script instead of using
yours. Upload a different export (.docx from Word or Google Docs, or plain
.txt/.csv both work), or hit Skip to build from your answers and website
only."*

**What it means**

The script, brochure or deck you attached opened, but there was no text
inside it to read — typically a scanned PDF (pictures of pages), a PowerPoint
whose text is baked into images, an empty file, or a format the panel can't
unpack.

**What to do**

1. Export the same document as **.docx** (Word or Google Docs → File →
   Download), **.txt** or **.csv**, and upload that instead.
2. For a scanned PDF, use the original document it was printed from, or paste
   the text into a .txt file.
3. If you don't have a text version, press **Skip** — the script is written
   from your answers and your website.

**If it keeps happening**

- Open the file on your computer and try selecting text in it. If you can't
  select any, there is none to read.
- Very large files can fail to unpack in the panel; split them or export just
  the pages that matter.

### BUILD-03 — "wire: unknown target"

**What you'll see**

- In the build log: `wire: unknown target "<slide name>" on <slide>`.
- In the finished presentation: a button that does nothing when clicked.

**What it means**

A button was told to go to a slide that isn't in the deck. Before 1.1.0 this
happened when the script left a slide's name blank (the answer slides of a
question, most often), so the slide it pointed at was never created.

Since 1.1.0 this is repaired automatically: blank slide names are filled from
the outline, slides that buttons point at are kept, and a button whose target
can't be found is re-pointed at the sensible fallback (the next slide after a
question's answers, or the return menu).

**What to do**

1. If you see this in a deck built before 1.1.0, or a button still does
   nothing when you play the presentation, open 🕘 History → **Ask for
   changes** and say where it should go, naming both ends:
   *"point the Learn More button on Main Menu - First at Pricing"*.
2. Play the presentation again to confirm the button works.

**If it keeps happening**

- The slide you named may be called something slightly different. Ask
  *"list the slide names"* first, then use the exact name.

### BUILD-04 — "(NO VOICEOVER)" is spoken aloud

**What you'll see**

A slide that should be silent narrates the words *"no voiceover"*, or the log
shows a narration of two words being restored *"from the script (2 words)"*,
or `audio missing after generation` on a slide that was meant to be silent.

**What it means**

Script writers mark a silent beat with a placeholder such as `(NO VOICEOVER)`,
`(no narration)`, `[silent]`, `(none)`, `n/a` or `—`. Older versions treated
those words as the narration and sent them to the voice service. **Fixed in
1.1.0**: every one of those markers now means "no narration", everywhere.

**What to do**

1. On a deck built before 1.1.0, open 🕘 History → **Ask for changes** and
   say *"make <slide name> silent — remove its narration"* for each affected
   slide. The audio is removed and the slide keeps its default timing.
2. If you're editing the script yourself, leave the narration cell empty
   rather than typing a marker.

**If it keeps happening**

- Check the slide's Notes in the myiDecide editor — if the marker text is
  still there, ask for the notes to be cleared as well.

### BUILD-05 — "iconify unreachable" / "display font unresolved"

**What you'll see**

In the build log:

- `iconify unreachable — using the built-in icon set`
- `display font unresolved (<reason>) — the engine default will be used`, or
  the same for `body font`

**What it means**

Icons come from an online icon library (Iconify) and fonts are looked up in
the editor's font catalogue. If the icon service can't be reached — a
firewall, a captive Wi-Fi login page, or the service being down — the build
uses the icon set bundled with the extension instead. If a font the brand
asked for isn't in the catalogue, the editor's default font is used. The build
completes either way; the slides just use the fallback.

**What to do**

1. Nothing is required. Look through the finished deck: if the icons or type
   look wrong, ask for changes — *"use icons from the online set"* or *"set
   the headline font to <font name>"* — once you're on a normal connection.
2. If the icon service was blocked by your network, check that
   `api.iconify.design` is allowed, or run the build on another network.

**If it keeps happening**

- A font that is never found is probably not one the myiDecide editor offers.
  Pick a similar one from the editor's font list and name that instead.

---

## The side panel

### PANEL-01 — The side panel was closed while a build was running

**What you'll see**

The panel disappears (Chrome closed it, you clicked its ✕, or Chrome was
restarted). When you reopen it, the build is not visibly running.

**What it means**

The build runs inside the side panel, so closing the panel pauses it. Nothing
is lost: every finished slide is saved in your presentation, and the run's
progress is saved in Chrome on your computer.

**What to do**

1. Reopen the panel with the extension's button next to the address bar.
   Since 1.1.0 it returns to the screen you were on and offers to continue.
2. If it doesn't offer, open 🕘 History, find the run, and press **Resume
   build**. It reconnects to the builder tab (see `TAB-01` if it can't) and
   continues from the last finished slide.

**If it keeps happening**

- A side panel belongs to one Chrome window and isn't visible from the
  others. Keep the panel and the builder tab in the same window and stay in
  it.
- Chrome's own updates can restart the browser mid-build; postpone them until
  the build finishes.

---

## Getting help

### HELP-01 — What to send to support

**Where the build record is**

Open 🕘 **History** in the panel. Every run that was saved has a **⬇** button
(*Download record*); it saves a file named
`build-record-<brand>-<date>.json` to your Downloads folder. It contains the
questionnaire answers, the script, every step of the build log, every edit
request and reply, and the timings — everything needed to see what happened.
It does **not** contain your API key.

**What to send**

Email **hi@idecide.com** with:

1. The build record file (⬇ from History).
2. The code from this page that matches what you saw (for example `TAB-04`),
   or the exact message if it isn't listed.
3. The presentation's address from the builder tab
   (`my.idecide.com/builder/create/<number>`).
4. Roughly when it happened and what you were doing (building, resuming,
   asking for a change).

**Before you send**

- Try the "What to do" steps for the matching code once — most stops resume
  cleanly from 🕘 History.
- Never send your Anthropic API key. Support does not need it, and a key sent
  by email should be treated as exposed and replaced.
