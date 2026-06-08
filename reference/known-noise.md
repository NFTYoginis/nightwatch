# known-noise.md

The recurring false alarms Nightwatch is allowed to SUPPRESS. **Every entry must carry an escalate-if tripwire** — suppression is a bet with a safety net, never a blind mute. If you can't write a safe tripwire, it doesn't belong here.

Add a row each time a real alert turns out to be noise. This file is what makes Nightwatch quieter over time without making it reckless.

## Format

```
### <signature>
- Normal pattern: <when/why it fires harmlessly>
- Normal band: <the metric range that is OK>
- Normal window: <the time window it self-resolves within>
- Escalate-if: <the condition that flips it back to PAGE/TICKET>
```

## Entries (examples — replace with your own)

### disk-space-warning:prod-db-1
- Normal pattern: nightly log rotation spikes usage 03:00–03:30, cleanup drops it back.
- Normal band: ≤92%.
- Normal window: clears below 80% by 03:30.
- Escalate-if: exceeds 92%, OR not below 80% by 03:45, OR free space trends below 5%.

### healthcheck-flap:worker-pool
- Normal pattern: workers briefly fail healthcheck during autoscale-up, then pass.
- Normal band: ≤2 consecutive failures per instance.
- Normal window: passes within 90 sec.
- Escalate-if: ≥3 consecutive failures, OR more than 25% of the pool failing at once, OR queue depth rising.

### 3rd-party-webhook-timeout:stripe
- Normal pattern: Stripe webhook retries occasionally time out and succeed on retry.
- Normal band: <5 timeouts / 10 min, all eventually delivered.
- Normal window: delivered within Stripe's retry schedule.
- Escalate-if: ≥5 timeouts / 10 min, OR any payment-affecting event undelivered after final retry. *(Money path — bias to escalate.)*

## Maintenance windows

Declare planned maintenance here so Nightwatch suppresses the expected fallout. **Maintenance suppression still yields to the security and data-loss hard-override gates** (rules STEP 1).

```
### <window name>
- Services: <what's affected>
- When: <start–end, with timezone>
- Expected alerts: <signatures to suppress>
```
