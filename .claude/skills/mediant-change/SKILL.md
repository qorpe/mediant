---
name: mediant-change
description: Run a change to Mediant through the delivery cycle — a feature, a defect, an analyzer or source-generator change. Use when working in the mediant repository on anything that will end in a pull request.
---

# mediant-change — the sequence, for a library other libraries are built on

Mediant is a dependency of the Goldpath platform and of everything built on it. A mistake here
does not stay here: it reaches every generated application through a package version, and the
people who meet it will not be reading this repository.

The sequence is `.claude/cycle.md`, the same nine steps every repository in the family runs.
This skill says what each step MEANS here and carries no rules of its own — the rules are
`CONTRIBUTING.md` and the README.

## Read first

1. `.claude/cycle.md` — the nine steps
2. `CONTRIBUTING.md` — the build, the suites, the public-API baselines, the commit convention
3. `README.md` — what the library promises. The promise is the thing you must not break.

## What each step means here

**3 — what must not break.** The public surface, and it is not a judgement call: it is frozen
in `tests/Mediant.UnitTests/PublicApi/<Assembly>.approved.txt`. If your change alters it, the
baseline test fails by design and tells you so. Changing the baseline is a decision you state
in the pull request, not a step you perform to make a test green.

**4 — prove the test.** Write it, watch it fail, then put the fault back and confirm it goes
red again. Source generators and analyzers make this easy to get wrong: a generator test that
asserts on generated TEXT can pass while the generated code does not compile. Assert on
behaviour where you can, and compile what you generate.

**5 — the layer that can fail.** Unit for dispatch logic; integration for the pipeline and its
ordering; EF Core tests for anything the database decides; the analyzer and generator suites
for diagnostics and emitted code. And the one people forget: **Native AOT**. Reflection- or
trim-unsafe code passes every other layer and fails only in the AOT publish of
`tests/Mediant.AotSample`, which is why CI runs it.

**6 — this repository's contract check.** Not an engine: the approved baselines plus a clean
build. `TreatWarningsAsErrors` and `EnforceCodeStyleInBuild` are on, so **the build is the
style and analyzer gate** — a warning is a failure, and suppressing it to get past the gate is
the one move `CONTRIBUTING.md` asks you not to make.

**7 — run it for real.** A library has no screen, so this step is the real consumer: exercise
the change from a sample or a test host the way an application would call it, and read what it
produced — the dispatched result, the generated source, the SQL. A change proven only by its
own unit test has been proven only against your own assumptions.

**8 — the as-is.** Multi-targeting means a change can be correct on .NET 10 and wrong on 8.
Does an existing test encode the old behaviour? Does the migration guide still describe what
the library does?

**9 — land.** Conventional Commits, one logical change per pull request, and an explicit note
when the public API moved — after 1.0, a breaking change needs a very good reason, and the
pull request is where that reason lives.

## The hook

`.claude/hooks/stop-gate.sh` will not let a turn end on a red build, and it builds only the
projects whose files changed. There is no separate gate script to run: with warnings as errors
and code style enforced in the build, the build already is that gate. The suites and the AOT
publish stay in CI, where they can take their time.
