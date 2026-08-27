# myiDecide Builder — a Claude plugin

Build and edit **interactive iDecide presentations** by talking to Claude.

[myiDecide](https://idecide.com) is a platform for presentations that branch. An
iDecide presentation is not a linear deck: viewers *choose* what they watch —
menus open into topics, questions branch on the answer, and every path ends at a
call to action. This plugin teaches Claude to design for that, then build it —
writing the script, composing the slides, sourcing stock footage, generating the
narration and wiring the menus and buttons, working directly in the myiDecide
editor while you watch.

## Install

**In Cowork**, in the Claude desktop app — open the **Cowork** tab first, then
**Customize**. There are two things to add: the *marketplace*, then the *plugin*
on it. Missing the second step is the usual reason nothing appears.

1. Open **Customize** in the left sidebar, then the **Plugins** tab
2. Click **Add** at the top right → **Add marketplace**
3. Choose **Add from a repository** — not *Browse Anthropic sources*
4. Paste `https://github.com/iDecideAI/myidecide-builder`, then click the
   **Use "…"** line that appears underneath
5. **Search the plugin list for “myiDecide Builder” and click Add.** Adding the
   marketplace on its own installs nothing
6. Start a **new** chat — plugins load when a conversation begins — then type
   `/` and pick **idecide-presentation-builder**

**A walkthrough with screenshots of every screen:**
[idecide.com/idecide-skill](https://idecide.com/idecide-skill)

<details>
<summary>Installing from Claude Code instead</summary>

```
/plugin marketplace add iDecideAI/myidecide-builder
/plugin install myidecide-builder@idecide
```

Same plugin, same marketplace. **Cowork is the supported way to run it** — that
is what we test against and what the walkthrough covers. These commands are here
because you are reading this on GitHub.
</details>

Updates arrive through the marketplace when we publish a new version. A plugin
added with *Upload plugin* is a fixed copy and cannot update itself, which is
why the steps above use a marketplace.

## What you need

- A **myiDecide account** and a builder URL — it looks like
  `https://my.idecide.com/builder/create/<sessionId>`
- **Cowork, in the Claude desktop app** for Mac or Windows, on a paid plan —
  Pro, Max, Team or Enterprise. Cowork is not on the Free plan. Claude builds by
  driving the myiDecide editor in a browser, through the editor's own agent API
  in the page.
- **No Anthropic API key.** It runs on your own Claude subscription.

## Using it

Type `/` and pick the skill, or just say what you want — it starts on its own.
Either way it opens by asking whether you are building something new or editing
a deck you already have, then asks for the matching builder URL.

For a new build it walks a short questionnaire — what the presentation is for,
who is watching, what they should be able to choose between, where they should
end up — and it will take an existing script, brochure, PowerPoint or logo if
you have one. Hand it a script and it keeps your structure and your wording
rather than inventing its own. Then it writes, builds, and stays open for edits.

To change a deck you already have, give it that deck's URL and say what you
want different. It reads the slide before it touches it, and it never redraws
something it did not design.

Claude re-reads the editor's API reference at the start of every session, so it
builds against the platform as it is today rather than a frozen snapshot.

## What it reads and writes

Everything this plugin knows is in this repository. `SKILL.md` and the files
under `references/` are the whole of its behaviour — plain Markdown you can
read — and nothing it reads at runtime changes them.

While a build runs, in the browser tab you opened:

- **Reads the editor's API reference** at `window.aiagent.instructions`: the
  method names and signatures published by the myiDecide editor. A call
  inventory. It carries no guidance about how Claude should behave.
- **Reads and writes the one presentation you pointed it at**, through that
  same API and your existing myiDecide login. Nothing else in your browser,
  and no other site.
- **Sends narration text and footage searches to myiDecide**, which generates
  the audio and fetches the clips server-side and saves them onto your
  presentation — the platform doing what it does when you build by hand.

It needs **no API key** and holds no credentials of its own. Your questionnaire
answers, and any script, brochure or logo you attach, reach Claude in your
conversation exactly the way anything else you type does; the plugin sends them
nowhere else. Data handling on the myiDecide side is covered by the
[iDecide policies page](https://idecide.com/policies/).

## What's inside

| File | What it covers |
|---|---|
| `SKILL.md` | The session flow, and the build and edit procedures |
| `references/aiagent-surface.md` | The editor's agent API, with worked examples |
| `references/composition.md` | Element contract and the design playbook |
| `references/deck-outline.md` | The per-slide plan contract |
| `references/platform-facts.md` | Verified platform behaviour and footguns |
| `references/script-craft.md` | Writing the narration and on-screen copy |
| `references/test-brands.md` | Prefilled answers for demo builds |

Claude loads `SKILL.md` first and pulls a reference in only when it needs it.

## Support

Questions and bug reports: [idecide.com](https://idecide.com)

Licensed for use with the myiDecide platform — see [LICENSE](LICENSE).
