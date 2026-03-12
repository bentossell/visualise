# Diagram Reference

Three diagram families. Pick based on intent, not subject matter.

## Diagram types

**Flowchart** — steps in sequence, decisions branching.
- Trigger: "walk me through the process", "what are the steps"
- Good for: workflows, pipelines, request lifecycles

**Structural** — things inside other things.
- Trigger: "what's the architecture", "where does X live"
- Good for: VPC/subnet/instance, file systems, containment

**Illustrative** — draw the mechanism. Build intuition.
- Physical things get cross-sections (engines, lungs, water heaters)
- Abstract things get spatial metaphors (LLM = stacked layers, hash table = row of buckets)
- Trigger: "how does X actually work", "explain X", "I don't get X"
- Default to illustrative for "how does X work" — don't retreat to a flowchart

## Route on the verb

| User says | Type | What to draw |
|-----------|------|-------------|
| "how do LLMs work" | Illustrative | Token row, stacked layers, attention threads |
| "transformer architecture" | Structural | Labelled boxes: embedding, attention, FFN |
| "how does attention work" | Illustrative | One query token, fan of weighted lines |
| "training steps" | Flowchart | Forward → loss → backward → update |

## Flowchart rules

**Sizing:** At 14px, each character ≈ 8px wide. A 13-char label needs a rect ≥ 140px wide.

**Spacing:** 60px between boxes, 24px padding inside, 10px arrowhead gap. Two-line boxes: 56px height minimum.

**Layout:** Single-direction flows. Max 4-5 nodes per diagram. 6+ components → decompose into overview + sub-flows.

**Arrows:** Must not cross any box. If direct path crosses something, use L-bend:
```svg
<path d="M x1 y1 L x1 ymid L x2 ymid L x2 y2" fill="none" class="arr" marker-end="url(#arrow)"/>
```

**Tier packing:** Compute total width BEFORE placing. 4 boxes at 130px + 3 gaps at 20px = 580px (fits in 640 safe area).

**Cycles:** Don't draw as rings. Use a stepper (HTML mode) or linear layout with return arrow.

**Single-line node (44px):**
```svg
<g class="node c-blue" onclick="sendPrompt('Tell me about this')">
  <rect x="100" y="20" width="180" height="44" rx="8" stroke-width="0.5"/>
  <text class="th" x="190" y="42" text-anchor="middle" dominant-baseline="central">Label</text>
</g>
```

**Two-line node (56px):**
```svg
<g class="node c-blue">
  <rect x="100" y="20" width="200" height="56" rx="8" stroke-width="0.5"/>
  <text class="th" x="200" y="38" text-anchor="middle" dominant-baseline="central">Title</text>
  <text class="ts" x="200" y="56" text-anchor="middle" dominant-baseline="central">Subtitle</text>
</g>
```

## Structural diagram rules

Large rounded rects = containers. Smaller rects inside = regions.

- Outermost: rx=20-24, lightest fill (50 stop), 0.5px stroke
- Inner regions: rx=8-12, next shade (100-200 stop), different ramp if semantically different
- 20px minimum padding inside every container
- Max 2-3 nesting levels
- External inputs/outputs sit outside with arrows pointing in/out

**Color:** Nested regions need distinct ramps. Same class on parent and child flattens the hierarchy.

**ERDs:** Use mermaid.js, not SVG. Import from `esm.sh/mermaid@11`.

## Illustrative diagram rules

The most ambitious type. Draw the mechanism, not a diagram about it.

**What changes from flowcharts:**
- Shapes are freeform — paths, ellipses, circles, polygons, curves
- Layout follows the subject's geometry, not a grid
- Color encodes intensity — warm = active/hot, cool = calm, gray = inert
- Layering and overlap encouraged for shapes (but never let strokes cross text)
- Small shape indicators allowed — triangles for flames, circles for bubbles, wavy lines for steam
- One gradient permitted — only for continuous physical properties. Two stops, same ramp.
- Animation permitted in HTML mode — CSS `@keyframes`, `transform` and `opacity` only

**Fidelity ceiling:** Schematics, not illustrations. A tank is a rounded rect. A flame is three triangles. Recognisable silhouette beats accurate contour.

**Labels:** Place outside the object with leader lines. Pick one side. Reserve 140px margin. Default to right-side labels.

**Composition:** 1. Main silhouette → 2. Internal structure → 3. External connections → 4. State indicators

**Prefer interactive over static.** If the system has a control, give the diagram that control. Water heater → thermostat slider. LLM → click tokens to see attention. Cache → drag hit rate.

## Common failures

1. Arrow through a box — trace coordinates against every box before writing
2. Text overflow — check (text_width + 2×padding) fits the rect
3. viewBox too small — content clipped
4. Floating labels — every text should be in a box, legend, or connected by leader
5. Connector without fill="none" — renders as black shape
6. Missing dominant-baseline="central" — text sits 4px too high
7. Missing arrow marker in defs — always include it
