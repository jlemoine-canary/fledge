# Back-to-back reservation design (Notion) — last edited 2026-08-17
Author: Jordan Avery

## Problem
Duplicate Guest Journey messages for consecutive stays.

## Proposal
At scheduling time, find the reservation's back-to-back partner. If the reservation is
the second of a pair, do not schedule pre-arrival or check-in messages for it.

## Open
- Do longer chains (3+) matter? Probably rare; skip for MVP.
