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

1. **`<style>`** — design tokens as CSS custom properties on `:root` (`--bg`, `--card`, `--ink`, `--blue`, `--green`/`--red` for gain/loss semantics, etc.). Layout is mobile-first with a `@media(min-width:1024px)` block that switches the page into a CSS-grid desktop layout (cards reflow from stacked to a 2-column grid), plus a narrower-desktop tweak at `max-width:1150px`.
2. **HTML body** — four `.card` sections, each independent:
   - `.hero-card` ("目标价格") — buy price + target return rate → target price.
   - `.quote-card` — a table of target prices for a fixed set of preset rates, populated by JS.
   - `.return-card` ("当前收益") — current/sell price + capital → current return rate and gain/loss amount.
   - `.loss-card` ("亏损回本") — current price after a loss → loss rate and the rebound % needed to break even.
   - `.formula-card` — static reference table of the four formulas used.
3. **`<script>`** — one `calc()` function reads all inputs by id (via the `$(id)` helper) and re-renders every derived value on every `input` event. There's no framework, no modules, and no separate state object — DOM input values *are* the state, and `calc()` is idempotent and re-run in full on any change. Preset rate buttons call `setRate(r)`, which sets the custom-rate input then calls `calc()`.

Core formulas (also documented in the on-page "公式速查" table):
- Return rate: `R = P₁ / P₀ − 1`
- Target price: `P₁ = P₀ × (1 + R)`
- Gain/loss amount: `G = C × (P₁ / P₀ − 1)`
- Breakeven rebound needed after a loss: `X = P₀ / P₁ − 1`

When editing, preserve the existing conventions: terse minified-style CSS/JS (no formatting/build pipeline reformats it), `pos`/`neg` CSS classes for gain/loss color semantics, and IDs used directly as JS hooks (`$('id')`) rather than data attributes or classes.
