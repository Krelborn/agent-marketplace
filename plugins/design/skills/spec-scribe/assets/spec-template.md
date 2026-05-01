# [Feature or Change Title]

> One- or two-sentence summary of what this spec is about. A reader who
> only ever sees this line should know roughly what's being proposed and
> for whom.

## Problem

What is the pain point and whose pain is it? What evidence do we have that
this is worth solving — support tickets, user research, telemetry, sales
feedback? What happens if nothing changes? Why now?

Keep this section grounded in observable user reality, not solution ideas.

## Goals

What this solution is intended to achieve, expressed as outcomes.

- ...
- ...

## Non-goals

What is explicitly out of scope for this work. Helps prevent scope creep
and clarifies expectations for design and engineering.

- ...
- ...

## Background

Relevant context about the current product or environment that a designer,
developer, or QA engineer would need to make sense of the requirements.
Existing flows, related features, prior decisions, constraints from
adjacent systems. Avoid implementation detail.

## Actors

The people and roles who interact with the system. Distinguish them where
they have different needs, permissions, or experiences.

- **End user** — ...
- **Administrator** — ...
- **(Other roles as needed)** — ...

## Scenarios

Narrative descriptions of how an actor experiences the solution. Optional
but valuable for design alignment. Each scenario is a story, not a
specification — written from the user's point of view.

### Scenario: [Short name]

[Step-by-step user journey, written in plain language. "Alex opens the app,
sees the new prompt at the top of their dashboard, taps it, and is taken
to the setup flow ..."]

## Functional requirements

Numbered EARS-syntax requirements, grouped into related areas.

### R1 — [Group name, e.g. Sign-in]

- **R1.1** The system shall ...
- **R1.2** When [trigger], the system shall ...
- **R1.3** While [state], the system shall ...
- **R1.4** Where [feature or role], the system shall ...
- **R1.5** If [unwanted condition], then the system shall ...

### R2 — [Group name]

- **R2.1** ...
- **R2.2** ...

## Acceptance criteria

How do we know we're done? Verifiable, observable outcomes that QA can
check. Reference requirement IDs where helpful.

- A new user can complete sign-up in a single session and receive their
  verification email within 1 minute (covers R1.2, R1.3).
- Attempting to submit the form with an invalid email address produces an
  inline error and prevents submission (covers R1.5).
- ...

## Open questions

Things still to be decided. Tag with an owner and date if known.

- [ ] Should administrators be able to ... — @owner, by [date]
- [ ] What happens when ... — needs input from design

## Glossary

Domain terms used in this document, especially where the team's vocabulary
differs from common usage.

- **Term** — definition.
- **Term** — definition.
