# myiDecide Builder — a Claude plugin

Build and edit **interactive iDecide presentations** by talking to Claude.

An iDecide presentation is not a linear deck. Viewers *choose* what they watch:
menus branch into topics, questions branch on the answer, and every path ends at
a call to action. This plugin teaches Claude to design for that — writing the
script, composing slides, sourcing stock footage, generating narration, and
wiring the menus and buttons — working directly in the myiDecide editor.

## Install

**Claude app or Cowork**

1. Open **Customize** in the left sidebar → **Plugins**
2. Under **Personal plugins**, click **+** → **Add marketplace**
3. Choose **Add from a repository** and paste:
   `https://github.com/iDecideAI/myidecide-builder`
4. Find **myidecide-builder** and click **Install**

**Claude Code**

```
/plugin marketplace add iDecideAI/myidecide-builder
/plugin install myidecide-builder@idecide
```

## What you need

- A **myiDecide account** and a builder URL — it looks like
  `https://my.idecide.com/builder/create/<sessionId>`
- A way for Claude to drive your browser (Claude in Chrome, or Cowork with
  browser access). The plugin works against the editor's own agent API in the
  page.
- **No Anthropic API key.** It runs on your own Claude subscription.

## Using it

Give Claude your builder URL and say what you want:

> Build me an eight-slide presentation about our onboarding process from this
> script — here's the builder link.

> Open this deck and re-time every slide so the graphics hold half a second
> past the narration.

Claude reads the editor's live instructions each session, so it stays current
with the platform rather than working from a frozen snapshot.

## What's inside

| File | What it covers |
|---|---|
| `SKILL.md` | The build and edit procedure |
| `references/aiagent-surface.md` | The editor's agent API, with worked examples |
| `references/composition.md` | Element contract and the design playbook |
| `references/deck-outline.md` | The per-slide plan contract |
| `references/platform-facts.md` | Verified platform behaviour and footguns |
| `references/script-craft.md` | Writing the narration and on-screen copy |

Claude loads `SKILL.md` first and pulls a reference in only when it needs it.

## Support

Questions and bug reports: [idecide.com](https://idecide.com)

Licensed for use with the myiDecide platform — see [LICENSE](LICENSE).
