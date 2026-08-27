# Multi-track planning guide

Most trips need no tracks. Fetch this chapter only when the group truly splits
into separate parallel, named itineraries. If the same room or session appears
in multiple tracks, fan-out requires independently bookable physical units;
player capacity never authorizes another copy. A one-off sit-out is a player
exception, not a track, and different arrival days are not tracks.

## What a track means

A track is a named itinerary lane with its own roster and sequence. It is not a
second way to describe trip membership, a player-capacity bucket inside one
room, or an inferred subgroup that happens to emerge during planning. Different
tracks may schedule different games. When they schedule the same room or
session, each copy must have its own independently bookable physical unit.

Use a track when two or more groups need to move through separate itineraries
in parallel, perhaps with shared meals or regrouping points between those
stretches. Two groups booking four places each in one eight-player room are
still one physical booking unit, not two copies. Fan-out of the same room or
session is allowed only for independently bookable physical units. Player
capacity never authorizes a second scheduled copy.

Keep track names stable and meaningful (for example, `Morning route` and
`Afternoon route`). Create the named tracks before laying out their items. A
new track starts with a pending roster, so it is valid to build and evaluate
its itinerary before deciding exactly who will be in it. Do not invent
temporary UUIDs or silently turn an observed spreadsheet subgroup into a
persisted track.

## Athens-style workflow

For a longer trip with parallel routes:

1. Create the named tracks. Leave their rosters pending if the group will make
   the assignments later.
2. Schedule games and activities into one track at a time. Use the track's
   roster version for each track-aware write. Use one track for a bulk game
   assignment; use separate calls for items that belong to different tracks.
3. Evaluate each draft and physical booking unit. Check the participant and
   track travel legs, endpoint slack, unresolved travel status, same-unit
   claims, concurrent membership, and player capacity separately. A room that
   supports eight players is not evidence that two four-person copies exist.
4. Add shared items without a track when everyone should attend. A track-tagged
   item uses that track's baseline. A one-off change on a tagged item is an
   item-level exception, not a new track.
5. When the group is ready, replace each pending baseline with the actual
   eligible full-member roster. Re-evaluate after each roster change. Read the
   consequence envelope and surface affected booked items, mail, costs, and
   unresolved reconciliation instead of claiming that the change was merely a
   label edit.

The roster is ground truth. Omitted players on a default item mean the eligible
whole-group default; an explicit empty roster means nobody attends. A pending
track is not an empty track with an active interpretation. Guests can attend
through the existing item invitation flow, but do not put guests into the
track baseline. Linked games in a target trip are read-only lenses: they may
carry a target-local track label, but never import source-trip people, costs,
booking units, or authorization into the target.

## Evaluate before committing

Use `evaluate_schedule` with top-level candidate-local tracks when exploring a
new layout. Give every candidate track an opaque `trackKey`, a name, an
optional positive `targetSize`, `rosterMode`, and explicit `participantIds`.
Items refer to that declaration by `trackKey`. Do not use a fake UUID for a
candidate track, reference a track by persisted `trackId` from an item, or send
both `playerIds` and `trackKey` on one item. Candidate evaluation is read-only.

The detailed result is participant-carried. Prefer `participantEdges`,
`participantTotals`, `trackEdges`, and `trackTotals` over reconstructing lanes
from array order. The scalar verdict is conservative across carriers. Pending,
unavailable, and no-route travel statuses remain distinct and unresolved travel
does not become a conflict. Inspect `physicalSession` and booking-unit status
per scheduled copy; do not aggregate player capacity into copy availability.

## Fairness is scratch analysis

Skeleton Key does not persist a fairness score, heat map, or threshold. If the
group wants to balance pairwise co-play, four-person versus five-person teams,
or another trip-specific objective, read the rosters and lengths from
`list_days` and the evaluated edges/totals from `evaluate_schedule`. Compute
the metric in agent scratch work, explain the tradeoff, and optionally record
the decision in a Discussion post. Keep the optimization separate from the
track model and from the worksheet UI.

## When not to use tracks

Do not create tracks merely because some people skip one game, because two
people arrive on different days, because a room has a player minimum or
maximum, or because the planner wants to compare two hypothetical orders. Use
the existing player toggle/exception, arrival and departure endpoints, or a
read-only draft evaluation instead. When the group no longer has genuinely
parallel named itineraries, untag items and preserve explicit rosters rather
than leaving empty scaffolding behind.
