# examples.md — Nightwatch decisions in action

Each example shows the full decision path. The point is not the format — it's the *judgment*. Watch where the raw numbers say one thing and the right call says another.

---

## Example 1 — The straightforward fire (PAGE)

**Input:**
> [Grafana] CRITICAL — 5xx error rate on `checkout` = 38% sustained 6 min. All regions. Trending up.

**Decision path:**
- STEP 1 gates: no security, no data-loss, no maintenance. Continue.
- STEP 2 noise: not a known-noise signature. Continue.
- STEP 4 severity: blast = all users · impact = hard-down on a core revenue path · trajectory = worsening → **SEV1**.
- STEP 6 route: SEV1 pages the rotation, off-hours included.

**Outcome: PAGE.**
> 🔴 **SEV1 — Checkout down (38% 5xx, all regions, rising)**
> Core revenue path failing for all users since 02:14 UTC, 6 min and climbing. No correlated deploy in the last 15 min. Paging primary on-call now.
> *Why: all-user hard-down on a core function, worsening — textbook SEV1.*

This one is easy. The alert label and the truth agree. Most won't be this clean.

---

## Example 2 — Raw numbers say "ignore," business says "fire" (EDGE CASE → PAGE)

**Input:**
> [Sentry] warning — 0.2% of API requests returning 503 on `/reports/export`. Stable. 1 affected org.

**Decision path:**
- A naive router sees `warning`, `0.2%`, `stable`, `1 org` → suppresses or tickets it. That would be the mistake.
- STEP 1 gates: none. STEP 2: not known noise. STEP 4: blast = single customer · impact = a feature degraded · trajectory = stable → looks like **SEV3 / TICKET**.
- **STEP 5 override:** the affected org is `Northwind Inc.` — on `reference/sla-customers.md`, Enterprise tier, contract clause: 99.9% on reporting features, renewal in 6 weeks. The 0.2% global rate is **100% of their export feature.** Bump SEV3 → **SEV2**.

**Outcome: PAGE (extended-hours rules apply).**
> 🟠 **SEV2 — Report export failing for Northwind (SLA at risk)**
> 503s on `/reports/export`, isolated to Northwind Inc. (Enterprise, 99.9% SLA on reporting, renewal in 6 wks). Globally tiny (0.2%) but it's 100% of their export. Paging per SLA-customer rule.
> *Why: raw volume is small, but business impact intersects a protected SLA customer — severity is about who's down, not how many.*

**This is the case that separates an operator from a dashboard.** The number didn't change; the *meaning* of the number did.

---

## Example 3 — Looks scary, is actually noise (SUPPRESS, with a tripwire)

**Input:**
> [Prometheus] WARNING — disk usage `prod-db-1` at 88%. 03:10 UTC.

**Decision path:**
- STEP 2 noise: signature `disk-space-warning:prod-db-1` is on `known-noise.md`. Documented: nightly log rotation pushes usage to ~90% between 03:00–03:30, then drops to ~70% on cleanup. Current 88% is inside the normal band, inside the window.

**Outcome: SUPPRESS — with the escalate-if recorded.**
> ⚪ Suppressed: `disk-space-warning:prod-db-1` — expected nightly log-rotation spike (normal band ≤92%, clears by 03:30).
> *Escalate-if: PAGE if usage exceeds 92%, or has not dropped below 80% by 03:45, or free space trends below 5%.*
> *Why: known self-healing pattern within its normal window — suppressing the human cost, not the safety net.*

Suppression here is a **bet with a tripwire**, not a blind mute. If the night is genuinely different, the escalate-if catches it.

---

## Example 4 — Ambiguous security signal (EDGE CASE → PAGE, by asymmetric risk)

**Input:**
> [Cloudflare] Spike in 401s on `/auth/login` — 4,000 in 10 min from ~600 IPs. User-facing error rate unchanged. Could be an attack; could be a bad mobile-app release sending stale tokens.

**Decision path:**
- The honest read: **impact is currently low and the cause is genuinely ambiguous.** STEP 8's FLAG conditions look tempting (novel-ish, hard to size). A timid operator tickets it or asks the human "is this an attack?"
- **STEP 1, gate 1 fires first and overrides all of that:** this is a security signal (auth anomaly, distributed). The asymmetric-risk rule in `reference/escalation-rules` applies — the cost of a missed credential-stuffing attack vastly exceeds the cost of one false page.

**Outcome: PAGE + `#security` flag (and name the fork).**
> 🔴 **SEV2 #security — Distributed 401 spike on login**
> 4k failed logins / 10 min from ~600 IPs. Two live hypotheses: (a) credential-stuffing attack, (b) bad mobile release sending stale tokens. User impact currently low but cause unconfirmed. Paging security-aware on-call.
> *First check: correlate IP/user-agent spread vs. a recent mobile release. If it's one app version → likely (b), downgrade. If spread across versions/geos → treat as (a).*
> *Why: security + ambiguous = page. I don't gamble that an attack is "probably nothing," and I hand over the fork to resolve it fast, not the whole question.*

Note what makes this an *operator* and not a chatbot: it didn't ask "what should I do?" It **decided to page**, gave its reasoning, and handed the human a sharp next check — not a blank.
