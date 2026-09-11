# Frontend Mentor - Four card feature section solution

This is my solution to the [Four card feature section challenge on Frontend Mentor](https://www.frontendmentor.io/challenges/four-card-feature-section-weK1eFYK). Frontend Mentor challenges help improve coding skills by building realistic projects.

![Static Badge](https://img.shields.io/badge/https%3A%2F%2Fimg.shields.io%2Fbadge%2FDifficulty-newbie-%236abecd?style=for-the-badge&logo=Frontend%20mentor&label=Diffilcuty&labelColor=%23555555&color=%236abecd)

## Table of contents

- [Overview](#overview)
  - [The challenge](#the-challenge)
  - [Screenshot](#screenshot)
  - [Links](#links)
- [My process](#my-process)
  - [Built with](#built-with)
  - [What I learned](#what-i-learned)
  - [Continued development](#continued-development)
  - [Useful resources](#useful-resources)
  - [AI Collaboration](#ai-collaboration)
- [Author](#author)

## Overview

### The challenge

Users should be able to:

- View the optimal layout for the site depending on their device's screen size.

The design shows a header block above a set of four feature cards. On mobile the cards stack in a single column; on desktop they form a three-column layout where the middle column stacks two cards, and the outer cards are centered vertically alongside them.

### Screenshot

![](./screenshot.jpg)

### Links

- Solution URL: https://github.com/gmansoain/frontend-mentor-four-cards-feature.git
- Live Site URL: https://gon-four-cards-feature.netlify.app/

## My process

### Built with

- Semantic HTML5 markup
- CSS custom properties (design tokens for colors)
- CSS Grid (desktop layout with row-spanning)
- Flexbox (icon alignment inside cards)
- Mobile-first workflow
- Self-hosted `@font-face` for Poppins (weights 200, 400, 600)

### What I learned

The layout looks simple, but building it robustly across screen sizes surfaced a handful of things worth writing down.

**1. Prefer layout-engine alignment over hardcoded margins.**

My first attempt centered the outer cards next to the two stacked middle cards by using magic-number margins:

```css
/* Before — brittle */
.card:nth-child(1),
.card:nth-child(4) {
    grid-row: 1 / 3;
    margin: 9rem 0;
}
```

Cleaner: let Grid handle the alignment on the container:

```css
/* After — adapts to content */
.cards-container {
    display: grid;
    align-items: center;
}
```

**2. `min-width` on the body causes horizontal scroll on small screens.**

I originally had `min-width: 23rem` on `body`, which meant that at 320px viewports the body couldn't shrink and the browser introduced a horizontal scrollbar. Removing it — and moving the width cap to the element I actually wanted to constrain (`.card`) — fixed it:

```css
.card {
    width: 100%;
    max-width: 28rem;
    margin: 0 auto;
}
```

The rule now lives on the thing it describes, which is more expressive than sizing the entire `<body>`.

**3. Semantic HTML matters even when it "looks the same."**

For the two-line heading with a different font-weight on the second line, I first used `<strong>`. But `<strong>` means "this text is semantically more important" — screen readers may emphasize it. The change was purely visual, so a neutral `<span>` is the honest choice:

```html
<h1>
    Reliable, efficient delivery<br>
    <span>Powered by Technology</span>
</h1>
```

**4. Don't set a default that every case overrides.**

Instead of setting a fallback border color on `.card` and then overriding it on every nth-child, I removed the default and let each card declare its own color:

```css
.card {
    border-top: 0.25rem solid; /* no color — each card sets its own */
}
.card:nth-child(1) { border-top-color: var(--color-primary-cyan); }
.card:nth-child(2) { border-top-color: var(--color-primary-red); }
.card:nth-child(3) { border-top-color: var(--color-primary-orange); }
.card:nth-child(4) { border-top-color: var(--color-primary-blue); }
```

**5. `rem` is relative to the root, not to `body`.**

Setting `font-size: 15px` on `body` matches the style guide's body copy size, but `1rem` still equals 16px because `rem` is relative to `<html>`. Useful to remember when sizing components with `rem`.

### Continued development

- **Tablet breakpoint**: right now the layout jumps from a single narrow column to a three-column grid at `48rem`. Between 448px and 768px, the single card sits centered with a lot of empty space. A dedicated 2-column intermediate layout would use that space better.
- **Design tokens**: I documented spacing and font-size scales as comments. Next step is to promote them to actual CSS custom properties so they're enforceable, not just conventional.
- **`justify-content` conventions**: I want to keep leaning into logical/axis-based Flexbox values (`flex-end`, `flex-start`) over physical directions (`left`, `right`).

### Useful resources

- [MDN — CSS Grid Layout](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout) — the definitive reference for grid tracks, lines, and `grid-row`/`grid-column`.
- [MDN — `min-width` / `max-width`](https://developer.mozilla.org/en-US/docs/Web/CSS/min-width) — helped me reason about which constraint actually belonged where.
- [MDN — `<strong>` vs `<span>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/strong) — clarified the semantic weight of `<strong>` and why it's not just "bold."

### AI Collaboration

I used **Claude Code** as a mentor during this challenge. The setup was configured (via `AGENTS.md`) so the assistant would guide me with questions and hints rather than write code for me.

- **What worked well:** Being asked *why* I made a choice ("What is `min-width` actually for?") caught mistakes I wouldn't have noticed on my own. The hint-progression style — conceptual nudge → more specific hint → near-solution — meant I kept ownership of the code.
- **What I noticed:** The biggest wins came from questions I couldn't answer confidently. Every time I couldn't justify a line, that line turned out to need fixing. Great forcing function for writing intentional CSS.
- **What I'd repeat:** Iterating through a review checklist and pushing fixes one at a time, then re-reviewing. The `min-width` → `max-width on body` → `max-width on card` progression is a good example of how a fix can be correct and *still* improve over multiple passes.

## Author

- Frontend Mentor - [@gmansoain](https://www.frontendmentor.io/profile/gmansoain)
