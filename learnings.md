# Four Card Feature Section — Learnings

Review notes and takeaways from the code review of my solution.

## What I did well from the start

- **Semantic HTML**: reached for `<header>`, `<main>`, `<article>` instead of generic `<div>`s.
- **CSS custom properties**: colors defined in `:root` — easy to theme and maintain.
- **Documented systems**: kept a font-size and spacing scale as comments at the top of the CSS.
- **Mobile-first approach**: started with single-column layout, expanded at `min-width: 48rem`.
- **CSS Grid for the desktop layout**: used `grid-row` to span the outer cards across two rows while the middle column stacks two cards.

---

## Corrections applied and the reasoning behind them

### 1. Base font-size wasn't set

**Before:** No `font-size` on `body` — browsers defaulted to 16px.
**After:** `font-size: 15px` on `body`, matching the style guide.

**Learning:** Setting a base `font-size` is worth doing explicitly. Note that `rem` is relative to the **root element** (`<html>`), not to `body`, so setting `font-size` on `body` doesn't change what `1rem` means — it only affects text that inherits from `body`.

### 2. Centering outer cards with hardcoded margins

**Before:** Used `margin: 9rem 0` on the outer cards to visually center them next to the two stacked middle cards. Magic number that would break if content changed.

**After:** Added `align-items: center` to `.cards-container`. Grid centers each item within its grid area — cards spanning both rows get centered against the combined height.

**Learning:** Prefer letting the layout engine (Grid/Flexbox) handle alignment over hardcoded spacing. If the content changes, the layout adapts on its own. Magic numbers are a smell — they're brittle.

### 3. `min-width` on body caused horizontal scroll

**Before:** `min-width: 23rem` on `body` (= 368px). At viewports narrower than that (e.g., 320px), the body couldn't shrink → **horizontal scroll on real phones**.

**After (v1):** Replaced with `max-width: 28rem` on `body`.

**After (v2 — better):** Moved the constraint from `body` to `.card`, added `width: 100%` and `margin: 0 auto` on `.card`.

**Learning:**
- `min-width` blocks the browser from shrinking → causes horizontal scroll below that threshold. Almost never what you want on responsive designs.
- **Where a rule lives matters as much as what it says.** If the intent was "cards shouldn't grow too wide," the constraint belongs on `.card`, not on `body`. Code becomes self-documenting when the rule sits on the thing it describes.
- Horizontal scrolling on mobile is one of the biggest UX complaints in web design — worth testing at 320px in DevTools before shipping.

### 4. `<strong>` used only for styling

**Before:** `<h1>Reliable, efficient delivery<br><strong>Powered by Technology</strong></h1>`

**After:** `<span>` instead of `<strong>`, targeted with `.header h1 span`.

**Learning:** `<strong>` means "this text is semantically more important" — screen readers may announce it with emphasis. When the change is purely visual (a different font weight), use a neutral element like `<span>`. Reserve semantic tags for their real meaning.

### 5. Redundant per-card default color

**Before:** `.card` had `border-top: 0.25rem solid var(--color-primary-cyan)`, then `.card:nth-child(1)` re-set the same cyan color explicitly.

**After:** `.card` declares only `border-top: 0.25rem solid` (no color). Each `.card:nth-child(N)` sets its own color. Every card declares its color, no duplication, no implicit default.

**Learning:** When a base rule needs to be overridden by every specific case, don't set a default at all — let each case declare itself. Cleaner and more honest.

### 6. Small polish

- **Extra whitespace** in the Supervisor `<p>` (leading double space) — trimmed.
- **Empty `<footer></footer>`** — filled with an attribution block.
- **Commented-out code** left after refactoring — removed once the new approach was verified.

**Learning:** Small details compound. Commented-out code left behind confuses future readers — trust the new solution and delete the old.

---

## Meta-lessons

- **Test at edges before shipping.** 320px, 448px, 768px, 1440px — walk the viewport widths intentionally. The bugs live between the design mockups.
- **Ask "why" for every line.** If I can't explain why a rule exists (e.g., "why `min-width: 23rem`?"), that's a hint it might not belong.
- **Prefer layout-engine solutions over magic numbers.** `align-items: center` beats `margin: 9rem 0`. The engine adapts; magic numbers don't.
- **Put the rule where the concept lives.** "Card max-width" belongs on `.card`, not on `body`. Naming and location tell the story.
- **Semantic tags carry meaning, not just style.** Use `<span>` for style hooks; reserve `<strong>`, `<em>`, `<article>` for their semantic roles.

---

## Progression to notice

The `min-width` → `max-width on body` → `max-width on card` sequence is a good example of how CSS thinking matures:

1. **First fix**: patch the immediate bug.
2. **Second fix**: use the right tool (`max-width` not `min-width`).
3. **Third fix**: put the rule in the right place (on the thing it describes).

Each step is correct — the third is just more expressive of intent.
