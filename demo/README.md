# Demo: make your own before/after

A blank Figma file, a small pricing page, five minutes.

`pricing.html` uses shadcn/ui's default "zinc" colours, so after capture every colour binds to a
matching variable. One badge colour (`#7c3aed`) is off-system on purpose, so the check has
something real to report.

## 1. Set up the Figma file

1. Create a new Figma design file. Call it `html-to-figma-sync demo`.
2. Add the shadcn colour variables. Either:
   - with [FigCli](https://github.com/silships/figma-cli): ask Claude to *"add the shadcn colour
     preset to this file"* (`figma-cli tokens preset shadcn`), or
   - by hand: create a collection called `shadcn/semantic` with a `Light` mode and these
     variables: `background #ffffff`, `foreground #09090b`, `primary #18181b`,
     `primary-foreground #fafafa`, `muted #f4f4f5`, `muted-foreground #71717a`, `border #e4e4e7`.
3. Optional: add a few local text styles for Inter (for example 36/40 Bold, 18/28 Semibold,
   14/20 Regular, 14/20 Medium, 12/16 Semibold) to see text styles apply too.

## 2. Capture the "before"

In Claude Code, inside this folder:

> *"Capture demo/pricing.html into my html-to-figma-sync demo file, but skip the design system
> pass, I want the raw capture."*

Select a few layers and screenshot the right-hand panel: every fill is a raw hex.

## 3. Capture the "after"

> *"Now export demo/pricing.html to the same file properly, using the shadcn/semantic
> collection."*

Claude binds the colours and styles, then reports what it couldn't bind (the purple badge).
Screenshot the same layers: fills now read `primary`, `foreground`, `border`.

## 4. Try a copy update

Change *"Start free trial"* to *"Start your free trial"* in `pricing.html`, then:

> *"I changed the Team button copy, update it in Figma."*

Only that layer changes, binding and style intact.

## 5. Put the image together

Place the two screenshots side by side and save as `assets/before-after.png`. That's the image the
main README shows.
