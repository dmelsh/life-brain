---
type: project
status: active
created: 2026-04-27
updated: 2026-04-27
audit_completed: 2026-04-27
outcome: "UniFi network fully configured, documented, and integrated with Home Assistant / OpenClaw"
target_date: 2026-06-01
area: "[[Home]]"
tags: [home-network, unifi, tech]
---

# UniFi Network Updates

> Track all config changes, fixes, and improvements to the home UniFi setup.

## Hardware

- **Router:** UDM Pro — `192.168.10.1`
- **Credentials:** `~/.openclaw/secrets/unifi.env`

---

## Status

| Item | Status |
|------|--------|
| UDM Pro reachable | ✅ |
| API/local auth | ⚠️ In progress |
| OpenClaw integration | ⬜ |
| HA integration | ⬜ |
| **Security audit** | ✅ Complete — 2026-04-27 |
| **Recommendations doc** | ✅ Written — see link below |

---

## Change Log

### 2026-04-27 — Full Network Security Audit ✅

Completed a comprehensive network security and performance audit covering all 15 devices, 6 VLANs, wireless config, firewall rules, IDS/IPS, port forwards, and 74 clients.

**→ Full recommendations:** [[2026-04-27-unifi-audit-recommendations]]

**Critical findings requiring immediate action:**
- 🔴 Port 22 (SSH) exposed to internet via dead Helium port forward → delete immediately
- 🔴 Port 443 exposed to internet via dead Helium port forward → delete immediately  
- 🔴 Plex port forward pointing to old IP `192.168.0.108` (NAS is now `192.168.10.108`) → fix
- 🔴 Zero inter-VLAN firewall rules — VLANs are decorative, all traffic flows freely
- 🔴 SSH password auth only on UDM Pro (no keys), weak password
- 🟠 All IoT and camera devices on Default VLAN — no segmentation enforced
- 🟠 AC Mesh offline + wrong subnet (192.168.0.x) — orphaned
- 🟠 Bobcat Helium miner still on network with open port forwards — decommission

**Recommendations summary:** 15 specific recommendations across 4 phases (~2h Phase 1 security fixes, ~6h Phase 2 VLAN migration, Phase 3 performance optimization, Phase 4 ongoing ops)

### 2026-04-26

- [ ] Confirmed UDM Pro reachable at `192.168.10.1`
- [ ] Saved credentials to `~/.openclaw/secrets/unifi.env`
- [ ] Auth failing — SSO (ui.com) account vs. local admin TBD
- **Next:** Verify login method at `https://192.168.10.1` OR generate API key via Settings → Control Plane → API Keys

---

## Open Tasks

### Auth / Access
- [ ] Determine if login is ui.com SSO or local account
- [ ] Generate local API key (Settings → Control Plane → API Keys)
- [ ] Validate API auth from OpenClaw

### Network / Config Changes

> See [[2026-04-27-unifi-audit-recommendations]] for full phased plan. Quick hits:

- [ ] **URGENT**: Delete port forwards: Helium 22 (SSH!), Helium 443, Helium 44158
- [ ] **URGENT**: Fix Plex port forward → change destination to `192.168.10.108`
- [ ] Set up SSH key auth on UDM Pro, disable password auth
- [ ] Fix Guest VLAN purpose: corporate → guest
- [ ] Decommission Bobcat miner (unplug + forget)
- [ ] Expand IPS to all 6 VLANs (currently Default only)
- [ ] Enable IGMP Snooping on all networks (fixes Sonos multicast flooding)
- [ ] Enable DNS filtering (Cloudflare Malware) — configured but inactive
- [ ] Create inter-VLAN firewall rules (major project — Phase 2)
- [ ] Migrate IoT devices to IoT VLAN 30
- [ ] Migrate cameras to Camera VLAN 50
- [ ] Fix Nano HD at 100Mbps (cable issue on Family Room switch port 5)
- [ ] Enable WPA3 mixed + PMF on all SSIDs
- [ ] Enable fast roaming (802.11r) + band steering
- [ ] Recover or retire AC Mesh (192.168.0.112 — wrong subnet, offline)

### Integrations
- [ ] Wire UniFi API into OpenClaw (device status, client counts, etc.)
- [ ] Connect to Home Assistant if needed

---

## Notes

<!-- Running notes, vendor info, decisions made -->
