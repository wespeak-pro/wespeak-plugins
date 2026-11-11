# Promotions and hotel services

These five tools require manager/owner access and the account's new Knowledge
version. A feature refusal is not an empty list. Use hotel IDs from the reads;
when there is only one hotel, do not ask the user to select it again.

## `getPromotions`

Call with `{}` to list each hotel (`complexId`, `hotelActive`), its `rooms`
(`key`, `label`) and its `promotions`.
Promotion status is evaluated **today in the account's timezone**: `live`,
`not_open_yet`, `expired` or `paused`. A deactivated hotel's promotions are not
served even if the promotion itself is active.

This list excludes legacy promotions written as ordinary knowledge text. Use an
explicitly read-only `knowledgeAssistant` inspection when investigating those.

## `searchPromotionRates`

Find the real engine rate that carries a promotion. Supply requested dates and
adult count; `checkout` must be 1–30 nights after `checkin`.

```json
{"complexId":"hotel-example-01","checkin":"2027-02-12","checkout":"2027-02-14","persons":2}
```

Copy exact rate names/codes from the response. No rates can mean sold out or a
minimum-stay restriction for that search; it does not prove the rate never exists.
Use another relevant date range when appropriate, and report what was actually
checked. This is a live availability read, not a tool for writing engine prices.

## `managePromotion`

Read the hotel's promotions first, show the full proposed change and obtain
confirmation. `op` is `create`, `update` or `delete`; update/delete require the
existing `promotionId`. **Update replaces the entire promotion**: preserve all
unchanged fields and restrictions from the current record.

Example after the hotel has approved the full terms and a real rate search has
returned the fictional name/code below:

```json
{
  "complexId":"hotel-example-01",
  "op":"create",
  "promotion":{
    "name":"Summer breakfast included",
    "description":"Breakfast is included for the selected stay dates.",
    "inEngine":true,
    "engineRates":[{"name":"Breakfast included","code":"BREAKFAST"}],
    "bookingTo":"2027-02-28",
    "stayFrom":"2027-02-01",
    "stayTo":"2027-02-28",
    "isActive":true
  },
  "rateCheck":{"checkin":"2027-02-12","checkout":"2027-02-14","persons":2}
}
```

- Always include the user-approved `bookingTo` so expired offers stop being
  mentioned. Booking dates govern when guests can book; stay dates govern check-in.
- `description` states the benefit and conditions, never amounts or a computed
  discount. Quote prices come from the engine.
- `inEngine: true` with named `engineRates` requires `rateCheck` showing all those
  rates. Empty rates means the benefit is already included in **every** engine
  rate; use that only when the hotel explicitly establishes it.
- `inEngine: false` means an advisor applies the offer: the assistant informs the
  guest without amounts and hands the booking to an advisor.
- Restrict rooms with `roomKeys`, copied from the `key` of that hotel's `rooms`;
  omit it only when the offer covers every room. A stored promotion comes back
  with `selectedRooms`, `bookingWindow` and `stayWindow` instead: the tool
  accepts those stored forms as they are, so an update can re-send the record
  it read. Leaving them out widens the offer to every room and removes its
  dates. Stay weekdays and people bounds are restrictions too; preserve all of
  them on updates.
- An unchanged stored engine rate can be preserved without a new rate check when
  only pausing an offer or changing dates. Follow the live tool's result if it
  rejects a newly selected rate.
- `warning: reindex_failed` means the write remains saved but the search card can
  be stale. Follow `warningGuidance`: after a create or an update, repeat an
  update with the stored content (it is idempotent); after a delete, re-save
  another remaining promotion with its own content, or have `knowledgeAssistant`
  re-save an Extra Information entry. Never repeat the create or the delete.
- After saving, the verification scenarios are a quote inside the offer, one
  outside, and a date-less promotion question. Run them within the authorized
  [live-test scope](knowledge-and-help.md#testassistant); otherwise report the
  saved promotion as behaviorally unverified. Never turn a quote test into a
  booking or payment request. Distinguish saved configuration from behavior.

For deletion after approval:

```json
{"complexId":"hotel-example-01","op":"delete","promotionId":"promotion-example-01"}
```

If migrating a live legacy offer out of knowledge text, first create the structured
promotion and verify that it is saved. Then remove the old text through an
authorized `knowledgeAssistant` edit. Do not leave two conflicting versions or
remove the only valid offer first. Unsupported cases such as cash-payment
discounts, per-person packages or holiday exclusions without dates need WeSpeak
support rather than a fabricated promotion representation.

## `getHotelServices`

Call with `{}` to read offered services and the catalog of service IDs and their
knowledge fields. Per hotel:

- `configured: true`: `activeServices` determines what the assistant offers. An
  inactive service is answered as unavailable, except the amenities (wifi, gym,
  coworking, laundry): an inactive amenity is only dropped from the services
  list the assistant receives. No “not available” instruction is added for it,
  and other knowledge text that mentions the amenity keeps doing so.
- `configured: false`: the service selection was never saved, so its existing
  knowledge still supplies services without this filtering. Some amenities only
  become available after selection; read `answeredAmenitiesNotServed` and
  `suggestedFromAnswers` rather than treating this as all services off.

## `setHotelServices`

Change only the services the user named, using IDs from the catalog. Read first,
preview with `dryRun: true`, show before/after and the notices, then apply the same
change with `dryRun: false` after confirmation.

```json
{"complexId":"hotel-example-01","activate":["wifi"],"dryRun":true}
```

The example assumes `wifi` was returned in this account's catalog. Use `deactivate`
for services being turned off. Omitted services retain their state, but the **first
ever selection** starts filtering: anything not in the resulting `after` set
stops being offered. Include `startsFilteringNotice` in the user's preview.

This only switches a service on/off. Set hours, prices and conditions separately
with an authorized `knowledgeAssistant` edit. An active service with no answer
can cause a handoff to an advisor. Relay returned `serviceActivation` (which
services are now denied, which stay unfiltered, which amenities are only omitted
and which active ones are deferred to an advisor) and the notice fields so the
hotel understands what guests will hear; then read back or test the relevant
response.
