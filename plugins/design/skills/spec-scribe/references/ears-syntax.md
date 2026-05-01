# EARS Syntax Reference

EARS (Easy Approach to Requirements Syntax) is a small set of structured
templates for writing clear, testable functional requirements. Every
requirement uses the modal verb **shall** — not "should", "will", "must", or
"may". This is a deliberate constraint that forces unambiguous obligation.

## The five patterns

### 1. Ubiquitous

Always true. No condition.

> The `<system>` shall `<response>`.

Examples:

- The system shall display the current user's name in the page header.
- The mobile app shall present content in the user's preferred language.
- The admin dashboard shall list every user account that has been created.

Use ubiquitous requirements sparingly — most behaviour is conditional. If
you find yourself writing many of these, you're probably under-specifying
the triggers and states.

### 2. Event-driven

Triggered by an event.

> When `<trigger>`, the `<system>` shall `<response>`.

Examples:

- When the user submits the registration form, the system shall send a
  verification email to the address provided.
- When the user uploads a file, the system shall display the upload progress
  until the transfer completes.
- When a new message arrives, the mobile app shall display a notification
  badge on the inbox icon.

The trigger should be an observable event the user (or another actor) causes,
not an internal mechanism. "When the user clicks Save" — yes. "When the
debounce timer expires" — that's implementation.

### 3. State-driven

True while a state holds.

> While `<state>`, the `<system>` shall `<response>`.

Examples:

- While the user is offline, the system shall queue any messages they
  compose for later delivery.
- While a download is in progress, the system shall display the elapsed time
  and remaining bytes.
- While the user has unsaved changes, the system shall warn before
  navigating away.

States are durations; events are instants. "Logged in" is a state. "Logging
in" is an event.

### 4. Optional feature

Applies only when a feature, role, or permission is present or enabled.

> Where `<feature>`, the `<system>` shall `<response>`.

Examples:

- Where the user has opted in to notifications, the system shall send a
  weekly summary by email.
- Where the user is a workspace administrator, the admin dashboard shall
  allow them to invite new members.
- Where the integration with the third-party calendar is enabled, the
  system shall display upcoming meetings on the home screen.

Use this when the requirement depends on a configurable capability or a role
that not every user has.

### 5. Unwanted behaviour

Handles things that go wrong.

> If `<unwanted condition>`, then the `<system>` shall `<response>`.

Examples:

- If the verification email cannot be delivered, then the system shall
  notify the user and prompt them to update their email address.
- If the user enters an invalid date range, then the system shall reject
  the input and explain which dates are valid.
- If the user attempts to upload a file larger than 100 MB, then the system
  shall refuse the upload and display the size limit.

Most specs under-cover this category. For every event-driven requirement,
ask: what if it fails? What if the input is wrong? What if a precondition
isn't met?

## Combining patterns

You can combine clauses for compound requirements:

> When the user clicks "Save", while editing a draft, if the draft has
> unsaved changes, then the system shall persist the changes and display
> a confirmation.

Avoid stacking more than two clauses — break complex requirements into
multiple simpler ones instead. A reader who can't tell at a glance whether a
requirement applies has lost the benefit of the syntax.

## Naming the actor

"The system" is fine when behaviour is observed from the product as a whole.
When multiple roles or surfaces exist, name them so each requirement is
attributable:

- "The mobile app shall ..."
- "The admin dashboard shall ..."
- "The notification service shall ..." (only if "service" is part of the
  user-facing model — otherwise this is leaking implementation)

Be consistent within a document. If you start with "the system", stick with
it.

## Numbering

Number requirements so they can be referenced from acceptance criteria, open
questions, and across documents:

> **R3.2** When the user uploads a file larger than 10 MB, the system shall
> display a progress indicator until the upload completes.

A common scheme is **R\<group>.\<number>**, where groups correspond to
sections of the spec (e.g. R1 = Authentication, R2 = Account management).
Pick a scheme and stay with it.

## Anti-patterns

- **Technical implementation:** "The system shall use OAuth 2.0." Rewrite
  as observable behaviour: "The system shall allow the user to sign in
  using their existing Google account."

- **UI prescription:** "The system shall display a modal asking for
  confirmation." Designers own UI. Rewrite: "The system shall require the
  user to confirm destructive actions before applying them."

- **Vague qualifiers:** "shall be fast", "shall be intuitive", "shall be
  reliable". Make it measurable: "shall return search results within 2
  seconds for 95% of queries".

- **Multiple requirements in one:** "The system shall validate the form,
  display errors, and submit on success." Three requirements; split them.

- **Mixed modal verbs:** "The user should be able to ..." Use **shall**
  consistently and put the system as the subject: "The system shall allow
  the user to ..."

- **Negative ubiquitous requirements:** "The system shall not lose data."
  Almost impossible to test as written. Reframe positively, with conditions:
  "While the user has unsaved changes, the system shall persist their work
  every 30 seconds."

- **Implementation-flavoured triggers:** "When the cache expires" or "When
  the worker picks up the job" — these are internal events, invisible to
  any actor. Rewrite around something the user actually does or sees.
