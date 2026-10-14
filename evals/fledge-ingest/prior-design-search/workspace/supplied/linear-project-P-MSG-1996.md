# Linear project P-MSG-1996 — Back-to-back reservation handling
Lead: Jordan Avery · Team: Messaging · Target: 2026-08-28 · Last edited: 2026-08-17

## Description
Guests who check out of one reservation and straight into another at the same hotel
("back-to-back") currently get every Guest Journey message twice: two pre-arrival
messages, two check-in links, two post-stay surveys. Send each Guest Journey message once
across consecutive reservations.

MVP: detect pairs of consecutive reservations (same guest, departure date == arrival
date of the next), designate the first one the owner, and skip scheduling messages for
the second one.

## Milestones
- M1 pilot (2026-08-28): pre-arrival and check-in messages deduplicated for pairs.

## Links
- Design doc: notion page `b2b-reservation-design` (supplied separately)
