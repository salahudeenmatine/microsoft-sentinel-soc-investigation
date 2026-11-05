# KQL queries

These are the investigation queries, tidied up from the steps I recorded during the lab. Files 01–03 each contain several separate queries. `04-detection-deployed.kql` is the exact KQL from my exported Sentinel rule.

| File | What's inside |
|---|---|
| `01-validation.kql` | Event count, null checks and schema |
| `02-exploration.kql` | Event-type counts, user/IP comparison and event timeline |
| `03-investigation.kql` | MFA reset, Mirage's activity, affected targets, IP correlation and Priya's events |
| `04-detection-deployed.kql` | Rolling-window detection actually deployed in Sentinel |

Run one section at a time from files 01–03. The original workspace has been deleted, so queries that use `OktaV2_CL` need a compatible table to run again.
