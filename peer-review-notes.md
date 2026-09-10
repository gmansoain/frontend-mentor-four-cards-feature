# Peer Review — Darionas' Solution

Notes from reading through a peer developer's solution to the same "Four card feature section" challenge. The goal here isn't to judge their code, but to spot **techniques worth borrowing** and **decisions worth thinking about**.

Peer: [Darionas on Frontend Mentor](https://www.frontendmentor.io/profile/Darionas)
Repo: <https://github.com/Darionas/four-card-feature-section-master>

---

## Overall impression

Solid, thoughtful solution with several **professional-grade patterns** I hadn't reached for. This person is clearly further along than a beginner — they're using industry conventions I want to internalize. That said, the sophistication comes with a few rough edges worth understanding before I copy the patterns.

---

## Things worth borrowing

### 1. Modern CSS reset (Andy Bell / Piccalilli)

Instead of a universal `* { margin: 0; padding: 0; }`, they use a **thoughtful reset** with clear intent behind each rule:

```css
html:focus-within { scroll-behavior: smooth; }

body {
  min-height: 100vh;
  text-rendering: optimizeSpeed;
  line-height: 1.5;
}

img, picture {
  max-width: 100%;
  display: block;
}

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    ...
  }
}
```

**Takeaway:** Adopt Andy Bell's reset (or a curated version of it) for future projects. It handles accessibility (reduced motion), image defaults (block + max-width), and sensible baselines — all things I was doing ad-hoc.

### 2. Design token naming (`text-preset-1`, `spacing-500`)

Their type system uses **numbered presets** as classes on elements, backed by CSS variables:

```html
<h1 class="text-preset-2">...</h1>
<p class="text-preset-4">...</p>
```

```css
:root {
    --spacing-500: 2.5rem;
    --spacing-400: 2rem;
    --spacing-300: 1.5rem;
    --spacing-200: 1rem;
    --spacing-100: 0.5rem;
}
```

This is how real design systems (Figma → code) map tokens. Numbered scales are **medium-agnostic** — they work whether the design is called "large" or "24px" in Figma.

**Takeaway:** In my next project, promote the spacing/font-size comments I wrote at the top of my CSS into actual CSS variables (`--font-size-100`, `--space-400`, etc.). The comments were the right instinct — variables are the honest execution.

### 3. Separate `design_system.html` file

They shipped a **living style guide** — a stand-alone HTML page that renders the color palette, typography scale, and spacing tokens. This is what serious teams do; it doubles as visual regression bait.

**Takeaway:** For any project with a design system worth naming, ship a `design-system.html` alongside `index.html`. Cheap insurance, huge signal of intent.

### 4. BEM-flavored naming (`feature--supervisor`)

Each card is `.feature .feature--supervisor` etc. — the base class holds shared styles, the modifier class holds the color variant:

```css
.feature { padding: ...; border-radius: 8px; }
.feature--supervisor { border-top: 4px solid var(--clr-cyan); }
```

Compare to my solution, which uses `:nth-child()` selectors. Their approach is **more expressive** and **doesn't break** if I reorder the cards in HTML. My version couples visual color to DOM position — theirs couples visual color to semantic name. Theirs is better.

**Takeaway:** Prefer explicit modifier classes over positional selectors when the styling is tied to *what the element is*, not *where it sits*.

### 5. Logical properties (`margin-block-start`)

They consistently use `margin-block-start` instead of `margin-top`. Same visual result today, but **future-proofs for RTL languages** (Arabic, Hebrew) and vertical writing modes (Japanese).

**Takeaway:** Start using `margin-block-*` and `margin-inline-*` by default. It's the same effort and it's the direction the platform is moving.

### 6. Fluid typography with `clamp()`

For the heading, they use:

```css
.text-preset-2 {
    font-size: clamp(1.3rem, 2vw, 2.25rem);
}
```

This lets the heading **scale smoothly** with the viewport between 1.3rem (min) and 2.25rem (max), sized at 2vw in between. No media-query jumps.

**Takeaway:** `clamp(min, preferred, max)` is a powerful pattern for responsive type. Worth practicing until it feels natural. Compare to my fixed `1.6rem` heading, which stays exactly 1.6rem at every viewport.

### 7. Three breakpoints, not two

They ship a **real tablet layout** at `41rem` (656px) where the four cards form a 2-column grid, then reflow to the classic 3-column layout at `48rem` (768px).

```css
@media (min-width: 41rem) { /* 2-column */ }
@media (min-width: 48rem) { /* 3-column */ }
```

I skipped the tablet layout entirely — my card just sits centered with empty margins between 448px and 768px. Their approach uses that middle range instead of wasting it.

**Takeaway:** For future challenges, ask "what's between mobile and desktop?" rather than jumping straight from one to the other.

### 8. `prefers-reduced-motion` honored

Their reset explicitly disables animations for users who've opted out at the OS level. A one-time cost that respects real users with vestibular disorders.

**Takeaway:** Include this rule in every project's reset from now on.

### 9. `preconnect` hints for Google Fonts

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
```

Performance micro-optimization — starts the TCP/TLS handshake to font servers before the CSS is parsed.

**Takeaway:** If loading fonts from a CDN, add `preconnect`. Free perf win.

---

## Things worth questioning

### 1. The h1 / p semantic split

The design shows two lines: *"Reliable, efficient delivery"* and *"Powered by Technology"*. The peer marked them up as:

```html
<h1 class="title text-preset-2">Reliable, efficient delivery</h1>
<p class="subtitle text-preset-1">Powered by Technology</p>
```

Two things stand out:
- The **bolder line** (which reads as more prominent in the design) is marked up as a `<p>`, not part of the heading.
- Two separate blocks — the second line isn't semantically connected to the first.

Compare to my choice: one `<h1>` containing both lines with a `<span>` on the second for the weight change. Mine keeps them semantically grouped; theirs splits them.

**Question worth sitting with:** Is *"Powered by Technology"* a heading? A subtitle? Body text? There's no single right answer, but the peer's choice makes screen readers announce "heading, Reliable efficient delivery. Powered by Technology." — treating the bold line as a paragraph. That might be less accurate than what the design intends.

### 2. Google Fonts loading *every weight*

The `<link>` imports Poppins in **all weights from 100 to 900, plus italics** — but the design only uses 200, 400, and 600. That's a lot of wasted bytes on the wire.

```html
<link href="https://fonts.googleapis.com/css2?family=Poppins:ital,wght@0,100;0,200;...;1,900&display=swap">
```

My solution self-hosts only the three weights needed via `@font-face`. Two different trade-offs:
- **Their approach:** easy, but heavier and depends on an external CDN.
- **My approach:** lean, offline-capable, but more setup.

**Takeaway:** If using Google Fonts, only request the weights actually used. `&family=Poppins:wght@200;400;600` would have been ~1/9th the size.

### 3. `justify-self: center` on `.container` — probably a no-op

```css
.container {
    width: 85%;
    max-width: 800px;
    justify-self: center;
    justify-items: center;
}
```

`.container` is applied to `<header>`, `<main>`, and `<footer>` — all direct children of `<body>`. **But `<body>` isn't a grid or flex container**, so `justify-self: center` on its children **does nothing**. The containers should be centered with `margin-inline: auto` instead.

Similarly, `justify-items: center` only takes effect if `.container` itself is a grid — which it isn't (except through the descendant `.features`, which is a different element).

Worth loading the peer's site and seeing whether the header/main/footer are actually centered or left-aligned. If they *look* centered, something else is doing the work; if not, this is a real bug they might not have noticed.

**Takeaway:** `justify-self` / `justify-items` only work inside grid/flex contexts. For centering a block element in normal flow, `margin-inline: auto` (or `margin: 0 auto`) is still the right tool.

### 4. Commented-out debug borders

```css
.main {
    margin-block-start: var(--spacing-500);
    /* border: 2px solid var(--clr-grey-500); */
}
.feature {
    ...
    /* border: 2px solid var(--clr-grey-500); */
}
```

Left in the final code. Not a bug, but noisy — the same lesson I applied to my own `margin: 9rem 0` cleanup.

**Takeaway:** Delete debug scaffolding before shipping.

### 5. Two text presets with identical `clamp()`

`.text-preset-1` and `.text-preset-2` both use `clamp(1.3rem, 2vw, 2.25rem)` — the only difference is font-weight (600 vs 200) and letter-spacing. This begs to be DRY'd up, e.g., a shared base preset with weight modifiers.

Not a bug — just an evolution the design system could go through as it grows.

---

## Comparison with my solution

| Aspect | My approach | Peer's approach | Winner |
|---|---|---|---|
| CSS reset | universal `*` reset | modern reset (Andy Bell) | **Peer** |
| Card color coupling | `:nth-child()` | `.feature--{name}` modifier | **Peer** |
| Card width cap | `max-width: 28rem` on `.card` | `max-width` on `.container` (body-level) | **Mine** (rule sits with the concept) |
| Heading semantics | one `<h1>` + `<span>` | `<h1>` + separate `<p>` | **Mine** (arguable, but keeps the concept together) |
| Font loading | self-hosted, 3 weights | Google Fonts, all weights | **Mine** (leaner payload) |
| Breakpoints | 1 (mobile → desktop) | 2 (mobile → tablet → desktop) | **Peer** |
| Type scaling | fixed rem | fluid `clamp()` | **Peer** |
| Design tokens | commented-out scale | real CSS variables | **Peer** |
| Logical properties | none (`margin-top`) | consistent (`margin-block-start`) | **Peer** |
| Reduced motion | no handling | respected via reset | **Peer** |
| Design system doc | none | separate `design_system.html` | **Peer** |

**Score:** Peer wins most categories on **maturity of technique**. I win a couple on **honesty of rule placement** and **payload discipline**.

---

## What I'm taking home

If I could distill this into three actions for my next challenge:

1. **Adopt a modern reset.** Andy Bell's or a curated version. Stop rewriting `* { margin: 0; padding: 0; }` every time.
2. **Promote comment scales to real CSS variables.** My documented spacing/font-size scale should be `--space-100..500` and `--font-size-100..500`, not a comment.
3. **Use modifier classes for state/variant styling, not positional selectors.** `.card--supervisor` beats `.card:nth-child(1)` — it survives reordering and reads better.

Bonus for the medium term:
- Start using `clamp()` for fluid typography.
- Default to logical properties (`margin-block-*`, `margin-inline-*`).
- Add a third breakpoint for tablet when the design has room for one.
