# Slack — #msg-eng — 2026-07-21 — thread started by Sam Ortiz

**Sam Ortiz** 14:02
Dumping my notes on the consecutive-stays duplicate-message problem since it keeps
coming up in support tickets. Not planning to work on it this quarter.

1. Don't try to decide at scheduling time. Reservations change after scheduling — legs
   get added, cancelled, dates move. Anything decided at schedule time goes stale.
2. Instead: schedule every message for every reservation exactly as today, and filter
   at send time. When a message is about to go out, look at the chain the reservation
   is in *right now* and suppress it if another leg already sent (or will send) the
   same message type.
3. Chains can be longer than two. Group bookings and extensions produce 3–5 leg chains
   regularly at long-stay properties.
4. Mid-stay messages need retiming against the chain's span, not each leg's.

**Priya Nair** 14:20
+1 on filter-at-send. We tried pre-scheduling dedupe in March (MSG-4410) and it broke
as soon as a chain grew after messages were already scheduled.

**Sam Ortiz** 14:22
Right, that's the ticket. Linking for whoever picks this up.
