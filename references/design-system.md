# Design System Reference

The core design tokens and rules that every visual must follow. Read this once per conversation before generating any visual output.

## Philosophy

- **Seamless**: The visual shouldn't feel embedded from somewhere else. It should look native to the host app.
- **Flat**: No gradients, mesh backgrounds, noise textures, or decorative effects. Clean flat surfaces.
- **Compact**: Show the essential inline. Explain the rest in prose outside the visual.
- **Text goes in your response, visuals go in the output** — all explanatory text, descriptions, and summaries belong in your normal response text. The visual output should contain only the visual element itself.

## CSS Variables

The client injects these into the iframe. Always use them — never hardcode colors.

**Backgrounds:**
- `--color-background-primary` — white / main content
- `--color-background-secondary` — surfaces, cards, metric boxes
- `--color-background-tertiary` — page background
- `--color-background-info` / `-danger` / `-success` / `-warning` — semantic

**Text:**
- `--color-text-primary` — near-black, main text
- `--color-text-secondary` — muted, labels
- `--color-text-tertiary` — hints, placeholders
- `--color-text-info` / `-danger` / `-success` / `-warning` — semantic

**Borders:**
- `--color-border-tertiary` — default (0.15 alpha), most borders
- `--color-border-secondary` — hover state (0.3 alpha)
- `--color-border-primary` — emphasis (0.4 alpha)

**Typography:**
- `--font-sans` — default font
- `--font-serif` — editorial/blockquote moments only
- `--font-mono` — code

**Layout:**
- `--border-radius-md` — 8px, most elements
- `--border-radius-lg` — 12px, cards
- `--border-radius-xl` — 16px, large containers

All variables auto-adapt to light/dark mode.

## Color Ramps

9 named ramps, 7 stops each (50=lightest, 900=darkest). For categorical coloring in diagrams and UI.

| Name | 50 | 100 | 200 | 400 | 600 | 800 | 900 |
|------|-----|-----|-----|-----|-----|-----|-----|
| purple | #EEEDFE | #CECBF6 | #AFA9EC | #7F77DD | #534AB7 | #3C3489 | #26215C |
| teal | #E1F5EE | #9FE1CB | #5DCAA5 | #1D9E75 | #0F6E56 | #085041 | #04342C |
| coral | #FAECE7 | #F5C4B3 | #F0997B | #D85A30 | #993C1D | #712B13 | #4A1B0C |
| pink | #FBEAF0 | #F4C0D1 | #ED93B1 | #D4537E | #993556 | #72243E | #4B1528 |
| gray | #F1EFE8 | #D3D1C7 | #B4B2A9 | #888780 | #5F5E5A | #444441 | #2C2C2A |
| blue | #E6F1FB | #B5D4F4 | #85B7EB | #378ADD | #185FA5 | #0C447C | #042C53 |
| green | #EAF3DE | #C0DD97 | #97C459 | #639922 | #3B6D11 | #27500A | #173404 |
| amber | #FAEEDA | #FAC775 | #EF9F27 | #BA7517 | #854F0B | #633806 | #412402 |
| red | #FCEBEB | #F7C1C1 | #F09595 | #E24B4A | #A32D2D | #791F1F | #501313 |

### How to assign colors

Color encodes meaning, not sequence. Don't cycle through colors like a rainbow.

- Group nodes by category — all nodes of the same type share one color
- Use gray for neutral/structural nodes (start, end, generic steps)
- Use 2-3 colors per diagram, not 6+
- Prefer purple, teal, coral, pink for general categories
- Reserve blue, green, amber, red for semantic meaning (info, success, warning, error)
- For illustrative diagrams, map colors to physical properties — warm for heat/energy, cool for cold/calm

### Text on colored backgrounds

Always use the 800 or 900 stop from the same ramp. Never black or generic gray on colored fills. Title darker (800), subtitle lighter (600).

### Light/dark mode color picks

- Light mode: 50 fill + 600 stroke + 800 title / 600 subtitle
- Dark mode: 800 fill + 200 stroke + 100 title / 200 subtitle

In SVG, apply `c-{ramp}` classes which handle this automatically.

## SVG Setup

```svg
<svg width="100%" viewBox="0 0 680 H">
  <defs>
    <marker id="arrow" viewBox="0 0 10 10" refX="8" refY="5"
      markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke"
        stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
  </defs>
</svg>
```

- viewBox width always 680 (matches container 1:1)
- Set H to fit content: last element bottom + 40px
- Safe area: x=40 to x=640
- Background transparent — no wrapper div
- Arrow marker uses `context-stroke` to inherit line color

### Pre-built SVG classes

**Text:** `t` (14px primary), `ts` (12px secondary), `th` (14px bold/500)
**Shapes:** `box` (neutral rect), `node` (clickable with hover), `arr` (arrow line), `leader` (dashed line)
**Colors:** `c-purple`, `c-teal`, `c-coral`, `c-pink`, `c-gray`, `c-blue`, `c-green`, `c-amber`, `c-red`

Apply color classes to `<g>` or shape elements (rect/circle/ellipse), NOT to `<path>`. They set fill+stroke and adjust child text automatically. Dark mode is automatic.

### Font width calibration

- 14px: ~8px per character
- 12px: ~7px per character
- Check: `box_width = max(title_chars × 8, subtitle_chars × 7) + 24`

SVG text never auto-wraps. If a subtitle needs wrapping, it's too long — shorten it.

### Text positioning

Every text inside a box needs `dominant-baseline="central"` with y at the center of its slot.

### viewBox checklist

1. Find lowest element: max(y + height)
2. Set viewBox height = that value + 40
3. Verify rightmost element within x=640
4. Check text-anchor="end" doesn't extend past x=0
5. No negative coordinates

## HTML Rules

- No emoji — use CSS shapes or SVG paths
- No font-size below 11px
- Sentence case always
- No mid-sentence bolding — use `code style` for entity names
- Round every displayed number (Math.round, toFixed, Intl.NumberFormat)
- When placing text on colored backgrounds, use darkest shade from same color family
- No rounded corners on single-sided borders
