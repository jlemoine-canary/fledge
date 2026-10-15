# Linear project P-MSG-1996 — Back-to-back reservation handling
*Description last edited 2026-09-01 by Jordan Avery*

## Description
Send each Guest Journey message once across a back-to-back chain.

**Redesign (2026-08-31):** we're no longer deciding at scheduling time. Schedule every message for
every leg as today, and filter at send: when a message is about to go out, resolve the chain the
reservation is in *now* and suppress the message if the chain's owner sends that message type.

- Chains of any length, not just pairs.
- All Guest Journey message types, not just pre-arrival and check-in.
- Mid-stay messages are retimed against the chain's span.
- New prerequisite ticket MSG-6119 (chain selector) added 2026-09-01.

## Milestones
- M1 pilot (2026-09-11)
