# Azure ingestion pipeline and validation

## Setup I used

- Subscription: Azure free-credit trial at the time of the lab.
- Region: UK South.
- Resource group: `rg-sentinel-soc-lab`.
- Log Analytics workspace: `law-sentinel-soc-lab`.
- Microsoft Sentinel connected to the workspace; I didn't investigate licensed Defender XDR workload telemetry.
- Data Collection Endpoint: `dce-sentinel-soc-lab`.
- Data Collection Rule: `dcr-okta-soc-lab`, stream `Custom-OktaV2_CL`.
- Custom Analytics-plan table: `OktaV2_CL`.
- Ingestion: Azure Monitor Logs Ingestion API, authenticated with an Entra app registration and client secret, granted Monitoring Metrics Publisher on the DCR only (secret and app later deleted).

## Source and timestamp handling

I used [Microsoft's Sentinel Training Lab `OktaV2_CL.csv`](https://github.com/Azure/Azure-Sentinel/blob/master/Tools/Microsoft-Sentinel-Training-Lab/Artifacts/Telemetry/Custom/OktaV2_CL.csv). The local CSV had 36 rows and 58 columns. The CSV was converted to JSON with AI help, keeping the source's original `TimeGenerated` value as `OriginalEventTime`.

The DCR applied this transform:

```kusto
source
| extend TimeGenerated = now()
```

The fictional events happened on 2 October 2026. They were ingested on 2 November 2026, so `TimeGenerated` represented ingestion time and `OriginalEventTime` kept the original sequence. The scheduled rule used `TimeGenerated` for the lookback and compared `OriginalEventTime` to find actions inside a five-minute window.

The daily run with a three-day lookback was fine for a one-off lab run, wrong for production. A live pipeline should use reliable event timestamps and allow for logs that arrive late, rather than giving historical events the current time.

## What I checked

- The Logs Ingestion API returned HTTP `204` after I submitted 36 events.
- My first Log Analytics count returned 0. I waited for ingestion to finish instead of uploading them again; the next check returned 36.
- `getschema` returned 63 columns after ingestion, including system fields.
- `OriginalEventTime` was typed as `datetime`, and the key username, event and IP fields were readable.
- Missing original timestamps, usernames and IPs: 0 each.
- An offline check of the source CSV confirmed 36 rows, 58 headers and the same event-type counts. This comparison was performed with ChatGPT's help. I didn't check every value in every imported column.

I didn't reingest the sample to change its timestamps.

## Security and cleanup

The client secret was revoked and the app registration deleted. On 5 November 2026, `az group exists` returned `false` for `rg-sentinel-soc-lab`, and `az resource list` showed no active resources left in that subscription. I saved the screenshots and rule export before deletion. The last displayed Azure cost was £0.00.

I haven't included secret values, live Azure IDs or scripts that automatically recreate resources. Rebuilding the lab would need new Azure resources and credentials.
