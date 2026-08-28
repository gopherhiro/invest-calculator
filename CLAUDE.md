# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file static web app: a Simplified-Chinese (zh-CN) investment return calculator ("价算｜投资收益率计算器"). No build system, no dependencies, no `package.json`, no tests, and not a git repository. The entire app — HTML, CSS, and JS — lives in `index.html.html`. Keep new UI text in Simplified Chinese.

## Running and verifying

Open `index.html.html` directly in a browser (no server needed; the app uses no external resources):

```sh
open index.html.html
```

To verify a change, open the file in a browser and exercise the calculators manually — there is no test suite, linter, or build step.

## Architecture

Note the deliberate double extension: `index.html.html` is the entry point; don't create a separate `index.html`. `README.md` is empty.

The page has four calculator sections, all recomputed by the single `calc()` function:

1. **① 目标价格** — target price = buy price × (1 + rate). Rate comes from preset buttons (`setRate`) or the custom-rate input.
2. **② 快速目标价格表** — target-price table for fixed rates (5%–50%), re-rendered as HTML on every calc.
3. **③ 当前收益** — return rate and profit/loss = capital × (sell ÷ buy − 1).
4. **④ 亏损回本** — loss rate and the gain needed to recover to the buy price.

State model: there is no app state — the DOM inputs are the state. All five inputs (`buy`, `customRate`, `sell`, `capital`, `lossPrice`) share one `input` event listener that calls `calc()`, which re-reads every input and re-renders every output. Empty or invalid inputs render `—` instead of a number.

Helper conventions in the `<script>` block:

- `$` = `getElementById`; `n(id)` = parseFloat of an input's value.
- `fmt(x, dec)` formats a number, stripping trailing zeros; `money(x)` formats with a forced +/− sign and 2 decimals.
- Formulas on the page: R = P₁ ÷ P₀ − 1; 目标价格 = P₀ × (1 + R); 盈亏 = C × (P₁ ÷ P₀ − 1); 回本涨幅 = P₀ ÷ P₁ − 1.

Style: the file uses a compact, mostly one-line-per-rule CSS and dense HTML idiom — match it when editing rather than reformatting. Theme colors are CSS custom properties in `:root`.
