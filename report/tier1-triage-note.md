# Tier 1 incident ticket note: Training simulation

Incident: #1 — Suspicious Okta Privileged Activity  
Source: Microsoft Sentinel scheduled rule; simulated Okta records  
Severity: High  
Simulated activity: 2 October 2026 (UTC)  
Alert and incident generated: 3 November 2026  
Resolved: 5 November 2026

## Finding

Five different successful sensitive actions were performed by `mirage@pkwork.onmicrosoft.com` within three minutes, all from `198.51.100.42`. The records show a super-admin grant and new TOTP factor for Mirage, SMS MFA deactivation for Priya Sharma, an MFA reset for the CEO and creation of `Backdoor API Token`.

## Assessment

High-priority privileged-account activity involving possible access abuse, persistence and changes to authentication controls. Initial access, subsequent token use, data theft and full compromise of the affected accounts were not established from the available records.

## Recommended actions

Escalate and preserve logs. Check Mirage's privileges and sessions, investigate and revoke suspicious tokens as appropriate, and verify Priya's and the CEO's identities before restoring trusted MFA settings.

## Disposition

After reviewing the generated alert, I added my findings and classified the incident as True alert before marking it Resolved. This was a controlled training incident; no live containment was carried out.
