# rules.md — Nightwatch decision logic

This is the heart of the operator. Run these steps **in order** for every incoming signal. The first step that produces an outcome wins — stop there and produce the artifact.

> **Core principle:** Severity is a function of **business impact**, not of the alert's own label. A monitoring tool that screams "CRITICAL" about a cosmetic issue is wrong, and a "warning" that means the one customer paying you $80k/year is down is a fire. I recompute every time. The incoming severity field is a hint, never a verdict.

---

## STEP 0 — Normalize the signal

Extract and write down (if absent, mark `unknown`):

- **Signature** — the recurring identity of this alert (e.g. `disk-space-warning:prod-db-1`, `5xx-spike:checkout`). Used for dedup and noise matching.
- **Source** — monitoring tool / log / customer report / human.
- **Raw severity** — what the tool *claims*. Recorded, then ignored for the decision.
- **Affected scope** — all users / a segment / a single customer / internal-only / unknown.
- **Impact level** — hard-down (core function unusable) / degraded / cosmetic / leading-indicator-only.
- **Trajectory** — worsening / stable / self-recovering / unknown.
- **Numbers** — error rate, latency, count, duration, and *for how long*.
- **Timestamp** + whether it is **business hours** (see `reference/routing-table.md`).

---

## STEP 1 — Hard-override gates (check FIRST, short-circuit on match)

These beat everything below, including low numbers. They exist because their downside is asymmetric — a missed one is catastrophic, a false positive merely annoying.

1. **Security signal** — auth anomaly, credential stuffing, unexpected privilege change, data exfiltration pattern, WAF/IDS hit. → **PAGE** and tag `#security`, *even if user impact is currently zero or unknown.* See `reference/escalation-rules` logic below.
2. **Data loss or corruption risk** — failed/missing backup, replication broken, write errors to the primary datastore, integrity-check failure. → **PAGE**, even if only a few users are visibly affected. Lost data doesn't come back.
3. **Active maintenance window** — signature matches a declared maintenance window in `reference/known-noise.md`. → **SUPPRESS** (log it), unless it ALSO trips gate 1 or 2.

If none match, continue.

---

## STEP 2 — Noise check (kill it before it costs a human anything)

Match the signature against `reference/known-noise.md`. SUPPRESS if **all** hold:

- The signature is on the known-noise list, **and**
- It is **self-resolving** within its documented normal window, **and**
- The current numbers are within the documented normal band.

When I SUPPRESS, I always record the **escalate-if condition** from the noise entry (e.g. "but PAGE if free disk < 5% or it fails to clear within 20 min"). Suppression is never unconditional — it's a bet with a tripwire.

**Flap detection:** if the same signature has fired and self-cleared **3+ times in 30 minutes**, SUPPRESS the individual flaps but open **one** TICKET titled "flapping: <signature>" so the underlying instability gets fixed in daylight.

---

## STEP 3 — Deduplicate / group

If an incident with the same signature (or the same root scope) is **already open**, do **not** create a new page or ticket. Attach the signal to the existing incident as a correlated event and, if it materially changes blast radius, append an update to the existing page. One incident, one notification stream.

---

## STEP 4 — Compute severity from the matrix

Using `reference/severity-matrix.md`, combine **blast radius × impact level × trajectory** into SEV1–SEV4. Use the numeric thresholds there, not intuition. Anchors:

- **SEV1** — core function hard-down for all/most users, or worsening fast. (Page, off-hours included.)
- **SEV2** — core function degraded for many, or hard-down for a segment; or any worsening issue you can't yet size. (Page during extended hours; off-hours judgment per routing table.)
- **SEV3** — limited or single-customer impact, stable, has a workaround. (Ticket, business hours.)
- **SEV4** — cosmetic, internal-only, or leading-indicator with no current user impact. (Ticket, low priority.)

---

## STEP 5 — SLA / high-value override

Cross-check affected scope against `reference/sla-customers.md`. If the impact intersects a customer on the SLA list, **bump severity by one level** (SEV3→SEV2, etc.) and note which SLA clause is at risk. This is the deliberate "raw numbers say small, business says big" override. A 0.2% global error rate that is 100% of one enterprise customer is not a 0.2% problem.

---

## STEP 6 — Map to action + route

Apply `reference/routing-table.md`: SEV level + business-hours flag → PAGE or TICKET, to which owner/queue. Special routings:

- **Post-deploy correlation:** if the error onset is within 15 minutes of a deploy, route to the **deployer** specifically (with the deploy window noted), rather than waking the whole rotation — *unless* it's SEV1, which pages the rotation regardless.

---

## STEP 7 — Draft the artifact

Produce the output using `reference/response-templates.md`. Always include a **one-line "why"** — the decision is auditable. A human should be able to read the why and either nod or override in five seconds.

---

## STEP 8 — When (and only when) to FLAG

FLAG for human judgment **only** when **all three** are true:

- The signal is genuinely **novel** (no signature match, no precedent), **and**
- I **cannot size impact** from the available data, **and**
- The trajectory is **unknown or worsening.**

Even then I do **not** go quiet waiting for an answer. The safe default I hold while flagging:

- Worsening + unsizeable → I **PAGE** anyway and *also* flag the open question. (When the trajectory is bad and I'm blind, waking a human is the safe bet.)
- Stable + unsizeable + novel → I open a **TICKET** and flag the open question for daylight.

A FLAG is "here's my best safe action AND the specific thing only you can decide" — never "I don't know, you handle it." If I find myself wanting to FLAG more than rarely, my `reference/` files are missing a rule, and that's the real fix.
