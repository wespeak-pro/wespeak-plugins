# Sales and reporting

Funnel, metrics and `lookupReservation` tools require supervisor or higher.
`lookupReservation` additionally requires WeSpeak Payments access.

## `getFunnelConfig`

Call with `{}` to read sections and stages, their IDs and names. Use it to resolve
IDs returned by reports and select a valid target for `moveLeadStage`. A stage with
`markAsWon: true` marks the lead won; moving there is not a cosmetic label change.

## `searchLeads`

List conversations that have a funnel state. Filters include `search`, `status`,
`staleDays`, `idFunnelState`, `metaStatus` and `lostReason`. Use zero-based `page`;
pages contain up to 20 rows. Follow `hasNextPage` and use `totalCount` for the
filtered total. The returned `conversationId`, not `remoteContactId`, identifies
the lead for writes.

```json
{"metaStatus":"open","staleDays":3,"page":0}
```

This finds open-outcome leads with no activity for at least three days. Inbox
`status` and funnel `metaStatus` are separate states. `metaStatus: open` includes
unrecorded outcomes; archived leads are excluded.

WeSpeak closes idle leads by itself. Every night it closes the conversations
with no activity for seven days and marks their open leads lost with
`no_activity`; when the intended check-in date passed four or more days ago, two
quiet days are enough. A snoozed lead and one with a scheduled message still
pending are exempt. So a `staleDays` of 7 or more finds few open leads, which
does not mean nobody needs a follow-up: look for those opportunities under
`{"metaStatus":"lost","lostReason":"no_activity"}`.

```json
{"metaStatus":"lost","lostReason":"price_too_high","page":0}
```

Use the live schema's loss-reason enum. `lostReason` implies lost; do not combine
it with won/open. For missing prices, query both `no_price_loaded_yet` and
`no_prices_loaded_yet`. `no_activity` is an automatic closure, not a guest's stated
reason. `other` includes missing reasons; free-text stored reasons may require
reading all lost leads. Separate recorded labels, guest-stated reasons and your
inferences when explaining why opportunities were lost.

## `moveLeadStage`

Apply the requested outcome or stage to an identified lead. Obtain explicit stage
IDs from `getFunnelConfig`; do not invent them or pass the guest's phone as the
conversation ID.

```json
{"conversationId":"conversation-example-01","metaStatus":"lost","lostReason":"price_too_high"}
```

For a stage move, pass the matching `idFunnelSection` and `idFunnelState` returned
by the configuration tool. Do not supply conflicting stage and outcome choices.

The stage decides part of the outcome. A lead is stored as won only while it sits
on a stage with `markAsWon: true`: to mark one won, move it to such a stage;
`metaStatus: "won"` on any other stage is stored as open. A lead on a won stage
stays won until it is moved to another stage, so marking it lost or open requires
that stage move as well. The example above applies to a lead that is not on a won
stage.

Marking won can trigger existing downstream purchase-attribution behavior; do it
only for a requested, supported outcome. Never mark won merely because the guest
received a payment link or said they were interested. Changing the outcome of an
**archived** lead un-archives it: the conversation reopens and the virtual
assistant resumes answering that guest. Archived rows appear in funnel reports
and in searches with `includeArchived`; confirm that effect before moving one.

Inspect `ok`/`error`, but the returned `metaStatus` only echoes the request: read
the lead back with `searchLeads` before reporting its outcome. A funnel change
does not confirm payment or create a reservation.

## `getMetrics`

Choose the report that measures the requested fact:

| Report | What it measures | Main limits |
| --- | --- | --- |
| `overview` | The virtual assistant's activity in a rolling `periodDays` window (default 7): `messages` counts the guest turns it answered, one per turn, and `conversations` the distinct contacts behind them | Excludes messages sent by the team, chats the assistant never answered and, outside web chat, the turns it handed to the team; no response-time or assistant-versus-human split |
| `funnel` | Current states of conversations **created** in a rolling period | Not closures occurring during the period or historical state transitions |
| `pixelConversions` | Browser-reported purchases during the conversion-report period | Soft evidence, not verified payments or all hotel bookings |

```json
{"report":"funnel","periodDays":30,"limit":50}
```

For funnel totals, use the first response's server-calculated `summary`. It covers
the entire created-conversation cohort, including archived conversations and ones
without funnel state. It includes outcome counts, loss reasons, stages, check-in
months, average nights and lead time. Read `period`, `summary.timezone`, exclusions
and category `otherCount`: omitted categories do not mean no such conversations.
Check-in months refer to intended stays, not conversation creation months.

Fetch details only when the task needs individual rows. Funnel and pixel reports
accept `periodDays` 1–365 and `limit` 1–100. While `hasMore` is true, pass
`cursor: nextCursor` with the same `periodDays`. Funnel pages can be shorter than
`limit` due to a size budget (`pageLimitedBy: size`); this does not end pagination.
`count` is the page size, and funnel `summary` is null after the first page. The
data can change during paging; do not claim a transactional snapshot.

```json
{"report":"pixelConversions","periodDays":30,"limit":50}
```

Keep purchase amounts separated by currency. Repeated exact transaction IDs are
collapsed within the report period; rows with missing identifiers retain their
incomplete-evidence warning. Pixel `status: disabled` is not zero bookings.
Pixel purchases, Payments reservations and won leads may describe the same sale:
never sum their totals as unique bookings. A sent payment link is not a conversion.

Conversation searches filter **last update** in the account's timezone; funnel
reports filter **creation**; pixel reports filter **purchase-report time** in UTC;
the overview counts **turns the assistant answered** in a rolling window that
ends now. State which population each number describes instead of comparing
them as one cohort. Do not fabricate unavailable period boundaries, revenue or
conversion denominators from a rolling report.

## `lookupReservation`

Use an actual reservation ID for one record, or zero-based `page` to list the most
recent Payments reservations (10 per page):

```json
{"reservationId":"reservation-example-01"}
```

```json
{"page":0}
```

Follow `hasNextPage`; `totalCount` describes this reservation source. Inspect the
returned status rather than assuming every returned record is paid/confirmed.
This is not an unrestricted PMS booking search. If Payments is unavailable, say
the tool cannot provide that source and use another source only when it actually
answers the question, identifying its limits.

## Example workflows

- “Which opportunities should we follow up today?” Read the funnel structure,
  search stale open leads and inspect their recent conversations. Prioritize with
  evidence and draft next steps; do not change stages or send messages on the
  strength of this analysis request.
- “Give me a commercial report for the last 30 days.” Use overview/funnel reports
  as appropriate, add pixel data only if relevant and available, and separate each
  population. Use full summary totals and disclose any sampled conversation review.
- “Why did we lose leads because of price?” Search the recorded loss reason, read
  the relevant threads and distinguish a label from what the guest actually said.
