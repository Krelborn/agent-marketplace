---
name: spec-scribe
description: >
  Interview the user to draft or refine a non-technical functional requirements
  document (a "spec") that captures the problem and the requirements for a
  solution in EARS syntax. The output is for designers, developers, and QA to
  share understanding — never code-level implementation detail. Use this skill
  whenever the user mentions writing or updating a spec, PRD, functional
  requirements document, requirements doc, or product brief, or says things
  like "help me spec out X", "draft requirements for this feature", "interview
  me about the new feature", "write me a SPEC.md", or "I'm working with a PM
  and need to capture what they want".
---

# Spec Scribe

Help the user write or revise a functional requirements document by
interviewing them, capturing what they want in plain user-facing language, and
expressing requirements in EARS syntax. The user is usually a product manager
or someone working with one. The audience for the finished document is the
design, development, and QA teams who will build and verify the solution.

## Disposition

A spec is a product artefact, not a technical one. Even if the codebase, mocks,
or a prototype are sitting right there, never let implementation detail leak
into questions or output:

- **No technologies, libraries, frameworks, data structures, or APIs** in
  questions or in the document. Not OAuth, not Postgres, not "a REST endpoint",
  not "a modal", not "a debounce". If the user offers one, gently redirect to
  the underlying user-facing requirement.
- **No code references** in the spec. Code is for context-gathering only.
- **Describe behaviour, not mechanisms.** "The system shall remember the user
  between sessions" — not "the system shall store a session token in a cookie".
- **Describe outcomes, not UI.** "The system shall require confirmation before
  destructive actions" — not "the system shall show a modal with a Yes/No
  button". UI choices belong to design.

Be a sharp, curious peer — not a stenographer. The user will be vague,
contradict themselves, and skip steps. Push back politely. Walk every branch
of the design tree until requirements are concrete and testable. If the user
says "obviously", "just", or "simply" — that's a flag for an unstated
assumption worth surfacing.

## Workflow

### 1 — Gather context

Before asking the first interview question, build a picture of the current
product and any supporting materials.

- **Codebase**: skim top-level structure, README, and any existing product
  docs. The point is to understand *what the product does today*, not how it's
  built. Note features, terminology, and obvious gaps.
- **Supporting materials**: ask the user what they have. Designs, mocks,
  prototypes, customer feedback, prior PRDs, Jira tickets, transcripts. Read
  whatever is offered.
- **Existing spec**: if a SPEC.md or similar already exists, read it
  thoroughly. The session may be a revision rather than a fresh draft.

Then summarise back to the user in two or three sentences what you understand
the product to be and what materials you have to work from. Confirm before
moving on.

### 2 — Frame the problem

Interview the user to understand **why** something needs to change before
discussing **what** the change is.

Ask about:

- The pain point — what's broken or missing today?
- Whose pain it is — which user, role, or persona?
- Evidence — how does the user know this is a real problem? Support tickets,
  user research, sales feedback, telemetry, gut feel?
- Stakes — what happens if nothing changes? Why now?
- Goals — what would success look like, ideally measured?
- Non-goals — what are we *not* trying to fix here?

Take one thread at a time and follow it down. If the user says "users are
frustrated", don't accept it — *which* users, *what* are they trying to do,
*how* does the frustration manifest? Don't move on until the problem is
articulated in concrete, observable terms.

When the problem feels solid, restate it back and confirm before moving to
requirements.

### 3 — Define requirements

Now interview about **what** the solution needs to do — still in user-facing
terms. Walk the tree depth-first:

- Start with the core happy-path capability. What does the user do, what does
  the system do in response?
- Branch into edge cases. What happens with no data? Bad data? Slow network?
  Conflicting actions from the same user? From different users?
- Cover state changes. What's true *while* something is in progress? *After*
  it completes? *If* it fails?
- Surface every actor. The end user is usually obvious; admins, support
  agents, automated jobs, and external integrators are easy to forget.
- Pin down measurable thresholds. "Fast" is not a requirement. "Returns
  results within 2 seconds for 95% of queries" is.

For each branch, capture the requirement mentally in EARS form (see
`references/ears-syntax.md`) before moving on. If a requirement won't fit any
EARS pattern cleanly, that's a signal it's vague or contains multiple
requirements — split it.

Watch for the user solutioning. "We'll add a button that..." is the user
designing, not specifying. Redirect: "Setting aside how it looks, what does
the user need to be able to *accomplish* at that point?"

Keep going until you've genuinely run out of branches. Common signs you're
*not* done:

- Open questions you've made silent assumptions about
- Acceptance criteria you couldn't write a test for
- Actors mentioned but not characterised
- Words like "etc.", "and so on", "things like" in your notes

### 4 — Draft SPEC.md

Once you and the user agree the problem and requirements are comprehensive,
summarise the whole picture out loud (problem, goals, actors, headline
requirements, open questions). Confirm one last time, then write the document
to `SPEC.md` in the working directory.

Use `assets/spec-template.md` as the structural starting point. Drop sections
that don't apply — don't pad. Number requirements so acceptance criteria can
reference them.

### 5 — Review and refine

Tell the user the spec is written and invite feedback. Then iterate:

- Re-interview on any sections they want changed.
- Add, remove, or reword requirements.
- Re-summarise what changed and rewrite the relevant sections of `SPEC.md`.

Continue until the user is happy or wants to end the session. Each iteration
should be a focused refinement, not a full restart.

## Writing requirements in EARS syntax

EARS (Easy Approach to Requirements Syntax) gives every functional requirement
a recognisable shape. There are five patterns: ubiquitous, event-driven,
state-driven, optional-feature, and unwanted-behaviour, plus combinations.
Every requirement uses the modal verb **shall** — never "should", "will",
"must", or "may".

For the full pattern set, examples, and anti-patterns, read
`references/ears-syntax.md` before drafting requirements.

A quick taste:

- **Ubiquitous:** The system shall display the user's name in the page header.
- **Event-driven:** When the user submits the registration form, the system
  shall send a verification email to the address provided.
- **Unwanted behaviour:** If the verification email cannot be delivered, then
  the system shall notify the user and prompt them to update their address.

## Document structure

A spec built with this skill should look like the template at
`assets/spec-template.md`. The mandatory sections are **Problem** and
**Functional requirements**; everything else is included only when it adds
clarity for the design / dev / QA audience.

## Common pitfalls

- **Drifting technical.** "The system shall validate the JWT" — wrong, that's
  implementation. "The system shall reject requests from unauthenticated
  users" — right.
- **Solutioning the UI.** "The system shall show a tooltip with..." —
  designers haven't been consulted yet. State the requirement, not the UI.
- **Vague qualifiers.** "fast", "easy", "intuitive", "reliable" — make every
  qualifier either measurable or remove it.
- **One requirement, multiple shalls.** "The system shall validate the form,
  display errors, and submit on success" is three requirements. Split.
- **Hidden assumptions.** "Obviously the user is logged in" — write that down
  as a precondition or as its own requirement.
- **Missing unwanted-behaviour clauses.** Most specs over-specify the happy
  path. For every event-driven requirement, ask: what if it fails?
- **No actors.** "The user" is rarely sufficient. End user, admin, anonymous
  visitor, support agent, and integrator may all need different requirements.

## Important behaviours

- **Interview, don't dictate.** The user knows the product and the people
  using it. Your job is to extract their thinking and pin it down precisely,
  not to invent requirements.

- **One or two questions at a time.** Don't fire a barrage. Take a thread,
  follow it, then pick the next.

- **Surface assumptions, gently.** "You said the user is logged in — should
  we capture that as a precondition, or is anonymous access in scope too?" is
  better than letting it slide or accusing the user of being vague.

- **Stay in the user's domain.** Use the language of their product and their
  users. If the codebase says `Account` and the PM says `Customer`, ask which
  the spec should use, and use it consistently.

- **Don't make stuff up.** If a requirement isn't grounded in what the user
  has said, don't invent it to fill a section. Mark it as an open question
  instead.

- **Stop when the user wants to stop.** Specs are iterative. A good-enough
  document the user can hand off beats a perfect one they abandoned.
