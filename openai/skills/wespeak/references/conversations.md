# Conversations

Read tools are available to agents and higher roles. `manageConversation` requires
supervisor or higher. Availability in the current connection is authoritative.
When the account restricts agent contacts, agents can search, read and report
only conversations assigned to them or unassigned. Caller-supplied assignment
filters cannot widen that access.

## `searchConversations`

Find conversations by name, phone or message text (`search`), state (`status`),
channel (`sourceChannel`), assigned agent (`assigned`), last eligible sender
(`lastMessageFrom`), or inclusive local calendar dates (`dateFrom`, `dateTo`).
Pages are zero-based and contain up to 20 rows; continue while `hasNextPage` is
true. `count` describes the page and `totalCount` the filtered result set.

`search` always matches names and phones. Message text is matched only for terms
of five or more characters, within about the last six weeks of messages and
through at most 100 matching messages: a shorter keyword, an empty result or a
small `totalCount` is not evidence that no other conversation mentions it. Read
the threads when the question is what was said.

```json
{"search":"Avery","status":"all","includeArchived":true,"page":0}
```

Use the returned conversation ID for subsequent reads and changes. Archived
conversations are excluded by default; repeat with `includeArchived: true` before
claiming a contact never wrote. An archived conversation can still be open, so
leave `status` as `all` for that check.

For an open-inbox triage, `{"status":"open","lastMessageFrom":"contact","page":0}`
finds threads whose last eligible stored message is from the contact. A complete
sweep should omit `lastMessageFrom`: rows with unknown direction match neither
filter and must be inspected separately. Do not infer direction from the text
preview: it is truncated and may start with a quote of the team's own words.

For attribution or follow-up evidence, add `includeInsights: true`:

```json
{"dateFrom":"2027-02-01","dateTo":"2027-02-07","includeInsights":true,"page":0}
```

- Dates select the conversation's **last update** in the account's timezone.
  Insights describe its retained history, not just events inside those dates.
- `sourceChannel` is the messaging channel; `leadAttribution` describes the stored
  acquisition touch. Missing attribution is unknown, not organic traffic. This is
  not a full advertising-touch history or guaranteed original acquisition source.
- Follow-up send counts are a lower bound of retained records. A reply after a
  follow-up shows chronology, not causation or a confirmed booking. A null reply
  indicator means unavailable evidence, not no response.
- `labels` contains the hotel's own named labels. Use those when the question is
  about a label-defined group, such as guests marked as having stayed; a won funnel
  outcome is a different fact.

## `getConversationMessages`

Read a thread by `conversationId` whenever available. Otherwise use
`contactNumber` plus its correct `channel`. `channelId` selects among several
WhatsApp API, Instagram or Facebook channels of the account; it does not select
a WhatsApp QR number, and an ID that matches no channel falls back to the first
one of that type. A conversation ID determines its own channel.

```json
{"conversationId":"conversation-example-01","limit":50}
```

Messages arrive newest first. Continue with the returned `nextCursor`, the same
conversation/contact and channel, and `limit` (1–100) while `hasMore` is true.
State the inspected coverage if stopping early. `count` is not total history size.

In the customer MCP, compact messages contain `id`, `at` (UTC), `from` and only
applicable fields. `from: contact` identifies the guest; `assistant` includes
automations, campaigns and API sends; `team` means a recorded account-side origin.
These categories are not proof of who typed a message, especially for native
social-channel replies. `agent`, when present, is the recorded sender's email.

- Read `transcription` for audio. If `transcriptionStatus` is `missing`, the user
  can generate it with the **Generate transcription** button under the audio in
  the panel (“Generar transcripción”, “Gerar transcrição”). If `failed`, the
  cause is ambiguous; ask the user to listen, rather than promising a
  transcription button or describing silence. There is no customer MCP
  transcription tool.
- Inspect a contact's attachment `url` with an available supported viewer before
  relying on it. If no viewer is available, disclose that its content is unread.
  A media label or quoted preview is not the attachment itself.
- Exclude `deleted` and `notDelivered` rows when deciding who last replied.
  `deliveryUncertain: true` means a failed provider response may still have been
  delivered. An empty or failed history read leaves the direction unknown.
- `quoted` is truncated; follow `quotedId` to the actual message when necessary.
- `adOrigin` is the latest recorded paid ad, not complete acquisition history.
  An ad may have a greeting that the thread does not store; this is a possibility,
  not a demonstrated explanation for an unexplained message.

## `manageConversation`

Change only the thread and operation requested. Resolve ambiguous names through
search/read first. Prefer `conversationId`; `close` and `snooze` require it, and
with `remoteContactId` an assign or a reopen applies to every conversation of
that contact.

| Request | Arguments after identifying the thread |
| --- | --- |
| Assign it | `{"op":"assign","conversationId":"conversation-example-01","assignTo":"agent@example.com"}` |
| Close it | `{"op":"close","conversationId":"conversation-example-01"}` |
| Reopen it | `{"op":"open","conversationId":"conversation-example-01"}` |
| Snooze it | `{"op":"snooze","conversationId":"conversation-example-01","snoozeUntil":"2027-02-08T09:00:00-03:00"}` |

Use the exact team email supplied or verified for this account; never guess it.
Assignment sets the assignee list to that one email. Resolve relative snooze times
in the user's intended timezone and send an ISO timestamp with an offset. On
`error`, explain what failed. `ok: true` means the operation was accepted, not
that a conversation changed: only `close` reports an unknown ID. Use IDs
returned by this account's tools and read back the assignee or status when the
outcome matters; the read tools do not return the snooze time. These operations
neither send a guest message nor mark a lead won/lost.

## Example workflow

“Review open conversations waiting for us and draft replies.” Search the open
inbox, inspect the latest eligible direction and the relevant histories, then
prioritize the findings and draft replies in the guest's language. Read further
when a promise, quoted price or attachment affects the draft. This request does
not authorize assigning, closing, snoozing or sending messages.
