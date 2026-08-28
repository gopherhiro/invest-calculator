# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file static web app: a Simplified-Chinese (zh-CN) investment return calculator. No build system, no dependencies, no `package.json`, no tests. The entire app — HTML, CSS, and JS — lives in `index.html` (renamed from `index.html.html` by the user on 2026-08-28). Keep new UI text in Simplified Chinese. The header is: blue tagline "简单 · 快速 · 计算投资收益" (`.brand .tag`), with the h1 as a small muted-gray regular-weight subtitle; there is no footer (removed at user request).

## Running and verifying

Open `index.html` directly in a browser (no server needed; the app uses no external resources):

```sh
open index.html
```

To verify a change, open the file in a browser and exercise the calculators manually — there is no test suite, linter, or build step.

## Architecture

`README.md` is empty. A git repo was initialized by the user (2026-08-28); follow git conventions if working with it.

The page has four calculator sections, all recomputed by the single `calc()` function:

1. **目标价格** — target price = buy price × (1 + rate). Rate comes from preset buttons (`setRate`) or the custom-rate input. The result renders in the "hero" answer block (`#targetPrice`), the page's signature visual element.
2. **快速目标价格** — target-price table for fixed rates (5%–50%), rendered by `calc()` as `.quote-row` rows (like a market quote table). The row matching the current target rate gets a `.hot` highlight class.
3. **当前收益** — return rate and profit/loss = capital × (sell ÷ buy − 1).
4. **亏损回本** — loss rate and the gain needed to recover to the buy price, plus a price-position bar (`#posBar`/`#posFill`): a track from 0 to buy price whose fill shows where the current price sits.

State model: there is no app state — the DOM inputs are the state. All five inputs (`buy`, `customRate`, `sell`, `capital`, `lossPrice`) share one `input` event listener that calls `calc()`, which re-reads every input and re-renders every output. Empty or invalid inputs render `—` instead of a number.

Helper conventions in the `<script>` block:

- `$` = `getElementById`; `n(id)` = parseFloat of an input's value.
- `fmt(x, dec)` formats a number, stripping trailing zeros; `money(x)` formats with a forced +/− sign and 2 decimals.
- Formulas on the page: R = P₁ ÷ P₀ − 1; 目标价格 = P₀ × (1 + R); 盈亏 = C × (P₁ ÷ P₀ − 1); 回本涨幅 = P₀ ÷ P₁ − 1.

Style: the file uses a compact, mostly one-line-per-rule CSS and dense HTML idiom — match it when editing rather than reformatting.

Responsive layout (2026-08-28, goal: every section visible on one desktop screen, no scrolling):
- **≥1024px**: 2×2 grid on `.wrap` (`grid-template-columns:1.1fr 1fr`, `align-items:stretch`): row 1 = `hero-card` (目标价格 — internal grid `30% 1fr`: the 30% left column holds 买入价格 input, 自定义收益率 input, 目标收益率 label + 3×2 button pad, all at the same 30% width; the right side is the hero answer with a left hairline divider, top-to-bottom reading order inside the column) + `quote-card` (快速目标价格, 6 big chips filling the card, 2 rows of 3; rate text is green with NO background pill, prices are ink-black); row 2 = `return-card` + `loss-card`, both following the "conditions on top, results below" pattern with a hairline divider. Results in both cards are side-by-side big-number tiles (32px bold, sign-colored green/red) with small labels above, per the user's stock-app reference (1.jpg): return-card pairs 卖出/当前价+投入本金 inputs on one row above 收益率/盈亏金额 tiles; loss-card has 30%-width input+posbar above 当前亏损率/回本涨幅 tiles. At the very bottom of the page (user moved it back down from the top) sits the formula reference, which is NOT a card — `formula-card` is transparent/borderless and left-aligned. Desktop layout is an explicit two-column grid: `h2` 公式速查 spans the full width on its own row, with `.sym-row` (the five symbol definitions, one per line, formatted `符号：释义` where `.sym b` is non-bold mono holding symbol+colon at `min-width:2.5em` so definitions align) in column 1 and `.formula-grid` (the four formulas stacked, left side in `.f-l` with `min-width:6em` so equal signs align, `.f-eq` has a half-space margin) in column 2. On mobile everything stacks as sequential blocks.
- **1024–1150px**: tighter values (h1 18px, smaller inputs/buttons, gap 6px) to keep the page within 768px height.
- **768–1023px**: 2-column grid (cards pair side by side).
- **<768px**: single stacked column with roomier mobile spacing; desktop internal grids do not apply.
- Height is deliberately tuned so `1024×768` and `1280×800` viewports fit; when adding content, keep an eye on total height.

Design system: modeled on the ui-ux-pro-max official demo "Vestia" (https://www.uupm.cc/demo/investment-platform), applied 2026-08-28 as the user's reference after several rejected directions:

- **Aurora gradient background** (user chose to keep it: `--bg: #eef2ff` + `body::before` with three fixed radial gradients and a 14s `drift` animation) + **simple pure-white cards** (user's final choice: `--card: #fff`, 16px radius, soft `--shadow`) — the dark-slate card skin was superseded and removed.
- Single light context, no card-level token overrides. Ink `#0f172a`, muted `#64748b` (slate-600), brand/navy `#1e293b`, interactive blue `#2563eb`, green `#10b981` (bar) / `#047857` (positive text), red `#b91c1c` (negative text).
- The hero number (目标价格 answer) is plain ink-black (`--ink`), per user request — the gradient and pos/neg coloring for it were removed; the target-rate label (`.unit`) still uses green/red by sign.
- Semantic color rules are `.metric .v.pos / .row strong.pos` etc. — specificity is deliberately higher than `.metric .v` because that rule sets `color: var(--ink)`.
- No dark mode and no glassmorphism — those directions were superseded; do not reintroduce them.
- Interaction polish (added with the reference-site pass): the active rate preset button gets `.sel` (blue fill), the hero number plays a `.tick` animation when its value changes (JS re-triggers the class in `calc()`), inputs have a hover border lighten. The `prefers-reduced-motion` block also disables `body::before` animation explicitly (`*` doesn't match pseudo-elements).
- Green = profit/up, red = loss/down (user-confirmed convention "涨绿跌红", 2026-08-28). The `+`/`−` text sign is always rendered alongside the color, so color is never the only indicator.
- All number displays follow the convention: hero target price and the rate label get `.pos`/`.neg` classes from `calc()` (rate > 0 green, < 0 red; the blue gradient text only shows when rate = 0), quote-table prices and the active-row highlight are green-themed, 收益率/盈亏/亏损回本 values use `.pos`/`.neg`. CSS overrides for `.hero-num.pos/.neg` cancel the gradient via `-webkit-text-fill-color:currentColor` and must stay after the `@supports` block.
- Light-only stands as a user aesthetic preference (dark was rejected twice; do not reintroduce `prefers-color-scheme` without asking).
- Accessibility floor: text contrast ≥ 4.5:1 on both card (`#fff`) and page (`#f8fafc`) surfaces; touch targets ≥ 44px tall.
- All numbers (inputs, prices, rates, metrics) use the `--num` monospace stack with `tabular-nums`; Chinese text uses the system sans stack. This pairing is deliberate.
- JS contains two presentation-only bits beyond the logic: the `#targets` innerHTML template (`.quote-row` markup) and the `#posFill` width + `#posBar.on` class toggle. Keep them in sync with their CSS.
- Respect `prefers-reduced-motion` and visible `focus-visible` styles when adding interaction.
