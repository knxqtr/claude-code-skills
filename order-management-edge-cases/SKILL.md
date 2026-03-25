---
name: order-management-edge-cases
description: Order lifecycle edge cases for trading systems. Use when writing code that places, monitors, cancels, or sizes orders on an exchange. Covers aggregate vs per-order position tracking, partial fills, cancel races, and stacking (multiple logical positions on one asset). Use when user says "wrong size", "aggregate position", "partial fill", "cancel race", or "stacking".
---

# Order Management Edge Cases

## When to Use

- Placing or monitoring orders on any exchange API
- Detecting fills via position changes or order status
- Multiple logical positions (strategies/triggers) trading the same asset
- Handling partial fills or cancel-with-partial scenarios
- Sizing exit orders based on fill data

## Aggregate vs Per-Order Position

Most exchange APIs return one aggregate position per asset, not per order. If two strategies both open on the same asset, `get_position("ETH")` returns the combined size. Using this aggregate for per-order decisions (fill size, exit order sizing) produces wrong results.

Track fills per order ID independently. Sources: WebSocket fill events (primary, includes order ID), order status API (secondary). Use the aggregate position only as a sanity confirmation, never as the source of truth for size or price.

When no per-order data is available AND other strategies have positions on the same asset, do not assume the aggregate is yours. Wait for per-order data or close as cancelled.

## Partial Fill Detection

A limit order can partially fill before being cancelled (by you, the exchange, or TP1 logic). The partial fill amount is not knowable from the aggregate position alone when stacking. Track cumulative fill size per order ID from WebSocket events. After cancelling, check tracked fills -- not the aggregate position.

## Exit Order Sizing

Exit orders (SL, TP1, TP2) must be sized for the specific fill, not the aggregate position. If strategy A filled 0.5 and strategy B filled 0.3 on the same asset, A's SL must be 0.5 and B's SL must be 0.3 -- not 0.8 for both.

reduce_only orders on most exchanges clip to the actual position size, so oversized exits won't open a reverse position. But they will interfere: one strategy's SL could close another strategy's position.

## Stacking Detection

Before using aggregate position as a fallback for fill detection, check if other active trades exist on the same asset. If they do, the aggregate is unreliable. Give WebSocket fill data time to arrive (2-3 poll cycles) before deciding.

## Order ID Replacement and Deferred Fills

When a fill callback replaces an order ID on a trade (e.g. TP1 handler cancels old SL and places a new BE stop), any queued fills for the old ID become unmatchable. This happens when fills are deferred -- network outage queuing, held fills during concurrent operations, or simply asyncio task scheduling.

Maintain a replaced-order-ID fallback map: `old_id -> (trade_id, order_type)`. When the main order lookup fails, check the fallback. Pop entries after use to prevent stale matches.

This applies whenever: (1) a fill callback mutates order tracking on a trade, AND (2) fills can be delivered out of band (queued, deferred, or concurrent). If both conditions are true, the mutation-before-lookup race exists.

## Common Mistakes

- Using `get_position(asset)` size as fill_size when multiple strategies trade the same asset.
- Using `get_position(asset)` entry_price as fill_price -- it's the weighted average of all entries, not yours.
- Assuming order gone = fully filled. Could be cancelled by exchange (margin call, liquidation).
- Not cleaning up per-order fill tracking after the monitor exits (memory leak for long-running bots).
- Sizing reduce_only exits for the aggregate position. They clip to actual size but interfere with other strategies' positions.
- Assuming order IDs are stable after fill callbacks. If a TP1 handler replaces the SL, queued fills for the old SL ID silently drop unless a fallback lookup exists.
