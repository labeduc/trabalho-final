# Nice Rooms: Hotel Revenue, MARS and Reviews

Two years of trading for fourteen fictional hotels in the invented waterfront city of Port Alder, published so that revenue management, rate positioning and review analytics can be demonstrated and tested without a licensed extract. Every hotel, rate, booking and review here is generated. No real property, guest or channel is represented.

- Coverage: 104 weeks, 2027-01-04 to 2028-12-31
- Scale: 14 hotels, 42 room types, 10,192 hotel-nights, 1,456 hotel-weeks, 12,077 reviews
- Revenue: 236,002,362 USD total revenue, 202,759,645 USD room revenue, 70.9% mean occupancy, 207 USD mean ADR, 153 USD mean RevPAR
- Rate calls: 1,442 weekly decisions, 44.2% raise, 43.0% hold, 12.8% lower
- Reviews: mean 3.31 stars, 25.1% at five stars, 29.2% at one or two, spread across 11 scenarios

## Files

| File | Contents | Grain | Rows |
| --- | --- | --- | ---: |
| `hotels.jsonl` | Hotel roster | one row per hotel | 14 |
| `room_types.jsonl` | Room types | one row per hotel and room type | 42 |
| `daily_market.jsonl` | Daily market panel | one row per hotel and night | 10,192 |
| `weekly_performance.jsonl` | Weekly performance | one row per hotel and ISO week beginning Monday | 1,456 |
| `rate_decisions.jsonl` | Rate decisions | one row per hotel and week, with the next week attached | 1,442 |
| `reviews.jsonl` | Reviews | one row per review | 12,077 |
| `review_scenarios.jsonl` | Review scenarios | one row per scenario | 11 |

Three notebooks are included: `01-market-and-mars.ipynb`, `02-reviews-and-scenarios.ipynb` and `03-horizon-projection.ipynb`. Each reads the files from `/kaggle/input/` when it runs on Kaggle and from the same folder when it runs beside the data.

## Projection notebook

`03-horizon-projection.ipynb` carries the last complete year forward to 5, 10 and 15 years under three scenarios, and prints the value at each horizon in nominal, real and present value terms. It is a scenario model, not a forecast. Nothing in it is fitted or sampled from the panel: every driver is a stated assumption, and all of them sit in one `ASSUMPTIONS` dictionary so a reader can change one number and rerun the page.

| Driver | Base | Downside | Upside |
| --- | ---: | ---: | ---: |
| ADR growth a year | 3.0% | 2.0% | 4.2% |
| Occupancy drift at stabilisation | 0.0 pts | -6.0 pts | +2.5 pts |
| GOP margin gain over 10 years | 2.0 pts | 0.0 pts | 3.0 pts |
| Exit cap rate | 7.5% | 8.5% | 6.8% |

Inflation is 2.5% a year and the discount rate is 9.0%. Value is NOI less a 3.0% capital reserve, capitalised at the scenario cap rate, with NOI taken after a 4.0% charge for management fee and fixed costs. Occupancy is floored at 45% and capped at 94%. The rating edge adds at most 75 basis points a year of ADR growth, so the review feed enters the projection but cannot dominate it.

The notebook also shows a 500 draw band around the base case, a year 10 sensitivity grid over ADR growth and occupancy drift, a bridge from frozen rates to the year 15 base value, and the hotel level CAGR. It writes the full hotel-year table to `/kaggle/working` when it runs on Kaggle.

## What MARS means

MARS is the market appropriate rate: what this hotel should charge for this night, given what its peer group is actually achieving and how the hotel is positioned against that group. It is a benchmark, not a forecast of demand, and it is built from realised peer rates rather than advertised ones.

```
MARS(h, d) = median ADR of h's comp-set peers on night d
             x positioning(h) / median positioning of those peers
```

Each hotel is benchmarked against its own tier peers: `luxury`, `upscale`, `boutique`, `midscale` or `economy`, two to four hotels in each group. A 79 USD motor inn is not a peer of a 352 USD resort, and mixing them would make every luxury hotel look overpriced and every budget hotel look underpriced. The neighbourhood and the distance to the water are location fields, not the rate peer group. The peer median is computed without the hotel itself, so no property sets its own benchmark.

The result is stored as `mars_rate`, with a usable band of plus or minus six percent in `mars_band_low` and `mars_band_high`.

`rate_position` is `adr / mars_rate`. A value of 1.00 means the hotel charged exactly MARS; 0.95 means five percent below it. `rate_gap_pct` is the same gap in percent.

A simple worked example: if the luxury peers achieve a median ADR of 318 USD and the hotel's positioning is 1.22 against a peer median of 1.15, MARS is 318 x 1.22 / 1.15 = 337 USD. A 352 USD rate is a position of 1.04, or 4 percent above MARS.

## How the rate action is labelled

`rate_action` is a rule applied to two observable signals on the night: the rate position and the on-the-books index for the next seven nights. The rule is deliberately conservative and is documented so it can be reproduced or replaced:

```
lower   if rate_position >= 1.12
raise   if an event night and otb >= 0.70 and rate_position <= 1.02
raise   if otb >= 0.80 and rate_position <= 0.98
raise   if otb >= 0.72 and rate_position <= 0.92
lower   if otb <= 0.54 and rate_position >= 1.00
lower   if otb <= 0.62 and rate_position >= 1.06
hold    otherwise
```

Three percent of labels are flipped by a fixed arithmetic marker, so a classifier is not scored against a perfect oracle. `suggested_change_pct` sizes the move, and a floor at 72 percent and a ceiling at 190 percent of the hotel's anchor rate are enforced in `rate_floor`, `rate_ceiling` and `rate_guardrail`.

`rate_decisions.jsonl` makes the same call weekly and attaches the following week, so the outcome can be scored. `realised_uplift_pct` compares the following week's RevPAR with the call week, and `outcome` bands it at plus or minus 1.5 percent. This is an observational outcome, not a causal lift: the counterfactual is not simulated.

## Revenue indicators

Every indicator is computed from the row's own components, and the validator re-checks the identities after the files are written.

| Indicator | Formula | What it answers |
| --- | --- | --- |
| Occupancy | rooms_sold / rooms_available | How much of the house was sold |
| ADR | room_revenue / rooms_sold | What a sold room earned |
| RevPAR | room_revenue / rooms_available | Revenue per key, sold or not |
| TRevPAR | (room_revenue + ancillary_revenue) / rooms_available | Total revenue per key |
| RevPOR | (room_revenue + ancillary_revenue) / rooms_sold | Total revenue per occupied room |
| Net RevPAR | net_revenue / rooms_available | Revenue after commission per key |
| GOPPAR | gross_operating_profit / rooms_available | Profit per key |
| CPOR | (total revenue - GOP) / rooms_sold | Cost per occupied room |
| RGI | hotel RevPAR / comp-set RevPAR x 100 | RevPAR index, 100 is par |
| ARI | hotel ADR / comp-set ADR x 100 | Rate index, 100 is par |
| MPI | hotel occupancy / comp-set occupancy x 100 | Occupancy index, 100 is par |
| Flow through | change in GOP / change in revenue, year on year | How much of a revenue change reaches profit |
| Ancillary capture | ancillary_revenue / total_revenue | Share of spend outside the room |
| CPOR | operating cost per occupied room | Cost efficiency at the room level |

The comp-set metrics are room-weighted for occupancy and room-night weighted for ADR, which is why `comp_set_revpar` is not simply `comp_set_adr * comp_set_occupancy` after rounding.

## Review scenarios

Reviews are not drawn from a single pool. Each one is produced by a scenario that fixes the rating band, the cause and the recovery action, so the 1 to 5 star feed can be joined to an explicit explanation. Hotel tier shifts the mix: economy and midscale properties see more condition, noise, billing and cleanliness cases; luxury properties see more delight, recovery and location cases. The rating always stays inside the scenario's declared band.

| Scenario | Share | Ratings | Trigger | Recovery |
| --- | ---: | --- | --- | --- |
| `delight` Stay exceeded expectations | 15% | 5, 4 | anticipatory service, a quiet upgrade or a personal touch | thank the guest and record the staff member |
| `location` Location and access | 10% | 5, 4 | walkable waterfront, ferry or old town access | none needed |
| `breakfast` Breakfast and food | 9% | 5, 4, 3 | breakfast quality, choice or service pace | invite the guest back and review the buffet line |
| `service_recovery` Service recovery | 10% | 5, 4, 3 | a problem raised during the stay and fixed well | log the recovery and brief the shift |
| `room_condition` Room condition | 11% | 3, 2, 1 | wear, maintenance or an unfinished room | block the room and raise a work order |
| `noise` Noise and sleep | 10% | 3, 2, 1 | street, corridor or neighbouring room noise | offer a move and a sleep kit, note the room |
| `checkin_wait` Check-in and queue | 9% | 4, 3, 2 | a wait at arrival or a room released late | add a queue check at peak arrival |
| `billing` Billing and charges | 8% | 3, 2, 1 | an unexpected charge, deposit or rate mismatch | audit the folio and refund where due |
| `wifi` Wifi and connectivity | 7% | 4, 3, 2 | dropped signal, slow link or a broken login | send the access point log to IT |
| `cleanliness` Cleanliness | 7% | 3, 2, 1 | housekeeping missed a surface, linen or bathroom detail | reclean, inspect and coach the section |
| `group_trip` Group or family stay | 4% | 5, 4, 3 | a group, wedding or family booking with mixed needs | confirm room allocations before arrival |

Observed stars: 1 star 1,087, 2 star 2,439, 3 star 3,212, 4 star 2,302, 5 star 3,037.

`sentiment_score` is the mean weight of matched phrases from a fixed lexicon, with word boundaries and masking so `not clean` is not also counted as `clean`. It is intentionally noisy: a three star review can read neutral, and a five star review can carry one mild complaint. `response_text` and `response_hours` are null or empty when the hotel did not reply, which is more common above three stars.

## Schema

### `hotels.jsonl`

Hotel roster, one row per hotel.

| Column | Type | Meaning |
| --- | --- | --- |
| `hotel_id` | string | Stable id, H01 to H14 |
| `name` | string | Fictional hotel name |
| `brand` | string | Fictional brand or Independent |
| `tier` | string | luxury, upscale, boutique, midscale or economy |
| `rooms` | integer | Keys in the house |
| `opened_year` | integer | Year the fictional property opened |
| `distance_to_water_km` | numeric | Walking distance to the harbour |
| `neighbourhood` | string | Fictional district |
| `comp_set` | string | Tier peer group used to build MARS |
| `positioning_index` | numeric | Quality and price position against the comp set, 1.00 is par |
| `base_rate` | numeric | Achieved ADR anchor in USD |
| `amenities` | string | JSON array of amenity labels |
| `city` | string | Fictional city, Port Alder |
| `review_count` | integer | Reviews in reviews.jsonl |
| `lifetime_rating` | numeric | Mean star rating, 1 to 5 |
| `share_rating_5` | numeric | Share of reviews at 5 stars |
| `share_rating_1_2` | numeric | Share of reviews at 1 or 2 stars |

### `room_types.jsonl`

Room types, one row per hotel and room type.

| Column | Type | Meaning |
| --- | --- | --- |
| `hotel_id` | string | Hotel id |
| `room_type_id` | string | Hotel id plus ladder position |
| `room_type` | string | Room name |
| `rooms` | integer | Rooms of this type |
| `max_occupancy` | integer | Adults and children the room sleeps |
| `size_sqm` | numeric | Floor area in square metres |
| `rate_premium_pct` | numeric | Premium over the hotel's anchor rate |
| `bar_rate` | numeric | Best available rate for the room type in USD |
| `view` | string | harbour, courtyard, city, street or park |
| `tier` | string | Hotel tier repeated for grouping |

### `daily_market.jsonl`

Daily market panel, one row per hotel and night.

| Column | Type | Meaning |
| --- | --- | --- |
| `hotel_id` | string | Hotel id |
| `hotel_name` | string | Hotel name |
| `tier` | string | Hotel tier |
| `comp_set` | string | Tier peer group used to build MARS |
| `date` | date | Stay night, ISO date |
| `weekday` | string | Day name |
| `month` | integer | Month number |
| `year` | integer | Year |
| `season` | string | winter, spring, summer or autumn |
| `demand_event` | string | Event name, empty on an ordinary night |
| `rooms_available` | integer | Keys available |
| `rooms_sold` | integer | Occupied rooms |
| `occupancy` | numeric | rooms_sold / rooms_available |
| `adr` | numeric | room_revenue / rooms_sold, USD |
| `room_revenue` | numeric | adr * rooms_sold, USD |
| `ancillary_revenue` | numeric | Food, beverage and other spend, USD |
| `commission` | numeric | Channel commission, USD |
| `net_revenue` | numeric | room_revenue + ancillary_revenue - commission |
| `gross_operating_profit` | numeric | net_revenue * the hotel's GOP margin |
| `bookings` | numeric | Gross bookings including those later cancelled |
| `cancellations` | numeric | Cancelled rooms |
| `cancellation_rate` | numeric | cancellations / bookings |
| `booking_window_days` | numeric | Mean lead time in days |
| `length_of_stay` | numeric | Mean nights per booking |
| `available_rooms` | integer | Alias of rooms_available |
| `revpar` | numeric | room_revenue / available_rooms, USD |
| `trevpar` | numeric | (room_revenue + ancillary_revenue) / available_rooms, USD |
| `revpor` | numeric | (room_revenue + ancillary_revenue) / rooms_sold, USD |
| `net_revpar` | numeric | net_revenue / available_rooms, USD |
| `goppar` | numeric | gross_operating_profit / available_rooms, USD |
| `cpor` | numeric | Operating cost per occupied room, USD |
| `ancillary_capture` | numeric | ancillary_revenue / total revenue |
| `market_occupancy` | numeric | All 14 hotels, room-weighted |
| `market_adr` | numeric | All 14 hotels, room-night weighted |
| `demand_index` | numeric | market_occupancy / 0.68 |
| `otb_occupancy_index` | numeric | Next seven nights' occupancy divided by the hotel's normal level |
| `comp_set_occupancy` | numeric | Peer group occupancy, room-weighted |
| `comp_set_adr` | numeric | Peer group ADR, room-night weighted |
| `comp_set_revpar` | numeric | Peer group RevPAR |
| `rgi` | numeric | RevPAR index against the comp set, 100 is par |
| `ari` | numeric | ADR index against the comp set, 100 is par |
| `mpi` | numeric | Occupancy index against the comp set, 100 is par |
| `mars_rate` | numeric | Market appropriate rate for this hotel on this night, USD |
| `mars_band_low` | numeric | mars_rate * 0.94 |
| `mars_band_high` | numeric | mars_rate * 1.06 |
| `rate_position` | numeric | adr / mars_rate, 1.00 is at MARS |
| `rate_gap_pct` | numeric | (adr - mars_rate) / mars_rate * 100 |
| `rate_action` | string | raise, hold or lower |
| `suggested_rate` | numeric | The rate the rule proposes, USD |
| `suggested_change_pct` | numeric | Change from adr, percent |
| `rate_guardrail` | string | none, floor or ceiling |
| `rate_floor` | numeric | Floor guardrail, USD |
| `rate_ceiling` | numeric | Ceiling guardrail, USD |
| `price_elasticity` | numeric | Assumed elasticity used by the rule |
| `mars_basis` | string | How MARS was built |
| `mars_confidence` | numeric | 0.4 to 0.95, wider when the signals disagree |

### `weekly_performance.jsonl`

Weekly performance, one row per hotel and ISO week beginning Monday.

| Column | Type | Meaning |
| --- | --- | --- |
| `hotel_id` | string | Hotel id |
| `hotel_name` | string | Hotel name |
| `tier` | string | Hotel tier |
| `comp_set` | string | Tier peer group used to build MARS |
| `week_start` | date | Monday of the week |
| `available_rooms` | integer | Room nights available |
| `rooms_sold` | integer | Room nights sold |
| `bookings` | integer | Gross bookings |
| `cancellations` | integer | Cancelled rooms |
| `occupancy` | numeric | rooms_sold / available_rooms |
| `adr` | numeric | Room revenue per sold room, USD |
| `revpar` | numeric | Room revenue per available room, USD |
| `room_revenue` | numeric | USD |
| `ancillary_revenue` | numeric | USD |
| `commission` | numeric | USD |
| `net_revenue` | numeric | USD |
| `total_revenue` | numeric | room_revenue + ancillary_revenue |
| `trevpar` | numeric | total_revenue / available_rooms, USD |
| `revpor` | numeric | total_revenue / rooms_sold, USD |
| `net_revpar` | numeric | net_revenue / available_rooms, USD |
| `gross_operating_profit` | numeric | USD |
| `goppar` | numeric | gross_operating_profit / available_rooms, USD |
| `cpor` | numeric | Operating cost per occupied room, USD |
| `length_of_stay` | numeric | Room-night weighted mean nights |
| `booking_window_days` | numeric | Mean lead time |
| `cancellation_rate` | numeric | cancellations / bookings |
| `comp_set_occupancy` | numeric | Peer occupancy |
| `comp_set_adr` | numeric | Peer ADR |
| `comp_set_revpar` | numeric | Peer RevPAR |
| `rgi` | numeric | RevPAR index, 100 is par |
| `ari` | numeric | ADR index, 100 is par |
| `mpi` | numeric | Occupancy index, 100 is par |
| `ancillary_capture` | numeric | Ancillary share of total revenue |
| `demand_event` | string | Event during the week, empty otherwise |
| `event_nights` | integer | Nights in the week touched by an event |
| `flow_through` | numeric | Change in GOP over change in revenue against the same week a year earlier, null in year one |
| `review_count` | integer | Reviews in the week, null-free |
| `mean_rating` | numeric | Mean star rating, null when no review landed |
| `positive_share` | numeric | Share at 4 or 5 stars |
| `sentiment_index` | numeric | Mean lexicon score, -1 to 1 |
| `response_rate` | numeric | Share of reviews with a management response |

### `rate_decisions.jsonl`

Rate decisions, one row per hotel and week, with the next week attached.

| Column | Type | Meaning |
| --- | --- | --- |
| `decision_id` | string | Stable id |
| `hotel_id` | string | Hotel id |
| `week_start` | date | Monday the call was made |
| `action` | string | raise, hold or lower |
| `current_adr` | numeric | The week's ADR, USD |
| `mars_rate` | numeric | Mean MARS for the week, USD |
| `rate_position` | numeric | Monday ADR / MARS |
| `rate_gap_pct` | numeric | Monday gap to MARS, percent |
| `booking_pace_index` | numeric | Next seven nights' occupancy against the hotel's normal level |
| `forecast_occupancy` | numeric | The week's occupancy scaled by pace |
| `suggested_rate` | numeric | Proposed rate, USD |
| `suggested_change_pct` | numeric | Proposed change, percent |
| `rate_guardrail` | string | none, floor or ceiling |
| `expected_revpar` | numeric | Elasticity-implied RevPAR, USD |
| `expected_lift_pct` | numeric | Expected RevPAR change, percent |
| `confidence` | numeric | 0.4 to 0.95 |
| `rationale` | string | Semicolon list: pace_high, pace_low, below_mars, above_mars, event_compression, cancel_risk, inside_band |
| `cancellation_rate` | numeric | The week's cancellation rate |
| `demand_event` | string | Event during the week |
| `realised_occupancy` | numeric | The following week's occupancy |
| `realised_adr` | numeric | The following week's ADR, USD |
| `realised_revpar` | numeric | The following week's RevPAR, USD |
| `realised_uplift_pct` | numeric | RevPAR change against the call week |
| `outcome` | string | better, in line or worse at a 1.5 percent band |

### `reviews.jsonl`

Reviews, one row per review.

| Column | Type | Meaning |
| --- | --- | --- |
| `review_id` | string | Stable id |
| `hotel_id` | string | Hotel reviewed |
| `review_date` | date | Date the review was posted |
| `stay_date` | date | Night being reviewed |
| `week_start` | date | Monday of the stay week, joins weekly_performance |
| `scenario_id` | string | Scenario that produced the review |
| `rating` | integer | Stars, 1 to 5 |
| `title` | string | Review title |
| `text` | string | Review body |
| `channel` | string | review site, OTA survey, direct survey or social |
| `segment` | string | ota, wholesale, group, direct or corporate |
| `nights` | integer | Nights stayed |
| `room_type` | string | Room type stayed in |
| `verified_stay` | boolean | Whether the booking is verified |
| `cleanliness` | integer | Subscore, 1 to 5 |
| `comfort` | integer | Subscore, 1 to 5 |
| `service_score` | integer | Subscore, 1 to 5 |
| `location_score` | integer | Subscore, 1 to 5 |
| `value_score` | integer | Subscore, 1 to 5 |
| `sleep_quality` | integer | Subscore, 1 to 5 |
| `would_return` | boolean | Whether the guest says they would return |
| `sentiment` | string | positive, neutral or negative |
| `sentiment_score` | numeric | Mean lexicon weight, -1 to 1 |
| `lexicon_version` | string | Scorer version |
| `response_text` | string | Management response, empty when none |
| `response_hours` | numeric | Hours to respond, null when none |

### `review_scenarios.jsonl`

Review scenarios, one row per scenario.

| Column | Type | Meaning |
| --- | --- | --- |
| `scenario_id` | string | Scenario key used in reviews.jsonl |
| `label` | string | Human label |
| `share_of_reviews` | numeric | Base share before the hotel tier adjustment |
| `rating_band` | string | Ratings the scenario can produce |
| `trigger` | string | What caused the stay to go this way |
| `expected_sentiment` | string | positive, mixed or negative |
| `recovery_action` | string | What the hotel should do about it |

## Provenance and reproducibility

The panel is generated by a deterministic simulation with seed 20271014. The same seed, start date and hotel roster produce the same rows line for line, so a rebuild can be diffed against this version. Nothing is sampled from a real hotel, OTA, review site or public dataset, and no external file is read at build time.

Each hotel-night starts from a tier occupancy level, a monthly seasonal curve, a day of week curve and six recurring demand events a year. ADR follows the hotel's anchor rate, the seasonal rate curve and the night's occupancy pressure. Ancillary spend, commission, cancellations, lead time and length of stay follow the hotel's tier and the night's occupancy. MARS is then computed from the realised peer rates, and the rate action from the position and the on-the-books index.

The three notebooks are executed before publication, so their charts are stored in the notebooks that ship with the data. The build also writes `dataset-metadata.json`, which Kaggle reads for the title, description and column types.

## Quality checks

The builder refuses to finish unless all of the following hold. Each is re-measured from the files after writing, not trusted from memory.

- Every line parses, and every file has exactly the documented keys in the documented order.
- Every hotel id, week start and review id is present in the table that owns it.
- Occupancy, shares, indices and star ratings sit inside their stated ranges.
- `room_revenue = adr * rooms_sold`, `revpar = room_revenue / available_rooms`, `net_revenue = room_revenue + ancillary_revenue - commission`, and `mars_band` brackets `mars_rate`.
- `rate_action` is one of raise, hold or lower, and `outcome` is one of better, in line or worse. All three of each appear.
- Every review's rating lies inside its scenario's declared band, and every scenario appears in the feed.
- There are exactly 104 weeks per hotel and 103 decisions per hotel, because the last week has no following week to score.

Limits worth knowing: the demand model is invented, so elasticities are assumed rather than estimated; the label has three percent noise; the on-the-books index uses the following seven nights and is therefore a simulation construct, not a booking curve; `flow_through` is null for the first 52 weeks; weekly review fields are null in the few hotel-weeks with no review. The data is suitable for testing pipelines, dashboards and models, not for inferring real market behaviour.

## Licence and citation

Released under the Creative Commons Attribution 4.0 International licence (CC BY 4.0). You may use, adapt and redistribute the data with attribution.

```
Nice Rooms: Hotel Revenue, MARS and Review Intelligence (2026).
Synthetic hotel market panel, version 1.0.0.
https://www.kaggle.com/datasets/crissymoon/nice-rooms-hotel-revenue-rate-intelligence
```

Generated 2026-10-24 for the Nice Rooms business intelligence instance. Names, brands and addresses are fictional; any resemblance to a real property is unintended.
