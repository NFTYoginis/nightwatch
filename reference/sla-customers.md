# sla-customers.md

The customers whose impact **overrides raw metrics**. When an incident's affected scope intersects anyone here, Nightwatch bumps severity by one level (rules STEP 5) and names the at-risk clause in the page/ticket.

This is the file that makes Nightwatch think like an account owner, not just a metrics dashboard. A problem affecting "only" one of these is not a small problem.

## Format

```
### <customer / org name>  — also list their org IDs / domains so signals can be matched
- Tier: <Enterprise / Priority / ...>
- SLA: <the commitment, e.g. 99.9% uptime on core API>
- Protected features: <which features the SLA actually covers>
- Renewal / sensitivity: <renewal date, expansion in flight, exec relationship — why this one stings extra>
- Notify: <account owner / CSM to loop in on customer comms>
```

## Entries (examples — replace with your own)

### Northwind Inc.  — orgs: `org_4812`, `@northwind.com`
- Tier: Enterprise
- SLA: 99.9% on reporting + export features
- Protected features: `/reports/*`, `/export/*`, the analytics dashboard
- Renewal / sensitivity: renews in 6 weeks; expansion deal in flight — any visible outage now is high-stakes
- Notify: Dana (CSM)

### Acme Co.  — orgs: `org_2290`
- Tier: Priority
- SLA: 99.5% on core API, 4-hour response on outages
- Protected features: API write path
- Renewal / sensitivity: largest single account by revenue
- Notify: Sam (account owner)

> Keep this list short and honest. If everyone is "high value," no one is, and the override loses its meaning. These are the accounts where you'd want to be woken up for a problem the raw numbers would let you sleep through.
