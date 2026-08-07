# Cold Chain Fault Response Automation

An autonomous IoT monitoring and remediation workflow built with n8n. Designed for retail cold chain management to detect, classify, auto-resolve, or escalate device failure events in real time.

## Architecture & Workflow Logic

1. **Ingestion**: Listens for device fault signals via Webhook (`/fault-signal`).
2. **Classification**: Maps fault codes to categories and determines autonomous fix eligibility.
3. **Playbook Lookup**: Retrieves remediation protocols from Google Sheets.
4. **Decision Gate**: Evaluates `autoResolvable` flags and severity thresholds (`CRITICAL` severity always routes to escalation, regardless of the fault's auto-fix eligibility).
5. **Path A (Auto-Resolve)**:
   - Dispatches remote fix commands via REST API.
   - Waits 60 seconds before proceeding. *(Note: this is a fixed delay, not a verification step — the workflow does not currently re-check the device's status after the wait. Adding an explicit confirmation call is a planned improvement.)*
6. **Path B (Escalation)**:
   - Constructs a structured diagnostic brief for field technicians.
7. **Audit & Alerting**: Logs every incident to Google Sheets and sends a notification email via Gmail.

## Known Limitations

- No automated verification that a fix actually resolved the fault — the 60-second wait is a placeholder, not a stabilization check.
- No retry or fallback logic if the remediation API call or the Google Sheets lookup fails.
- Unrecognized fault codes default to `Unknown` / `ESCALATE`, but no distinct alert is raised for this case.
- Notifications are sent via Gmail only.

## Setup & Deployment

1. Import `workflow.json` into your n8n instance.
2. Configure credentials for **Google Sheets API** and **Gmail**.
3. Ensure the target Google Sheet contains `Resolution_Playbook` and `Incident_Log` tabs.
4. Activate the workflow.

## Testing Payloads

### Scenario A: Auto-Resolvable Temperature Spike
```json
{
  "deviceId": "FRG-0042",
  "deviceType": "walk-in-freezer",
  "faultCode": "ERR_TEMP_HIGH",
  "currentTemp": -6,
  "targetTemp": -18,
  "severity": "high",
  "location": "Branch-AUH-007",
  "timestamp": "2026-07-10T09:14:22Z"
}
```
