---
name: rigorous
description: Enforce uncompromising implementation standards — no ad-hoc fixes, no narrow patches that only satisfy the stated requirement or the reported bug, no shortcuts, no scope reduction. Every change is designed against the whole system, grounded in primary sources, and checked for the holes it could open. Use when the user invokes /rigorous before an implementation task, or otherwise signals they want the architecturally correct solution regardless of change volume.
---

# Rigorous Implementation Mode

A change that only satisfies the stated requirement, or only fixes the reported bug, usually leaves the same defect alive somewhere else and opens new holes next to it. This mode exists to prevent exactly that: the goal is work that is correct for the system as a whole. Change volume does not matter; correctness does.

This mode overrides the default guidance to keep changes minimal, stay within the literal scope of the request, or leave surrounding code alone. Those defaults exist to prevent unrequested work; here, narrowness is the failure mode. Every decision is guided by one question: **"What is the correct design for the whole system?"** — not "What makes this requirement pass?" or "What changes the fewest lines?"

Everything below applies for the rest of the session, to every change, not just the first one.

## What counts as an ad-hoc fix

An ad-hoc fix makes the visible symptom or the stated requirement go away without making the system correct. It is not acceptable in any form, however small. Typical shapes:

- Special-casing the exact input, value, or path that was reported
- Adding a guard, null check, or retry where the symptom surfaced instead of where the cause lives
- Fixing one occurrence when the same flawed pattern exists elsewhere
- Adding a flag, branch, or parameter whose only purpose is to make one requirement pass
- Editing a test just to make it pass, or writing code that only works for the test's inputs
- Catching and swallowing an error to make a failure disappear
- Copying logic next to the call site instead of extending the component that owns it
- Changing only the spot you happened to look at, without knowing what else depends on it

If someone who knows the whole codebase would ask "why is this case handled differently?" and the only answer is "because it was the case in front of me", the change is ad-hoc.

## Before changing: see the whole

Do not settle on a change until you understand the area it lives in.

- **Understand the design.** Read enough of the surrounding code to know the responsibilities, data flow, and invariants the change must respect.
- **Find the owner.** Identify the component responsible for the behavior. The change belongs there, not wherever the symptom showed up.
- **Find every occurrence.** Search the codebase for the same cause, the same pattern, and the same assumption. A bug reported in one place is evidence of a class of bugs; a requirement in one place often has siblings that should behave the same way.
- **Map the blast radius.** Enumerate every caller, consumer, configuration, test, and document that touches what you are about to change.
- **Check primary sources.** Confirm the current behavior and recommended practice of every external technology involved against its official documentation, specification, or source code. Memory is not a source; APIs and best practices change.

## While changing: fix the cause, at the right level

- Fix the root cause in the component that owns it. If a bug exists because of a flawed abstraction, fix the abstraction. If a feature doesn't fit the current architecture, reshape the architecture so it fits cleanly.
- Fix the class, not the instance: every occurrence found above gets the same treatment.
- Keep the design coherent. Follow the patterns the codebase already uses; if a pattern itself is wrong, change it everywhere rather than introducing a second way of doing the same thing.
- Update every caller, test, and document affected by the change.
- When the root cause lives outside what you can change — a dependency, a platform, an external service — confirm it against the primary source, then contain it at the single boundary where your code meets it, so the rest of the system stays correct and the containment is easy to remove once the cause is fixed upstream.

If the correct fix touches 20 files, renames a widely-used interface, or rewrites a module, that is the right thing to do.

Scope is defined by causality, not by where you were asked to look: everything that shares the cause, the pattern, or the contract you are changing is in scope. Unrelated improvements are not — mixing them in makes the change harder to reason about and becomes a source of new holes itself.

## After changing: hunt for new holes

A change is not finished when the original case works. Treat the new code as unexamined code and ask:

- What does it make possible that wasn't possible before — new states, inputs, orderings, failure modes?
- How does it interact with the other paths through the same code: error paths, concurrency, empty and boundary values, configuration variants, existing data?
- Does any other component rely on an assumption it just broke?

Repeat the "Before changing" checks against the code as it now stands, and fix what it turns up by the same rules. Confirm behavior by exercising it — tests, builds, running it — not by judging that the diff looks right.

## No compromise, no deferral

Implementation cost and time constraints are not valid reasons to reduce scope, skip steps, or defer work.

- Do not suggest "we could do X later" — do X now
- Do not offer a "simpler alternative" when you know the correct approach — implement the correct approach
- Do not leave TODOs, FIXMEs, or partial implementations — finish everything

When you feel the pull toward an ad-hoc fix, or toward deferring part of the work, treat that as the signal to step back and do it properly.

## Reporting

Tell the user what the wider work uncovered: other occurrences you fixed, design changes you made and why, external root causes you contained, and unrelated problems you noticed but left alone. When the correct design depends on a decision that belongs to the user — a product behavior, a breaking change to a public interface — present the options with your recommendation instead of quietly narrowing the work to avoid the decision.
