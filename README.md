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

**In the Claude app** (or Cowork — open the Cowork tab first, then Customize).
There are two things to add: the *marketplace*, then the *plugin* on it. Missing
the second step is the usual reason nothing appears.

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

Same plugin, same marketplace. Most people use the app; this is here because
you are reading it on GitHub.
</details>

Updates arrive through the marketplace when we publish a new version. A plugin
added with *Upload plugin* is a fixed copy and cannot update itself, which is
why the steps above use a marketplace.

## What you need

- A **myiDecide account** and a builder URL — it looks like
  `https://my.idecide.com/builder/create/<sessionId>`
- A way for Claude to drive your browser (Claude in Chrome, or Cowork with
  browser access). The plugin works against the editor's own agent API in the
  page.
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

Claude reads the editor's live instructions at the start of every session, so it
stays current with the platform rather than working from a frozen snapshot.

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
