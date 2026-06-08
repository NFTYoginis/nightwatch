# routing-table.md

Maps a computed SEV (after any SLA bump) + time of day to a concrete action and owner. **Edit names, channels, and hours to match your team.**

## Business hours

> _Define yours (replace):_ Mon–Fri 09:00–18:00 in the team's primary timezone. Everything else is **off-hours**.

## The table

| SEV | Business hours | Off-hours |
| --- | --- | --- |
| **SEV1** | PAGE primary on-call immediately + post to `#incidents` | PAGE primary on-call immediately (phone/push, not just Slack) + auto-escalate to secondary if unacked in 10 min |
| **SEV2** | PAGE primary on-call | PAGE primary on-call. *(If you run a "no off-hours SEV2 paging" policy, downgrade to a high-priority ticket queued for 1st business hour — set that choice here and be explicit.)* |
| **SEV3** | TICKET → on-call queue, this-week priority | TICKET → queue for next business day. **Do not page.** |
| **SEV4** | TICKET → backlog, low priority | TICKET → backlog. **Do not page.** |

> **Set your off-hours SEV2 policy explicitly.** The single most common small-team mistake is leaving it ambiguous, so Nightwatch either over-pages (burnout) or under-pages (missed SLA). Pick one and write it above.

## Owner-routing overrides

- **Post-deploy correlation:** error onset within **15 min** of a deploy → route to the **deployer** by name, with the deploy time/diff window noted, instead of the whole rotation — *unless SEV1*, which always pages the rotation.
- **Security (`#security` tag):** always also notify the security-aware on-call / channel, in addition to the standard route.
- **Single-customer SEV3:** route to the account/support owner for that customer too, so customer comms can start in parallel.

## Acknowledgement & escalation

- A PAGE unacknowledged for **10 min** escalates to the secondary on-call. Unacked for **20 min** → escalate to the team lead. (Adjust to your roster size; for a true solo on-call, set the secondary to a backup contact or a louder channel.)
