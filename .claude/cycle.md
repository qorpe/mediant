# The delivery cycle

Nine steps. A defect and a feature differ only in the first; everything after is identical,
which is why this is one flow and not two.

This file is the SEQUENCE. It deliberately carries no rules of its own — every rule lives in
the document that owns it (`conventions.md` here, the manifest, the ADRs in the Goldpath repo).
A skill that restates a rule becomes a second source of truth waiting to disagree with the
first.

Skipping a step is a decision, not a shortcut. State it in the merge request with the reason.

## 1 — Prove the cause, or prove the need

**A defect:** evidence, never a hypothesis. Read the logs, query the data to size it, open what
the report attached. "Four orders blocked since Tuesday" is a fact; "orders are failing" is a
feeling. If two explanations fit, rule one out **in writing** before building on the other.

**A feature:** who asked, what evidence there is, what done means, and the part everyone skips
— what is deliberately NOT in scope.

## 2 — Agree before building

Put the diagnosis, or the approach and its trade-off, and wait for an answer. For a feature
this is required, not optional. When the choice is expensive to reverse, record it where this
repository keeps decisions.

## 3 — Read what must not break

The rules this change touches, by id, and the manifest. A change that contradicts a rule or an
ADR is a conversation, never a workaround.

## 4 — Write the failing test, then prove the test

Write it. Watch it fail **for the reason in the issue**. Then do the step that is easy to skip:
put the fault back and confirm the test goes red again.

A test that is green on both sides of a fix is worse than no test, because it advertises a
guarantee it does not provide. Cite the rule id the test proves — a test that names its rule
explains itself to the next reader.

## 5 — Choose the layer that can actually fail

| Layer | Use it for | What it cannot tell you |
|---|---|---|
| Unit | pure logic, mapping, decision rules | anything the database or a layout decides |
| Integration | constraints, unique indexes, transactions, concurrency — on real infrastructure | anything a user sees |
| End-to-end | routing, the rendered screen, what the reader actually gets | behaviour under load or concurrency |
| Mutation | whether the tests above would notice if the code were wrong | whether the behaviour is the right one |

A unique-index collision cannot be reproduced in memory. Clipped text cannot be reproduced
without layout. Pick the layer that can fail; a test in the wrong layer passes for the wrong
reason.

## 6 — Implement, then ask the machine

The smallest change that makes the test pass. Then run **this repository's own contract check**
— whatever proves that what is committed still equals what is built — and get it clean.
**"Done" without that run is not done.**

Every repository has one, and its skill names it, because the check is stack-specific while
this step is not: a generated application asks the engine (`spec_validate` on the manifest,
`spec_drift` on the repository, and the committed OpenAPI in `specs/` re-exported rather than
hand-edited — SPEC0212 exists because that is the one people forget); a library asks its
public-surface files and its own gates. If the change touches a committed contract, the
committed copy changes with it **in the same commit**, never in a follow-up.

## 7 — Run it for real and measure

Automated green is not the same as seen working.

**Where the change has a user-facing surface, drive it.** Bring the stack up, open the screen,
sign in if it asks, and use the feature the way a person would. Then measure rather than
squint: read the computed style, the element's width, the request payload, the row in the
database, the message that was sent.

**Where it has none**, call the real endpoint or trigger the real job and read what it
produced.

"It looks right" is not a result in either case. This step is conditional by surface; it is
never optional.

## 8 — Watch what you did not intend to change

- Does an existing test encode the old contract? Move it deliberately and say so — never delete
  a test to make a build pass.
- Did a field's meaning change while its name did not? Rename it. A flag that no longer means
  what it says is a lie the next reader will believe.
- Does removing a path leave a stale screen, translation or job behind?
- Update every document this change made untrue, in the same change. Stale documentation is
  confidently wrong, which is worse than absent.

## 9 — Land with evidence

The merge request carries what ran and what it produced: the cause with its proof, what was
ruled out, the test results, what you saw when you drove it, and the as-is you checked.

A green local run is not a green pipeline. If the change touches CI, or adds a test that reads
anything outside the project it runs in, run the job the way CI runs it before pushing.

Close an issue only with evidence. An issue you could not reproduce stays open, with what you
ruled out written down.
