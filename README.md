# html-to-figma-sync

**Your HTML in Figma at full fidelity: the same page, not a redraw. Bound to your design system,
and kept in sync in both directions.**

A [Claude Code](https://claude.com/claude-code) skill. You describe what you want, Claude does the
capture, the binding and the updates.

![The same pricing page twice, side by side and identical: on the left in the browser, on the right in Figma as editable layers exported by html-to-figma-sync. Below: 90 of 90 colours bound to variables, 48 of 48 text layers on text styles, and sync in both directions.](assets/browser-vs-figma.png)

---

## Why

Redrawing a coded page in Figma by hand takes hours and is never quite right. Capturing it is fast,
but what you get back is a frame full of raw hex values and loose text: it looks right, but it isn't
connected to anything. Change a token and nothing updates. Change the copy in the HTML and the Figma
file is out of date. Change it in Figma and the code is.

`html-to-figma-sync` fixes all three:

- **Full fidelity.** A real capture of your real page, every state you need, as editable layers.
  What's in Figma is what's in the browser, down to the pixel.
- **Bound to your design system.** Colours become your variables, text gets your text styles,
  shadows get your effect styles. Only where it's an exact match, so nothing quietly changes.
- **Kept in sync, both ways.** Change the HTML, tell Claude, and the Figma frame follows. Copy
  changes update in place, so anything you tweaked by hand in Figma survives. Change copy or a
  token in Figma, and Claude brings it back into the code.

---

## Install

**As a Claude Code plugin** (recommended). Inside Claude Code:

```
/plugin marketplace add tonioloz/html-to-figma-sync
/plugin install html-to-figma-sync
```

**Or copy the skill by hand.** Download this repo and copy `skills/html-to-figma-sync` into
`~/.claude/skills/`.

**You'll also need** the official [Figma MCP server](https://help.figma.com/hc/en-us/articles/32132100833559)
connected to Claude Code, and the fonts your HTML uses installed on your machine.

---

## Use it

Just ask. There are no commands to learn.

| Say | What happens |
|---|---|
| *"Export pricing.html to Figma"* | First time only: Claude asks which Figma file and page, and which design system to use, then shows you what it found. Then it captures the page and binds it. |
| *"I changed the button to 'Start your free trial', update it in Figma"* | Only that text layer changes. Claude shows you the change first. |
| *"I reworked the pricing page, update it in Figma"* | A fresh capture lands **next to** the old frame, bound and checked. Nothing is deleted. |
| *"I changed the headline and `primary` in Figma, update the code"* | Claude finds what changed in Figma since the last sync, shows you the code changes (the text in your HTML, the `--primary` value in your CSS), and applies them on your yes. |
| *"Pull my Figma changes"* | Same, for everything that changed in Figma. |
| *"Check this frame"* | A read-only report: hardcoded colours, text without a style, styles from outside your system, layers still called `Frame 427`. |

Claude remembers your answers in a small `figma-map.json` next to your HTML, so it only asks once.
It's plain JSON, edit it whenever you like.

Works with a **published Figma library** or with **variables and styles defined in the file**.

### What syncs in each direction

| | HTML → Figma | Figma → HTML |
|---|---|---|
| Copy | Updated in place | Updated in your HTML source |
| Colour tokens | Bound to your variables | Variable values written to your CSS custom properties |
| Text styles | Applied by size, weight, line height | Style values written to the matching CSS rule |
| Layout and structure | Full re-export, placed beside the old frame | Not synced. Listed for you to change by hand |

Layout from Figma to code is left out on purpose: doing it badly breaks your HTML, and a list of
"moved this, resized that" is more useful than a guess.

**When both sides changed the same thing,** Claude shows you both versions and asks. It keeps a
small snapshot after every sync (in `.figma-sync/`), so it always knows which side changed.

---

## The rules it follows

Each of these exists because breaking it damaged a real file.

| Rule | Why |
|---|---|
| **Binds a colour only when it's an exact visual match** | A nearest-match approach turned navy panels black. Anything that doesn't match stays as hex and is listed, so you can decide whether the HTML or the system is wrong. |
| **Never binds a translucent paint** | Binding resets opacity to 100%, so scrims and frosted cards turn solid. |
| **Never binds to a translucent variable** | A solid fill bound to one turns see-through. |
| **Matches text styles by what they are, not their names** | Size, weight, line height. Works with any naming scheme. Exact by default, nearest if you choose it. |
| **Updates copy in place** | Re-capturing a frame to change three words throws away your hand tweaks. |
| **Never overwrites the other side** | If the HTML and Figma both changed the same text, you pick. Silent last-write-wins loses work. |
| **Never deletes your frames** | Re-exports land next to the old frame. You decide what to remove. |
| **The check never fixes anything** | It reports and offers. Every fix needs your yes. |

---

## Try it in five minutes

The [`demo/`](demo) folder has an editorial pricing page for a made-up invoicing app, built on six colour tokens and eight text styles, and the
steps to make your own before/after in a blank Figma file. See [`demo/README.md`](demo/README.md).

---

## Optional: FigCli for faster checks

If you use [FigCli](https://github.com/silships/figma-cli), the end-of-job check runs through it.
Claude confirms FigCli is connected to the right file (by file key, not name) before reading
anything. Without FigCli, the same check runs through the Figma MCP.

---

## Credits

Built on the official [Figma MCP server](https://help.figma.com/hc/en-us/articles/32132100833559)
and its HTML capture. Optional checks via [FigCli](https://github.com/silships/figma-cli) by
Sil Bormüller.

Made by [Antonio Lozano](https://github.com/tonioloz). MIT licence.
