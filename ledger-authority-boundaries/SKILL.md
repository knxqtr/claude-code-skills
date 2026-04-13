---
name: ledger-authority-boundaries
description: Keep canonical ledgers and realized-PnL summaries limited to authoritative economics, and route degraded or non-economic events elsewhere. Use when trade logs, audit journals, or reporting readers may mix authoritative rows with degraded closes, cleanup rows, synthetic prices, or rows for trades that never had confirmed exposure.
---

# Ledger Authority Boundaries

Use this skill when a system has both canonical and degraded journals and the boundary between them is getting fuzzy.

The core question is not “should this be logged” but “is this row authoritative enough to enter canonical realized-PnL?”

## Workflow

1. Audit readers before writers.

- Identify every consumer of the canonical and degraded journals.
- Confirm whether a writer change is safe before touching routing.

2. Inventory writers with explicit columns.

- Record:
  - trade state or shape
  - position-backed status
  - current row shape
  - current routing
  - proposed routing
  - reachability

3. Separate cleanup from ledger emission.

- State hygiene may still need cancel, close, or cleanup work for non-economic siblings.
- That does not imply a canonical or degraded row should be written.

4. Test mixed and single-trade cases explicitly.

- Mixed cases usually expose the real bug.
- Single-trade cases protect the unchanged authoritative path from collateral drift.

5. Write the policy at the branch.

- Use one short comment at the continue or routing split.
- Record the cutover and operator-visible consequence in history.

## Canonical Row Standard

A canonical row should have:

- authoritative economic evidence
- no invented price or fee component
- no row for a trade that never had confirmed exposure

If any of those fail, default away from the canonical ledger.

## Common Failure Shapes

- authority computed from one subset, but logging loops over a broader set
- degraded rows entering canonical summaries by convenience
- risk booking already gated, but ledger write still ungated
