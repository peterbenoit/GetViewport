# GetViewport — Technical Documentation

## Overview

**GetViewport** is a lightweight, zero-dependency JavaScript utility for responsive breakpoint detection. It bridges the gap between CSS media queries and JavaScript logic by using a CSS custom property (`--viewport`) as a live communication channel — letting your JS code know the current breakpoint without polling `window.innerWidth` on every resize.

**Version:** 1.1.2
**License:** MIT
**Author:** Pete Benoit
**npm:** [`get-viewport`](https://www.npmjs.com/package/get-viewport)
**Repository:** [github.com/peterbenoit/GetViewport](https://github.com/peterbenoit/GetViewport)

---

## Why It Exists

Responsive JavaScript logic is a recurring pain point. The common approaches all have trade-offs:

| Approach | Problem |
|---|---|
| `window.innerWidth` checks | Brittle — magic numbers scattered across your codebase |
| CSS pseudo-element hacks (`::before { content: "md" }`) | Fragile, non-standard, hard to maintain |
| Importing SCSS breakpoints into JS | Requires a build step; tightly couples CSS and JS configs |
| `matchMedia()` listeners | Verbose boilerplate; a separate listener per breakpoint |
| Resize observer polling | Overhead; still requires you to define breakpoints twice |

GetViewport solves this by:

1. **Defining breakpoints once** — in JS, passed as a plain object
2. **Injecting CSS automatically** — media queries are generated and inserted at runtime
3. **Reading the CSS variable from JS** — a single `getComputedStyle` call gives you the current breakpoint name and index
4. **Providing a fallback** — if the CSS variable read diverges from the real window width (e.g., during SSR hydration or CSSOM delays), it falls back to direct `window.innerWidth` comparison

---

## Target Audience

GetViewport is built for:

- **Front-end developers** who need to make JavaScript decisions based on breakpoints (toggling behaviors, lazy-loading, conditional rendering) without duplicating breakpoint definitions
- **Teams using vanilla JS or jQuery** who want Bootstrap-compatible breakpoint names without pulling in the full Bootstrap library
- **Framework-agnostic projects** — works as a CommonJS module, AMD module, or plain browser global
- **Legacy codebases** that can't adopt modern CSS Container Queries or a full JS framework

It is intentionally small (~3 KB unminified) and has no runtime dependencies.

---

## How It Works

### Initialization

When you call `new GetViewport()`, the constructor:

1. Accepts an optional custom breakpoints object (or uses Bootstrap-compatible defaults)
2. Generates a sorted internal breakpoint map for fallback detection
3. Calls `injectBreakpointStyles()` — appends a `<style>` tag to `<head>` with one `@media` rule per breakpoint, each setting `--viewport: 'name,index'` on `:root`
4. Attaches a `resize` listener that updates the internal state (with both immediate and debounced updates)
5. Calls `updateViewportVariable()` once to set the initial state

### CSS Variable Bridge

The injected CSS looks like this (for default breakpoints):

```css
@media (min-width: 0px)   { :root { --viewport: 'xs,1'; } }
@media (min-width: 576px) { :root { --viewport: 'sm,2'; } }
@media (min-width: 768px) { :root { --viewport: 'md,3'; } }
@media (min-width: 992px) { :root { --viewport: 'lg,4'; } }
@media (min-width: 1200px){ :root { --viewport: 'xl,5'; } }
@media (min-width: 1400px){ :root { --viewport: 'xxl,6'; } }
```

Because CSS cascade applies the last matching rule, the CSS variable always holds the current breakpoint. JavaScript reads it with:

```js
getComputedStyle(document.documentElement).getPropertyValue('--viewport')
```

### Mismatch Verification

After reading the CSS variable, `verifyBreakpoint()` cross-checks it against a direct width calculation. If they disagree (can happen during rapid resize events or CSSOM staleness), it overwrites the internal state with the width-derived value. This dual-source approach makes the utility resilient to edge cases.

### Resize Handling

The resize listener performs two updates:

- **Immediate** — updates state right away so UI reacts instantly
- **Debounced** — fires again after 100 ms to settle on the final value

---

## Installation

### npm / yarn

```bash
npm install get-viewport
# or
yarn add get-viewport
```

```js
// CommonJS
const GetViewport = require('get-viewport');

// ESM (bundler)
import GetViewport from 'get-viewport';
```

### CDN

```html
<script src="https://unpkg.com/get-viewport/getViewport.js"></script>
<!-- GetViewport is now available as window.GetViewport -->
```

### Manual

Download `getViewport.js` and include it directly:

```html
<script src="path/to/getViewport.js"></script>
```

---

## API Reference

### Constructor

```js
new GetViewport(customBreakpoints?)
```

| Parameter | Type | Default | Description |
|---|---|---|---|
| `customBreakpoints` | `Object` | Bootstrap defaults (see below) | Key/value map of breakpoint name → min-width in px |

**Default breakpoints:**

```js
{
  xs: 0,
  sm: 576,
  md: 768,
  lg: 992,
  xl: 1200,
  xxl: 1400
}
```

---

### Methods

#### `getBreakpoint()`

Returns the name of the current breakpoint.

```js
viewport.getBreakpoint(); // 'md'
```

---

#### `getBreakpointValue()`

Returns the numeric index of the current breakpoint (1-based, ordered from smallest to largest).

```js
viewport.getBreakpointValue(); // 3  (for 'md' with default breakpoints)
```

Useful for range comparisons:

```js
if (viewport.getBreakpointValue() >= 4) {
  // lg and above
}
```

---

#### `isMobile()`

Returns `true` when the current breakpoint value is ≤ 2 (`xs`, `sm`).

```js
viewport.isMobile(); // true at 480px
```

---

#### `isTablet()`

Returns `true` when the current breakpoint value is exactly 3 (`md`).

```js
viewport.isTablet(); // true at 800px
```

---

#### `isDesktop()`

Returns `true` when the current breakpoint value is > 3 (`lg`, `xl`, `xxl`).

```js
viewport.isDesktop(); // true at 1024px
```

---

#### `getWidth()`

Returns the current `window.innerWidth` in pixels.

```js
viewport.getWidth(); // 1280
```

---

#### `getHeight()`

Returns the current `window.innerHeight` in pixels.

```js
viewport.getHeight(); // 800
```

---

#### `getDimensions()`

Returns an object with both `width` and `height`.

```js
viewport.getDimensions(); // { width: 1280, height: 800 }
```

---

#### `getBreakpoints()`

Returns a copy of the breakpoint definitions in use.

```js
viewport.getBreakpoints();
// { xs: 0, sm: 576, md: 768, lg: 992, xl: 1200, xxl: 1400 }
```

---

## Usage Examples

### Basic initialization and logging

```js
const viewport = new GetViewport();

console.log(viewport.getBreakpoint());      // 'lg'
console.log(viewport.getBreakpointValue()); // 4
console.log(viewport.isMobile());           // false
console.log(viewport.isDesktop());          // true
console.log(viewport.getDimensions());      // { width: 1024, height: 768 }
```

---

### Responding to resize events

```js
const viewport = new GetViewport();

window.addEventListener('resize', () => {
  if (viewport.isMobile()) {
    collapseNavigation();
  } else {
    expandNavigation();
  }
});
```

---

### Conditional feature loading

```js
const viewport = new GetViewport();

if (viewport.isDesktop()) {
  import('./desktop-charts.js').then(module => module.init());
}
```

---

### Debug table on load and resize

```js
const viewport = new GetViewport();

['load', 'resize'].forEach(event => {
  window.addEventListener(event, () => {
    console.clear();
    console.table({
      Breakpoint:      viewport.getBreakpoint(),
      BreakpointValue: viewport.getBreakpointValue(),
      IsMobile:        viewport.isMobile(),
      IsTablet:        viewport.isTablet(),
      IsDesktop:       viewport.isDesktop(),
      Width:           viewport.getWidth(),
      Height:          viewport.getHeight(),
    });
  });
});
```

---

### Custom breakpoints (Tailwind-style)

```js
const viewport = new GetViewport({
  sm:  640,
  md:  768,
  lg:  1024,
  xl:  1280,
  '2xl': 1536,
});

console.log(viewport.getBreakpoint()); // 'lg' at 1100px
```

---

### Range guards with `getBreakpointValue()`

When you need to target a range of breakpoints rather than a single named point:

```js
const viewport = new GetViewport();
const bp = viewport.getBreakpointValue();

if (bp >= 2 && bp <= 3) {
  // sm through md — small tablets and large phones
}
```

---

## Module Format Support

GetViewport uses a UMD wrapper and works in all three common module environments:

| Environment | How to use |
|---|---|
| Browser `<script>` tag | `window.GetViewport` is set automatically |
| CommonJS (`require`) | `const GetViewport = require('get-viewport')` |
| AMD (`define`) | Compatible via `define([], factory)` |
| ESM bundler (`import`) | Resolves to the CommonJS export via `package.json` `exports` |

---

## Limitations & Considerations

- **Browser-only** — depends on `document`, `window`, and `getComputedStyle`. Not suited for Node.js server-side rendering without a DOM shim (e.g., jsdom).
- **Single instance per breakpoint set** — if you need multiple independent breakpoint systems on one page, instantiate separately with different custom breakpoints.
- **CSS injection is global** — the injected `<style>` tag targets `:root`. If another library sets `--viewport` on `:root`, there will be a conflict.
- **`isMobile()` / `isTablet()` / `isDesktop()` are hardcoded to the default 6-breakpoint structure** — if you use custom breakpoints with a different count, use `getBreakpointValue()` directly for range comparisons.

---

## Contributing

Bug reports and pull requests are welcome at [github.com/peterbenoit/GetViewport/issues](https://github.com/peterbenoit/GetViewport/issues).

---

## License

MIT © Pete Benoit
