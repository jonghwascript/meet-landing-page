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

**Hover states need to account for shared classes, not just the base color.**

The challenge requires visible feedback on interactive elements, so both hero buttons needed a `:hover` state:

```css
.download:hover {
  background-color: color-mix(in srgb, var(--Cyan-600) 85%, black);
}
```

`color-mix()` darkens the actual brand color instead of a hand-picked hex value, so the hover shade stays correct even if the base color changes later. The catch was the footer's download button, which reuses the `.download` class (`class="footer__download download"`) but should darken from *purple*, not cyan. `.download:hover` and `.footer__download:hover` have equal specificity, so whichever rule comes **later** in the stylesheet wins — without a `.footer__download:hover` rule placed after `.download:hover`, the footer button would flash cyan on hover instead of a darker purple.

**A size that's fixed in pixels will eventually overflow a flexible container.**

The desktop hero images (`hero__left`/`hero__right`) were fixed at `394px × 303px` — the exact resolution of their source PNGs — but they sit inside a `minmax(0, 1fr)` grid track, which actually shrinks to as little as ~224px at 1024px and never grows past ~272px even at the widest supported width. The images were overflowing their track at every desktop size; it just wasn't visible as a scrollbar because the earlier `overflow-x: hidden` fix was quietly clipping it. The real fix is to let the image respond to its track instead of assuming the track will always be wide enough:

```css
.hero__left,
.hero__right {
  width: 100%;
  max-width: 394px;
  height: auto;
  aspect-ratio: 394 / 303;
}
```

`width: 100%` fills whatever the track actually resolves to, `max-width` stops it from upscaling past its native resolution, and `aspect-ratio` keeps it from distorting as the width changes.

**An image wrapped in a link needs a name, even if it's "just" a logo.**

`<a href="#"><img src="./logo.svg" alt=""></a>` left the link with no accessible name — a screen reader would announce "link" with no indication of where it goes or what it represents. An empty `alt` is only correct for images that are purely decorative; a logo that identifies the site isn't decorative, so it needs real text (`alt="Meet"`).

**`width: min(X, 100%)` only looks fluid — inside the media query where it lives, `X` is always smaller than the viewport, so it's just a fixed width in disguise.**

`.wrapper` used `width: min(375px, 100%)` at the base, then re-declared `width: min(768px, 100%)` and `width: min(1440px, 100%)` inside the tablet and desktop `@media` blocks. Each override only applies once the viewport is *already* past that breakpoint, so within the block's own active range (376–767px, 769–1023px) the viewport is always wider than the pixel value — `min()` picks the fixed number every time. The result: any window size between two breakpoints renders the *previous* tier's exact design width, centered with dead space on both sides — including the `.footer`'s full-bleed background band, since `.footer` is a child of `.wrapper`.

The fix isn't a `min()` → `max-width` syntax swap (they compute to the identical value); it's giving `.wrapper` a single cap that never shrinks:

```css
.wrapper {
  width: 100%;
  max-width: 1440px;
  margin-inline: auto;
}
```

with no per-breakpoint override left in either media query. `.landing` keeps its own per-tier caps (369px / 680px / 1120px) unchanged, because that one *is* supposed to freeze at each tier's exact design width — the hero grid and image sizes inside it are built for that specific width. The bug was specific to the outer full-bleed wrapper, not the inner content column.

**`display: none` doesn't cancel an `<img>`'s network request — but it does for a CSS `background-image`.**

`.hero__left`, `.hero__right`, and `.hero__media` were all real `<img>` elements, one per breakpoint, hidden with `display: none` outside their own tier. The browser fetches an `<img src>` the moment it parses the tag, regardless of CSS — so every visitor downloaded all three images and threw away whichever two didn't apply to their viewport. `.hero__media` also had no `aspect-ratio`, so before it loaded there was no reserved box: the headline and buttons sat higher on the page, then jumped down once the image arrived.

The fix was to drop the `<img>`s for empty, purely-decorative `<div>`s (they already carried `alt=""`, so nothing was lost for screen readers) and move each image into a CSS `background-image`, declared **only inside the media query where it's actually shown**:

```css
.hero__media {
  display: none;
  width: 100%;
  aspect-ratio: 820 / 303;
  background-size: cover;
}

@media (min-width: 768px) and (max-width: 1023px) {
  .hero__media {
    display: block;
    background-image: url(../images/tablet/image-hero.png);
  }
}
```

A `background-image` is only fetched once the rule that declares it actually applies *and* the element gets a layout box — so a `display: none` element's background is never requested at all. `.hero__left`/`.hero__right` needed the same treatment split across two ranges (`max-width: 767px` and `min-width: 1024px`, since they're shown at both mobile and desktop but hidden at tablet), rather than one unconditional declaration. Verified with a local static server and Playwright network logs at 375px/900px/1440px: each width now requests exactly the two images it displays, never the other four.

**An unsupported `anchor()` doesn't just fail loudly — it quietly falls back to a broken layout, with no `@supports` guard to catch it.**

The section-2 divider badge used to straddle the content/footer seam with CSS Anchor Positioning:

```css
.divider--two {
  position-anchor: --boundary;
  position: absolute;
  top: anchor(top);
  left: anchor(center);
  transform: translate(-50%, -79%);
}
```

On a browser without Anchor Positioning support, `anchor()` isn't a parse error — the declaration is just invalid at compute time, so `top`/`left` fall back to `auto`. The element is still `position: absolute`, so it renders at its static in-flow position instead of at the anchor, and the `transform` (tuned for the anchored position) drags it up and to the left over whatever content happens to be there. `overflow-x: hidden` on `html`/`body` (added earlier for the intentional mobile avatar bleed) hides the horizontal half of that damage, which made it easy to miss.

The replacement needs no anchor lookup at all — pull the footer up over the badge instead of pulling the badge down onto the footer:

```css
.divider--two {
  position: relative;
  z-index: 1;
  margin-block-start: 4rem;
}

.footer {
  margin-block-start: -1.75rem; /* half the 56px circle, so it straddles the seam */
}
```

The `z-index` is easy to skip and looks redundant — `.footer` isn't positioned, so shouldn't a positioned `.divider--two` already paint above it? Not here: `.divider--two` and `.footer` aren't siblings (`.divider--two` is inside `.landing`, `.footer` is `.landing`'s sibling), and `position: relative` with `z-index: auto` doesn't create a new stacking context. Without an explicit `z-index`, `.divider--two` stays in the normal in-flow paint order, and `.footer` — coming later in the DOM — paints over it. Giving `.divider--two` a real `z-index` promotes it into its own stacking context, which paints after (above) any non-positioned content regardless of DOM order.

**An orphaned `border-top: 1px solid #000` with no matching design and a same-property override at the next breakpoint is a tell that it was leftover debugging, not intentional.**

`.footer` had `border-top: 1px solid #000`, immediately cancelled by `border-top: none` in the tablet media query — a one-breakpoint-only line with nothing in the design comp to justify it. Deleted both declarations.

### Continued development

- Do a BEM naming pass **before** writing markup next time, instead of retrofitting it after the fact.
- When sizing an image with a fixed pixel value, check it against its container's actual available space (especially inside a flexible grid track) instead of assuming the container will always be big enough.

### Useful resources

- [MDN - CSS anchor positioning](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_anchor_positioning) - explains the `anchor()` function and `position-anchor`/`anchor-name` and clarified why `transform` is still needed alongside it. The section-divider badge no longer uses this (see "What I learned"), but the API itself was still worth understanding.
- [BEM - Block Element Modifier methodology](https://getbem.com/) - the reference I checked against while auditing class names for modifier syntax and flat naming.

### AI Collaboration

This project was reviewed and iterated on with **Claude Code**.

- Used for: a full code review pass (invalid CSS values like `160xp`/`row-gap: 72` with no unit, hardcoded non-responsive background/hero images despite per-breakpoint assets already existing in `/images`, a BEM naming audit), then applying the agreed fixes directly to the HTML/CSS.
- It also diagnosed and fixed a mobile-only horizontal scroll issue, and verified the responsive hero-image swap (mobile/tablet/desktop) by spinning up a local static server and taking Playwright screenshots at three viewport widths before calling the change done.
- A follow-up review pass (styled as inline PR-style comments) caught three more issues in one go: missing `:hover` states on the buttons (a stated challenge requirement), the desktop hero images silently overflowing their grid track, and the header logo's `alt=""` leaving its link with no accessible name.
- A later PR-style comment diagnosed `.wrapper`'s `width: min(X, 100%)` re-declared per breakpoint as a fixed-width bug in disguise, and correctly separated it from `.landing`'s per-tier caps, which needed to stay as-is.
- Another comment caught all three hero images being downloaded on every visit regardless of breakpoint, plus a missing `aspect-ratio` causing a layout jump on tablet; after converting them from `<img>` to breakpoint-scoped `background-image` divs, it verified the fix with real network-request logs from a headless browser at three viewport widths instead of just reading the CSS.
- A final round flagged the Anchor Positioning divider badge as a no-`@supports`-guard progressive-enhancement risk and an orphaned `border-top` with no matching design; replacing the badge with an in-flow negative-margin technique was verified by measuring the actual rendered gap between the divider and the footer in a headless browser (confirming the circle overlaps the seam by exactly half its height) rather than trusting the CSS by inspection alone.
- What worked well: catching typos and invalid CSS that are easy to miss by eye, and cross-checking existing image assets against what the markup/CSS actually reference.
- What required back-and-forth: a couple of fixes (like the `overflow-x: hidden` scroll fix, and which selector a border rule should actually target) needed a second round after real-browser testing showed the first attempt wasn't enough.

## Author

- Frontend Mentor - [@jonghwascript](https://www.frontendmentor.io/profile/jonghwascript)
