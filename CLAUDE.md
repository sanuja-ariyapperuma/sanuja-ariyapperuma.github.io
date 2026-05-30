# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository shape

A single-file static portfolio site. The only checked-in source is `Sanuja - Profile.html` (note the spaces in the filename — quote it on the CLI). There is no package manager, no build step, no test runner, and no git history in this directory.

## Running / previewing

There is no build pipeline. To view changes, open the HTML file in a browser. Prefer serving over a local HTTP server rather than opening via `file://`, because the React tweaks panel loads an external `.jsx` via `<script type="text/babel" src="tweaks-panel.jsx">`, which some browsers refuse to fetch from `file://`:

```sh
python3 -m http.server 8000
# then open http://localhost:8000/Sanuja%20-%20Profile.html
```

## Architecture

The page is a single document with two co-existing runtimes:

1. **Vanilla-JS main page** (inline `<script>` block, lines ~283–388). Owns the canvas node-network animation (`#net`), scroll-progress bar, reveal-on-scroll, tilt cards, and counter animation. Exposes a single global, `window.applyTweaks(t)`, which is the integration point: it mutates `:root` CSS custom properties (`--blue`, `--cyan`, `--glow-rgb`, …) and the `NET` config object so the animation re-tunes live.

2. **React island** (UMD React 18 + Babel Standalone loaded from `unpkg`, mounted into `#tweak-root` at the bottom of the body). It renders a `TweaksPanel` UI that calls `window.applyTweaks` whenever its state changes. The panel's components (`TweaksPanel`, `TweakSection`, `TweakColor`, `TweakRadio`, `TweakToggle`, `useTweaks`) are expected to come from a sibling file **`tweaks-panel.jsx`** that is referenced but not present in this directory. Without it, the panel mount silently fails — the page still works, the defaults still apply (the inline script calls `applyTweaks(TWEAK_DEFAULTS)` once on load), but no UI appears.

The contract between the two runtimes is just `window.applyTweaks` and the shape of the tweak object (`{ accent: [primary, highlight], motion: "Calm"|"Balanced"|"High", network: boolean }`). Keep that shape stable when editing either side.

### Sentinel markers — do not strip

In the inline React script there is a block:

```js
const TWEAK_DEFAULTS = /*EDITMODE-BEGIN*/{ … }/*EDITMODE-END*/;
```

The `EDITMODE-BEGIN` / `EDITMODE-END` comment pair brackets a JSON-shaped literal so an external tool can find and replace the defaults in place. Preserve the markers and keep the inner content valid JSON (double-quoted keys, no trailing commas) so that round-trip continues to work.

### Missing / dangling references

The HTML also links to `index.html` (the brand mark in the nav, and a "compare directions" link near the footer). That file isn't in this directory either. If a task involves the nav brand or the compare link, confirm with the user whether `index.html` is expected to live alongside this file or whether those links should be changed.
