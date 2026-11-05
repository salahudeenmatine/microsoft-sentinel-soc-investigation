# Screenshot evidence

I captured these screens while working through the lab. I renamed them for the repository and redacted my personal email in screenshot 10. The other screenshots haven't been rebuilt or altered.

| File | What it shows |
|---|---|
| `01-data-ingestion.png` | 36 rows in `OktaV2_CL` |
| `02-event-distribution.png` | 9 event types and their counts |
| `03-mirage-timeline.png` | Six Mirage events in timestamp order |
| `04-affected-targets.png` | Role grant, token and accounts targeted by MFA changes |
| `05-actor-ip-comparison.png` | Events and distinct IPs per user |
| `06-deployed-query-and-result.png` | Deployed rolling-window KQL returning 5 actions |
| `07-disabled-analytics-rule.png` | Rule settings, High severity and Disabled status |
| `08-generated-incident.png` | Incident #1 as created, with Mirage and IP entity graph |
| `09-alert-query-result.png` | Alert row with `TotalActions = 5`, `UniqueActions = 5` |
| `10-resolved-incident-redacted.png` | Incident marked Resolved and classified True alert |

Some labels are cut off by the Sentinel interface. The [investigation queries](../queries/03-investigation.kql) and [incident report](../report/incident-report.md) contain the full field values.
