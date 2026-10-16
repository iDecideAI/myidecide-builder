> **Reference for the myiDecide Presentation Builder skill.** The plain-English
> troubleshooting KB shared with the myiDecide AI: one
> entry per situation under a stable code (`ACCT-01`, `TAB-04`, …) — what the
> user sees, what it means, what to do. Read it when the user reports an error
> or asks what a message means; answer from it before retrying anything. The
> `ACCT-*`, `API-*` and `NET-*` entries concern the Extension's connection to
> the person's myiDecide account (since 2.3.0 its AI runs through myiDecide on
> their sign-in; this skill runs on the user's own Claude subscription), and
> `BUILD-07` the Extension's own reading of the AI's plan (here you write and
> act on the plan yourself); none of those apply to this skill.

# Troubleshooting — myiDecide AI

Plain-English help for the messages the myiDecide Builder can show while it writes,
builds or edits a myiDecide presentation. Each entry has a short code
(`ACCT-01`, `TAB-02`, …) that the side panel quotes next to an error, so you can
jump straight to the matching section. Nothing here needs technical knowledge;
where a step does, it says so.

The `ACCT-*`, `API-*` and `NET-*` entries are about the **Chrome extension's**
connection to your myiDecide account — since 2.3.0 every AI request goes
through myiDecide on your own sign-in, and there is no API key to manage. (The
`KEY-01`…`KEY-06` codes of earlier versions, about a pasted Anthropic key, are
retired; a link to one of them lands on `ACCT-01`.) The Claude plugin (the
"myiDecide Builder" plugin for Cowork) runs on your Claude subscription and
never shows those — every other entry applies to both.

## Codes at a glance

| Code | What you'll see (short) | In one line |
|---|---|---|
| `ACCT-01` | "You're signed out of myiDecide" · `API 401` | Your myiDecide session ended — sign in on the tab that opens; the panel carries on by itself |
| `ACCT-02` | "Today's AI allowance is used up" · `API 429 rate_limit_error` | Your account's daily iDecide AI allowance is spent — it comes back at the reset time in ⚙ Settings |
| `ACCT-03` | "That model isn't available right now" · `404 not_found_error` "model: …" · `403 permission_error` | The model iDecide AI builds with couldn't be reached through myiDecide — wait a few minutes |
| `API-01` | "The AI request was refused" · `API 400` / `invalid_request_error` | The request was sent and refused; usually temporary — resume, then send the record |
| `API-02` | "iDecide AI is busy right now" · `529 overloaded_error` · `API 502/503` | The AI service is overloaded or briefly down — wait a minute or two |
| `NET-01` | "myiDecide couldn't be reached" · "Failed to fetch" | No answer from my.idecide.com — connection, captive Wi-Fi page, firewall or VPN |
| `TAB-01` | "Open your presentation in the myiDecide Builder first" · "I'm not seeing a myiDecide tab in front" | The panel can't find the myiDecide Builder tab it should work in |
| `TAB-02` | "Your presentation's tab has to be the one you can see" · "the build tools never loaded … because <reason>" | Chrome is not drawing the myiDecide Builder tab (minimised, behind another window, or behind another tab) so it never finishes starting |
| `TAB-03` | "builder error dialog detected" · "The myiDecide Builder crashed N times … mid-update" | The myiDecide Builder showed its Unknown Error dialog |
| `TAB-04` | "slides.changeSlide did not answer in 30s" · "could not open <slide> — skipped" · "export timed out after 20s" | The tab lost focus or the computer slept during a long pass |
| `TAB-05` | "This presentation is open in another tab" · `409 SESSION_MISMATCH` | The same presentation is open somewhere else and holds the lock |
| `BUILD-01` | "A build is running in your myiDecide Builder tab right now" | Edits wait until the running build finishes |
| `BUILD-02` | "I couldn't read any text out of <file>" | The uploaded script or brochure has no readable text |
| `BUILD-06` | "…is 140 MB — the limit for videos is 96 MB" · "Six files at a time is the limit" · "could not be uploaded into the presentation" | A file attached with the paperclip is over a limit, or the presentation tab wasn't reachable to upload it |
| `BUILD-03` | "wire: unknown target" | A button pointed at a slide that doesn't exist — repaired automatically since 1.1.0 |
| `BUILD-04` | "(NO VOICEOVER)" is spoken aloud | A silent-slide marker was narrated — fixed in 1.1.0; older decks can be cleared |
| `BUILD-05` | "iconify unreachable" · "display font unresolved" | An icon or font service couldn't be reached; built-in fallbacks were used |
| `BUILD-07` | "The AI's plan came back garbled" · `non-JSON reply (N chars…)` | The AI's plan wasn't in the exact format the extension reads — resume and it almost always comes back clean |
| `PANEL-01` | The side panel was closed while a build was running | Reopen it — History → Resume carries on where it stopped |
| `GEN-01` | "Something went wrong" | A problem with no specific explanation yet — try again, then send the build record |
| `HELP-01` | You need to send something to support | Where the build record is and what to send to hi@idecide.com |

---

## Your myiDecide account (Chrome extension only)

Since 2.3.0 the extension sends every AI request to **myiDecide** (the backend
at my.idecide.com), which runs it on iDecide's own account and counts it
against a **daily iDecide AI allowance** for your myiDecide user. You never
paste a key: the only thing the extension needs is that you are **signed in to
myiDecide in Chrome**. ⚙ Settings shows who you are signed in as and how much
of today's allowance is used; the slim bar at the bottom of the panel shows the
same figure while you work.

### ACCT-01 — You're signed out of myiDecide

**What you'll see**

- A bar under the panel's header: *"Sign in to myiDecide — iDecide AI builds
  through your myiDecide account"* with a **Sign in** button, and a myiDecide
  login tab opening by itself.
- Before a build: *"One thing first: you're signed out of myiDecide. Sign in
  with the button at the top and I'll start writing straight after."*
- Before a resume: *"Waiting on your myiDecide account"* with this card.
- In the build log: `API 401: not signed in to myiDecide`.

**What it means**

myiDecide no longer has a signed-in session for this Chrome profile. The
session expired, you signed out in another tab, or you signed in to myiDecide
in a different browser or profile. Nothing already built is lost — the
presentation lives in your myiDecide account, and the build record is kept in
the panel.

**What to do**

1. Sign in on the myiDecide tab that opened (or press **Sign in** at the top
   of the panel; it opens the login page and brings it forward).
2. Come back to the panel. It checks by itself as soon as the login page
   moves on, says *Signed in as …*, and the bar folds away. A build you had
   asked for while signed out starts on its own.
3. If a build had stopped: 🕘 History → **Resume build**. Progress is kept.

**If it keeps happening**

- Sign in to myiDecide in the **same Chrome profile** the extension is
  installed in — a different profile or an incognito window has its own
  cookies, and the panel cannot see them.
- An ad blocker or privacy extension that deletes cookies for my.idecide.com
  signs you out between visits; allow the site.
- Still stuck? See `HELP-01`.

### ACCT-02 — Today's AI allowance is used up

**What you'll see**

- *"Today's AI allowance is used up (ACCT-02)."*
- ⚙ Settings → *Today's AI usage*: **Limit reached**, with the reset time.
- In the build log: `API 429: … rate_limit_error …` and the build stopping
  rather than retrying.

**What it means**

Every myiDecide user has a daily iDecide AI allowance, and today's is spent.
It is a cap on how much AI work your account can ask for in a day — a
long Creative Build, several builds in a row, or a lot of edit requests will
reach it. The panel stops the build at once instead of retrying until the
reset, so no allowance is wasted on refused calls. Nothing already built is
lost.

**What to do**

1. Note the reset time in ⚙ Settings (*Resets at …*). The allowance comes
   back in full then.
2. After the reset, open 🕘 History and press **Resume build**. Finished
   slides are never redone; the build carries on from where it stopped.
3. If your work regularly needs more than a day's allowance, email
   **hi@idecide.com** from your myiDecide account.

**If it keeps happening**

- Quick Build uses roughly half the allowance of a Creative Build on the same
  presentation; a build that stops near the end can be resumed the next day
  and finishes in minutes.
- Still stuck? See `HELP-01`.

### ACCT-03 — That model isn't available right now

**What you'll see**

- *"That model isn't available right now (ACCT-03)."*
- In the build log: `API 404: … not_found_error … model: …` or
  `API 403: … permission_error …`.

**What it means**

myiDecide accepted the request but the AI model it builds with could not be
reached — the model is being updated on the service's side, or myiDecide's
own configuration is mid-change. That is on iDecide's side, not yours.

**What to do**

1. Wait a few minutes, then 🕘 History → **Resume build**.
2. If it is still refused after a few tries, email **hi@idecide.com** with the
   *Technical detail* line from the card.

### API-01 — The AI request was refused

**What you'll see**

- *"The AI request was refused (API-01)"* with the exact wording under
  *Technical detail*.
- In the build log: `API 400: … invalid_request_error …`.

**What it means**

The request reached myiDecide and was sent on, but was refused — most often a
request the model could not accept as sent (a very large attachment, say) or a
temporary fault on the service's side.

**What to do**

1. Try again in a minute (🕘 History → **Resume build**, or send your message
   again).
2. If the same message repeats, download the build record (🕘 History → ⬇)
   and email it to **hi@idecide.com** — see `HELP-01`.

### API-02 — iDecide AI is busy right now

**What you'll see**

- *"iDecide AI is busy right now (API-02)"*.
- In the build log: `API 529: … overloaded_error …`, `API 502` or `API 503`.

**What it means**

The AI service behind myiDecide is overloaded or briefly unavailable. The
panel retries short outages by itself; this card means they lasted longer
than it was willing to wait.

**What to do**

1. Wait a minute or two, then 🕘 History → **Resume build**.
2. If it keeps happening for more than a few minutes, email **hi@idecide.com**.

### NET-01 — myiDecide couldn't be reached

**What you'll see**

- *"myiDecide couldn't be reached (NET-01)"*.
- In the build log: `Failed to fetch`, `NetworkError`, `ERR_INTERNET_DISCONNECTED`,
  `ERR_NAME_NOT_RESOLVED`.

**What it means**

The request to my.idecide.com never got an answer. Usually a dropped
connection, a captive Wi-Fi login page that has not been clicked through yet,
or a firewall or VPN that blocks the address.

**What to do**

1. Check that **my.idecide.com** opens in a tab.
2. Try again in a moment. If a build had stopped: 🕘 History → **Resume
   build**.

## The myiDecide Builder tab

### TAB-01 — "Open your presentation in the myiDecide Builder first" / "I'm not seeing a myiDecide tab in front"

**What you'll see**

- While connecting: *"I'm not seeing a myiDecide tab in front. Switch to the
  tab with your presentation open in the myiDecide Builder, then tell me again."*
- *"That's myiDecide, but not the myiDecide Builder. Open the presentation itself (the
  editing view)…"*
- When resuming or editing: *"Open your presentation in the myiDecide Builder first
  (the tab I built it in), then try again."*
- Older versions: *"myiDecide Builder tab not found"*.

**What it means**

The panel reads whichever tab is **active** when you confirm, and it needs
that tab to be the myiDecide **Builder** — the address looks like
`my.idecide.com/builder/create/<number>`. The dashboard, the presentation
list or the player don't count, and neither does a tab in another window.

**What to do**

1. In the same Chrome window as the side panel, click the tab with your
   presentation open in the myiDecide Builder (you should see the slides).
2. If you haven't opened one yet: sign in at my.idecide.com and open (or
   create) the presentation so you're looking at the myiDecide Builder. When you
   start a new build the panel can also make a blank one for you — choose
   **Make me a new one**.
3. Back in the panel, press **Try again**.

**If it keeps happening**

- Make sure you're signed in — a signed-out tab lands on the login page,
  which is "myiDecide, but not the myiDecide Builder".
- Chrome side panels belong to a window. If the presentation is open in a
  different window, either move the tab into this window or open the panel
  from that window.

### TAB-02 — "Your presentation's tab has to be the one you can see"

**What you'll see**

- *"Your presentation's tab has to be the one you can see."* — with a line
  above it naming what the panel actually found, for example *"the build tools
  never loaded because Chrome is not drawing that tab — the window looks
  minimised"*, *"…because another tab is in front of it in that window"*, or
  *"…because its window is behind another app"*.
- Older wording, before 2.1.2: *"The myiDecide Builder tab hasn't finished
  loading."* / *"The myiDecide Builder reloaded but its tools never came up."*
  Before 1.1.0: *"myiDecide Builder page never mounted (engine present but no
  current page after 45s)"*.

**What it means**

Chrome stops drawing a tab that nobody is looking at, and the myiDecide Builder
**does not finish starting while it isn't being drawn** — minimised window,
window behind another app, or another tab in front of it in the same window all
count. The panel brings the tab forward itself first; when Chrome still isn't
drawing it, the panel stops after a few seconds instead of leaving you watching
a spinner for a minute and a half, and it names the reason it found.

**Nothing in your presentation is changed by this.** The failure happens before
any edit is written, so the deck is exactly as it was. Since 2.1.2 the panel
also prints what it was about to do, so you can see the change is only waiting.

Since 2.1.2 the panel no longer reloads the tab when the build tools are already
live in the page — the myiDecide platform sometimes rewrites its own address and
drops the `aiagent=` switch while the tools are still running, and reloading on
that was costing a wait for nothing.

**What to do**

1. Click the tab with your presentation so it's the one you can see, and wait
   until the slides appear.
2. Press **"Try that again ↻"** under the message. Your request is still
   there — you don't have to retype it. (**"Show me the tab"** next to it jumps
   straight to the presentation.)
3. Keep that window in front while a change runs. It only has to be visible;
   once the change finishes you can work in another app again.

**If it keeps happening**

- Reload the myiDecide Builder tab yourself (⌘R / F5), wait for the slides, then
  try again.
- A second Chrome window sitting on top counts as "behind another window", even
  when the presentation's own window is un-minimised.
- On a very slow connection the myiDecide Builder can still be loading when the
  tab comes forward — wait until the slides are drawn before pressing Try again.
- If the panel says it can't find a myiDecide tab at all, that's `TAB-01`, not
  this one.

### TAB-03 — "builder error dialog detected" / "The myiDecide Builder crashed N times … mid-update"

**What you'll see**

- In the build log: *"builder error dialog detected — refreshing the page"*,
  or a step name followed by *"builder error dialog detected"*.
- In the myiDecide Builder tab: a grey box titled **Unknown Error** — *"The application
  has encountered an unknown error. Please try to reload the page"* — with a
  single **Reload Page** button.
- If it happens repeatedly: *"The myiDecide Builder crashed 3 times in the last 8
  minutes and isn't recovering. This usually means the myiDecide platform is
  mid-update. Wait a minute, then hit Resume on the History screen — the
  build continues from where it stopped."*

**What it means**

The myiDecide Builder itself crashed. A single crash is normal on a long build
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

The myiDecide Builder stopped answering for a while. The usual cause is that its tab
**lost focus**: the myiDecide Builder only renders — and only moves between slides —
while its tab is visible, so switching to another tab, minimising Chrome,
locking the screen or the computer going to sleep freezes it mid-step. The
panel waits a fixed time, marks the slide as skipped and carries on, so one
stall can cost a slide its design, its wiring or its thumbnail.

**What to do**

1. **Keep the myiDecide Builder tab visible for the whole run.** Don't switch tabs in
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
- If every slide stalls, the myiDecide Builder is not really loaded: see `TAB-02`.

### TAB-05 — "This presentation is open in another tab" / 409 SESSION_MISMATCH

**What you'll see**

- A myiDecide message saying the presentation is open in another tab, or
  asking you to reload.
- In the build log: a save or upload failing with `409` (sometimes with
  `SESSION_MISMATCH` in the text), or repeated `400`s on every slide.

**What it means**

myiDecide lets **one tab at a time** write to a presentation. Every myiDecide Builder tab
gets its own session ticket, and the newest tab holds the lock. If the same
presentation is open in a second tab, a second window, another browser or
another device — or you duplicated the tab — the myiDecide Builder's tab is now the
stale one and the platform rejects its saves.

**What to do**

1. Close every other tab, window and device that has this presentation open.
2. Reload the myiDecide Builder tab (⌘R / F5) so it takes the lock back, and wait for
   the slides to show.
3. In the panel, open 🕘 History → **Resume build** (or Try again). Slides
   that failed to save are rebuilt; the rest are kept.

**If it keeps happening**

- Someone else on your team may have the presentation open. Ask them to close
  it while the build runs.
- Check the player too: a preview tab of the same presentation can count.

---

## During a build or an edit

### BUILD-01 — "A build is running in your myiDecide Builder tab right now"

**What you'll see**

*"A build is running in your myiDecide Builder tab right now — I can make changes as
soon as it finishes. Your message is safe to resend then."*

**What it means**

You asked for a change while the build (or a rebuild started by an earlier
change) was still working in the tab. Only one thing can drive the myiDecide Builder at
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

- Check the slide's Notes in the myiDecide Builder — if the marker text is
  still there, ask for the notes to be cleared as well.

### BUILD-05 — "iconify unreachable" / "display font unresolved"

**What you'll see**

In the build log:

- `iconify unreachable — using the built-in icon set`
- `display font unresolved (<reason>) — the engine default will be used`, or
  the same for `body font`

**What it means**

Icons come from an online icon library (Iconify) and fonts are looked up in
the myiDecide Builder's font catalogue. If the icon service can't be reached — a
firewall, a captive Wi-Fi login page, or the service being down — the build
uses the icon set bundled with the extension instead. If a font the brand
asked for isn't in the catalogue, the myiDecide Builder's default font is used. The build
completes either way; the slides just use the fallback.

**What to do**

1. Nothing is required. Look through the finished deck: if the icons or type
   look wrong, ask for changes — *"use icons from the online set"* or *"set
   the headline font to <font name>"* — once you're on a normal connection.
2. If the icon service was blocked by your network, check that
   `api.iconify.design` is allowed, or run the build on another network.

**If it keeps happening**

- A font that is never found is probably not one the myiDecide Builder offers.
  Pick a similar one from the myiDecide Builder's font list and name that instead.

### BUILD-06 — "the limit for videos is 96 MB" / "Six files at a time is the limit" / "could not be uploaded into the presentation"

**What you'll see**

- *"logo-final.png is 18 MB — the limit for images is 12 MB."*
- *"Six files at a time is the limit — send these first."*
- In a reply: *"the file … could not be uploaded into the presentation"*, or
  *"the attached file(s) could not be uploaded — the myiDecide Builder tab was
  not reachable"*.

**What it means**

The 📎 paperclip beside the message box (2.1.2 and later) attaches your own
files to a change request. Pictures and video are uploaded **into the
presentation's own media library** — the myiDecide platform stores them, the
extension stores nothing — and the assistant then places them by name.
Documents (.docx, .pptx, .pdf, .txt, .csv, .md) are read and handed over as
text, so their real copy and figures can be used.

Because each message carries the file to the presentation in one go, there are
limits: **12 MB per picture, 96 MB per video, 24 MB per document, six files per
message**. A file over its limit is refused before anything is sent. A file
that is within its limit but still won't go up almost always means the
presentation's tab wasn't reachable at that moment (see `TAB-02`) — the upload
travels through that tab.

**What to do**

1. Export or compress the file under its limit. A 1558×720 deck never needs a
   picture wider than about 1920px, and a long video is better trimmed to the
   seconds that appear on the slide.
2. If you attached more than six files, send them in two messages.
3. If the message says it couldn't be uploaded into the presentation, click the
   presentation's tab so it's the one you can see, then attach the file again.
4. Once a file is in, it stays available for the rest of the session — you can
   say *"put the logo on slide 3"* later without attaching it again.

**If it keeps happening**

- A document that opens but has no text inside is `BUILD-02`, not this one.
- SVG logos upload as pictures, but a very complex SVG can fail to draw; export
  it as a PNG at the size you need instead.
- Files attached in an earlier session are not carried over — the list only
  covers the session you're in.

### BUILD-07 — "The AI's plan came back garbled" / "non-JSON reply (N chars; head: … tail: …)"

**What you'll see**

- *"The AI's plan came back garbled"*, with a technical detail beginning
  `non-JSON reply (19613 chars; head: "{\"theme\":…`.

**What it means**

The build asks the AI for each stage's plan in a strict format (JSON) that the
extension then reads. This time part of the reply wasn't in that format — a
single wrong character is enough — so none of it could be used.

The extension already handles most of these on its own. It mends the slips that
have only one possible meaning (a number written `06` instead of `6`, a stray
comment, a trailing comma, a line break inside a sentence, an unescaped quote
mark), and if that isn't enough it asks the AI once more, telling it exactly
what was wrong and where. You only see this code when a reply got past both.

**What to do**

1. Resume the build: 🕘 **History → Resume build**. Your answers and your
   script are kept, and a second attempt almost always comes back clean.
2. If you changed the **Build mode** or the model since the build started,
   that's fine — a resumed build uses the settings on screen.

**If it keeps happening**

- Download the build record (🕘 History → ⬇) and send it to hi@idecide.com.
  Since 2.2.0 the record carries the **exact reply that failed** (both
  attempts), so the faulty character can be found rather than guessed at.
- Two replies failing the same way usually means the instructions the AI was
  given contradict each other — that's a fault on our side, not yours.

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
   build**. It reconnects to the myiDecide Builder tab (see `TAB-01` if it can't) and
   continues from the last finished slide.

**If it keeps happening**

- A side panel belongs to one Chrome window and isn't visible from the
  others. Keep the panel and the myiDecide Builder tab in the same window and stay in
  it.
- Chrome's own updates can restart the browser mid-build; postpone them until
  the build finishes.

---

## Getting help

### GEN-01 — "Something went wrong"

**What you'll see**

- *"Something went wrong — myiDecide AI hit a problem it doesn't have a
  specific explanation for"*, with the exact message under **Technical detail**.

**What it means**

This is the catch-all: the extension stopped on something it has no specific
explanation for yet. Every failure we have seen and understood has its own code
on this page — GEN-01 means this one is new to us.

**What to do**

1. Try the step again: 🕘 **History → Resume build** for a build, or send your
   message again for an edit. Your progress is saved.
2. If it happens a second time, don't keep retrying — send it to us (below).

**If it keeps happening**

- Download the build record and email it, with the message, to
  hi@idecide.com — see `HELP-01` for exactly what to send. A GEN-01 that
  repeats is how new codes get added to this page.

---

### HELP-01 — What to send to support

**Where the build record is**

Open 🕘 **History** in the panel. Every run that was saved has a **⬇** button
(*Download record*); it saves a file named
`build-record-<brand>-<date>.json` to your Downloads folder. It contains the
questionnaire answers, the script, every step of the build log, every edit
request and reply, and the timings — everything needed to see what happened.
It does **not** contain your myiDecide password or any sign-in cookie.

**What to send**

Email **hi@idecide.com** with:

1. The build record file (⬇ from History).
2. The code from this page that matches what you saw (for example `TAB-04`),
   or the exact message if it isn't listed.
3. The presentation's address from the myiDecide Builder tab
   (`my.idecide.com/builder/create/<number>`).
4. Roughly when it happened and what you were doing (building, resuming,
   asking for a change).

**Before you send**

- Try the "What to do" steps for the matching code once — most stops resume
  cleanly from 🕘 History.
- Never send your myiDecide password. Support does not need it — the
  address of the presentation and the record are enough.
