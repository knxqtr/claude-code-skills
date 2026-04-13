---
name: masked-contract-audit
description: Find code where defensive max/min/fallback arithmetic is hiding an undefined or inconsistent field contract, then replace that masking with one explicit writer/reader contract. Use when one field seems to have two meanings in different branches, when close paths use max(...) or fallback sums to stay green, or when review suspects a safety net is papering over a deeper invariant gap.
---

# Masked Contract Audit

Use this skill when arithmetic is correct only because fallback logic hides disagreement about what a stored field means.

The goal is to pin one contract, not just remove a helper.

## Workflow

1. Inventory all writers of the field.

- Record what each writer adds, excludes, or overwrites.
- Look for one outlier writer that forced the reader-side masking.

2. Inventory all readers and annotate their branch gates.

- Prove whether “entry-excluded” and “post-TP1” or similar paths are truly branch-exclusive.
- If an entry-excluded reader is reachable after the field already includes entry, stop and treat that as a production bug first.

3. Hand-check the arithmetic on at least two representative sites.

- Show that the chosen contract preserves the common-case totals before removing the mask.

4. Choose the narrowest consistent contract.

- Prefer the contract that already matches most writers and readers.
- If one write site is the outlier, change that writer instead of redesigning the whole lifecycle.

5. Pin the contract before removing the mask.

- Add or update a regression that states the contract in plain language.
- Add a field-level comment if the field meaning was previously implicit.

6. Remove the masking in the same commit as the contract change.

- Update the regression expectations only when they are part of the new contract.

7. Run the focused accounting tests and the full suite.

## Red Flags

- `max(field, entry_fee + tp1_fee)`
- `min(...)` or `or` fallback around a stored total
- comments like “safety net” or “defensive”
- one branch adds `entry_fee` while another assumes it does not

## Split Bug Fix From Cleanup When Needed

- If a new regression exposes an earlier transition-order or data-loss bug, fix that bug first in its own commit.
- Only remove the masking after the underlying bug is closed and reverified.
