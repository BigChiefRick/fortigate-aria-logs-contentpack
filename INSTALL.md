# Installation Guide

## Prerequisites

- VMware Aria Operations for Logs 8.12 or later
- FortiAnalyzer configured to forward syslog to your Aria Operations for Logs instance
- Admin access to Aria Operations for Logs

## FortiAnalyzer Syslog Forwarding

In FortiAnalyzer, configure syslog forwarding to your Aria Operations for Logs instance:

1. Go to **System Settings** → **Advanced** → **Syslog Server**
2. Add your Aria Operations for Logs IP/FQDN
3. Set format to **Default** (key=value)
4. Recommended: UDP 514 or TCP 514

Verify logs are arriving in Aria before importing the content pack. You should see events with `event_type` containing `v4_2187792b` in the Explorer view.

## Installing the Content Pack

1. Log in to VMware Aria Operations for Logs as an admin
2. Navigate to **Content Packs** (left sidebar)
3. Click **+ Import Content Pack**
4. Browse to `fortigate-aria-logs.vlcp` and select it
5. Click **Import**
6. When prompted, select **Install for all users**

## Verifying Installation

After import:

1. Go to **Content Packs** → confirm **FortiGate** appears with the Fortinet icon
2. Open the **Explorer** view
3. In the **Content Packs** dropdown, check **FortiGate**
4. Run a query against your FortiGate logs and open the **Field Table** tab
5. Confirm `fg_devname`, `fg_srcip`, `fg_action` etc. appear as extracted fields

> **Note:** Extracted fields only apply to events ingested after the pack is installed. Allow a few minutes for fresh logs to flow through before verifying field extraction.

## Accessing Dashboards

1. Navigate to **Dashboards** → **Content Pack Dashboards**
2. Select **FortiGate** to expand the dashboard list:
   - EMS Connectivity
   - User Activity by Site
   - SD-WAN Path Usage
   - Traffic Actions

## Customizing Queries

After installation, individual dashboard widgets can be edited via the gear icon on each widget. Key queries to adapt for your environment:

| Query | Default Value | What to Change |
|-------|--------------|----------------|
| EMS inbound policy | `policyname="FortiClient EMS Inbound"` | Match your EMS DNAT policy name |
| SD-WAN rule | `vwlname="sdr.directinternet"` | Match your SD-WAN rule name |
| FortiClient users | `unauthusersource="forticlient"` | Standard — no change needed |

## Updating the Content Pack

To install a newer version:

1. Go to **Content Packs** → find **FortiGate**
2. Click the gear icon → **Uninstall**
3. Follow the installation steps above with the new `.vlcp` file

> Uninstalling removes the dashboards and alerts. Export any custom dashboards first if you have made modifications.

## Troubleshooting

**Fields not extracting (showing as "No results" in grouped widgets)**
- Confirm the content pack is selected in the Explorer Content Packs dropdown
- Confirm logs are arriving with `event_type` containing `v4_2187792b`
- Wait for fresh log ingestion — extraction does not apply retroactively to historical events

**"No results" on line chart widgets**
- Verify the base query matches your environment (policy names, SD-WAN rule names)
- Check the time range — ensure the dashboard window covers a period with active log ingestion

**Import fails with version error**
- Ensure you are running Aria Operations for Logs 8.12+

**Fields show as encoded hashes instead of fg_* names**
- Uninstall and re-import the pack
- Confirm no other content pack is using the `com.usmr.fg` namespace

## Developer Notes

### Content Pack Format (.vlcp)

`.vlcp` files are JSON with the following top-level structure:

```json
{
  "name": "...",
  "namespace": "com.usmr.fg",
  "contentPackId": "com.usmr.fg",
  "framework": "#9c4",
  "version": "1.6",
  "icon": "<base64 PNG>",
  "extractedFields": [...],
  "queries": [],
  "alerts": [...],
  "dashboardSections": [...],
  "agentClasses": [],
  "aliasFields": [],
  "aliasRules": []
}
```

### internalName Encoding

Aria Operations for Logs uses a specific encoding for extracted field `internalName` values:

```python
import base64

def make_internal_name(namespace, field_name):
    raw = f"@@9_{namespace}{field_name}"
    return base64.b32encode(raw.encode()).decode().lower().replace('=', '0')
```

The namespace last segment is prepended to the `displayName` in the UI. With namespace `com.usmr.fg` and displayName `_devname`, the field renders as `fg_devname`.

### Dashboard Widget groupByFields

For grouped widgets (bar/pie charts) to work, each widget's `chartQuery` must:

1. Include the field definition in the `extractedFields` array within the chartQuery
2. Include a `fieldConstraints` entry with `"operator": "EXISTS"` for the group-by field
3. Reference the field in `groupByFields` with the correct `internalName` and `displayNamespace`

This is not documented — it was reverse-engineered from the VMware Linux content pack.

### Field Extraction Constraint

All fields use a constraint scoping extraction to FAZ-forwarded events:

```json
{
  "filters": [{
    "internalName": "event_type",
    "operator": "CONTAINS",
    "value": "v4_2187792b"
  }]
}
```

If your FAZ forwards with a different `event_type` value, update this constraint in the `extractedFields` array and regenerate the pack.
