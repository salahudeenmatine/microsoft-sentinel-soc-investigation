# Detection engineering notes: Deployed v1

## Threat hypothesis

The aim was to catch a cluster of sensitive identity-admin actions rather than alert every time someone changed an MFA factor. My hypothesis was that several different successful changes by one actor in a few minutes could point to privileged-account abuse.

## Rule I deployed

The deployed query is in [`queries/04-detection-deployed.kql`](../queries/04-detection-deployed.kql). I also exported the [sanitised ARM rule](rule-export-sanitized.json). I replaced its original Azure rule identifier; the query and settings were retained. I haven't tested re-importing that template.

- Event types: `user.account.privilege.grant`, `system.api_token.create`, `user.mfa.factor.deactivate`, `user.mfa.factor.reset_all`, `user.mfa.factor.update`.
- Outcome: `Success`.
- Grouping: the same `ActorUsername` and `SrcIpAddr`.
- Time window: every event is a potential `WindowStart`. Matching events must fall between `WindowStart` and `WindowStart + 5m`, inclusive.
- Threshold: `dcount(EventOriginalType) >= 3`.
- Result: one highest-count burst per actor/IP, including `TimeGenerated`, total actions, distinct actions and the first/last original timestamps.
- Sentinel settings: High severity, `P1D` frequency, `P3D` lookback, more than 0 results to trigger, 24-hour suppression, a single alert for all matches, account/IP entity mapping and incident creation enabled. I disabled the rule after the incident.

## Why `bin()` missed an event

I first tried `bin(OriginalEventTime, 5m)` to group the five sensitive actions. That split the activity at 10:45 UTC:

| Fixed window (UTC) | Actions |
|---|---:|
| 10:40–10:45 | 1 |
| 10:45–10:50 | 4 |

The action at 10:44:55 fell into the earlier bucket. The query showed 4, not 5. Switching to a rolling window using a self-join picked up all 5 actions from 10:44:55 to 10:47:55.

A later revision briefly omitted `OriginalEventTime >= WindowStart` while changing the time limit to 20 seconds. Without the lower bound, the join could include events before the supposed start. I corrected it before running the final negative test.

The rule checks event types and timing. It doesn't search for labels like `Backdoor API Token` or `attacker-controlled device` from the training messages.

## Known accuracy and engineering limitations

1. Duplicate or tied timestamps can inflate action totals. Each raw event on the left of the self-join creates a window start. Two records with the same actor, IP and timestamp can duplicate the joined window and overstate `TotalActions`; the distinct-type threshold is less affected. I haven't tested deduplication or duplicate event IDs.
2. Requiring one IP can miss an actor moving between networks or proxies. An actor-only join with the IPs shown as context is one possible change, but could add false positives. I didn't deploy or test that version.
3. `dcount()` estimates the number of distinct event types. With only five monitored types, an exact set using `make_set()` and `array_length()` would be clearer, but I didn't test that change.
4. `arg_max(TotalActions, *)` chooses the busiest matching window, which may not have the most distinct types. It also keeps only one burst per actor/IP per evaluation; ties aren't explicitly resolved.
5. Legitimate helpdesk work can trigger the rule when several MFA changes happen close together. A slower or differently ordered attack might evade it. I picked the three-type, five-minute threshold after looking at the training sample, and I don't have production data to measure false positives or missed detections.
6. The table is a custom Okta schema and the event messages are simulated. In a live environment I'd check event types, target fields and administrator roles against the actual Okta connector or parser before relying on them.
7. The DCR changed `TimeGenerated` to ingestion time while I used `OriginalEventTime` for the simulated activity. Running the rule daily with a three-day lookback overlaps the same records and can generate repeat alerts. A live rule would need a schedule and filtering that account for when events arrive, especially late records.
8. The rule groups all matching results into one alert. If several accounts matched, that would make triage less clear than one alert per actor or burst. I didn't set up a notification playbook.

## What I'd change in a v2

- Deduplicate window-start timestamps and use an exact action set.
- Try correlating on actor alone, keeping the IPs in the result for context.
- Normalise usernames, handle empty actor fields and use stronger account identifiers where available.
- Test duplicate timestamps, IP changes, fixed-bucket boundaries and two accounts sharing an IP.
- Add `FirstAction`, `LastAction` and the event types as custom alert details.
- Compare against several days of normal admin activity and change tickets before choosing thresholds for a real deployment.
- Consider a separate single-event alert for a high-risk super-admin role grant.

These are design ideas for a later version. I didn't implement or test them in this project.

## MITRE ATT&CK mapping

| Observed simulated action | Possible ATT&CK technique | What I could confirm |
|---|---|---|
| Super-admin role grant | [T1098.003 — Additional Cloud Roles](https://attack.mitre.org/techniques/T1098/003/) | A successful grant event; I couldn't verify the grantor's permissions or before/after privilege state |
| API token creation | [T1098.001 — Additional Cloud Credentials](https://attack.mitre.org/techniques/T1098/001/) | The token was created; no later token use was established |
| MFA reset, deactivation and enrolment | [T1556.006 — Multi-Factor Authentication](https://attack.mitre.org/techniques/T1556/006/) | Changes to MFA factors; no independent evidence of an MFA bypass |

In Sentinel I configured the broader Privilege Escalation and Persistence tactics, not these technique IDs.
