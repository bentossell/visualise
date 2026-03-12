# visualise

An [Agent Skill](https://agentskills.io) that lets coding agents render rich interactive visuals — SVG diagrams, HTML widgets, charts, and explainers — directly inline in conversations.

Instead of describing a flowchart or pasting ASCII art, the agent generates a real interactive visual. Diagrams you can click, sliders you can drag, charts that update live.

built by [droid](http://factory.ai/)

## Installation

**User-level (recommended)**

```
git clone https://github.com/bentossell/visualise.git ~/.agents/skills/visualise
```

**Project-level**

```
git clone https://github.com/bentossell/visualise.git .agents/skills/visualise
```

## Usage

The skill activates automatically when you ask an agent to:

- Visualize, diagram, or chart something
- Explain how something works (spatial/sequential concepts)
- Compare options side-by-side
- Draw, map out, or illustrate an idea

Or just say: *"show me"*.

## How It Works

The skill uses progressive disclosure to stay lean. At startup, only the name and description are loaded (~100 tokens). When activated, the agent reads the main instructions, then pulls in only the reference docs it needs:

- **design-system.md** — CSS variables, color ramps, typography
- **components.md** — interactive explainers, comparisons, cards, steppers
- **diagrams.md** — flowcharts, structural, illustrative diagrams
- **charts.md** — Chart.js and data viz patterns

Output is raw HTML/SVG fragments rendered in a sandboxed iframe. No build step, no dependencies.

## Client support

This skill teaches agents *what to generate* — it doesn't include a renderer. The output is HTML/SVG wrapped in a `visualizer` code fence.

If your agent client supports rendering that fence in a sandboxed iframe, visuals appear inline. If not, the output is still valid HTML you can open in a browser.

See `references/client-implementation.md` for a full guide on adding renderer support to an agent client.

## License

MIT
