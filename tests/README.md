# KQL detection tests

The `datatable()` scenarios were run during the Sentinel lab and their results recorded. Files 01–03 were rebuilt afterwards from those inputs and outcomes because I hadn't saved the editor text as separate files.

The tests copy the detection logic rather than calling the deployed query directly. The exact rule is in [`queries/04-detection-deployed.kql`](../queries/04-detection-deployed.kql).

| File | Test input | Result I recorded |
|---|---|---|
| `01-positive-three-distinct.kql` | Three distinct actions at 10:00, 10:01 and 10:03 | 1 match; `TotalActions = 3`, `UniqueActions = 3` |
| `02-negative-spread-out.kql` | Three different actions at 10:00, 10:07 and 10:14 | 0 rows |
| `03-negative-identical-action.kql` | Three MFA resets in three minutes | 0 rows |
| `04-negative-20-second-window.kql` | Historical query with the window reduced to 20 seconds | 0 rows |

The real Okta dataset separately returned five actions. I didn't run additional duplicate-timestamp, IP-rotation or bucket-boundary regression tests. I haven't run the rebuilt files themselves since the lab.

Files 01–03 use temporary synthetic tables and don't ingest anything. File 04 requires `OktaV2_CL`, which is no longer available in my deleted lab. I can't calculate a real false-positive or false-negative rate from these checks.
