# Feature Roadmap & Verification Checklist

## Methodology

- One phase at a time. Do **not** start the next phase until the current phase's checklist is fully checked off and verified on the test matrix below.
- Every phase that touches the shared state (`all_lines`, `all_labels`, `all_levels`, `all_is_high`, `all_taken`, `all_created_bar`, `all_days_old`) must re-verify Phase 1 and Phase 2 still pass before being marked done — these arrays are shared across every later feature, so a bug introduced in Phase 4 can silently break Phase 1.
- "Verified" = checked against the test matrix below, not just "looked right on one chart."

## Test matrix (pick before starting Phase 1)

- [ ] Instrument 1: ________ (e.g. EURUSD), timeframe(s): ________
- [ ] Instrument 2: ________ (e.g. ES futures), timeframe(s): ________
- [ ] Instrument 3 (optional): ________ (e.g. BTCUSD — no real "session" concept, useful edge case)
- [ ] Confirm chart timezone vs. session timezone (`America/New_York`) assumption is understood and won't silently shift levels

## Phase 0 — Baseline (new, not in original list)

- [ ] Tag or branch the current "Decent first version working" commit as a known-good fallback before invasive changes begin
- [ ] Confirm rollback plan if a phase breaks something downstream

## Phase 1 — Level identification correctness

- [ ] Time-based liquidity (session highs/lows: London/NY/Tokyo/Sydney) match manual chart inspection across multiple days
- [ ] Event-based liquidity (news/data-release levels) — define the event source (manual input list vs. some external feed; Pine has no native economic calendar) before implementing
- [ ] Level expiration: levels are ignored/removed after a configurable time period
- [ ] Distance filter: levels too far from current price are ignored (define "too far" — % move or ATR multiple)
- [ ] No duplicate or missing levels across days
- [ ] No desync between internal arrays and chart objects when `max_lines_count` / `max_labels_count` (500) is hit
- [ ] Behavior verified across a DST transition and across a weekend/holiday (no session that day)

## Phase 2 — Line drawing

- [ ] Lines extend correctly to the right edge of the chart as new bars form
- [ ] Lines stop/cut off correctly when a level expires or is swept
- [ ] No repainting of historical line position (or, if some repaint is intentional — e.g. level only confirmed after session close — that's documented)

## Phase 3 — UI / styling

- [ ] Colors (per session, high vs. low, swept vs. active)
- [ ] Fonts / label size / style
- [ ] Line style, width, material
- [ ] All styling configurable via inputs, grouped logically
- [ ] Readable when many levels are on screen at once (label collision/overlap handled)

## Phase 4 — Sweep detection

- [ ] Define "swept" precisely: wick-through-and-reject (liquidity grab) vs. close-through (breakout/continuation) — pick one or support both
- [ ] Sweep detected on the correct bar, no repaint
- [ ] Swept levels visually distinguished (vs. current naive "taken" flag) and confirmed correct against the test matrix

## Phase 5 — Higher-timeframe gaps (FVG)

- [ ] Define HTF gap/FVG calculation (note: `request.security` is a common repaint trap — verify explicitly)
- [ ] HTF gap levels match manual inspection of the actual HTF chart
- [ ] Confluence handling: how HTF gaps interact with/display alongside liquidity levels

## Phase 6 — Reporting / dashboard

- [ ] Define dashboard contents (active levels, swept status, distance from price, age)
- [ ] Build via `table.*`
- [ ] Dashboard contents verified to match what's actually drawn on the chart (no mismatch)

## Phase 7 — Custom alerts

- [ ] Define alert triggers (level created, level swept, price approaching within X)
- [ ] Implement via `alertcondition()` / `alert()` with dynamic messages
- [ ] Alerts fire once per event (not once per tick/realtime update), verified on both historical and live bars

## Cross-cutting (check after every phase)

- [ ] Performance: compile time and runtime still acceptable as features accumulate
- [ ] Regression: earlier phases re-verified, since later phases mutate shared arrays
- [ ] Definitions documented (what "swept," "too far," "expired" mean) so dashboard/alerts in Phase 6-7 stay consistent with Phase 1-4 logic
