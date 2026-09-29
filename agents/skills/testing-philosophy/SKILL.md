---
name: testing-philosophy
description: >
  Our stance on test types and boundaries — the testing trophy (favor component/integration tests
  over heavily-mocked unit tests) and concrete heuristics for where to draw a test boundary and
  what to mock. Use this skill whenever writing a new test, deciding what layer a test belongs at
  (unit vs. component vs. e2e), choosing what to mock/stub/fake, or reviewing a PR that touches
  tests — including when the request doesn't name "testing" directly (e.g. "add coverage for
  this," "this test is flaky," "should this be an integration test," a PR review that edits a test
  file). Not for test-framework setup or tooling/CI configuration questions that don't involve a
  boundary or mocking decision.
---

# Testing philosophy

## The shape: testing trophy, not pyramid

We follow Kent Dodds' testing trophy, not the classic test pyramid: static → unit → component/
integration → e2e, widest in the middle. Most of our test effort goes into the component layer —
several real units wired together, exercised through a realistic entry point. We don't target
fixed percentages; the ratio is a consequence of drawing boundaries well, not a goal to hit
directly.

## What we mean by "component test"

A component test exercises a meaningful slice of real behavior — several units cooperating,
unmocked — through the same kind of entry point a real caller would use (a public function, an
HTTP handler, a rendered UI component), asserting on outcomes a caller would actually observe.
This is what Martin Fowler calls a **sociable** unit test, as opposed to a **solitary** one that
mocks every collaborator into silence.

## Where to draw the boundary (the art)

- **Mock what you don't own, not what you do.** The mocking boundary is the edge of *your*
  system: network, filesystem, clock, third-party APIs, external processes. Everything inside
  that edge — your own functions, classes, modules, DB access through your own code — runs for
  real.
- **Test through the same door a real caller uses.** Assert on what's observable from the
  outside (return value, persisted row, rendered DOM node, emitted event), not on internal state
  or call order.
- **A boundary should make sense without narration.** If you can name what the slice does in one
  sentence without saying "and then it calls X which internally does Y," it's a good boundary.
- **Group by what changes together.** If two modules only ever change and ship together, test
  them together rather than subdividing to the class level.

## Where each layer still fits

- **Unit test, with mocking:** a pure function or small algorithm with many edge cases worth
  enumerating (parsing, validation, a calculation) — or the thin adapter *at* the boundary itself,
  where the "real collaborator" is the external thing you don't own.
- **E2e:** the true golden path, plus maybe one or two of the highest-stakes failure modes.
  Nothing else — if a component test could cover the case, use that instead.

## Gotchas

- A "unit test" that mocks every collaborator, including ones the team owns, is testing that the
  mocks were called — not that the code works. Push it to the component layer instead.
- If a test file's `mock`/`patch`/`stub` setup outnumbers its assertions, the boundary is wrong:
  either stop mocking your own code, or the logic genuinely is isolated and belongs at the unit
  layer.
- An e2e test added to cover a case a component test could already cover is ice-cream-cone drift,
  not extra safety — flag it in review.
- Don't compensate for missing static analysis (types/lint) with more tests. Fix the cheap layer
  first.

## Further reading

For the primary sources this stance is grounded in — Kent Dodds, Martin Fowler, Google's Testing
Blog, Testing Library's guiding principles — see `references/FURTHER_READING.md`.
