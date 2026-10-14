# Linear project P-MSG-1996 — Back-to-back reservation handling
*Description last edited 2026-09-01 by Jordan Avery*

## Description
Send each Guest Journey message once across a back-to-back chain. Every message is still
scheduled for every leg; when one is about to go out, it is suppressed if the chain's owner
sends that message type.

- Chains of any length.
- All Guest Journey message types.
- Mid-stay messages are timed against the chain's span.
- Depends on MSG-6119 (chain selector).

## Milestones
- M1 pilot (2026-09-11)
