---
name: html-to-figma-sync
description: Turn any HTML page into editable Figma layers bound to the user's design system (colour variables, text styles, effect styles), then keep the two in sync in both directions. Use when someone asks to export, capture, send or push HTML into Figma, to "update this copy in Figma", to "update this screen in Figma", to bring Figma changes back into the code ("I changed the headline in Figma, update the HTML", "pull my Figma changes"), or to check that frames in Figma use their design system's variables and styles. Uses the official Figma MCP (generate_figma_design, use_figma, search_design_system). FigCli is optional, only for the final check.
---

# HTML to Figma sync

Captures HTML pages into Figma as real, editable layers, binds them to the user's design system,
and keeps the HTML and Figma in step in both directions.

Nothing syncs on its own. The user changes one side, then asks for that change on the other:

| The user says | Job |
|---|---|
| "Export these pages to Figma" | [First export](#first-export) |
| "I changed the button copy, update it in Figma" | [Copy update in place](#copy-update-in-place) |
| "I reworked the pricing page, update it in Figma" | [Page re-export](#page-re-export) |
| "I changed the headline in Figma, update the HTML" / "pull my Figma changes" | [Figma to code](#figma-to-code) |
| "Check this frame" | [The check](#the-check) |

## Requirements

- The **official Figma MCP server** connected (`generate_figma_design`, `use_figma`,
  `search_design_system`). If it isn't, stop and tell the user how to connect it.
- **Figma Desktop or the browser**, with the target file the user can edit.
- The brand fonts installed on the machine, or captured text falls back to a generic sans.

## Ground rules

These hold for every job. Each one exists because breaking it damaged a real file.

- **Never delete the user's nodes.** Re-exports go beside the old frame, never over it.
- **Bind only exact matches.** A colour binds to a variable only when they are visually identical.
  Nearest-match binding was tried and turned navy panels black.
- **Never bind a paint whose opacity is below 1.** Binding resets paint opacity to 100%, so
  scrims and translucent cards turn solid.
- **Never bind to a variable whose value has alpha below 1.** A solid fill bound to one turns
  see-through.
- **Show changes before making them** when they touch existing frames (copy updates, fixes after
  the check). Captures into an empty page are fine to run.
- **Read-only means read-only.** The check never fixes. It reports and offers.
- **Never overwrite a change made on the other side.** When the same thing changed in both the HTML
  and Figma, show both and let the user pick.

## The map file

Each project keeps a `figma-map.json` next to its HTML. It records where things go in Figma and
which design system to bind to, so Claude asks once.

```json
{
  "figmaFile": { "key": "AbC123xyz", "name": "Pricing page", "url": "https://www.figma.com/design/AbC123xyz" },
  "page": { "id": "0:1", "name": "Exports" },
  "designSystem": {
    "source": "library",
    "libraryName": "Acme Design System",
    "colorCollection": { "key": "4f1c...", "name": "Semantic", "mode": "Light" },
    "textStyles": "library",
    "textStyleMatch": "exact",
    "effectStyles": "library",
    "fonts": ["Inter"]
  },
  "tokenMap": { "primary": "--primary", "muted-foreground": "--muted-foreground" },
  "styleMap": { "Heading/Display": "h1", "Body/Default": "body" },
  "screens": [
    {
      "source": "pricing.html",
      "state": "default",
      "frameName": "Pricing / Default",
      "nodeId": "12:345",
      "exportedAt": "2026-10-21"
    }
  ]
}
```

- `designSystem.source` is `library` (a published Figma library enabled in the file) or `local`
  (variables and styles defined in the file itself).
- `textStyleMatch` is `exact` (default) or `nearest`. See [Text styles](#3-apply-text-styles).
- `tokenMap` links each colour variable to the CSS custom property that holds it. `styleMap` links
  each text style to the CSS selector that sets it. Both are filled in the first time
  [Figma to code](#figma-to-code) needs them.
- The user can edit this file by hand. Respect what it says.

## The snapshot

After every sync, in either direction, save what the frame looked like to
`.figma-sync/<frameName>.json` next to the map (slashes and spaces in the name become `-`):

```json
{
  "nodeId": "12:345",
  "savedAt": "2026-10-21T10:42:00Z",
  "texts": { "12:401": "Start free trial", "12:388": "Simple pricing for growing teams" },
  "variables": { "primary": "#18181b", "foreground": "#09090b" },
  "textStyles": { "Heading/Display": { "family": "Inter", "style": "Bold", "size": 36, "lineHeight": 40, "letterSpacing": -0.72 } }
}
```

The snapshot is what makes two-way sync safe. Comparing Figma now with the snapshot shows what
changed **in Figma**. Comparing the HTML now with the snapshot shows what changed **in the HTML**.
The same thing changed on both sides means a conflict, and the user decides.

Read texts with `use_figma` (node id and `characters` for every TEXT node in the frame), variable
values from the mapped collection and mode (hex, aliases resolved), and text style values from the
mapped styles.

## First run: set up

Only when there is no `figma-map.json`, or it is missing a part. One question at a time.

### 1. Where should the frames go?

Ask for the Figma file link and the page. Offer to create a new page (for example `Exports`) so
existing work is never disturbed. Record `figmaFile` and `page`.

### 2. Which design system?

Ask: "Which design system should these frames use? A published library, or variables and styles
in this file?" Then discover it and **show what you found before saving**:

**Published library**
- List the colour collections with `figma.teamLibrary.getAvailableLibraryVariableCollectionsAsync()`
  (via `use_figma` on the target file). Only libraries enabled in that file appear. If the user's
  library is missing, ask them to enable it (Assets panel, Libraries) and try again.
- Text and effect styles: `search_design_system` (entity `style`), restricted to that library.

**Local**
- Colour collections: `figma.variables.getLocalVariableCollectionsAsync()`.
- Text and effect styles: `figma.getLocalTextStylesAsync()`, `figma.getLocalEffectStylesAsync()`.

**Pick the colour collection.** If there are several (for example `primitives` and `semantic`),
recommend the semantic one: binding to `primary` survives a rebrand, binding to `blue/600`
doesn't. Ask which mode to match against (usually Light).

**Fonts.** Ask which brand fonts the HTML uses, or read them from the CSS.

Then report it in one short block and save it:

```
Found: Acme Design System
  Colours   "Semantic" collection, Light mode, 42 variables (6 skipped: translucent)
  Text      24 text styles
  Effects   5 shadow styles
  Fonts     Inter
Save this to figma-map.json?
```

## First export

1. Confirm the exact pages and states (one frame per state). For each, build one export-ready
   HTML file:
   - Strip any prototype controls, debug bars or scenario switchers (product UI only).
   - Bake the target state on load by driving the page's own controls (click the real toggles,
     buttons, menus), so modals and menus are open at capture time.
   - Kill CSS transitions and animations so overlays capture at their final state.
   - Keep font paths resolving.
   - Add the capture script in the head:
     `<script src="https://mcp.figma.com/mcp/html-to-design/capture.js" async></script>`

   Keep a small commented generator script that produces these files, so the set is reproducible
   and a re-export uses the same recipe.
2. Serve the files from a local static server, rooted so font paths resolve. `file://` URLs are
   not supported.
3. Call `generate_figma_design` with the file key (no captureId) to get a capture id, then open
   the local URL with the hash:
   `open "<url>#figmacapture=<id>&figmaendpoint=<endpoint>&figmadelay=1200"`.
   Poll `generate_figma_design` with the file key and capture id every 5 seconds until the status
   is `completed`.
4. **Capture one page at a time, in the foreground.** Browsers freeze JS in background tabs, so
   only the active tab captures. Open a tab, give it about 7 seconds in front, then move on.
5. Move each frame onto the mapped page, name it `<Page> / <State>`, and record `nodeId`,
   `frameName`, `source`, `state` and `exportedAt` in `screens`.
6. Run the [design system pass](#the-design-system-pass).
7. Save the [snapshot](#the-snapshot).
8. Run [the check](#the-check).

For external sites (not localhost), use the Playwright capture path instead of the script tag.

## Copy update in place

For a change to words only. The frame, its bindings, its styles and anything the user tweaked by
hand in Figma all stay. Only the text changes.

1. Find the frame from the map (`source`, `state`). If the `nodeId` no longer exists or was
   moved, **stop and ask**. Never recreate it silently.
2. Read the frame's text layers with `use_figma` (read-only): id, name, `characters`.
3. Work out the change from what the user said, or by diffing the HTML text against the Figma
   text. Match each change to a layer by its **current text**. Captured layer names aren't
   reliable. If the layer's text in Figma differs from the [snapshot](#the-snapshot), someone
   edited it in Figma too: show both versions and ask which one wins.
4. Show the list, then wait for a yes:

   ```
   Pricing / Default
     "Start free trial"  →  "Start your free trial"   layer 12:401
     "Save 20%"          →  (no match found, fix by hand)
   ```
5. Apply with `use_figma`:
   - Load every font in the layer first (`getStyledTextSegments(['fontName'])`, then
     `figma.loadFontAsync` for each).
   - **One style throughout:** set `characters`. Text style and colour binding carry over.
   - **Mixed styles** (a bold word in a sentence, a link): don't rewrite it. List it for the user.
   - **Copy split across several layers:** list it. Don't merge layers.
6. Report any layer whose height changed (the new copy wraps onto another line).
7. Update `exportedAt`, save the [snapshot](#the-snapshot), then run [the check](#the-check) on the
   frame.

## Page re-export

For a change to layout or structure.

1. Re-run the generator for that page and capture it as in the first export.
2. Place the new frame **next to** the old one. Don't delete the old one. Give the user both node
   ids. Say that hand tweaks on the old frame don't carry over.
3. Point the map's `nodeId` at the new frame.
4. Run the [design system pass](#the-design-system-pass), save the [snapshot](#the-snapshot), and
   run [the check](#the-check).

## Figma to code

The other direction: the user changed something in Figma and wants the HTML to follow. v1 covers
**copy** and **design tokens**. It does not cover layout.

1. **Find what changed in Figma.** Read the frame and the design system as described in
   [The snapshot](#the-snapshot) and compare with the saved snapshot. If there is no snapshot (the
   frame was exported before this existed), compare Figma with the HTML directly and treat every
   difference as "unknown side": show it and ask which side is right.
2. **Sort each change** into one of these:

   | Changed in Figma | Goes to the code as |
   |---|---|
   | Text in a layer | The same text replaced in the HTML **source** (`source` in the map, never the export copy) |
   | A colour variable's value | The CSS custom property in `tokenMap` |
   | A text style's size, weight, line height or letter spacing | The CSS rule for the selector in `styleMap` |
   | Anything else (moved, resized, new or deleted layers, a layer bound to a different variable) | **Not applied.** Listed as "change by hand in the code", with the layer name |

3. **Check the other side.** For each change, look at the HTML. If the HTML still has the snapshot's
   value, it's a clean Figma change. If the HTML has a third value, both sides changed: it's a
   conflict.
4. **Show the plan, then wait for a yes:**

   ```
   From Figma → pricing.html
     Copy     "Simple pricing for growing teams"  →  "Pricing that grows with you"
     Token    --primary   #18181b  →  #2563eb   (variable "primary")
     Style    h1 font-size   36px  →  40px      (text style "Heading/Display")
   Conflict   "Start free trial": HTML says "Try it free", Figma says "Start your free trial". Which one?
   By hand    "Team" card moved above "Starter" (layout, not applied)
   ```
5. **Apply only what was approved:**
   - **Copy:** find the old text in the source HTML. One match: replace it. Several: show each
     with its surrounding line and ask which. No match (the text is built by a script, or split
     across tags): list it, don't guess. Keep the HTML around the text untouched (tags, entities,
     whitespace).
   - **Token:** if `tokenMap` has no entry for the variable, look for a custom property with the
     same name (`primary` to `--primary`, `bg/surface` to `--bg-surface`). Confirm it with the
     user the first time, then save it to `tokenMap`. Change only that declaration's value. Match
     the file's existing format (hex case, `rgb()`, `oklch()`).
   - **Text style:** if `styleMap` has no entry, ask which selector the style belongs to and save it.
     Change only the properties that changed.
6. **Make the two sides agree again.** If tokens or styles changed, Figma already shows the new
   values, so nothing to push back. Save a new [snapshot](#the-snapshot).
7. Report what changed in the code, file by file, and what was left for the user to do by hand.

Never edit layout CSS, never add or remove elements, and never touch anything the user didn't
approve in step 4.

## The design system pass

Captures land as raw hex and loose text. Run these with `use_figma` on the file key, scoped to the
captured nodes, in chunks of about 300 layers.

### 1. Refresh the fonts

Captured text can name the right font in the font picker but still draw a fallback. Figma only
re-lays out text when it is edited, so force it: for each text run, load its font, switch it to
another installed style of the **same family**, and straight back.

```js
// Final font, size and colour are unchanged. Only forces a re-layout.
const fonts = await figma.listAvailableFontsAsync();
const otherStyle = f => fonts.find(x => x.fontName.family === f.family && x.fontName.style !== f.style)?.fontName;
for (const t of textNodes) {
  for (const seg of t.getStyledTextSegments(['fontName'])) {
    const tmp = otherStyle(seg.fontName);
    if (!tmp) continue; // family has one style; nothing to flip to
    await figma.loadFontAsync(seg.fontName);
    await figma.loadFontAsync(tmp);
    t.setRangeFontName(seg.start, seg.end, tmp);
    t.setRangeFontName(seg.start, seg.end, seg.fontName);
  }
}
```

Export one text layer as PNG and check the glyphs really are the brand font. Report any layer that
now wraps onto an extra line.

### 2. Bind colours

Bind each SOLID fill, stroke and text-run fill to a variable in the mapped colour collection.

- **Candidates:** for a library, `getVariablesInLibraryCollectionAsync`, then
  `importVariableByKeyAsync`. For local, the collection's variables. Resolve aliases to the mapped
  mode's value. Drop any variable whose value has alpha below 1.
- **Match only when visually identical:** Lab distance 2.3 or less. Anything further stays as hex
  and is listed.
- **Role tie-break:** when several variables share a value, prefer the one whose name fits the
  role. Text prefers names with `foreground`, `text` or `fg`. Strokes prefer `border`, `stroke`,
  `outline`. Fills prefer `background`, `bg`, `surface`, `card`, `primary`, `button`. If it is still
  a tie, list it and ask rather than guess.
- **Skip** any paint with opacity below 1, and list it.
- Bind with `figma.variables.setBoundVariableForPaint`; text runs with `setRangeFills`. Wrap each
  node in try/catch, since some captured layers carry broken gradient stops that make `fills =`
  throw. Skip and list them.
- Report the hex to variable mapping with counts. Screenshot a frame to check nothing went
  see-through or lost contrast.

### 3. Apply text styles

Match by what the style **is**, not what it's called: font family, weight, font size, line height
(and letter spacing when set).

- `textStyleMatch: "exact"` (default): apply only when family, weight and size match exactly and
  line height is within 1px. Everything else is listed.
- `textStyleMatch: "nearest"`: same family and weight, nearest size (ties go to the smaller one).
  Report every size that moved (for example 22 to 20) and how many layers changed height.
- Apply per text run with `setRangeTextStyleIdAsync` after loading the run's font and the style's
  font. For library styles, import with `importStyleByKeyAsync` first. Batches of about 900 layers.

### 4. Apply effect styles, exact matches only

Compare each layer's drop-shadow stack (offset, blur, spread, colour, alpha) with the effect styles.
Captures can come in slightly scaled, so also compare at 1.05x. Apply with `setEffectStyleIdAsync`
on an exact match. List the rest for the user.

### 5. Verify

Screenshot the frames (modal and menu states especially) and compare each against the HTML
before calling it done.

## The check

The last step of every job, and available on its own. **Read-only.** Scope it to the frames the
job touched (or the frame the user names).

It reports four things:

1. **Hardcoded colours:** SOLID fills, strokes or text runs with opacity 1 and no bound variable.
2. **Text with no style.**
3. **Text with a style from somewhere else:** look each style up with `figma.getStyleByIdAsync`
   and compare its key with the mapped design system's styles. A local or third-party style is
   not the design system, even though the layer has one.
4. **Default layer names:** `Frame 427`, `Group 12`, `Rectangle 3` and the like.

```
Pricing / Default
  3 hardcoded colours                #7C3AED (2), #FEF3C7 (1)   no variable within 2.3
  2 text layers with no style        "$29", "/month"
  1 text layer with another style    "Most popular" (local: Badge Bold)
  5 default layer names
```

Then offer, don't do: binding what can be bound, applying styles, renaming layers. Each needs a yes.

### With FigCli (optional, faster)

If the user has [FigCli](https://github.com/silships/figma-cli) installed and connected:

1. **Confirm it's on the right file.** Only the most recently opened FigCli plugin tab is
   connected. Run `node src/index.js eval "figma.fileKey"` from the FigCli folder and compare it
   with `figmaFile.key`. Compare keys, not names: duplicated files share a name. **If they differ,
   stop** and ask the user to close FigCli in other tabs and reopen it in the right file. If
   `figma.fileKey` comes back empty, ask for the URL of the file FigCli is open in.
2. Run the checks as a read-only `eval` script scoped to the frames' node ids. Never use `eval` to
   create or change nodes here.
3. Never run `lint --fix`.

### Without FigCli

Run the same read-only script with `use_figma` on the file key. Same report.
