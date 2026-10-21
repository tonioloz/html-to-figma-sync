# Demo: make your own before/after

A blank Figma file, a small pricing page, ten minutes.

`pricing.html` is a pricing page for Tally, a made-up invoicing app. Every colour, text size and
the logo come from a small token set, so after capture every layer binds to a Figma variable or
text style:

| CSS | Figma variable (collection `Tally`) | Light | Dark |
|---|---|---|---|
| `--paper` | `paper` | `#fafafa` | `#111113` |
| `--ink` | `ink` | `#18181b` | `#f4f4f5` |
| `--ink-muted` | `ink-muted` | `#52525b` | `#a1a1aa` |
| `--rule` | `rule` | `#d4d4d8` | `#3f3f46` |
| `--accent` | `accent` | `#047857` | `#10b981` |
| `--on-accent` | `on-accent` | `#fafafa` | `#052e1f` |

| CSS class | Figma text style | Font |
|---|---|---|
| `.t-display` | `Display` | Newsreader Regular 72/72, -2% |
| `.t-figure` | `Figure` | Newsreader Regular 56/56, -2% |
| `.t-title` | `Title` | Newsreader Medium 28/32 |
| `.t-lead` | `Lead` | Schibsted Grotesk Regular 18/28 |
| `.t-body` | `Body` | Schibsted Grotesk Regular 15/24 |
| `.t-body-strong` | `Body Strong` | Schibsted Grotesk Medium 15/24 |
| `.t-label` | `Label` | Schibsted Grotesk Medium 13/20 |
| `.t-wordmark` | `Wordmark` | Schibsted Grotesk SemiBold 22/24, -3% |

## 1. Set up the Figma file

1. Create a new Figma design file.
2. Ask Claude: *"Create the Tally variables and text styles from demo/README.md in this file"*,
   and paste the file link. Or make them by hand from the tables above.

## 2. Capture the "before"

> *"Capture demo/pricing.html into my demo file, in light mode, but skip the design system pass. I
> want the raw capture."*

Select a few layers and look at the right-hand panel: every fill is a raw hex.

Capturing tip: keep the browser tab in front while it captures. Browsers pause background tabs.
If your computer is in dark mode, ask for light mode explicitly, or the capture comes in dark.

## 3. Capture the "after"

> *"Now export demo/pricing.html to the same file properly, using the Tally collection."*

On the run this demo was built with: 90 of 90 colours bound, 48 of 48 text layers styled, the
logo's strokes bound to `ink` and its strike to `accent`. Select the same layers: fills now read
`ink`, `rule`, `accent`.

## 4. Try both directions

**HTML to Figma.** Change the headline in `pricing.html` to *"Get paid on time, every time."*, then:

> *"I changed the headline, update it in Figma."*

**Figma to HTML.** In Figma, change the `accent` variable to another colour, then:

> *"I changed accent in Figma, update the code."*

`--accent` in `pricing.html` follows, and the button, the strike in the logo and the "Most teams
pick this" label all change with it.

## 5. The image in the README

`assets/before-after.png` was made from this demo: a board in the Figma file built from the two
real frames, with the values read straight from the layers, then exported.
