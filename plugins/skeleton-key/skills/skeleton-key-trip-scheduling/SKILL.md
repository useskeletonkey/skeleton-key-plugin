---
name: skeleton-key-trip-scheduling
description: >-
  Use when building or refining the schedule of an escape-room trip — which rooms go on which days,
  daily intensity, meals and travel buffers, geographic clustering, booking order, arrival/departure
  days, multi-city routing, lodging moves, rest or tourism days, day-trips, or scheduling around an
  anchor event or convention (e.g. an organized escape-room tour) — and whenever operating the
  Skeleton Key trip-planner tools (schedule_game, book_game, evaluate_schedule, etc.). Trigger even
  on short asks like "plan my escape room trip," "build the schedule," or "what should each day
  look like."
---

# Skeleton Key Trip Scheduling

Arrange a pool of escape rooms into a day-by-day itinerary the way experienced escape-room trip
planners do it. This skill covers **scheduling**: shaping days, placing meals and travel,
sequencing bookings, arrival/departure, lodging, rest days, and teams. It does **not** pick which
rooms are worth playing — if the user has no pool yet, help them search and add rooms
(`search_escape_rooms`, `add_game`), but the picks and priorities are theirs; this skill takes
over once a prioritized pool exists.

Two principles frame everything below:

1. **Schedules are personal — never assume a preference.** How intense a day should be, how meals
   work, how late is too late, how much sightseeing belongs: these differ enormously between
   users, and a technically-perfect schedule built on assumed preferences is a failed schedule.
   Elicit the trip profile (below) before proposing days. The small set of rules that are NOT
   preferences is under "Fixed methodology."
2. **When soft preferences collide and rooms win, name what was sacrificed** and what it would
   cost to get it back. The finite days force real tradeoffs (tourism vs. an extra top-priority
   room, a lighter day vs. a fourth game). Surface them as explicit flags or questions — never
   decide silently.

## Workflow

1. **Check tools.** If the Skeleton Key tools aren't available, see "If the tools aren't
   connected" — do not fake the full workflow without them.
2. **Check the dates.** Compare the trip's `endDate` from `list_trips` against today's date. If
   the trip is already over, this is recording, not planning — jump to "Recording a trip that
   already happened" and skip steps 3–8.
3. **Gather trip facts** (Step 1 below).
4. **Establish the trip profile** (Step 2) — recall a saved one or run the intake. Do this
   BEFORE drafting any schedule, even a "rough" one.
5. **Draft days**: profile dials + fixed methodology.
6. **Verify** every multi-room or day-trip day with `evaluate_schedule`; fix or flag, never fudge.
7. **Present** with per-time provenance labels, tradeoff flags, and open questions.
8. **Booking order**, when it comes up: priority × scarcity, after asking the booking-risk
   question (a just-in-time question, not part of intake).

### Multi-track gate

Use tracks if and only if the group splits into separate parallel named
itineraries. If the same room or session appears in multiple tracks, fan-out
requires independently bookable physical units; player capacity never
authorizes another copy. A one-off sit-out uses `toggle_player`, different
arrival days are not tracks, and most trips should have zero tracks.
When this test passes, read the
[multi-track planning guide](references/multi-track-planning-guide.md) (or
fetch the `multi_track_planning_guide` tool chapter) before creating or
evaluating tracks.

## Step 1 — Trip facts

Facts about this trip, not preferences about the user. Collect before scheduling; never persist
to memory.

1. **Anchor event?** Organized tour or convention (e.g. a Room Escape Artist tour)? Reserve its
   dates as fixed and empty — schedule around them, never into them.
2. **Arrival & departure**: times and delay risk (see the arrival rule for what raises risk).
3. **Region geography**: which venues cluster into which areas; travel times between them.
4. **Venue operating hours** — they set the earliest feasible start each day; a constraint, not
   a preference. But they are the *weakest* source in this list: escape-room hours are sparse and
   stale on Google, especially outside the US, and a venue will often open a listed-closed day for
   a booked group. Slot data for a specific date always outranks them, and a listed-closed day with
   no slot data is a "confirm with the venue" flag, never a reason to rule the day out. This is why
   `evaluate_schedule` reports hours as `openingHoursAdvisories` and never scores a day infeasible
   on them.
5. **Per-room priority and popularity** — needed for placement and booking order. Priority is
   the user's call; `bookingPressure` on `list_games` is the data-driven popularity/scarcity read
   that feeds booking order (see Booking order).
6. **Game comments** — read them for every game with `commentCount > 0` (see Comments rule).

## Step 2 — The trip profile

The profile is a small set of dials that parameterize every scheduling decision. It is the
contract the schedule is built against.

**Recall first.** If this environment has a memory facility and it holds a saved Skeleton Key
trip profile, present that profile and ask whether it still applies — discuss only the deltas.
Otherwise run the intake.

**Intake: ONE batched message, not an interrogation.** Ask the following together, with the
default in brackets after each question so a partial answer works ("Balanced, defaults fine" is a
complete answer). Don't proceed to a schedule until the user has answered at least the day-shape
question.

1. **Day shape.** Offer these four, described exactly like this — the descriptions are the
   load-bearing part, the names are just mnemonics. Ask which day the user would actually want to
   live, and say "not sure" is fine (treat as Balanced and adjust as real days take shape):
   - **Sampler** — 2 rooms (~2–3 h inside games). Late start, a real sit-down lunch and dinner,
     evenings free. Trip has plenty of non-game time by design.
   - **Balanced** — 3 rooms (~3.5–4.5 h). First room late morning, unhurried meals (~75–90 min),
     usually done by ~9 pm. *(Default.)*
   - **Enthusiast** — 3–5 rooms (~4.5–6 h). The day is built around the room grid; meals fill the
     gaps bookings leave (75-min floor); an occasional late finish, balanced by a lighter
     neighboring day.
   - **Marathon** — 4–6 rooms (~6–7 h). Every feasible slot considered, meals compressed into
     gaps, late finishes routine. Suits short blitzes and single-venue back-to-back days.
2. **What makes a day feel like too much?** Total hours in games / number of rooms / finishing
   late / not enough downtime between things. [default: total hours] This calibrates which
   warnings matter for this user.
3. **Meals.** Planned sit-down blocks, or grab-and-go between rooms? Minimum comfortable meal
   length? Any destination-dining ambitions? [default: real blocks, 75-min floor]
4. **Getting around.** Renting a car, willing to rent just for specific days (e.g. a rural
   day-trip), or transit + rideshare only? No default — always ask; it changes which day-trips
   and venue chains are feasible.
5. **Group.** How many players travel as the base team? Are extra players ever recruitable
   (friends, event attendees), or is the base team fixed? [default: the trip's member count, no
   recruiting]
6. **Rest & tourism.** None, half-days, or full days? Should it scale with how far from home the
   destination is? [default: a half-day on trips of 4+ days, more when the destination is
   far-flung or unusual]

Late-night tolerance is implied by the chosen day shape; treat it as adjustable when a real late
finish shows up. Booking risk is deliberately NOT asked here — ask it just-in-time when booking
order first comes up, with the actual rooms on the table.

**Emit the profile block** after intake, ask the user to confirm, and keep it visible in
proposals:

```
Trip profile
- Day shape: Balanced (~3.5–4.5 h in games, 3 rooms/day typical)
- "Too much" means: finishing late
- Meals: planned blocks, 75-min floor; one nice dinner if it fits
- Getting around: transit + rideshare only
- Group: 2, open to recruiting for specific games
- Rest & tourism: one museum half-day
```

**Offer (never force) to save the confirmed profile to memory** so future trips can skip the
intake. When a just-in-time answer reveals a durable preference, update the block and re-offer
the save.

## Just-in-time questions

When a drafting decision isn't answered by the profile, ask — with the concrete situation in the
question — rather than assuming. Classify each answer: **one-off** ("keep it just this day")
stays local; **durable** ("we never do late nights") updates the profile.

| Situation | Ask |
|---|---|
| A day trends past the profile band and real rooms still fit | "Day 3 hits 6.5 h if we add X — keep or trim?" |
| A late finish, or two late nights adjacent | "This ends ~11:30 pm, and so does tomorrow — OK, or restructure?" |
| Slot grid squeezes a meal below the user's floor | "Real slots leave only 45 min for dinner — eat fast, or drop/move a room?" |
| A dense slate squeezes out tourism entirely | "No sightseeing fits this trip; trading [specific room] buys a half-day — worth it?" |
| A room's player minimum exceeds the group | "Needs 4 — recruit, pay for empty slots, or drop?" |
| First booking discussion | The booking-risk question: lock everything early, or book the `high`/`medium`-pressure ones and gamble on the `low`-pressure rest? |
| Arrival day with meaningful delay risk | "Booking anything that evening risks the fee if you're delayed — book or keep it free?" |
| A light day, especially all-projected rooms | "This day is light and nothing on it is bookable yet — make it the tourism half-day and cluster its rooms to one end?" |

## Fixed methodology

These are correctness and craft rules, not preferences. They hold for every user; the profile
does not override them.

### Budget days by time inside games, not room count

Total game/experience time is what makes a day exhausting or not — the profile's band is in
hours for that reason. Long rooms substitute for short ones: three long rooms can outweigh five
short ones; count each experience's full duration, immersive/large-format included. Room count
is a consequence, not the target. Schedule long/immersive experiences within normal days rather
than giving them a dedicated lighter day. Stack 5–6 rooms only at a single venue that supports
back-to-back plays — never spread that many across scattered venues; travel kills it.

The band stretches for a top-priority room — a day exceeding the band to fit one more
top-priority game is usually right, but flag the stretch and let the user call it. Balance a
marathon or late-finish day with a lighter neighbor, and never schedule late finishes
back-to-back unless the profile's day shape treats them as routine.

### Pin only real times

The highest-stakes rule here: a schedule that looks bookable but isn't is worse than one that
honestly shows rough edges.

- Every pinned time must be an actual slot returned by `list_game_slots` (a provider-imported or
  manually-entered slot), or a projected estimate labeled as such. **Never invent, round, or
  nudge a time to make a day flow better.** If the real slots are 10:00 / 12:15 / 14:30, you may
  pin one of those and nothing else.
- **No slot data at all** (venue not synced, no manual times): do not put a precise clock time in
  the itinerary — labeling a fabricated time "estimated" doesn't make it real. Give a target
  *window* derived from the venue's operating hours ("Friday evening — venue is open 18:00–22:00;
  confirm directly") and flag it as unverified. When the day needs a concrete block to verify or
  to protect the space, use a clearly-labeled **hold** inside that window ("HOLD — window, not a
  slot; contact venue") — a hold reserves room in the day; it is never presented as a bookable
  time.
- **A slot after midnight is a real slot.** Athens venues in particular run a nightly
  17:00 / 19:30 / 22:00 / 00:30 pattern and sell that last one as part of the evening
  ("00:30 +1"). Pin it on the day whose evening it belongs to, with `dayOffset: 1` — a 00:30
  session ending Friday's outing is `time: "00:30", dayOffset: 1` on **Friday**. Putting it on
  Saturday instead is the mistake to avoid: it lands before Saturday's breakfast, and the drive
  gets measured from the hotel rather than from the venue the group is actually leaving.
- Anything outside Skeleton Key's data (train schedules, flight times the user hasn't provided)
  is a guess — present it as "verify against the carrier," never as a settled time.
- Real slots are lumpy. They leave awkward midday gaps, force late finishes, fail to chain
  neatly. That IS the schedule — surface the gap or the tight transfer and flag it. An honest
  "these four rooms only fit as 10:00, 12:15, 16:00, 19:00 — a 2.5 h midday gap and a ~21:30
  finish" is correct; a tidy fake is not.

### Verify every multi-room and day-trip day

`evaluate_schedule` is a stateless simulation — it runs on a scratch copy and persists nothing,
so it is always safe to call, including in read-only contexts. Use it on every multi-room or
day-trip day before presenting. Check: each start is a real slot; previous room's end + travel +
buffer ≤ next start; the day's own transport (drive/train each way) is a real block, not
ignored. Treat any verdict other than `feasible` / `feasible_provisional` as a day you must fix
(move to other real slots or drop a room and flag it) — never fudge. On `indeterminate`
(uncached travel legs), call `warm_travel` on the same draft and re-evaluate — once. Transit
legs cannot be warmed, so when `warm_travel` comes back reporting those legs in `unwarmable` /
`unwarmablePairs`, the verdict will stay `indeterminate` no matter how many times you re-run it:
accept it, present the day with its transit legs flagged as unverified, and move on. Never loop
warm-then-evaluate on legs the tool already told you it can't warm. It does not check player
minimums — that arithmetic is yours.

### Meals are schedule items, not leftovers

On full days, place lunch and dinner as explicit blocks in the proposal (and as `add_activity`
items when writing to the trip) — a day of games with no meals in it is a bug, not a lean
schedule, unless the profile explicitly says grab-and-go. Meal length is a residual of the
day's fixed slots — let it stretch to absorb slack — but keep the profile's floor. Place each
meal where the user will physically be, in a town or neighborhood with real options along the
day's route; this matters most on drive-heavy days through rural areas. Give a meal its own
extended block only for genuine destination dining.

### Geography and travel

- A room's real location is the source of truth — resolve each room to its actual venue address
  or coordinates, not its cluster tag or booking-page URL (a "city" booking page can sell a room
  an hour outside the city). If cluster and address disagree, trust the address.
- Cluster each day's rooms geographically. On travel-heavy days build the day as an explicit
  chain: travel → room → meal-in-town → travel → room. Inter-city transport gets its own block.
- Default 15-min buffer between venues on top of travel time (the trip's `travel_buffer_minutes`
  setting; adjust via `update_trip` if the user wants more slack).
- Respect the profile's getting-around answer: travel mode is per leg in Skeleton Key
  (`drive | walk | cycle | transit`), so set each scheduled leg's mode to match — transit-only
  users get `transit`/`walk` legs (model rideshare as `drive` where transit can't work, and say
  so), and day-trips that require a car get flagged when the user isn't renting one.
- Keep a geographic bench of wanted-but-unbooked rooms, grouped by area, ready to slot into
  whichever day passes through.

### Priorities fill scarce slots

Days are the scarce resource. Fill them highest-priority-first: don't burn a slot on a
lower-priority room while a reachable top-priority room goes unscheduled. Actively place every
confirmed top-priority room that has open slots — do the real slot arithmetic and pin a concrete
time; don't leave a schedulable top-priority room as a vague "optional / if time" mention.
Cutting a distant, low-availability cluster entirely is reasonable; leaving a nearby
top-priority room off the schedule in favor of lesser ones is not. If a top-priority room truly
can't be placed (availability, geography, players), surface a flag rather than quietly dropping
it.

### Confirmed vs. projected — strictly from the slot's source

`source: "projected"` (and provisional slots) are read-only estimates for future dates the venue
hasn't released — never generated for a date that has already passed, though one written while
the date was still ahead can linger on a trip nothing refreshes any more. A provider name or
`manual` is a real, bookable slot. Never infer "confirmed" from times merely appearing. Label
every proposed time confirmed vs. projected. Projected rooms can be *planned* (place them, hold
the space) but not *booked* — "not yet released" is a hard gate on booking order, independent of
priority: book the confirmed scarce rooms now, and set the projected high-priority ones aside to
book the moment availability opens. Don't downgrade a strong room for being projected — plan it,
flag it pending.

### Comments are first-class input

Trip members leave freeform comments on games; they carry intent the structured fields don't.
Find them via `commentCount` on `list_games`, then call `list_game_comments` for every game with
count > 0 — including rooms you'd otherwise exclude, since a comment is often *why* a room is
out or bumped. Build an explicit gameId → game name → comment table and re-verify each pairing
before acting; mis-attributing a comment to the wrong game is a high-cost failure. Typical
readings: "already played" → exclude even at high priority; "must do / book first" → hard
priority; "skip / only if time" → deprioritize; logistics notes ("needs 4," "do after dark,"
"pairs with X") → fold into placement. When a comment conflicts with a structured field, the
comment wins — it's the newer, more specific human signal. Whenever a comment changes the plan,
say so.

### Cut games are out of the pool

A **cut** game (`isCut: true`) is one the owner deliberately set aside — the structured
equivalent of an "already played" comment. **Never schedule a cut game.** `list_games` hides cut
games by default, so the pool you place from is already clean; `get_trip_summary` still lists
them (labeled `isCut`) so you can see the whole pool. Once per session, do one
`list_games` with `includeCut: true` to see what was cut and read those games' comments (the
*why* often lives there) — and if something high-priority was cut, surface it rather than
silently honoring it ("you've cut [room], a top-priority pick — intended, or should it go back
in?"). Don't un-cut a game yourself; flag it and let the user decide.

### Arrival, departure, and city changes

- **Arrival day:** gate on arrival time + delay risk, not on it being "the arrival day." Risk
  rises with long-haul/international flights, connections, overnight or last-flight arrivals,
  weather-prone seasons/airports, and slackless itineraries; the real test is whether a delay
  would blow up a booked room. High risk or late arrival → book nothing. Low-risk early arrival →
  building an afternoon around it is fine.
- **Departure day:** early/midday flight → empty. Late-afternoon flight → a tight 1–3 room
  morning cluster ending before the flight, with an airport buffer confirmed for that airport.
- **Mid-trip city change:** you can only play where you physically are — give the transport its
  own block and cluster that day's rooms around it.

### Lodging, teams, and events

- Single base when the region is compact; when the trip spans regions, move lodging to follow
  the play area, with luggage moves as their own evening blocks. Treat any lodging placed before
  key facts land (e.g. an unannounced event hotel) as provisional — don't over-optimize around it.
- When a trip is not using tracks, keep a consistent team across a multi-part
  or campaign game. With tracks, keep the roster consistent within each named
  track unless an item-level exception is intentional.
- A room's advertised player minimum is usually soft — bookable under-strength for a fee, or
  fillable by recruiting where the context offers people (an organized event, a group chat).
  When a minimum exceeds the base team, still schedule it and surface the gap as a per-room flag
  ("needs N — recruit or pay for empty slots"); use the profile's recruiting answer, and only
  drop the room if the gap genuinely can't close on this trip.
- **Anchor events:** reserve the event's dates as fixed and empty; fill the other days with
  self-planned play before, after, or both — no inherent bias; follow availability.

### Booking order — when to commit, not the final grid

Sequence bookings by **priority × scarcity**, not by date. Read the scarcity axis from
`bookingPressure.tier` on `list_games` — a data-driven read of how hard each room is to book:
`high` (books solid weeks out, or barely runs) > `medium` (tightens only near-in) > `low`/none.
(The app shows these to users as "hottest" / "hot" / no-tag, but the API returns `high`/`medium`/
`low`.) Book high-priority + `high`-pressure rooms first — even before an anchor event's schedule
publishes; let the trip build around those anchors. Gamble on low-priority, `low`-pressure rooms
per the user's booking-risk answer: leave them unbooked and squeeze them into gaps once the fixed
schedule lands; a sellout is an acceptable loss for a low-priority room only if the user said so.
`bookingPressure` is an estimate (carries a `confidence`; `null` for irregular or off-platform
rooms) — a guide for ordering, never an override of a stated priority or of real availability.

## Operating the Skeleton Key tools

- `create_trip` / `update_trip` — dates, region, travel buffer (15-min default). Past dates are
  allowed and mean recording (see Recording a trip that already happened).
- `copy_trip` — run an old trip again at new dates: `mode: "games"` for the pool alone,
  `mode: "layout"` for the whole day-by-day itinerary shifted onto the new dates.
- `add_cluster` — encode geographic day-groupings (remember: address beats cluster).
- `schedule_game` / `move_game` / `pin_time` — lay rooms onto days; pin only real slot times.
- `update_scheduled_game` — per-leg `travelMode` and buffer overrides to match the profile.
- `add_activity` — meals, tourism, transport, and rest as first-class items; use `isTravel` +
  `transitMode` (`flight`/`train`/`ferry`/`bus`/`other`) for inter-city legs.
- `set_endpoint` — arrival/departure; keep a high-risk arrival day empty.
- `book_game` — commit in priority × scarcity order.
- `list_games` — the schedulable pool; hides cut games by default (pass `includeCut` to audit
  what was set aside — see Cut games).
- `list_game_slots` — the only legitimate source of pinnable times.
- `set_game_availability` — record availability the user gives you (or you observed) for a date: a
  verdict (`available` with its open/sold-out `slots`, or `sold_out`/`closed`) on any date inside
  the trip window, past dates included. The writable layer; provider-imported times are read-only.
- `list_game_comments` — honor member intent (see Comments).
- `evaluate_schedule` / `warm_travel` — the required verification pair (see Verify).
- `list_days` / `get_trip_summary` — review the interleaved schedule and per-day load.

In a read-only context ("propose, don't change anything"), the full workflow still applies:
every `list_*`/`get_*` tool, `evaluate_schedule` (stateless), and `warm_travel` (cache-only) are
safe; present the proposal instead of writing it.

## Recording a trip that already happened

Not every trip is ahead of the user. Sometimes the trip is over and they want it written down —
which rooms they played, when, what it cost — so it sits alongside the rest of their trips. That
is **recording**, and most of this skill does not apply to it.

**Recognize it from the dates, not from memory.** Read the trip's `endDate` off `list_trips` (or
`get_trip`) and compare it against today's date as established in this conversation. Don't infer
the answer from the trip's name — "Austin 2024" may be a trip taken, a trip being planned, or a
name nobody updated. And don't assume you know today's date: if the conversation hasn't
established it, ask ("is this trip already over, or still ahead?") rather than picking one. A
trip whose `endDate` is before today is past; a trip that spans today is live and still plans
normally — though its days that are already behind you are records too, and everything below
applies to those days.

Once the trip is past:

- **Skip the trip profile intake entirely.** Step 2 exists to shape days that haven't happened
  yet. Asking someone which day shape they want for a weekend they already lived — or whether
  three rooms in a day is too many — is noise. The just-in-time questions go too: a late finish
  that already happened is not a decision anyone can make.
- **Skip booking pressure and booking order.** `bookingPressure` reads how hard a room is to
  book right now; it says nothing about a room played last March, and it is not refreshed for a
  trip that is over. Don't sequence a recorded trip by priority × scarcity, and don't tell the
  user to book anything early. "The Vault is `high` pressure, book it first" is nonsense advice
  about a room they already played.
- **Record the real times, then pin them.** No availability import runs for a trip that is over,
  and projections are never generated for a past date, so what the user remembers (or has in
  their confirmation emails) is the only source of times. Record them with `set_game_availability`
  — `{date, status: "available", slots: [{time: "10:00", available: true}, ...]}` — then `pin_time`
  the scheduled game to a time you just recorded. That is how "pin only real times" stays
  satisfiable here: a recorded time is a real time, it just came from the user instead of the
  venue's booking page (the room was available then; list_game_availability reports such a date by its data rather than as 'historical', having no local clock to judge past-ness). If they only
  remember "sometime that afternoon," leave the game unpinned and say so — an invented 14:00 is
  still invented.
- **Don't warm travel, and don't verify feasibility.** The day already happened; a simulator's
  opinion about whether it fits changes nothing, and `warm_travel` is the one tool that spends
  external travel budget. Call `evaluate_schedule` / `warm_travel` on a past trip only if the
  user explicitly asks for travel times or a retrospective ("how much of that day was driving?").
- **Book what they played, and omit `sendInvite`.** Mark each room they actually played with
  `book_game`, recording what it cost (`bookingPriceCents` + `bookingCurrency`) if they track
  spend. Leave `sendInvite` out: nobody needs a calendar invite to an evening they already spent,
  and the server's own suppression is a backstop, not a licence — it only kicks in once a date is
  a full day behind UTC, so a trip that ended yesterday can still mail real invites to real
  people. When something clearly older is booked with `sendInvite: true` anyway, the booking
  still succeeds and the response reports the suppression as `emailError: "Past date - calendar
  invites skipped"` — that is the expected outcome, not a failure, so report the booking as done.
- **Replanning is a different tool.** "Let's do that trip again next year" is `copy_trip`, which
  builds a NEW trip at new dates from the old one's pool (`mode: "games"`) or its whole layout
  (`mode: "layout"`) — never edit the record of what happened into next year's plan. Reserve
  `shift_trip_dates` for moving the recorded trip itself, when it was filed under the wrong dates:
  it carries the recorded times along as unconfirmed predictions, but it fails while any game on
  the trip is booked. **Unbooking to clear that is destructive** — `unbook_game` deletes every
  payment recorded on the game (all of `costs[]`, not just the first), and on a Splitwise-linked
  trip it deletes the linked expenses too, which re-booking recreates as new ones. So read each
  game's `costs[]` off `list_days` and write the amounts down first, unbook, shift, re-book, and
  restore every payment (`bookingPriceCents` on `book_game` for the first, `add_booking_cost` for
  the rest). If that is more than the user bargained for, say so before touching anything — a
  mis-dated record they can live with beats losing what it cost.

### A recorded day, end to end

> "Add the Portland day from last month — we did Cabin in the Woods at 10, then The Vault at
> 1:30. The Vault was $140 for the four of us."

1. `list_trips` → the Portland trip's `endDate` is `2026-06-14`, before today, so this is
   recording: no intake, no booking-order talk.
2. `get_trip_summary` `{tripId}` → the day `2026-06-13` exists and both rooms are already in the
   pool but unscheduled. Note their `gameId`s and the `dayId`.
3. `schedule_game` `{tripId, dayId, gameIds: [<cabin>, <vault>]}` → two `scheduledGameIds`.
4. `set_game_availability` `{tripId, gameId: <cabin>, date: "2026-06-13", status: "available",
   slots: [{time: "10:00", available: true}]}`, and again for the Vault with `[{time: "13:30",
   available: true}]`. Those times are now real, confirmed slots on that date.
5. `pin_time` each scheduled game to its recorded time: `{tripId, scheduledGameId: <cabin sg>,
   time: "10:00"}`, then `{tripId, scheduledGameId: <vault sg>, time: "13:30"}`.
6. `book_game` `{tripId, scheduledGameId: <vault sg>, bookingPriceCents: 14000,
   bookingCurrency: "USD"}` → booked, with no invite mailed (no `sendInvite` passed).

Every call carries `tripId`; it is required on all of these, and omitting it fails the call.

No `evaluate_schedule`, no `warm_travel`, no intake questions. Present it back as what it is: a
record of the day, with anything the user didn't remember left blank rather than filled in.

**Nothing in Skeleton Key yet?** Same flow, two steps earlier: `create_trip` with the dates the
trip actually ran (past dates are accepted, and the days are generated from them), then
`add_game` per room — `search_escape_rooms` first if you need the `mortyGameId`, which fills in
the venue and duration — and pick up at step 3 above.

## If the tools aren't connected

Say so, and tell the user how to connect: Skeleton Key's MCP server is
`https://useskeletonkey.com/api/mcp` — added as a custom connector on claude.ai, or via
`claude mcp add` in Claude Code; their Skeleton Key account page has instructions. Meanwhile you
can still run the intake, discuss methodology, and sketch **structural** day plans — how many
rooms, which areas, what sequence — but never clock times: with no slot source, every specific
time would be invented, and the "pin only real times" rule does not relax just because the data
is missing.

## Common mistakes

| Mistake | Instead |
|---|---|
| Proposing a schedule on assumed pace ("I went with enthusiast intensity") | Run the intake first; the day-shape question is non-negotiable |
| A full day of games with no meal anywhere in it | Lunch + dinner are explicit blocks unless the profile says grab-and-go |
| Precise clock times for venues with no slot data, "labeled as estimates" | A window from operating hours + a confirm-with-venue flag |
| Two late finishes back-to-back because the slots happened to chain | Ask (JIT), or restructure — late-night spacing is a default, not an accident |
| Silently picking rental car vs. transit | It's an intake question; it changes day-trip feasibility |
| Dropping tourism without a word because rooms filled the days | Surface the tradeoff with a named, priced alternative |
| Treating a lodging pin as the fixed center of gravity | Lodging placed before key facts land is provisional — say so in placement reasoning |
| Smoothing a lumpy slot grid into a tidy fake schedule | Report the real gaps, late finishes, and tight transfers as flags |
| Running the intake (or booking-order advice) on a trip that already ended | Compare `endDate` to today first — a past trip gets recorded, not planned |
| Re-warming travel over and over on an `indeterminate` day | Once `warm_travel` reports the legs `unwarmable`, accept the verdict and flag the transit legs |
