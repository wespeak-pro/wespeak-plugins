# Assistant configuration

These four tools require manager/owner access. They affect the live account. Use
the current returned catalog and editability flags: capabilities vary by account.

## `getAssistantConfig`

Call with `{}` to read settings and their current `value`. Respect
`editable: false` and its `reason`. Available settings include global pause,
answering schedule, automatic follow-ups, reduced response mode, quote price
modifier, currency instructions, photo collages, the mandatory post-quote message,
and booking mode. `schedule` is in the account's timezone; `source: alwaysOn`
means no schedule is stored. When `followUpAgent` is true, the first follow-up
switch controls the AI follow-up agent.

## `setAssistantConfig`

1. Read the current configuration.
2. Pass only the requested `changes` with `dryRun: true`.
3. Explain `before` → `after`, including account-wide effects, and obtain the
   user's confirmation for this concrete preview.
4. Apply the same changes with `dryRun: false`. A confirmation already given for
   that preview remains valid; ask again only if the proposed change changes.
5. Check the actual `before`, `after`, `unchanged`, `failed`, `warnings`,
   `unverified` and `notice` fields. Report partial success precisely; a saved
   setting is not proof of guest-facing behavior.

Preview example for “Set the global markup over engine prices to 5%”:

```json
{"changes":{"priceModifierPercent":5},"dryRun":true}
```

After the user approves the concrete preview, repeat with `dryRun: false`.
`priceModifierPercent` is the resulting percentage applied to every quote, not an
increment to the existing modifier. Positive means markup, negative means discount.
Never infer an amount or silently turn a global adjustment into a per-hotel one.

Other supported changes, subject to the live schema and account:

```json
{"changes":{"schedule":{"sunday":"off","monday":[{"start":"08:00","end":"22:00"}]}},"dryRun":true}
```

- Omitted settings and weekdays keep their stored values. Each schedule range
  stays within one day; split an overnight shift across two weekdays. End time
  can be `24:00`.
- `assistantPaused` pauses/resumes the account for **all guests**; explicitly
  confirm that effect.
- The second automatic follow-up requires the first. This tool changes switches,
  not the send timing or message text.
- `mandatoryMessage: ""` removes the post-quote message. Preserve the user's
  requested wording rather than inventing terms or promises.
- Changing `bookingMode` replaces the booking script with that mode's standard
  script. Entering or leaving WeSpeak Payments is not supported here.
- A refused/invalid setting prevents the planned write. Do not retry by removing
  restrictions or changing other fields without the user choosing a new scope.

## `getBehaviorSettings`

Call with `{}` to read the current account's available automatic-pause rules,
pause messages, next-available-date offer, quote-photo switch where offered,
chat-label AI instructions and room text. `catalog` explains the accepted
`fieldPath`, type and bounds. `settings`, label `values` and room `values` are raw
stored values; null means unset, with the platform default applying.

Use returned `labelId`, exact `roomName` and `complexId` as selectors; leave
`complexId` out when the reader returns it as null. Top-level `errors`, keyed by
`fieldPath`, lists the settings that could not be read and are absent from
`settings`; a label or room that could not be resolved carries `error` instead
of `values`. Never read either as unset or use it as a write baseline.

## `setBehaviorSetting`

Read the field, present the specific before/after value and obtain confirmation.
Write one setting per call with the exact read value as `expectedValue`:

```json
{"fieldPath":"enableNextAvailableDate","value":true,"expectedValue":false}
```

This example applies only if that field is offered, the current value really is
false and the incompatible no-availability pause is off. Use the actual current
baseline, including null when unset. If another user changed it, the comparison
fails: read again and explain the newly relevant difference before applying.

- `chatLabel.*` requires `selector.labelId`; `room.*` requires
  `selector.roomName`, plus `complexId` for multi-hotel accounts. Copy the field
  and selector from the reader rather than guessing a path.
- Preserve types: booleans for switches, numbers for limits, strings for text and
  the full weekly object for behavior schedules. Text that resembles JSON or
  `true` is still text. An empty text string clears that text.
- A pause's switch, limit and message are distinct writes. Treat a multi-call
  change as potentially partial; report which calls succeeded.
- `enableNextAvailableDate` and `extraConfiguration.enableStopOnNoAvailability`
  are mutually exclusive. Include disabling the incompatible setting in the
  preview and authorized change before enabling the requested one.
- `enableSecondStopMessage` requires its `secondStopMessageTimeTables` first.
- Read back after success. `changed: false` means the value was already identical.

## Choose the correct editing surface

Hotel facts and reservation-flow guidance belong to `knowledgeAssistant`.
Promotions belong to `managePromotion`; offered-service switches belong to
`setHotelServices`. Do not use knowledge text to bypass an unavailable setting.
Room/common-area photo uploads, follow-up timing, notification recipients and
other platform settings without a matching tool belong in the WeSpeak panel;
look up the current help article. Integrations, payment methods and unsupported
engine behavior require the appropriate product workflow or WeSpeak support.

When a test run is authorized, `testAssistant` can demonstrate the observed reply;
follow its [live-test limits](knowledge-and-help.md#testassistant). Otherwise read
back the setting and identify behavior as untested. If the
test does not reflect a saved change, say both facts. Do not invent a propagation
delay or promise it will work later; offer an authorized reversal or support review.
