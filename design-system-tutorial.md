# Build a Design System From Scratch

A step-by-step tutorial for building a minimal design system in HTML + CSS, plus a small demo project that uses it. Written for a beginner/intermediate developer who has done a few CSS projects but never intentionally built tokens or a design system.

By the end you'll have:

- A **tokens file** (colors, spacing, typography, radii, shadows) — the source of truth.
- A **design-system.html** page that renders every token visually — your living style guide.
- A **minimal demo project** (a profile card) that uses only the tokens — proving the system works.

---

## Table of contents

1. [What is a design system? (5-minute concept)](#1-what-is-a-design-system)
2. [The project structure](#2-the-project-structure)
3. [The CSS reset](#3-the-css-reset)
4. [Defining your tokens](#4-defining-your-tokens)
5. [Building the design-system.html page](#5-building-the-design-systemhtml-page)
6. [The minimal demo project](#6-the-minimal-demo-project-a-profile-card)
7. [Proving the system works](#7-proving-the-system-works)
8. [Where to go next](#8-where-to-go-next)
9. [Appendix A — Responsive grid columns with `repeat(auto-fill, minmax(...))`](#appendix-a--responsive-grid-columns-with-repeatauto-fill-minmax)
10. [Appendix B — Handling dark mode with semantic tokens](#appendix-b--handling-dark-mode-with-semantic-tokens)

---

## 1. What is a design system?

A design system is **a shared vocabulary for design decisions**. Instead of writing `color: #FCAE4A` in seventeen places, you define `--color-primary-orange: #FCAE4A` once, and everywhere else you refer to that name.

The pieces:

- **Tokens** — the atomic values (a color, a size, a spacing unit).
- **Presets / patterns** — reusable combinations (a "heading" isn't just a size; it's a size + weight + line-height).
- **Style guide** — a visible page that shows all of them.

**Why bother?**

1. **Consistency.** Every button uses the same corner radius. Every heading uses the same font size. You don't drift.
2. **Change once, update everywhere.** Rebrand from orange to green? Change one line.
3. **Communication.** A designer says "use spacing 400" and there's no ambiguity.
4. **Onboarding.** A new dev opens the style guide and sees the whole visual language in 60 seconds.

The core insight: **the design system is separate from any single page**. It's a library your pages consume.

---

## 2. The project structure

Start with this folder layout:

```
my-project/
├── index.html                ← the demo page (profile card)
├── design-system.html        ← the living style guide
└── css/
    ├── reset.css             ← modern CSS reset
    ├── tokens.css            ← the source of truth (variables + presets)
    ├── design-system.css     ← styles ONLY for the DS page
    └── styles.css            ← styles for the demo project
```

Why the split:

- **`tokens.css`** is loaded by both HTML pages. If it changes, both update.
- **`design-system.css`** contains only rules that lay out the DS page (swatch grids, sample blocks). These shouldn't pollute your product CSS.
- **`styles.css`** is your actual product/project styles.
- **`reset.css`** is shared boilerplate — safe defaults for everything.

Create the folder and empty files now. We'll fill them in order.

---

## 3. The CSS reset

Every browser has its own defaults (margins on `<h1>`, list bullets on `<ul>`, etc.). A **CSS reset** neutralizes those so *you* decide what things look like.

Copy this into `css/reset.css`. It's Andy Bell's modern reset, lightly annotated.

```css
/* Box sizing — makes width/height include padding and border. Always want this. */
*, *::before, *::after {
    box-sizing: border-box;
}

/* Wipe default margins/paddings. */
* {
    margin: 0;
    padding: 0;
}

/* Sensible defaults on the body. */
body {
    min-height: 100vh;
    line-height: 1.5;
    text-rendering: optimizeSpeed;
    -webkit-font-smoothing: antialiased;
}

/* Images shouldn't overflow their containers. */
img, picture, svg {
    max-width: 100%;
    display: block;
}

/* Form elements should inherit the page's font. */
input, button, textarea, select {
    font: inherit;
}

/* Respect users who prefer less motion. */
@media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
        animation-duration: 0.01ms !important;
        transition-duration: 0.01ms !important;
        scroll-behavior: auto !important;
    }
}
```

That's it. Save and move on.

---

## 4. Defining your tokens

This is the heart of the system. Everything else references what we write here.

Open `css/tokens.css`. We'll fill it in sections: colors, spacing, typography, radii, shadows.

### 4.1 Colors

Two families: **primary** (brand accents) and **neutral** (grays / white). Numeric scales are more flexible than names like "light" and "dark" — you can always add `--color-neutral-450` later without renaming everything.

```css
:root {
    /* Primary — brand accents */
    --color-primary-500: hsl(212, 86%, 55%);   /* main brand blue */
    --color-primary-400: hsl(212, 86%, 65%);   /* lighter tint */

    /* Neutral — text, backgrounds, borders */
    --color-neutral-100: hsl(0, 0%, 100%);     /* white */
    --color-neutral-200: hsl(210, 20%, 96%);   /* page background */
    --color-neutral-300: hsl(210, 15%, 88%);   /* borders */
    --color-neutral-500: hsl(210, 10%, 50%);   /* muted text */
    --color-neutral-700: hsl(210, 15%, 25%);   /* body text */
    --color-neutral-900: hsl(210, 20%, 10%);   /* headings */
}
```

**Why HSL?** It's the most human-readable color format: *hue, saturation, lightness*. Once you learn it, you can tweak a color by nudging one number.

### 4.2 Spacing

A T-shirt-style numeric scale. Every gap, padding, and margin in your project should pick one of these values.

```css
:root {
    /* ...colors above... */

    /* Spacing scale (matches an 8px base) */
    --space-100: 0.25rem;    /* 4px  */
    --space-200: 0.5rem;     /* 8px  */
    --space-300: 0.75rem;    /* 12px */
    --space-400: 1rem;       /* 16px */
    --space-500: 1.5rem;     /* 24px */
    --space-600: 2rem;       /* 32px */
    --space-700: 3rem;       /* 48px */
    --space-800: 4rem;       /* 64px */
}
```

**The discipline:** never write `padding: 15px`. If your design says 15px, pick the nearest scale value (16px, `--space-400`). Constraints are what makes it a *system*.

### 4.3 Typography

Font-family, font-size scale, weight scale, and then **preset classes** that combine them into ready-to-use styles.

```css
:root {
    /* ...spacing above... */

    /* Font family */
    --font-family-base: 'Inter', system-ui, sans-serif;

    /* Font sizes */
    --font-size-100: 0.75rem;      /* 12px — captions */
    --font-size-200: 0.875rem;     /* 14px — small text */
    --font-size-300: 1rem;         /* 16px — body */
    --font-size-400: 1.25rem;      /* 20px — subheadings */
    --font-size-500: 1.5rem;       /* 24px — h3 */
    --font-size-600: 2rem;         /* 32px — h2 */
    --font-size-700: 2.5rem;       /* 40px — h1 */

    /* Font weights */
    --font-weight-regular: 400;
    --font-weight-medium: 500;
    --font-weight-bold: 700;

    /* Line heights */
    --line-height-tight: 1.2;
    --line-height-base: 1.5;
}

body {
    font-family: var(--font-family-base);
    font-size: var(--font-size-300);
    color: var(--color-neutral-700);
    background: var(--color-neutral-200);
}

/* Typography presets — combinations you use as classes */
.text-heading-lg {
    font-size: var(--font-size-700);
    font-weight: var(--font-weight-bold);
    line-height: var(--line-height-tight);
    color: var(--color-neutral-900);
}

.text-heading-md {
    font-size: var(--font-size-500);
    font-weight: var(--font-weight-bold);
    line-height: var(--line-height-tight);
    color: var(--color-neutral-900);
}

.text-body {
    font-size: var(--font-size-300);
    font-weight: var(--font-weight-regular);
    line-height: var(--line-height-base);
}

.text-caption {
    font-size: var(--font-size-200);
    font-weight: var(--font-weight-medium);
    color: var(--color-neutral-500);
    text-transform: uppercase;
    letter-spacing: 0.05em;
}
```

**The pattern:** raw tokens (`--font-size-500`) are the atoms; preset classes (`.text-heading-md`) are the molecules. You mostly consume the presets, and reach for atoms only when composing new ones.

### 4.4 Radii and shadows

Small but visually important.

```css
:root {
    /* ...typography above... */

    /* Border radius */
    --radius-sm: 0.25rem;    /* 4px  */
    --radius-md: 0.5rem;     /* 8px  */
    --radius-lg: 1rem;       /* 16px */
    --radius-full: 9999px;   /* pill / circle */

    /* Shadows */
    --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
    --shadow-md: 0 4px 12px rgba(0, 0, 0, 0.08);
    --shadow-lg: 0 12px 32px rgba(0, 0, 0, 0.12);
}
```

That's the full token set. Nothing fancy — but it's now a **language** the rest of the project speaks.

---

## 5. Building the design-system.html page

Now the fun part: render everything visually.

### 5.1 The HTML skeleton

Open `design-system.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Design System</title>
    <link rel="stylesheet" href="css/reset.css">
    <link rel="stylesheet" href="css/tokens.css">
    <link rel="stylesheet" href="css/design-system.css">
</head>
<body>
    <main class="ds">
        <header class="ds__header">
            <h1 class="text-heading-lg">Design System</h1>
            <p class="text-body">The visual vocabulary for this project.</p>
        </header>

        <!-- Sections go here -->
    </main>
</body>
</html>
```

### 5.2 Base styles for the DS page

Open `css/design-system.css`:

```css
.ds {
    max-width: 60rem;
    margin: 0 auto;
    padding: var(--space-800) var(--space-500);
}

.ds__header {
    margin-bottom: var(--space-800);
}

.ds__section {
    margin-bottom: var(--space-800);
}

.ds__section-title {
    font-size: var(--font-size-500);
    font-weight: var(--font-weight-bold);
    color: var(--color-neutral-900);
    margin-bottom: var(--space-500);
    padding-bottom: var(--space-200);
    border-bottom: 1px solid var(--color-neutral-300);
}
```

### 5.3 Colors section

Add inside `<main class="ds">` in the HTML:

```html
<section class="ds__section">
    <h2 class="ds__section-title">Colors</h2>
    <div class="swatch-grid">
        <div class="swatch">
            <div class="swatch__color" style="background: var(--color-primary-500);"></div>
            <p class="swatch__name">primary-500</p>
        </div>
        <div class="swatch">
            <div class="swatch__color" style="background: var(--color-primary-400);"></div>
            <p class="swatch__name">primary-400</p>
        </div>
        <div class="swatch">
            <div class="swatch__color" style="background: var(--color-neutral-100); border: 1px solid var(--color-neutral-300);"></div>
            <p class="swatch__name">neutral-100</p>
        </div>
        <div class="swatch">
            <div class="swatch__color" style="background: var(--color-neutral-200);"></div>
            <p class="swatch__name">neutral-200</p>
        </div>
        <div class="swatch">
            <div class="swatch__color" style="background: var(--color-neutral-300);"></div>
            <p class="swatch__name">neutral-300</p>
        </div>
        <div class="swatch">
            <div class="swatch__color" style="background: var(--color-neutral-500);"></div>
            <p class="swatch__name">neutral-500</p>
        </div>
        <div class="swatch">
            <div class="swatch__color" style="background: var(--color-neutral-700);"></div>
            <p class="swatch__name">neutral-700</p>
        </div>
        <div class="swatch">
            <div class="swatch__color" style="background: var(--color-neutral-900);"></div>
            <p class="swatch__name">neutral-900</p>
        </div>
    </div>
</section>
```

And the CSS in `design-system.css`:

```css
.swatch-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(8rem, 1fr));
    gap: var(--space-400);
}

.swatch__color {
    width: 100%;
    height: 5rem;
    border-radius: var(--radius-md);
    margin-bottom: var(--space-200);
}

.swatch__name {
    font-size: var(--font-size-200);
    color: var(--color-neutral-500);
    font-family: monospace;
}
```

> 📎 **New pattern:** `grid-template-columns: repeat(auto-fill, minmax(8rem, 1fr))` is a self-adjusting responsive grid that reflows the number of columns based on available width — no media queries needed. See [Appendix A](#appendix-a--responsive-grid-columns-with-repeatauto-fill-minmax) for a full breakdown of how each piece works.

**Note the inline `style="background: var(--color-...)"`.** In production CSS this would be a smell, but here it's intentional: the whole point of the design system page is to **name the token next to the value**. Inline style is the simplest way to bind them.

### 5.4 Typography section

```html
<section class="ds__section">
    <h2 class="ds__section-title">Typography</h2>
    <div class="type-list">
        <div class="type-sample">
            <p class="type-sample__meta">text-heading-lg — 2.5rem / bold</p>
            <p class="text-heading-lg">The quick brown fox</p>
        </div>
        <div class="type-sample">
            <p class="type-sample__meta">text-heading-md — 1.5rem / bold</p>
            <p class="text-heading-md">The quick brown fox</p>
        </div>
        <div class="type-sample">
            <p class="type-sample__meta">text-body — 1rem / regular</p>
            <p class="text-body">The quick brown fox jumps over the lazy dog.</p>
        </div>
        <div class="type-sample">
            <p class="type-sample__meta">text-caption — 0.875rem / medium / uppercase</p>
            <p class="text-caption">The quick brown fox</p>
        </div>
    </div>
</section>
```

CSS:

```css
.type-list {
    display: flex;
    flex-direction: column;
    gap: var(--space-500);
}

.type-sample__meta {
    font-family: monospace;
    font-size: var(--font-size-200);
    color: var(--color-neutral-500);
    margin-bottom: var(--space-100);
}
```

Reload — you'll see each preset with its name and a live sample. Change a size token, refresh, watch it update everywhere it's used.

### 5.5 Spacing section

Trick: **use a colored bar whose width equals the spacing value**. That way "big" values are visually big.

```html
<section class="ds__section">
    <h2 class="ds__section-title">Spacing</h2>
    <div class="space-list">
        <div class="space-sample">
            <p class="space-sample__name">space-100</p>
            <div class="space-sample__bar" style="width: var(--space-100);"></div>
            <p class="space-sample__value">4px</p>
        </div>
        <div class="space-sample">
            <p class="space-sample__name">space-200</p>
            <div class="space-sample__bar" style="width: var(--space-200);"></div>
            <p class="space-sample__value">8px</p>
        </div>
        <div class="space-sample">
            <p class="space-sample__name">space-300</p>
            <div class="space-sample__bar" style="width: var(--space-300);"></div>
            <p class="space-sample__value">12px</p>
        </div>
        <div class="space-sample">
            <p class="space-sample__name">space-400</p>
            <div class="space-sample__bar" style="width: var(--space-400);"></div>
            <p class="space-sample__value">16px</p>
        </div>
        <div class="space-sample">
            <p class="space-sample__name">space-500</p>
            <div class="space-sample__bar" style="width: var(--space-500);"></div>
            <p class="space-sample__value">24px</p>
        </div>
        <div class="space-sample">
            <p class="space-sample__name">space-600</p>
            <div class="space-sample__bar" style="width: var(--space-600);"></div>
            <p class="space-sample__value">32px</p>
        </div>
        <div class="space-sample">
            <p class="space-sample__name">space-700</p>
            <div class="space-sample__bar" style="width: var(--space-700);"></div>
            <p class="space-sample__value">48px</p>
        </div>
        <div class="space-sample">
            <p class="space-sample__name">space-800</p>
            <div class="space-sample__bar" style="width: var(--space-800);"></div>
            <p class="space-sample__value">64px</p>
        </div>
    </div>
</section>
```

CSS:

```css
.space-list {
    display: flex;
    flex-direction: column;
    gap: var(--space-300);
}

.space-sample {
    display: grid;
    grid-template-columns: 8rem 1fr auto;
    gap: var(--space-400);
    align-items: center;
}

.space-sample__name {
    font-family: monospace;
    font-size: var(--font-size-200);
    color: var(--color-neutral-700);
}

.space-sample__bar {
    height: 1.5rem;
    background: var(--color-primary-500);
    border-radius: var(--radius-sm);
}

.space-sample__value {
    font-family: monospace;
    font-size: var(--font-size-200);
    color: var(--color-neutral-500);
}
```

### 5.6 Radii and shadows section

```html
<section class="ds__section">
    <h2 class="ds__section-title">Radii & Shadows</h2>
    <div class="chip-grid">
        <div class="chip" style="border-radius: var(--radius-sm);">radius-sm</div>
        <div class="chip" style="border-radius: var(--radius-md);">radius-md</div>
        <div class="chip" style="border-radius: var(--radius-lg);">radius-lg</div>
        <div class="chip" style="border-radius: var(--radius-full);">radius-full</div>
    </div>
    <div class="chip-grid" style="margin-top: var(--space-500);">
        <div class="chip" style="box-shadow: var(--shadow-sm);">shadow-sm</div>
        <div class="chip" style="box-shadow: var(--shadow-md);">shadow-md</div>
        <div class="chip" style="box-shadow: var(--shadow-lg);">shadow-lg</div>
    </div>
</section>
```

CSS:

```css
.chip-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(10rem, 1fr));
    gap: var(--space-400);
}

.chip {
    padding: var(--space-500);
    background: var(--color-neutral-100);
    border: 1px solid var(--color-neutral-300);
    text-align: center;
    font-family: monospace;
    font-size: var(--font-size-200);
    color: var(--color-neutral-700);
}
```

**That's the whole design-system.html.** Reload the page — you should see a clean, scrollable reference of every token you'll use.

---

## 6. The minimal demo project: a profile card

Time to prove the system pays off. We'll build a single component — a profile card — using **only tokens and preset classes**. No hardcoded colors, no arbitrary spacing.

### 6.1 The HTML

Open `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Profile Card</title>
    <link rel="stylesheet" href="css/reset.css">
    <link rel="stylesheet" href="css/tokens.css">
    <link rel="stylesheet" href="css/styles.css">
</head>
<body>
    <main class="page">
        <article class="profile-card">
            <img class="profile-card__avatar"
                 src="https://i.pravatar.cc/160?img=12"
                 alt="Avatar of Alex Rivera">
            <p class="text-caption">Product Designer</p>
            <h1 class="text-heading-md">Alex Rivera</h1>
            <p class="text-body profile-card__bio">
                Designing intuitive tools for creative teams.
                Formerly at Studio Nord, now freelancing.
            </p>
            <button class="button">Get in touch</button>
        </article>
    </main>
</body>
</html>
```

### 6.2 The CSS

Open `css/styles.css`:

```css
.page {
    min-height: 100vh;
    display: grid;
    place-items: center;
    padding: var(--space-500);
}

.profile-card {
    background: var(--color-neutral-100);
    border-radius: var(--radius-lg);
    box-shadow: var(--shadow-md);
    padding: var(--space-600);
    max-width: 22rem;
    width: 100%;
    text-align: center;
}

.profile-card__avatar {
    width: 6rem;
    height: 6rem;
    border-radius: var(--radius-full);
    margin: 0 auto var(--space-400);
    object-fit: cover;
}

.profile-card__bio {
    margin-top: var(--space-300);
    margin-bottom: var(--space-500);
    color: var(--color-neutral-500);
}

.button {
    background: var(--color-primary-500);
    color: var(--color-neutral-100);
    border: none;
    padding: var(--space-300) var(--space-500);
    border-radius: var(--radius-md);
    font-weight: var(--font-weight-medium);
    cursor: pointer;
    transition: background 0.2s;
}

.button:hover {
    background: var(--color-primary-400);
}
```

**Read that CSS carefully.** Notice something? **There are zero hardcoded colors, zero raw pixel values, zero arbitrary radii.** Every single visual decision references a token. That's the goal.

The three preset classes (`text-caption`, `text-heading-md`, `text-body`) come from `tokens.css` for free — no need to define them again.

---

## 7. Proving the system works

Here's the payoff moment. Open `tokens.css` and change one line:

```css
--color-primary-500: hsl(280, 65%, 55%);   /* was blue, now purple */
```

Reload **both** pages:

- `design-system.html` — the primary swatch is now purple, the spacing bars are purple, everything else stays consistent.
- `index.html` — the button on the profile card is now purple. On hover, the lighter shade (which we didn't change) is now visually wrong — a great forcing function to also update `--color-primary-400`.

**That's the value.** One line changes the entire visual identity across every page that consumes the system. This is impossible if you write `background: #4A90E2` in seventeen places.

Try more experiments:

- Change `--radius-md` to `0` — every card, chip, and button in both pages becomes sharp-cornered.
- Change `--space-500` from `1.5rem` to `3rem` — every gap using that token grows.
- Add a new color like `--color-accent-500` and render it as a swatch on the DS page before using it anywhere real.

---

## 8. Where to go next

You now have a working design system. Real teams grow theirs like this:

### Adding component patterns

Add reusable class-based components in `tokens.css` (or a new `components.css`):

- `.button`, `.button--secondary`, `.button--ghost`
- `.card` (base card styles)
- `.input`, `.input--error`

For each one, render it in `design-system.html` under a **Components** section so anyone can see the available options.

### Documenting usage rules

Add prose notes to `design-system.html` — e.g., *"Use `text-heading-lg` for page titles only, never in cards"*. Rules turn a token library into a proper system.

### Handling dark mode

Because every element references *tokens* instead of raw colors, adding dark mode becomes a **token-layer concern, not a component-layer concern** — you rewire the variables and the whole UI repaints. The mechanics look like this at a high level:

```css
:root {
    /* light theme defaults */
}

@media (prefers-color-scheme: dark) {
    :root { /* dark overrides — follows the OS */ }
}

[data-theme="dark"] {
    /* dark overrides — user toggle wins */
}
```

There's a subtle but important twist: naming your tokens by *value* (like `--color-neutral-100`) makes dark mode confusing to read later. Naming them by *role* (like `--color-surface`) makes it clear. This is called the **semantic token** pattern.

> 📎 **See [Appendix B](#appendix-b--handling-dark-mode-with-semantic-tokens)** for the full walkthrough: how the cascade actually flips, the palette-vs-semantic token split, a working `tokens.css` with both OS-preference and user-toggle triggers, and the ~10-line JS to power a theme toggle button.

### When to graduate

For teams and larger projects, this HTML approach hits a ceiling. Consider:

- **[Storybook](https://storybook.js.org/)** — automates the "render every component" pattern for React/Vue/etc.
- **[Figma tokens](https://tokens.studio/)** — sync design tokens between Figma and code.
- **[Style Dictionary](https://amzn.github.io/style-dictionary/)** — export tokens as CSS variables, Sass variables, iOS Swift, etc.

But **don't reach for these too early**. A hand-rolled HTML design system is enough for solo projects and small teams. It teaches you the concepts before the tooling abstracts them away.

---

## Recap — the mental model

- Tokens are **atoms** (a color, a size).
- Presets are **molecules** (a heading style combining several atoms).
- The design system page is the **shelf** where you see all of them.
- The demo project is the **customer** — it consumes the shelf, never invents its own.
- One change to a token cascades everywhere. That's the entire point.

Now try it on your next Frontend Mentor challenge: define the tokens first from the style guide, build the design system page for them, *then* build the actual layout. The layout will practically write itself. 🎨

---

## Appendix A — Responsive grid columns with `repeat(auto-fill, minmax(...))`

In section 5.3 we used this one-liner:

```css
grid-template-columns: repeat(auto-fill, minmax(8rem, 1fr));
```

It's doing a *lot* of work, and it's one of the most powerful CSS Grid patterns you can learn. Translated to English:

> *"Fill the row with as many columns as will fit, where each column is at least 8rem wide but can grow to share the leftover space equally."*

Let's peel it back layer by layer, from the inside out.

### Layer 1 — `1fr`

`fr` = "fractional unit." It means *"one share of the available leftover space."*

If a container has 900px available and three columns are `1fr 1fr 1fr`, each gets 300px. If you write `2fr 1fr 1fr`, the first gets half (450px) and the other two split the rest.

Think of it like **splitting a pizza after everyone else has taken their fixed slices** — `fr` is what's left divided among the flexible eaters.

### Layer 2 — `minmax(8rem, 1fr)`

`minmax(min, max)` sets a **size range** for a column.

- **min: `8rem`** — the column can never be *narrower* than 8rem (128px).
- **max: `1fr`** — the column can be as *wide* as its fair share of leftover space.

So each column says: *"I refuse to go below 128px, but if there's extra room, I'll happily grow to share it."*

**Why not just `8rem`?** Because then columns would be exactly 8rem, leaving weird empty space on the right of the row. Using `1fr` as the max lets them stretch to fill the row edge-to-edge.

**Why not just `1fr`?** Because then very narrow viewports would squish columns to unreadable widths.

`minmax` is the sweet spot: **stretchy, but with a floor**.

### Layer 3 — `repeat(auto-fill, ...)`

`repeat()` is shorthand. Instead of writing:

```css
grid-template-columns: minmax(8rem, 1fr) minmax(8rem, 1fr) minmax(8rem, 1fr) ... ;
```

...you write `repeat(N, minmax(...))` and it repeats the pattern N times.

**But how many times?** That's where `auto-fill` comes in.

`auto-fill` tells the browser: *"You figure it out. Fit as many 8rem-minimum columns as you can in the current row width."*

- Container is 400px wide? Room for ~3 columns of 8rem+ → grid becomes 3 columns.
- Container is 800px wide? Room for ~6 columns → grid becomes 6 columns.
- Container is 200px wide? Room for only 1 → single column stack.

**No media queries needed.** The grid rewires itself as the viewport changes.

### Putting it all together — visual

Imagine a container 900px wide with `minmax(8rem, 1fr)` (min ~128px):

```
Container:  900px
Min column: 128px
Fit count:  900 / 128 = 7 columns (rounded down to 7 that all fit)
```

The browser creates 7 columns. Each starts at 128px minimum, then grows via `1fr` to share the leftover: `(900 − 7×128) / 7` ≈ 4px extra per column → each ~132px.

Resize the browser wider to 1200px:

```
Container: 1200px
Fit count: 1200 / 128 = 9 columns
```

Now 9 columns, each ~133px. **The grid adapted itself.**

Shrink to 300px:

```
Container: 300px
Fit count: 300 / 128 = 2 columns
```

Two columns, each 150px. Still readable, no overlap, no scroll bar.

### Why this is amazing

Compare it to the "old way":

```css
/* Old way — brittle */
.swatch-grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
}
@media (max-width: 768px) { grid-template-columns: repeat(3, 1fr); }
@media (max-width: 500px) { grid-template-columns: repeat(2, 1fr); }
@media (max-width: 320px) { grid-template-columns: 1fr; }
```

That's four separate declarations, arbitrary breakpoint numbers, and it still doesn't adapt if you add more items or change the item's min-content.

The one-liner replaces all of that. **The layout responds to the *content's needs*, not to arbitrary viewport widths.**

### One important cousin — `auto-fit`

There's a nearly identical alternative:

```css
grid-template-columns: repeat(auto-fit, minmax(8rem, 1fr));
```

`auto-fit` vs `auto-fill`:

- **`auto-fill`** creates empty invisible columns to fill the row, even if you don't have enough items.
- **`auto-fit`** collapses empty columns, letting your existing items **stretch to fill the whole row**.

Example — 3 items in a 1000px container:

- **`auto-fill`**: 3 items in their fair-share slots, then empty phantom tracks to the right → items stay narrow.
- **`auto-fit`**: 3 items each stretch to ~333px, filling the whole row → items look "spread out."

**Rule of thumb:** use `auto-fill` when you want items to stay their natural size (like our swatches — you don't want three swatches ballooning to 300px each). Use `auto-fit` when you want items to always fill the row (like a hero grid or feature cards).

### Try it yourself

Open the `design-system.html` page and experiment:

1. Change `8rem` to `4rem` — swatches get smaller, more per row.
2. Change `8rem` to `20rem` — swatches get bigger, fewer per row.
3. Change `auto-fill` to `auto-fit` — resize the browser and watch the difference.

Playing with the numbers is the fastest way to internalize what each part does.

### The one-line takeaway

```css
grid-template-columns: repeat(auto-fill, minmax(<min>, 1fr));
```

**Memorize this pattern.** You'll use it in nearly every project — card grids, image galleries, product listings, dashboards. It's the closest thing CSS has to a "just make it responsive" button. 🚀

---

## Appendix B — Handling dark mode with semantic tokens

Section 8 mentioned dark mode almost in passing. Because dark mode is one of the biggest payoffs of building a design system, it deserves a proper walkthrough. This appendix covers:

- The core idea (why dark mode is a *token* concern, not a component concern).
- The two triggers: OS preference vs. explicit user toggle.
- The **critical insight**: literal tokens vs. semantic tokens.
- A full working `tokens.css` with both triggers combined.
- The ~10-line JavaScript toggle.

### The core idea

Dark mode works by **overriding the token values** based on context. Everything downstream (buttons, cards, text) keeps referring to the same variable names — the variables just point to different values depending on the mode.

Think of it like a **light switch for your CSS variables**. The wiring (`background: var(--color-surface)`) stays the same; the current running through the wire changes.

### The mechanic — CSS variable inheritance

CSS variables cascade down the DOM tree. If you declare:

```css
:root {
    --my-color: white;
}
```

Then every element inside `<html>` can read `--my-color` and get `white`. But if some element **redefines** the variable on itself (or an ancestor), its subtree gets the new value:

```css
:root {
    --my-color: white;
}

.dark {
    --my-color: black;
}
```

Anything inside `<div class="dark">…</div>` now sees `--my-color: black`. Everything outside still sees white. **That's the entire mechanism** — no JavaScript needed, just cascade.

### Two ways to trigger dark mode

There are **two separate systems** — you can use either, or combine both.

#### Method 1 — Follow the user's OS preference (`prefers-color-scheme`)

Modern operating systems (macOS, Windows, iOS, Android) have a global "dark mode" toggle in Settings. Browsers expose it to CSS via a media query:

```css
@media (prefers-color-scheme: dark) {
    /* rules that only apply when the OS is set to dark mode */
}
```

- **Pro:** zero JavaScript. If a visitor's phone is in dark mode, your site respects that automatically.
- **Con:** the user can't override *your site alone*. If they want your site dark while the rest of their OS is light, they can't.

#### Method 2 — Explicit toggle (`[data-theme="dark"]`)

You add an attribute to `<html>` (or `<body>`) and style based on it:

```html
<html data-theme="dark">
```

```css
[data-theme="dark"] {
    /* rules that apply when the attribute is set */
}
```

The user flips it with a button — you need a tiny bit of JavaScript to toggle the attribute. But now the user is in charge.

- **Pro:** user has full control, preference can be saved.
- **Con:** you have to write the toggle button + JS.

**Best practice:** combine both. Follow OS by default, let the user override if they want.

### The critical insight — literal vs. semantic tokens

Here's the part I glossed over in the main tutorial. Imagine you wrote your palette like this:

```css
:root {
    --color-neutral-100: hsl(0, 0%, 100%);   /* white */
}

[data-theme="dark"] {
    --color-neutral-100: hsl(210, 15%, 10%); /* dark */
}
```

**This works**, but it's *confusing*. `--color-neutral-100` was named after its **value** ("100 = the lightest neutral"). Now in dark mode it's the *darkest* color. **The name lies.**

There are two ways to structure your tokens. Every serious design system uses **Approach B**.

#### Approach A — literal tokens (simpler, but loses meaning)

Keep your palette named by lightness (100 = lightest, 900 = darkest). In dark mode, invert them:

```css
:root {
    --color-neutral-100: hsl(0, 0%, 100%);
    --color-neutral-900: hsl(210, 20%, 10%);
}

[data-theme="dark"] {
    --color-neutral-100: hsl(210, 20%, 10%);
    --color-neutral-900: hsl(0, 0%, 100%);
}
```

Works, but reading the CSS is confusing. If someone sees `background: var(--color-neutral-100)`, they can't tell if that means "light" or "dark" — depends on mode.

#### Approach B — semantic tokens (recommended)

Add a **second layer** of tokens named by **role**, not by value. The palette stays literal; the semantic tokens *point at* palette tokens:

```css
:root {
    /* Layer 1 — the raw palette (never used directly in components) */
    --palette-white: hsl(0, 0%, 100%);
    --palette-grey-100: hsl(210, 20%, 96%);
    --palette-grey-500: hsl(210, 10%, 50%);
    --palette-grey-900: hsl(210, 20%, 10%);

    /* Layer 2 — semantic tokens (this is what components use) */
    --color-surface: var(--palette-white);
    --color-surface-muted: var(--palette-grey-100);
    --color-text-primary: var(--palette-grey-900);
    --color-text-muted: var(--palette-grey-500);
    --color-border: var(--palette-grey-100);
}

[data-theme="dark"] {
    /* Only remap the semantic layer — palette stays the same */
    --color-surface: var(--palette-grey-900);
    --color-surface-muted: hsl(210, 15%, 15%);
    --color-text-primary: var(--palette-white);
    --color-text-muted: var(--palette-grey-500);
    --color-border: hsl(210, 15%, 25%);
}
```

Now components look like this:

```css
.card {
    background: var(--color-surface);
    color: var(--color-text-primary);
    border: 1px solid var(--color-border);
}
```

**Read that CSS.** In *any* mode, the meaning is obvious: card sits on a "surface," has "primary text," has a "border." The card doesn't care which palette color those resolve to. **That's the whole point of semantic tokens.**

- **Palette tokens** = the paint cans in your garage.
- **Semantic tokens** = the label on the wall that says "living room wall color."

When you redecorate, you change what "living room wall color" points to. You don't rename the paint cans.

### A full working example — light + dark with both triggers

Put this together in `tokens.css`:

```css
:root {
    /* Palette */
    --palette-white: hsl(0, 0%, 100%);
    --palette-grey-100: hsl(210, 20%, 96%);
    --palette-grey-500: hsl(210, 10%, 50%);
    --palette-grey-900: hsl(210, 20%, 10%);
    --palette-primary: hsl(212, 86%, 55%);

    /* Semantic — light theme defaults */
    --color-surface: var(--palette-white);
    --color-surface-muted: var(--palette-grey-100);
    --color-text-primary: var(--palette-grey-900);
    --color-text-muted: var(--palette-grey-500);
    --color-accent: var(--palette-primary);
}

/* Dark theme — via OS preference */
@media (prefers-color-scheme: dark) {
    :root {
        --color-surface: var(--palette-grey-900);
        --color-surface-muted: hsl(210, 15%, 15%);
        --color-text-primary: var(--palette-white);
        --color-text-muted: hsl(210, 10%, 65%);
    }
}

/* Dark theme — via explicit user toggle (overrides OS) */
[data-theme="dark"] {
    --color-surface: var(--palette-grey-900);
    --color-surface-muted: hsl(210, 15%, 15%);
    --color-text-primary: var(--palette-white);
    --color-text-muted: hsl(210, 10%, 65%);
}

/* Optional: let user force LIGHT mode even if OS is dark */
[data-theme="light"] {
    --color-surface: var(--palette-white);
    --color-surface-muted: var(--palette-grey-100);
    --color-text-primary: var(--palette-grey-900);
    --color-text-muted: var(--palette-grey-500);
}
```

**Reading order (the cascade):**

1. `:root` sets defaults (light).
2. `@media (prefers-color-scheme: dark)` overrides them *if* the OS is in dark mode.
3. `[data-theme="dark"]` overrides *both* of the above if the user explicitly chose dark.
4. `[data-theme="light"]` lets a user force light even in a dark OS.

Each layer wins over the ones above it because it appears later in the stylesheet (or is more specific).

### The toggle button (~10 lines of JavaScript)

```html
<button id="theme-toggle">Toggle theme</button>
```

```javascript
const toggle = document.getElementById('theme-toggle');
const html = document.documentElement;

// On page load, apply saved preference (if any)
const saved = localStorage.getItem('theme');
if (saved) html.setAttribute('data-theme', saved);

toggle.addEventListener('click', () => {
    const current = html.getAttribute('data-theme');
    const next = current === 'dark' ? 'light' : 'dark';
    html.setAttribute('data-theme', next);
    localStorage.setItem('theme', next);
});
```

That's it. The click flips `data-theme` on `<html>`, the CSS cascade repaints the whole page, and `localStorage` remembers the choice for next visit.

### Why this is powerful

Because your components use **semantic tokens** (`--color-surface`, `--color-text-primary`), **no component code changes** when you add dark mode. The card component doesn't know or care whether the surface is white or dark. You wire dark mode entirely in the token file.

**That's the emotional payoff of a design system:** theming is a token-layer concern, not a component-layer concern. Components stay simple.

### The one-line takeaway

Design for theming from day one by naming tokens by **role** (surface, text-primary, border) rather than by **value** (grey-100, grey-900). Then dark mode is just remapping semantic → palette on a `[data-theme="dark"]` selector. 🌗
