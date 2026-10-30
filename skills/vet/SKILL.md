---
name: vet
description: Verify each /code-review finding against the code, keep only the real ones, and report each with whether users can see it, what triggers it and how it looks, and how to fix it, ending with a recommendation of which findings to fix and how. Use when the user invokes /vet.
argument-hint: "[low|medium|high|xhigh|max] [target]"
---

# Vet

Run `/code-review`, keep only the real findings, decide which of them are worth fixing, and return them with what they mean for users and how to fix them. Reviews at high and above cast a wide net and mix in low-confidence findings, so this is where the keep-or-drop call gets made.

Change no code. The user reads the report to decide what to fix, so it must end with your recommendation of which findings to fix and how; the user then applies it or overrides it.

## 1. Run the review

Invoke `code-review` with the Skill tool. Pass `$ARGUMENTS` through unchanged; if it contains no level (`low` / `medium` / `high` / `xhigh` / `max`), prepend `high`.

The review runs in a separate subagent, and its findings arrive in the conversation later. Wait for them before moving on.

## 2. Verify

For each finding, read the flagged lines and trace where the value or state comes from and where it goes (callers, callees, the owner of the state). Then classify it as one of:

- **Real**: you can show in the code the condition that triggers it and what breaks when it does
- **False positive**: a misreading / prevented upstream so it never happens / intended behavior (shown by docs or a consistent implementation)
- **Unknown**: the deciding fact cannot be confirmed

When a classification depends on how a library or framework behaves, confirm it against the source of the version this project pins or the official docs. If you cannot confirm it, classify it as unknown. Filling the gap from memory either drops a real finding or lets a fake one through.

When there are many findings and they are independent of each other, you may split verification across subagents, one per finding, in parallel. For a handful, read them in place.

## 3. Decide

For each real finding, decide whether to fix it, weighing:

- How much it hurts: who sees it and what breaks
- How likely it is: whether ordinary use triggers it or it needs rare conditions to line up
- Whether the change under review introduced it or it was already there
- What the fix costs and what else it could break

Give each finding one of three verdicts:

- **Fix**
- **Skip**: with the reason it is not worth fixing now
- **Your call**: when the decision turns on something only the user knows (a product requirement, a trade-off they own); state the question they need to answer

Then look across the findings marked Fix: which share a root cause and are resolved by one change, which fixes interact, and the order to apply them in.

## 4. Report

List the real findings in order of user impact, largest first. Write each one in this form:

```markdown
### 1. <one-line summary>

`path:line`

- **Visible to users**: yes / under certain conditions / no
- **Trigger and symptom**: <which action or internal event> → <what happens on screen or in behavior>
- **Evidence**: <the code that decided it, and what it shows>
- **How to fix**: <the root cause and the fix you recommend there; mention an alternative only when it differs materially, with its trade-off>
- **Verdict**: Fix / Skip / Your call — <the reason>
```

Write "Trigger and symptom" in terms of user actions and what appears on screen (e.g. pressing the send button twice quickly shows the same message twice). An explanation in function names alone does not convey how much it hurts. For findings users cannot see, write what goes wrong internally and where it can spread.

Next, list the dropped findings (false positives and unknowns) one per line with the reason. For a false positive, give which kind; for an unknown, what could not be confirmed. Without visible reasons for the drops, the report cannot be trusted.

End the report with the conclusion:

```markdown
## Conclusion

- **Fix**: #1, #3 — <reason, e.g. one change at the shared root cause resolves both>
- **Skip**: #2 — <reason>
- **Your call**: #4 — <the question to answer>
- **Plan**: <the order and approach for applying the fixes>
```

Keep it to a few lines, refer to findings by number instead of restating them, and leave out any line with nothing to put in it. When any finding is marked Fix, close with one sentence asking whether to proceed with the plan.

If there are no real findings, say so in one sentence, then list the dropped findings.
