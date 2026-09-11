# GON Design System

A minimal, token-based design system built with plain HTML and CSS — no frameworks, no build step. It doubles as a hands-on tutorial for anyone who wants to understand how design tokens, style guides, and consuming projects fit together.

## What's inside

- **Design tokens** — colors, spacing, typography, radii, and shadows as CSS custom properties.
- **Living style guide** — a page that renders every token visually so the system is self-documenting.
- **Demo project** — a small profile card that uses only the tokens, proving the system works.
- **Full tutorial** — a step-by-step walkthrough of how everything was built.

## Preview

Open the two HTML files directly in your browser:

- `design-system.html` — the living style guide (colors, type, spacing, radii, shadows).
- `index.html` — the demo profile card built on top of the tokens.

No server, no dependencies — just double-click the files or drag them into a browser tab.

<img width="373" height="536" alt="image" src="https://github.com/user-attachments/assets/3967a910-bafe-43aa-b524-c0d2dff63505" />


## Project structure

```
gon-design-system/
├── index.html                  ← demo project (profile card)
├── design-system.html          ← living style guide
├── design-system-tutorial.md   ← step-by-step tutorial
└── css/
    ├── reset.css               ← modern CSS reset
    ├── tokens.css              ← source of truth (variables + typography presets)
    ├── design-system.css       ← styles for the style guide page
    └── styles.css              ← styles for the demo project
```

Both HTML pages load `reset.css` and `tokens.css`. If a token changes, every page updates.

## The tokens

All defined in `css/tokens.css` as CSS custom properties:

- **Colors** — a primary blue scale plus a neutral grayscale (`--color-primary-*`, `--color-neutral-*`).
- **Spacing** — an 8-point-inspired scale from `--space-100` (4px) to `--space-800` (64px).
- **Typography** — Inter font family, a size scale from `--font-size-100` to `--font-size-700`, three weights, two line heights, and four semantic presets (`.text-heading-lg`, `.text-heading-md`, `.text-body`, `.text-caption`).
- **Radii** — `--radius-sm`, `--radius-md`, `--radius-lg`, `--radius-full`.
- **Shadows** — `--shadow-sm`, `--shadow-md`, `--shadow-lg`.

## Using the system in your own project

1. Copy the `css/` folder into your project.
2. Link `reset.css` and `tokens.css` in your HTML `<head>`:

   ```html
   <link rel="stylesheet" href="css/reset.css">
   <link rel="stylesheet" href="css/tokens.css">
   ```

3. Reference tokens in your own stylesheet:

   ```css
   .card {
     padding: var(--space-600);
     background: var(--color-neutral-100);
     border-radius: var(--radius-lg);
     box-shadow: var(--shadow-md);
   }
   ```

4. Use the typography presets directly on elements:

   ```html
   <h1 class="text-heading-lg">Hello</h1>
   <p class="text-body">Some copy.</p>
   ```

## Tutorial

`design-system-tutorial.md` walks through the whole build from an empty folder to a working system, including:

- What a design system actually is and why tokens matter
- Setting up a modern CSS reset
- Choosing and organizing tokens
- Building the living style guide page
- Wiring up a demo project that consumes only tokens
- Appendices on responsive grid tricks and dark-mode-ready semantic tokens

## License

MIT
