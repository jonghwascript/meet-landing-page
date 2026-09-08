# Frontend Mentor - Meet landing page solution

This is a solution to the [Meet landing page challenge on Frontend Mentor](https://www.frontendmentor.io/challenges/meet-landing-page-rbTDS6OUR). Frontend Mentor challenges help you improve your coding skills by building realistic projects.

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

- View the optimal layout depending on their device's screen size
- See hover states for interactive elements

### Screenshot

![Screenshot of the finished Meet landing page solution](./screenshot.jpg)

### Links

- Solution URL: [Repository](https://github.com/jonghwascript/meet-landing-page.git)
- Live Site URL: [Live site](https://jonghwascript.github.io/meet-landing-page)

## My process

### Built with

- Semantic HTML5 markup
- CSS custom properties (color palette and typography presets)
- CSS Grid (hero layout, per-breakpoint `grid-template-areas`)
- Flexbox (content media grid, footer layout)
- Mobile-first workflow (375px / 768px / 1024px breakpoints)
- CSS Anchor Positioning (experimental) for the section-divider badge
- BEM naming convention

### What I learned

**`anchor()` only tells you where the anchor line is — it doesn't recenter your box on it.**

I used the new CSS Anchor Positioning API to make a small circular badge straddle the boundary between two sections:

```css
.divider--two {
  position-anchor: --boundary;
  position: absolute;
  top: anchor(top);
  left: anchor(center);
  transform: translate(-50%, -79%);
}
```

My first attempt only used `top`/`left` with `anchor()` and the badge ended up hanging *below* the boundary instead of straddling it. The reason: `position: absolute` always positions an element's top-left corner, so `top: anchor(top)` pins the badge's top edge to the anchor's top edge — it doesn't center the badge on that line. `anchor()` is purely a "where is the target line?" lookup; `transform: translate(-50%, ...)` is still required to pull the element back by half its own size and actually center it on that line. The two techniques are a pair, not a substitute for one another.

**BEM naming is easy to violate without noticing.**

While reviewing the markup I found a few naming issues that are worth remembering for the next project:

- Nesting elements with multiple layers (e.g. `content__text-message` where `content__text` is itself supposed to be an element) blurs which node is actually the "block." BEM wants **flat** naming under one block, not a hierarchy:
  ```
  content            (block)
  ├─ content__media
  ├─ content__body
  │   ├─ content__eyebrow
  │   ├─ content__title
  │   └─ content__desc
  ```
- Modifiers must use `--`, not be tacked on as bare classes. `class="divider one"` looks like an independent class; `class="divider divider--one"` makes the block/modifier relationship explicit both in the markup and in the CSS selector (`.divider--one` vs. `.divider.one`).
- A class name should describe a role, not the literal text/content it happens to hold right now. `what-is-it` (named after the button's copy) breaks the moment the copy changes; `secondary-action` doesn't. Same idea applied to `footer__sub-title`, which was actually the *only* (i.e. main) heading in the footer — renamed to `footer__title`.
- Bare classes with no block namespace (`logo`, `download`) aren't wrong by themselves, and mixing a bare class with a BEM element on the same node (`class="footer__download download"`) is a legitimate BEM "mix" technique — but if a bare class is meant to be a reusable component, giving it its own block name (`btn`, `btn--primary`) keeps the system consistent.

**A horizontal scrollbar can appear from "intentional" bleed, not just a bug.**

On mobile, the hero's circular avatar images are supposed to be cropped by the edge of the viewport — that's the actual design, not an accident. The scrollbar didn't come from a broken layout; it came from the browser correctly reporting `document.scrollWidth > document.clientWidth` once those images (fixed at `208.05px` each, `two + gap` wider than a 375px viewport) pushed the content wider than the screen. The fix isn't to shrink the images to "fit" — that would change the design — it's to clip the overflow instead:

```css
html {
  overflow-x: hidden;
}

body {
  margin: 0;
  padding: 0;
  overflow-x: hidden;
}
```

It needed to be set on **both** `html` and `body`, because which element ends up acting as the page's actual scrolling container differs across browsers/engines — setting it on only one of the two left the scrollbar in place for some browsers.

### Continued development

- Explore a safe fallback for the Anchor Positioning badge for browsers that don't yet support it (e.g. giving a positioned ancestor a sane default `top`/`left` so the badge doesn't jump to the top of the page instead of straddling the section boundary).
- Do a BEM naming pass **before** writing markup next time, instead of retrofitting it after the fact.

### Useful resources

- [MDN - CSS anchor positioning](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_anchor_positioning) - explains the `anchor()` function and `position-anchor`/`anchor-name` and clarified why `transform` is still needed alongside it.
- [BEM - Block Element Modifier methodology](https://getbem.com/) - the reference I checked against while auditing class names for modifier syntax and flat naming.

### AI Collaboration

This project was reviewed and iterated on with **Claude Code**.

- Used for: a full code review pass (invalid CSS values like `160xp`/`row-gap: 72` with no unit, hardcoded non-responsive background/hero images despite per-breakpoint assets already existing in `/images`, a BEM naming audit), then applying the agreed fixes directly to the HTML/CSS.
- It also diagnosed and fixed a mobile-only horizontal scroll issue, and verified the responsive hero-image swap (mobile/tablet/desktop) by spinning up a local static server and taking Playwright screenshots at three viewport widths before calling the change done.
- What worked well: catching typos and invalid CSS that are easy to miss by eye, and cross-checking existing image assets against what the markup/CSS actually reference.
- What required back-and-forth: a couple of fixes (like the `overflow-x: hidden` scroll fix, and which selector a border rule should actually target) needed a second round after real-browser testing showed the first attempt wasn't enough.

## Author

- Frontend Mentor - [@jonghwascript](https://www.frontendmentor.io/profile/jonghwascript)
