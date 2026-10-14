---
name: vet
description: Verify each /code-review finding against the code, keep only the real ones, and report each with whether users can see it, what triggers it and how it looks, and how to fix it. Use when the user invokes /vet.
argument-hint: "[low|medium|high|xhigh|max] [target]"
---

# Vet

Run `/code-review`, keep only the real findings, and return them with what they mean for users and how to fix them. Reviews at high and above cast a wide net and mix in low-confidence findings, so this is where the keep-or-drop call gets made.

Change no code. Whether to fix anything is the user's call after reading the report.

## 1. Run the review

Invoke `code-review` with the Skill tool. Pass `$ARGUMENTS` through unchanged; if it contains no level (`low` / `medium` / `high` / `xhigh` / `max`), prepend `high`.

The review runs in a separate subagent, and its findings arrive in the conversation later. Wait for them before moving on.

## 2. Verify

For each finding, read the flagged lines and trace where the value or state comes from and where it goes (callers, callees, the owner of the state). Then classify it as one of:

- **Real**: you can show in the code the condition that triggers it and what breaks when it does
- **False positive**: a misreading / prevented upstream so it never happens / intended behavior (shown by docs or a consistent implementation)
- **Unknown**: the deciding fact cannot be confirmed

When a verdict depends on how a library or framework behaves, confirm it against the source of the version this project pins or the official docs. If you cannot confirm it, classify it as unknown. Filling the gap from memory either drops a real finding or lets a fake one through.

When there are many findings and they are independent of each other, you may split verification across subagents, one per finding, in parallel. For a handful, read them in place.

## 3. Report

List the real findings in order of user impact, largest first. Write each one in this form:

```markdown
### 1. <one-line summary>

`path:line`

- **Visible to users**: yes / under certain conditions / no
- **Trigger and symptom**: <which action or internal event> → <what happens on screen or in behavior>
- **Evidence**: <the code that decided it, and what it shows>
- **Fix**: <the root cause and how to fix it there; if there are several options, their trade-offs>
```

Write "Trigger and symptom" in terms of user actions and what appears on screen (e.g. pressing the send button twice quickly shows the same message twice). An explanation in function names alone does not convey how much it hurts. For findings users cannot see, write what goes wrong internally and where it can spread.

Finally, list the dropped findings one per line with the reason. For a false positive, give which kind; for an unknown, what could not be confirmed. Without visible reasons for the drops, the report cannot be trusted.

If there are no real findings, say so in one sentence, then list the dropped findings.
