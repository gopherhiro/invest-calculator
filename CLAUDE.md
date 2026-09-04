# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single self-contained static HTML page: `index.html`. It's a Chinese-language investment return calculator ("投资收益计算器") — no build step, no dependencies, no package manager. All CSS and JavaScript are inline in the one file.

## Development

There is no build/lint/test tooling in this repo. To work on it:

- Edit `index.html` directly.
- Preview by opening the file directly in a browser (no server needed).
- There is no test suite; verify changes by exercising the calculator in a browser (enter values, click the rate preset buttons, check the three result cards update).

## Architecture

Everything lives in `index.html` in three parts:

1. **`<style>`** — design tokens as CSS custom properties on `:root` (`--bg`, `--card`, `--ink`, `--blue`, `--green`/`--red` for gain/loss semantics, etc.). Each card sets its own `--accent`/`--accent-rgb` (blue/teal/violet/amber) that inputs, headings, and focus rings inherit. Dark mode is a `@media(prefers-color-scheme:dark)` block that redefines the same tokens plus the `body::before` glow; two `theme-color` `<meta>` tags match. Layout is mobile-first with a `@media(min-width:1024px)` block that switches the page into a CSS-grid desktop layout (cards reflow from stacked to a 2-column grid, each card getting its own internal grid), plus a narrower-desktop tweak at `min-width:1024px and max-width:1150px` and a small-phone tweak at `max-width:390px`.
2. **HTML body** — five `.card` sections, each independent:
   - `.hero-card` ("目标价格") — buy price + target return rate → target price.
   - `.return-card` ("当前收益") — current/sell price + capital → current return rate and gain/loss amount.
   - `.quote-card` — a table of target prices for a fixed set of preset rates, populated by JS.
   - `.loss-card` ("亏损回本") — current price after a loss → loss rate and the rebound % needed to break even, plus a `.posbar` progress bar of the price's position relative to buy price.
   - `.formula-card` — static reference table of the four formulas used.
3. **`<script>`** — one `calc()` function reads all inputs by id (via the `$(id)` helper) and re-renders every derived value on every `input` event. There's no framework, no modules, and no separate state object — DOM input values *are* the state, and `calc()` is idempotent and re-run in full on any change. Preset rate buttons call `setRate(r)`, which sets the custom-rate input then calls `calc()`.
   - **Validation lives in `calc()`, not in markup.** Buy price must be `> 0`, or `calc()` marks `.buy-wrap` `.invalid`, shows "请输入买入价格", blanks every other card, and returns early. A negative custom rate is clamped to `0` (and the input rewritten). Missing sell/capital/loss inputs render the `—` sentinel rather than `NaN`.
   - **The preset rate list `[.05,.10,.15,.20,.25,.30,.50,.60]` is repeated three times** — the `.btnrow` buttons' `onclick`, the `rates` array in `calc()` (drives button `.sel` state), and the same array again for the `.quote-card` table. Changing the presets means editing all three, and the button count must stay a multiple of 3 for the grid.
   - Formatting helpers: `fmt()` (2-decimal `Intl.NumberFormat('zh-CN')`, `—` when not finite), `money()` (same but with a forced `+`/`−` sign). "Selected" preset matching uses `Math.abs(r-x)<0.005`.
   - Per-input wiring at the bottom of the script: the `['buy','customRate','sell','capital','lossPrice']` array attaches the `calc` and `has-val` listeners; the `.clear-btn` (×) handlers clear their sibling input and re-dispatch `input`. A new input id must be added to that array to become reactive.

Core formulas (also documented in the on-page "公式速查" table):
- Return rate: `R = P₁ / P₀ − 1`
- Target price: `P₁ = P₀ × (1 + R)`
- Gain/loss amount: `G = C × (P₁ / P₀ − 1)`
- Breakeven rebound needed after a loss: `X = P₀ / P₁ − 1`

When editing, preserve the existing conventions: terse minified-style CSS/JS (no formatting/build pipeline reformats it), `pos`/`neg` CSS classes for gain/loss color semantics, and IDs used directly as JS hooks (`$('id')`) rather than data attributes or classes.
