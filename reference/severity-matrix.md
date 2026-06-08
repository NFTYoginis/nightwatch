# severity-matrix.md

Severity is computed from three axes, never read off the alert label. Find the row that matches and take the SEV. When two rows could apply, take the **higher** severity.

## The axes

- **Blast radius** — All users · A segment (region/plan/feature cohort) · A single customer · Internal-only.
- **Impact level** — Hard-down (core function unusable) · Degraded (slow/partial, workaround exists) · Cosmetic · Leading-indicator (no user impact *yet*).
- **Trajectory** — Worsening · Stable · Self-recovering.

## The grid

| Blast radius | Impact | Trajectory | SEV |
| --- | --- | --- | --- |
| All / most users | Hard-down | Any | **SEV1** |
| All / most users | Degraded | Worsening | **SEV1** |
| All / most users | Degraded | Stable | **SEV2** |
| A segment | Hard-down | Any | **SEV2** |
| A segment | Degraded | Worsening | **SEV2** |
| A segment | Degraded | Stable | **SEV3** |
| Single customer | Hard-down | Any | **SEV3** *(→ SEV2 if SLA, see step 5)* |
| Single customer | Degraded | Any | **SEV3** |
| Any | Cosmetic | Any | **SEV4** |
| Any | Leading-indicator | Worsening | **SEV3** |
| Any | Leading-indicator | Stable | **SEV4** |
| **Cannot size impact** | unknown | **Worsening / unknown** | **Treat as SEV2 and PAGE** (fail safe, see rules STEP 8) |

## Numeric thresholds (tune these to your stack)

These convert raw metrics into the "impact" axis. **Edit the numbers** — they're starting points for a small SaaS, not law.

- **Error rate on a core endpoint:** ≥5% sustained ≥5 min = Hard-down. 1–5% = Degraded. <1% = not impact on its own (check trajectory).
- **Latency on a core endpoint:** p95 > 4× baseline sustained ≥5 min = Degraded; > 10× or timeouts = Hard-down.
- **Sustained-duration rule:** anything must persist past its threshold for **≥5 min** to count, *unless* it's already Hard-down for all users (page immediately — don't wait out the clock on a full outage).
- **Trajectory = worsening** if the metric has risen across the last 2 sample windows. A single spike that's already receding is Self-recovering.

## What "core" means

Define your **core functions** here — the 2–5 things that, if broken, mean customers can't get value. Everything else is non-core and caps lower by default.

> _Example (replace):_ core = sign-in, checkout, the main dashboard load, the API write path. Non-core = settings page, email digests, the marketing site, exports (unless SLA).
