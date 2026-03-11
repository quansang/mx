# 02-preview-pipeline

The preview pipeline transforms raw markdown into rendered HTML through three stages: markdown-it parsing, KaTeX math rendering, and Mermaid diagram rendering. Output displays in the preview pane.

## System Diagram

```mermaid
flowchart LR
    Raw[Raw Markdown] --> MdIt[markdown-it]
    MdIt --> HTML[HTML string]
    HTML --> KaTeX[renderKaTeX]
    KaTeX --> Mermaid[processMermaidBlocks]
    Mermaid --> DOM[innerHTML]
    DOM --> MermaidRun[mermaid.run]
```

## 1. markdown-it

Configured with `html: true`, `linkify: true`, `typographer: true`. Handles standard markdown, tables, code blocks, blockquotes, lists.

## 2. KaTeX Math

`renderKaTeX()` processes the HTML string with regex replacement:

| Pattern | Mode | Example |
|---------|------|---------|
| `$$...$$` | Display (block) | `$$\int_0^1 x dx$$` |
| `$...$` | Inline | `$E=mc^2$` |

Both use `throwOnError: false` for graceful degradation. Errors render as `<pre>` or `<code>` with class `katex-error`.

## 3. Mermaid Diagrams

Two-phase rendering:

1. `processMermaidBlocks()` — regex replaces `<pre><code class="language-mermaid">` blocks with `<div class="mermaid">` elements, decoding HTML entities
2. `renderMermaidDivs()` — async call to `mermaid.run()` on all `.mermaid` divs in the preview pane

Mermaid initialized with `startOnLoad: false`, `theme: "dark"`. Render errors are silently caught.

## 4. Render Trigger

`updatePreview()` skips rendering if preview pane is hidden (`display === "none"`). Called on:
- Content change (debounced 300ms)
- View mode switch to split or preview
- Initial load with sample content

## File Reference

| File | Purpose |
|------|---------|
| `src/main.ts:30` | markdown-it instance |
| `src/main.ts:32-46` | `renderKaTeX()` |
| `src/main.ts:54-63` | `processMermaidBlocks()` |
| `src/main.ts:65-71` | `renderMermaidDivs()` |
| `src/main.ts:75-84` | `updatePreview()` orchestrator |

## Cross-References

| Doc | Relation |
|-----|----------|
| [01-editor-engine](01-editor-engine.md) | Triggers preview on content change |
| [05-ui-layout](05-ui-layout.md) | Preview pane display |
| [04-pdf-export](04-pdf-export.md) | Separate Mermaid rendering for PDF |
