# FortiGate Content Pack for VMware Aria Operations for Logs

A community content pack for VMware Aria Operations for Logs that provides extracted fields, dashboards, and alerts for FortiGate firewall logs forwarded via FortiAnalyzer syslog.

## Features

- **47 extracted fields** covering all key FortiGate traffic and UTM log attributes
- **4 dashboards** with 22 widgets:
  - EMS Connectivity — FortiClient EMS traffic, actions, users, source IPs
  - User Activity by Site — FortiClient endpoint sessions, users, OS/device breakdown
  - SD-WAN Path Usage — SD-WAN rule matching, SNAT egress IPs, destination countries
  - Traffic Actions — Forward traffic volume, action distribution, denied destinations, WAN inbound
- **4 alerts** for EMS resets, denied traffic spikes, elevated-risk apps, and high EMS inbound rate
- Fortinet branding icon

## Requirements

- VMware Aria Operations for Logs (tested on 8.12+)
- FortiGate firewalls running FortiOS 7.x
- FortiAnalyzer forwarding syslog to Aria Operations for Logs
- Logs must arrive with `event_type` field containing `v4_2187792b` (standard FAZ syslog format)

## Architecture

```
FortiGate → FortiAnalyzer → Syslog → Aria Operations for Logs
```

Logs are forwarded from FortiAnalyzer in standard key=value format. The content pack uses regex-based field extraction scoped to FAZ-forwarded events via the `event_type` constraint.

## Installation

See [INSTALL.md](INSTALL.md) for full installation instructions.

## Extracted Fields

All fields are prefixed with `fg_` in the Aria UI (e.g. `fg_devname`, `fg_srcip`, `fg_action`).

| Field | Description |
|-------|-------------|
| fg_devname | FortiGate device name |
| fg_devid | FortiGate device serial number |
| fg_type | Log type: traffic, utm, event |
| fg_subtype | Log subtype: forward, app-ctrl, vpn |
| fg_policyid | Firewall policy ID |
| fg_policyname | Firewall policy name |
| fg_srcip | Source IP address |
| fg_srcport | Source port |
| fg_srcintf | Source interface |
| fg_srcname | Source endpoint hostname |
| fg_srcintfrole | Source interface role: wan, lan, dmz |
| fg_dstip | Destination IP address |
| fg_dstport | Destination port |
| fg_dstintf | Destination interface |
| fg_dstintfrole | Destination interface role |
| fg_action | Traffic action: allow, deny, client-rst, server-rst |
| fg_proto | IP protocol number |
| fg_service | Matched service object |
| fg_duration | Session duration in seconds |
| fg_sentbyte | Bytes sent |
| fg_rcvdbyte | Bytes received |
| fg_srccountry | Source IP country |
| fg_dstcountry | Destination IP country |
| fg_user | Authenticated AD user |
| fg_unauthuser | FortiClient identified user |
| fg_unauthusersrc | User identity source |
| fg_srcdomain | Source Windows domain |
| fg_fctuid | FortiClient endpoint UID |
| fg_osname | Endpoint OS name |
| fg_devtype | Endpoint device type |
| fg_srcmac | Source MAC address |
| fg_app | Detected application |
| fg_appcat | Application category |
| fg_apprisk | App risk level |
| fg_applist | App control profile |
| fg_sni | SNI hostname from TLS |
| fg_direction | Traffic direction |
| fg_msg | Log message text |
| fg_tranip | DNAT translated IP |
| fg_transip | SNAT egress IP |
| fg_vwlname | SD-WAN rule name |
| fg_vwlquality | SD-WAN path quality |
| fg_srcremote | FortiClient public source IP |
| fg_authserver | Auth server used |
| fg_utmaction | UTM overall action |
| fg_eventtype | UTM event type |

## Customization

The pack was developed for a multi-site hub-and-spoke Fortinet environment with FortiClient EMS. Dashboard queries can be modified after import via the Aria Operations for Logs UI.

Key queries you may want to adapt:
- `policyname="FortiClient EMS Inbound"` — rename to match your EMS inbound policy name
- `vwlname="sdr.directinternet"` — rename to match your SD-WAN rule name
- `unauthusersource="forticlient"` — this is standard FortiGate and should not need changing

## Contributing

Pull requests welcome. See [INSTALL.md](INSTALL.md) for notes on the content pack format and lessons learned reverse-engineering the `.vlcp` schema.

## Known Issues

- Extracted fields only apply to events ingested **after** the content pack is installed. Historical logs will not be retroactively indexed.
- The `event_type` constraint (`v4_2187792b`) is specific to FAZ syslog forwarding. If your logs arrive via a different path, update the constraint in each extracted field definition.

## License

MIT
