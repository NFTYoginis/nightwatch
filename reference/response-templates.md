# response-templates.md

The formats Nightwatch fills in. Every output ends with a one-line **why** so the decision is auditable and overridable in seconds. Keep them short — a page read at 3am must be scannable.

## PAGE

```
🔴 SEV<n>[ #security] — <one-line headline: what's broken, for whom>
<scope + key metric + since when + trajectory>. <correlated deploy? SLA at risk? known unknowns?>
<the single most useful first check, if there is an obvious one>.
Why: <the one sentence that justifies waking a human>.
```

## TICKET

```
🟡 SEV<n> — <headline>
Impact: <who / what, and the workaround if any>
Evidence: <metric, source, timestamp>
Suggested owner: <queue / person>
Why: <why this waits for daylight instead of paging>.
```

## SUPPRESS (log line)

```
⚪ Suppressed: <signature> — <which known-noise rule matched>.
Escalate-if: <the tripwire copied from the noise entry>.
Why: <known self-healing pattern within its normal window>.
```

## FLAG (rare — genuine human judgment call)

```
🟣 FLAG — <the decision that is genuinely yours to make>
What I see: <the facts, honestly including what I can't determine>.
What I've done meanwhile: <the safe default I'm holding — usually PAGE if worsening, TICKET if stable>.
What I need from you: <one specific question, not "what should I do?">.
Why I can't call it: <the missing rule or data — so you can add it to reference/ and I won't flag it again>.
```

## Severity emoji key

🔴 SEV1 · 🟠 SEV2 · 🟡 SEV3 · ⚪ SEV4 / suppressed · 🟣 FLAG
