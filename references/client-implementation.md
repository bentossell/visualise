# Client Implementation Guide

The renderer that makes this skill work. This is the only code you build.

## Architecture

```
Model generates code (guided by SKILL.md + references)
        ↓
Code wrapped in ```visualizer fence
        ↓
Client detects fence, strips it
        ↓
Sandboxed iframe created
        ↓
Theme CSS + SVG classes + code injected
        ↓
ResizeObserver auto-sizes height
        ↓
sendPrompt bridge connects clicks to chat
```

## Renderer component (React/TypeScript)

```tsx
import { useRef, useState, useEffect } from 'react';

export function VisualWidget({ code, title, onSendPrompt }) {
  const iframeRef = useRef(null);
  const [height, setHeight] = useState(200);
  const isDark = matchMedia('(prefers-color-scheme: dark)').matches;

  useEffect(() => {
    const iframe = iframeRef.current;
    if (!iframe) return;
    const doc = iframe.contentDocument;
    if (!doc) return;

    // Inject theme + SVG classes + widget code
    doc.open();
    doc.write(`<style>${getThemeCSS(isDark)}\n${SVG_CLASSES}</style>${code}`);
    doc.close();

    // Bridge: sendPrompt from iframe → chat
    iframe.contentWindow.sendPrompt = (text) => onSendPrompt?.(text);
    iframe.contentWindow.openLink = (url) => window.open(url, '_blank');

    // Auto-size
    let timer;
    const ro = new ResizeObserver(([entry]) => {
      clearTimeout(timer);
      timer = setTimeout(() => setHeight(Math.ceil(entry.contentRect.height) + 16), 50);
    });
    if (doc.body) ro.observe(doc.body);
    return () => { ro.disconnect(); clearTimeout(timer); };
  }, [code, isDark, onSendPrompt]);

  return (
    <iframe ref={iframeRef} title={title} sandbox="allow-scripts"
      style={{ width: '100%', height, border: 'none', display: 'block', overflow: 'hidden' }} />
  );
}
```

## Theme CSS function

Returns CSS variables matching your app's design tokens. Switch on dark mode:

```typescript
function getThemeCSS(isDark: boolean): string {
  return isDark ? `
    :root {
      --color-text-primary: #E5E7EB;
      --color-text-secondary: #9CA3AF;
      --color-text-tertiary: #6B7280;
      --color-text-info: #60A5FA;
      --color-text-success: #34D399;
      --color-text-warning: #FBBF24;
      --color-text-danger: #F87171;
      --color-background-primary: #1A1A1A;
      --color-background-secondary: #262626;
      --color-background-tertiary: #111111;
      --color-border-tertiary: rgba(255,255,255,0.15);
      --color-border-secondary: rgba(255,255,255,0.3);
      --font-sans: system-ui, -apple-system, sans-serif;
      --font-mono: 'SF Mono', Menlo, monospace;
      --border-radius-md: 8px; --border-radius-lg: 12px;
    }` : `
    :root {
      --color-text-primary: #1F2937;
      --color-text-secondary: #6B7280;
      --color-text-tertiary: #9CA3AF;
      --color-text-info: #2563EB;
      --color-text-success: #059669;
      --color-text-warning: #D97706;
      --color-text-danger: #DC2626;
      --color-background-primary: #FFFFFF;
      --color-background-secondary: #F9FAFB;
      --color-background-tertiary: #F3F4F6;
      --color-border-tertiary: rgba(0,0,0,0.15);
      --color-border-secondary: rgba(0,0,0,0.3);
      --font-sans: system-ui, -apple-system, sans-serif;
      --font-mono: 'SF Mono', Menlo, monospace;
      --border-radius-md: 8px; --border-radius-lg: 12px;
    }`;
}
```

## SVG classes string

Inject alongside theme CSS. Handles text classes, shape classes, color ramps (light+dark):

```typescript
const SVG_CLASSES = `
  .t { font: 400 14px var(--font-sans); fill: var(--color-text-primary); }
  .ts { font: 400 12px var(--font-sans); fill: var(--color-text-secondary); }
  .th { font: 500 14px var(--font-sans); fill: var(--color-text-primary); }
  .box { fill: var(--color-background-secondary); stroke: var(--color-border-tertiary); }
  .node { cursor: pointer; } .node:hover { opacity: 0.85; }
  .arr { stroke: var(--color-border-secondary); stroke-width: 1.5; fill: none; }
  .leader { stroke: var(--color-text-tertiary); stroke-width: 0.5; stroke-dasharray: 3 2; fill: none; }

  .c-purple > rect, .c-purple > circle, .c-purple > ellipse { fill: #EEEDFE; stroke: #534AB7; }
  .c-purple > .th { fill: #3C3489; } .c-purple > .ts { fill: #534AB7; }
  .c-teal > rect, .c-teal > circle, .c-teal > ellipse { fill: #E1F5EE; stroke: #0F6E56; }
  .c-teal > .th { fill: #085041; } .c-teal > .ts { fill: #0F6E56; }
  .c-coral > rect, .c-coral > circle, .c-coral > ellipse { fill: #FAECE7; stroke: #993C1D; }
  .c-coral > .th { fill: #712B13; } .c-coral > .ts { fill: #993C1D; }
  .c-gray > rect, .c-gray > circle, .c-gray > ellipse { fill: #F1EFE8; stroke: #5F5E5A; }
  .c-gray > .th { fill: #444441; } .c-gray > .ts { fill: #5F5E5A; }
  .c-blue > rect, .c-blue > circle, .c-blue > ellipse { fill: #E6F1FB; stroke: #185FA5; }
  .c-blue > .th { fill: #0C447C; } .c-blue > .ts { fill: #185FA5; }
  .c-amber > rect, .c-amber > circle, .c-amber > ellipse { fill: #FAEEDA; stroke: #854F0B; }
  .c-amber > .th { fill: #633806; } .c-amber > .ts { fill: #854F0B; }

  @media (prefers-color-scheme: dark) {
    .c-purple > rect, .c-purple > circle, .c-purple > ellipse { fill: #3C3489; stroke: #AFA9EC; }
    .c-purple > .th { fill: #CECBF6; } .c-purple > .ts { fill: #AFA9EC; }
    .c-teal > rect, .c-teal > circle, .c-teal > ellipse { fill: #085041; stroke: #5DCAA5; }
    .c-teal > .th { fill: #9FE1CB; } .c-teal > .ts { fill: #5DCAA5; }
    .c-coral > rect, .c-coral > circle, .c-coral > ellipse { fill: #712B13; stroke: #F0997B; }
    .c-coral > .th { fill: #F5C4B3; } .c-coral > .ts { fill: #F0997B; }
    .c-gray > rect, .c-gray > circle, .c-gray > ellipse { fill: #444441; stroke: #B4B2A9; }
    .c-gray > .th { fill: #D3D1C7; } .c-gray > .ts { fill: #B4B2A9; }
    .c-blue > rect, .c-blue > circle, .c-blue > ellipse { fill: #0C447C; stroke: #85B7EB; }
    .c-blue > .th { fill: #B5D4F4; } .c-blue > .ts { fill: #85B7EB; }
    .c-amber > rect, .c-amber > circle, .c-amber > ellipse { fill: #633806; stroke: #EF9F27; }
    .c-amber > .th { fill: #FAC775; } .c-amber > .ts { fill: #EF9F27; }
  }

  button { background: transparent; border: 0.5px solid var(--color-border-secondary); border-radius: var(--border-radius-md); padding: 6px 14px; font-size: 13px; color: var(--color-text-primary); cursor: pointer; font-family: var(--font-sans); }
  button:hover { background: var(--color-background-secondary); }
  input[type="range"] { -webkit-appearance: none; height: 4px; background: var(--color-border-tertiary); border-radius: 2px; }
  input[type="range"]::-webkit-slider-thumb { -webkit-appearance: none; width: 18px; height: 18px; border-radius: 50%; background: var(--color-background-primary); border: 0.5px solid var(--color-border-secondary); cursor: pointer; }
  * { box-sizing: border-box; margin: 0; font-family: var(--font-sans); }
  body { background: transparent; color: var(--color-text-primary); line-height: 1.5; }
`;
```

## Detecting visualizer output in messages

Split message content on the `visualizer` code fence:

```tsx
function ChatMessage({ content }) {
  const parts = content.split(/```visualizer\n([\s\S]*?)```/g);
  return (
    <div>
      {parts.map((part, i) => {
        if (i % 2 === 1) {
          return <VisualWidget key={i} code={part} title={`visual-${i}`}
            onSendPrompt={(text) => sendMessage(text)} />;
        }
        return <Markdown key={i}>{part}</Markdown>;
      })}
    </div>
  );
}
```

## Streaming support

Write to the iframe incrementally as tokens arrive:

```typescript
function startStreaming(iframe, isDark) {
  const doc = iframe.contentDocument;
  doc.open();
  doc.write(`<style>${getThemeCSS(isDark)}\n${SVG_CLASSES}</style>`);
  // Leave open for appending
}

function appendChunk(iframe, chunk) {
  iframe.contentDocument.write(chunk);
}

function finishStreaming(iframe) {
  iframe.contentDocument.close(); // Triggers script execution
}
```

## CSP for the iframe

```
default-src 'none';
script-src 'unsafe-inline' https://cdnjs.cloudflare.com https://esm.sh https://cdn.jsdelivr.net https://unpkg.com;
style-src 'unsafe-inline';
img-src data: blob:;
font-src https://cdnjs.cloudflare.com https://cdn.jsdelivr.net;
```

## Optional: MCP server wrapper

Only if you need this to work with clients you don't control:

```typescript
import { Server } from '@modelcontextprotocol/sdk/server';
import { z } from 'zod';
import fs from 'fs';

const server = new Server({ name: 'inline-visualizer', version: '1.0.0' });

server.tool('visualizer_read_me', {
  modules: z.array(z.enum(['design-system', 'diagrams', 'components', 'charts'])),
}, async ({ modules }) => {
  const docs = modules.map(m =>
    fs.readFileSync(`./references/${m}.md`, 'utf-8')).join('\n\n---\n\n');
  return { content: [{ type: 'text', text: docs }] };
});

server.tool('render_visual', {
  title: z.string(),
  widget_code: z.string(),
}, async ({ title, widget_code }) => ({
  content: [{ type: 'resource', resource: {
    uri: `visual://${title}`, mimeType: 'text/html', text: widget_code
  }}]
}));

server.listen();
```
