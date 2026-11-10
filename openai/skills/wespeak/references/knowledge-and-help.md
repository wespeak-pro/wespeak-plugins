# Knowledge, testing and product help

`knowledgeAssistant` and `testAssistant` require manager/owner access. Documentation,
error diagnosis and reporting are available to agents and higher roles. Diagnosis
and knowledge delegation can run the virtual assistant; their live-test effects
are described under `testAssistant` below.

## `knowledgeAssistant`

This is the customer MCP's single entry point for reading and editing knowledge.
Pass a natural-language `instruction`, including the intended hotel when needed.
Its internal field editors are not separate tools available to this plugin.

For an analysis request, make the absence of edit authorization explicit:

```json
{"instruction":"Read only: inspect the hotel's breakfast information and reservation guidance. Report gaps or conflicting answers with the current text. Do not edit, delete, undo, save, test or replay anything."}
```

The delegate reads in full the account-wide fields (the assistant's introduction,
the communication and booking guidelines), the Extra Information entries and each
hotel's value proposition, website and booking link. Of the other per-hotel
fields (hotel facts, services, policies, payment texts) it only gets short
excerpts, a few fields per search. Present an inspection of those as a sample of
what is stored, not as a complete audit.

For an authorized edit, pass the complete text the field should hold:

```json
{"instruction":"The user has approved this exact text for the Breakfast field of Hotel Example: \"Breakfast is served from 07:00 to 10:30 in the main restaurant and is included in every rate.\" Replace the field with that text and change nothing else. Do not run tests or replays; report behavior as untested."}
```

An edit replaces the whole text of a field or entry, and for most per-hotel
fields the delegate cannot see the rest of what is stored. An instruction to
change one detail and keep the other content can drop that content while the
result still says `applied: true`: have the user provide or confirm the full
text first.

The delegated agent validates edits, keeps change snapshots and can undo knowledge
changes. Use it for hotel facts/policies, service details, website and booking links,
reservation-flow guidance, guest-facing checkout information and advisor-handoff
rules. Keep operational settings, structured promotions and service activation in
their dedicated tools.

- Inspection can correctly return `applied: false`: no edit was requested.
- For a requested edit, claim it was saved only when `applied: true`. Check
  `summary` and `error`; do not treat a fluent explanation as write confirmation.
- A summary pointing to another configuration tool is a routing result, not a
  reason to keep retrying the knowledge editor.
- Behavioral rules are best-effort. Knowledge cannot override a pipeline-owned
  behavior or an unavailable setting. Do not guarantee a language rule, changed
  quote behavior or future answer without checking the account's capabilities and
  a relevant test.
- An undo is another write, and it is not selective. It restores the knowledge to
  its state before the assistant's most recent run of edits, which also reverts
  every knowledge text changed since then, edits made by hand in the panel
  included; promotions and the service selection are kept. It cannot pick an
  older change: correct that one with a new explicit edit. Tell the user what an
  undo will revert, get their confirmation and relay what the summary reports as
  also reverted.

## `testAssistant`

Run a realistic message when testing/reproduction is within the user's authorized
scope. This is the account's **live preview path**, not an isolated sandbox: a
handoff can send a real `[TEST]` email to the hotel's notification address, and
configured booking/payment tools remain available to the tested assistant. Explain
that before testing. Do not use testing to bypass a missing guest-message or
booking tool. A real booking/payment action or guest contact requires explicit
authorization for that effect; a general request to “test the answer” is not it.

For an authorized factual-answer test:

```json
{"message":"What time is breakfast served?"}
```

Read `assistantResponse`, `assistantStopped`, `stopReason` and `sessionId`.
Continue a multi-turn scenario with the same returned `sessionId`; omit it for an
independent scenario. If the returned ID changes or the answer loses context, do
not claim the prior turns were retained. Dates, party size and room choices must
match what the test is intended to establish. Do not supply real guest identities
or advance from a quote into reservation/payment creation merely to complete a
test. A generated answer does not itself prove a payment or reservation occurred.

The customer MCP does not guarantee a separate structured verification block from
`knowledgeAssistant`. Call `testAssistant` yourself within the authorized test
scope when you need to show an observed question and answer. If testing is not
authorized, rely on the `applied` result and say behavior has not been tested.
If the saved edit is absent from the test response, say so explicitly. Do not
invent indexing delays or promise eventual success.

## `analyzeAssistantError`

Use when the hotel identifies a problematic assistant reply and explains what
was wrong. If the expected behavior is already clear, use that explanation without
asking again; otherwise obtain it before analysis. Preserve the real conversation
ID, exact assistant message and the user's complaint:

```json
{
  "conversationId":"conversation-example-01",
  "assistantMessage":"Breakfast ends at 10:00.",
  "complaint":"Breakfast now ends at 10:30, and we updated the hotel information yesterday."
}
```

The tool investigates and can reproduce the case through the same live-test path;
do not invoke it as a strictly read-only conversation summary. When reproduction
could trigger booking/payment operations or notifications outside the authorized
scope, explain that limit and use the stored conversation/documentation evidence
instead. Explain its verdict in plain language:

| Verdict | Next step |
| --- | --- |
| `not-an-error` | Explain the evidence or relevant setting; do not report it as a bug |
| `knowledge-gap` | Explain what is missing/wrong and propose a specific knowledge correction |
| `real-error` | Explain the observed problem and offer to report it |
| `improvement-suggestion` | Describe it as a request for improvement, not a confirmed defect |

The verdict does not authorize applying a fix or notifying operations.

## `reportToOperations`

This creates a **real support ticket** in WeSpeak's operations queue, visible to
the hotel under `/tickets`. It also sends a real confirmation email to the
hotel's own ticket recipients and raises an alert in the hotel's panel. Use it
after analysis and only with an explicit request or confirmation to report that
issue. Pass `conversationId`, exact `assistantMessage`, the diagnosis-based
`summary`, `category`, `type` and `priority`. Include `messageId` only when
known: it is the **guest message that prompted the reply**, not the assistant
message's ID. Copy it exactly; omit a blank/unavailable ID.

Example following a matching diagnosis and the user's report authorization:

```json
{
  "conversationId":"conversation-example-01",
  "assistantMessage":"Breakfast ends at 10:00.",
  "summary":"The hotel reports a 10:30 breakfast end time. Analysis found conflicting breakfast hours in the stored knowledge: the policy says 10:30 but an extra-information entry says 10:00. The reported reply used 10:00.",
  "category":"error",
  "type":"general",
  "priority":"2"
}
```

Choose `category: error` or `improvement`, and `type: priceAndAvailability`,
`general` or `noIntervention` from the case. Priority uses the tool's unusual
direction: `1` low impact, `2` important but not urgent, `3` urgent. Do not infer
cross-customer impact from one hotel's complaint.

Use the returned `ticketId`. `alreadyExists: true` means an existing report was
found for that message, which may already be closed; tell the user no new ticket
was created instead of presenting it as just opened. Without a known message ID,
do not assume retries are deduplicated. On uncertain creation, do not blindly
repeat the write. A ticket is not a resolved issue or a promised resolution date.

## `searchDocumentation`

Search the public WeSpeak help center for a topic before giving feature instructions
that are not established by the available tools:

```json
{"query":"automatic follow-ups configuration"}
```

Results include article slugs, summaries and public links in available languages.
This is global product documentation; a successful search does not verify which
hotel account is connected or whether its plan has a feature.

## `getDocumentation`

Read the full article with a slug copied from the search results:

```json
{"slug":"article-slug-from-search","language":"en"}
```

Use `es`, `en` or `pt` to match the user when available, and link the returned
public article. Do not invent a slug, URL or menu sequence. Distinguish general
product instructions from a capability verified on this account.

If the tools and documentation cannot establish that a request is supported,
say that plainly and direct the user to WeSpeak support. Managers and owners
reach it through the **Support** button on the panel's dashboard (“Soporte”,
“Suporte”); agents and supervisors cannot open that page, so point them to the
help center's support article, found with `searchDocumentation`, or to their
account's manager. Do not automatically send a message or open a ticket, and do
not invent a support link.
