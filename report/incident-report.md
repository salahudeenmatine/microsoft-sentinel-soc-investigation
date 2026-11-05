# SOC Investigation Report — Simulated Okta Privileged Activity

Incident ID: 1  
Rule: Suspicious Okta Privileged Activity  
Platform: Microsoft Sentinel / Azure Log Analytics  
Data: Microsoft Sentinel Training Lab Okta records (`OktaV2_CL`, 36 events)  
Simulated incident date: 2 October 2026 (UTC)  
Incident generated: 3 November 2026  
Resolved: 5 November 2026  
Sentinel severity: High  
Final classification: True alert

## 1. Executive summary

The investigation covered six successful Okta events performed by `mirage@pkwork.onmicrosoft.com`. The account received a super-admin role grant, created an API token and changed MFA settings affecting other accounts. Five of those actions happened within three minutes from `198.51.100.42`.

The sequence was treated as a high-priority privileged-identity abuse scenario in the training data. The MFA changes involving Priya Sharma and the CEO were more significant than the raw number of events.

## 2. Investigation scope and methodology

Microsoft's 36-event Okta sample was imported into the custom `OktaV2_CL` table. My investigation covered the event count, field types, missing values, event distribution, accounts, IPs, `OriginalTarget` values and timestamps.

The source CSV had 36 rows and 58 columns. Log Analytics returned 36 records and 63 columns after ingestion, including system fields. There were no missing `OriginalEventTime`, `ActorUsername` or `SrcIpAddr` values. A complete field-by-field comparison of the imported rows wasn't performed.

Event sequencing used `OriginalEventTime`. The [pipeline notes](../ingestion/pipeline.md) explain why the lab's `TimeGenerated` field reflects ingestion instead.

## 3. Timeline of confirmed logged events

| Time (UTC), 2 October 2026 | Actor | Action and target | Result |
|---|---|---|---|
| 10:43:55 | Mirage | Okta session start; Salesforce app user | Success |
| 10:44:55 | Mirage | Super-admin role granted to Mirage | Success |
| 10:45:55 | Mirage | Token named `Backdoor API Token` created | Success |
| 10:46:55 | Mirage | SMS MFA deactivated for Priya Sharma | Success |
| 10:47:25 | Mirage | All MFA factors reset for CEO PKWork | Success |
| 10:47:55 | Mirage | TOTP factor enrolled for Mirage; event text describes an attacker-controlled device | Success |

All six events came from `198.51.100.42`. This is a TEST-NET-2 documentation IP, not an address I could attribute to a real attacker or location.

## 4. Other activity and affected accounts

### Priya Sharma

Priya performed 13 events, with one additional event showing Mirage deactivating her SMS MFA. Her own activity ran from 10:23:22 to 11:15:22 and included sessions, MFA authentications, OAuth grants and sign-on policy evaluations. Those events came from 13 different IPs.

The rapid IP changes initially stood out. Looking across the other accounts showed that every account except Mirage used a different IP in every event. That pattern wasn't strong enough to treat as evidence of compromise on its own.

### CEO PKWork

The reset of all MFA factors targeted the CEO. Earlier successful OAuth token grants were also associated with the CEO, but the logs didn't establish a connection between those grants and Mirage's actions.

### Mirage and the API token

The logs record a super-admin grant targeting Mirage and successful creation of `Backdoor API Token`. The grantor's permissions and Mirage's existing admin rights before the self-targeted grant were not established. The record also doesn't show the token being used.

## 5. Assessment and proposed severity

The sequence involved privileged role changes, token creation and changes to other users' authentication controls. An equivalent incident would warrant immediate escalation.

| Level | Meaning here |
|---|---|
| High | The actual Sentinel rule and generated incident severity; High is the highest category in the portal |
| Potential P1 / Critical business priority | A proposed internal priority if unauthorised super-admin activity and executive MFA tampering were confirmed in a real organisation; I had no organisation-specific severity matrix |

## 6. Recommended response (not executed)

1. Escalate the case and preserve the Okta audit records.
2. Verify the grantor and Mirage's current permissions; restrict the account and revoke sessions if authorised.
3. Revoke or investigate the new API token and check for any use.
4. Verify Priya's and the CEO's identities, review their sessions and restore trusted MFA methods.
5. Look for initial access clues, session identifiers and additional persistence.
6. Search for other sensitive identity changes and record the findings in the ticket.

These were proposed actions for a real SOC response. No real users or accounts were changed.

## 7. Detection engineering and results

The rolling-window KQL rule looked for three or more distinct successful sensitive actions by the same username and IP within five minutes. It matched Mirage with `TotalActions = 5` and `UniqueActions = 5`, from 10:44:55 to 10:47:55.

The earlier `bin(OriginalEventTime, 5m)` attempt split the actions between buckets of one and four. A rolling window using each event as a possible starting point fixed that problem. Synthetic positive and negative cases exercised the time and distinct-type conditions; the standalone test files were reconstructed from the recorded inputs and results afterwards.

The rule ran once per day and looked back three days to pick up the recently ingested sample. It generated one alert and Incident #1. After checking the account, IP and counts, I added an investigation note, classified the incident as True alert and marked it Resolved. The rule was disabled after the test.

## 8. Limitations

The investigation used just 36 training events, too few to establish normal account behaviour or calculate a meaningful false-positive rate. The synthetic message labels, including `Backdoor API Token` and the attacker-controlled device description, also reveal what the scenario is meant to show. My detection did not use those labels.

The available records couldn't confirm Mirage's initial access, lasting admin privileges, whether the token was used, the TOTP device's real provenance, data theft or a full takeover of Priya's or the CEO's account.

The [engineering notes](../detection/engineering-notes.md) explain the separate detection issues: duplicate counts, IP changes, timing, lookback, event grouping and other production gaps.

## 9. Outcome

The project finished with an investigated incident, a recorded triage decision and a scheduled detection that produced the expected alert from the training data. The evidence and source files are saved here for review. The strongest learning point was fixing the five-minute bucket boundary problem rather than accepting the first query result.

Supporting evidence: [KQL](../queries/README.md) · [Detection notes](../detection/engineering-notes.md) · [Screenshots](../screenshots/README.md) · [Official source](https://github.com/Azure/Azure-Sentinel/blob/master/Tools/Microsoft-Sentinel-Training-Lab/Artifacts/Telemetry/Custom/OktaV2_CL.csv).
