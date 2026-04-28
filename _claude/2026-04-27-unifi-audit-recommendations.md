---
type: reference
status: active
created: 2026-04-27
updated: 2026-04-27
area: "[[Home]]"
tags: [home-network, unifi, security, audit]
related: "[[2026-04-27-unifi-updates-project]]"
---

# UniFi Network Audit — Security & Performance Recommendations

> **Audit Date:** 2026-04-27 | **Auditor:** Chief (AI CoS) | **Network:** Warson Woods, MO
> **Scope:** Full hardware inventory, VLAN config, firewall rules, wireless config, IDS/IPS, port forwards, client analysis
> **Devices audited:** 15 (14 online, 1 offline) | **Clients:** 74 total

---

## Executive Summary

The network has solid hardware bones — a UDM Pro backbone, 10G uplink to the basement aggregation switch, and good AP coverage — but the security posture has significant gaps. The most critical issues are an **internet-exposed SSH port forward** pointing to a dead IP, **zero inter-VLAN firewall rules** (making the VLAN segmentation entirely cosmetic), and **IoT/camera devices living on the same flat network as workstations and servers**. These three issues alone represent serious attack surface. The good news: the firewall fixes and port forward cleanup are low-effort, high-impact wins that can be done in a single evening.

---

## Part 1: Top 15 Recommendations (Prioritized)

---

### 🔴 REC-01 — Kill the internet-facing SSH port forward immediately
**Priority: CRITICAL | Effort: Low (5 min)**

**What:** Port forward "Helium 22" is forwarding internet port 22 (SSH) to `192.168.0.59` — the old Bobcat Helium miner on the dead legacy subnet. The target IP is unreachable, but the UDM Pro is still accepting TCP connections on port 22 from the open internet and attempting to forward them.

**Why it matters:** Any SSH brute-force attack, exploit against the UDM Pro's forwarding stack, or future IP collision (if `192.168.0.59` ever becomes reachable again) represents direct shell access to a device on your network. The open port is also generating log noise and may trigger ISP abuse flags.

**Fix:**
1. UniFi UI → Settings → Firewall & Security → Port Forwarding
2. Delete rule: **"Helium 22"** (ext port 22 → 192.168.0.59:22)
3. While there: also delete **"Helium 443"** (ext port 443 → 192.168.0.59:443) and **"Helium"** (ext port 44158 → 192.168.0.59:44158)
4. Confirm no port 22 in port forwards after deletion

**Rollback:** Re-add the rule if needed (but don't — Helium is dead).

---

### 🔴 REC-02 — Fix Plex port forward pointing to dead IP
**Priority: CRITICAL | Effort: Low (5 min)**

**What:** Port forward "Plex" routes external port 32400 to `192.168.0.108` — the old NAS IP on the legacy 192.168.0.x subnet. The NAS is now at `192.168.10.108`. Plex remote access is broken and has likely been broken since the subnet migration.

**Why it matters:** Remote Plex access is non-functional. Also, `192.168.0.108` is unreachable — if Plex ever tried to relay through Plex.tv servers, it may be generating failed auth attempts.

**Fix:**
1. UniFi UI → Settings → Firewall & Security → Port Forwarding
2. Edit rule: **"Plex"** — change destination IP from `192.168.0.108` to `192.168.10.108`
3. Confirm Plex remote access works in Plex dashboard (Settings → Remote Access)
4. Consider locking the Plex forward to only allow Plex Media Server relay IPs via a firewall rule (optional hardening)

---

### 🔴 REC-03 — Replace SSH password auth with key-based auth on UDM Pro
**Priority: CRITICAL | Effort: Low-Medium (30 min)**

**What:** SSH to the UDM Pro uses password authentication with no key configured (`x_ssh_keys: []`). The current password `3hq7sc4DE` is 9 characters, no symbols — weak by modern standards. Anyone with network access can attempt SSH brute-force.

**Why it matters:** The UDM Pro is the gateway to your entire network. SSH access = game over. Password auth is unnecessary risk when key auth exists.

**Fix:**
1. Generate SSH key pair (if not already): `ssh-keygen -t ed25519 -C "unifi-udmpro"`
2. UniFi UI → Settings → System → Administration → SSH
3. Add your public key to the SSH Keys field
4. Disable "Password Authentication" toggle
5. Test key-based login: `ssh -i ~/.ssh/your_key root@192.168.10.1`
6. Update `~/.openclaw/secrets/unifi.env` or a new `~/.openclaw/secrets/unifi-ssh.env` with key path

**Bonus:** Change the SSH username from the current random string `lE4D0s9WrMg` to something meaningful like `admin` — or keep it obscure as a minor defense-in-depth measure.

---

### 🔴 REC-04 — Create inter-VLAN firewall rules (the VLANs are currently decorative)
**Priority: CRITICAL | Effort: Medium (2-3 hours)**

**What:** There are 6 VLANs defined (Default, Guest, IoT, Secure, Camera, AI-Quarantine) but **zero custom firewall rules**. Every VLAN can freely reach every other VLAN. A compromised IoT device (Roborock vacuum, LG fridge, Helium miner) has full Layer 3 access to your NAS at `192.168.10.108`, workstations, Home Assistant at `192.168.10.242`, and everything else.

**Why it matters:** VLANs without firewall rules are a false sense of security. The whole point of segmentation is to limit blast radius from a compromised device.

**Fix — create these rules in UniFi UI → Firewall & Security → Firewall Rules → LAN In:**

**Block IoT → Default (LAN):**
- Rule: DROP | Source: IoT Network (30) | Destination: Default Network (untagged)
- Except: Allow IoT → Home Assistant (`192.168.10.242`) for device control (add allow rule above the drop)

**Block IoT → Secure:**
- Rule: DROP | Source: IoT Network (30) | Destination: Secure Network (40)

**Block Camera → Default:**
- Rule: DROP | Source: Camera Network (50) | Destination: Default Network
- Exception: Allow Camera → NVR/Video-Recorder (`192.168.10.51`) if recorder is being moved to Camera VLAN

**Block AI-Quarantine → Default (except specific IPs):**
- Rule: ALLOW | Source: AI-Quarantine (70) | Destination: `192.168.10.1` (UDM Pro/gateway only)
- Rule: DROP | Source: AI-Quarantine (70) | Destination: Default Network
- NOTE: `DESKTOP-AU0QVMV` is on AI-Quarantine VLAN — if intentional, add specific allow rules for it

**Block Guest → All LAN:**
- UniFi handles this if you set Guest network purpose to "Guest" (see REC-05) — but verify

**Order matters:** Allow rules must precede Drop rules. UniFi processes top-to-bottom.

---

### 🔴 REC-05 — Fix Guest VLAN network purpose (currently "corporate")
**Priority: CRITICAL | Effort: Low (5 min)**

**What:** The Guest VLAN (VLAN 20, 192.168.20.0/24) is configured with network purpose "corporate" instead of "guest." This disables the automatic client isolation and inter-VLAN blocking that UniFi provides when a network is marked as guest.

**Why it matters:** Guests on your WiFi can potentially reach LAN resources. Setting purpose to "guest" enables automatic isolation — guests can only reach internet, not each other or your LAN.

**Fix:**
1. UniFi UI → Settings → Networks → Guest → Edit
2. Change "Network Purpose" from **Corporate** to **Guest**
3. Verify "Apply Guest Policies" is checked
4. Save and verify existing Guest SSID maps to this network

---

### 🟠 REC-06 — Expand IDS/IPS coverage to all VLANs
**Priority: HIGH | Effort: Low (10 min)**

**What:** IDS/IPS is configured in active IPS mode (good!) but only applied to the Default network (`5ed7f1c9de932e025fa96847`). IoT VLAN (30), Camera VLAN (50), and AI-Quarantine VLAN (70) have no intrusion detection. A compromised IoT device doing C2 callouts or lateral movement won't trigger any alerts.

**Why it matters:** IoT devices are historically the most-exploited segment. Your Bobcat miner, smart appliances, and cameras are the highest-risk devices — and they're unmonitored.

**Fix:**
1. UniFi UI → Settings → Security → Threat Management → IPS Settings
2. Under "Networks to Protect" — enable all networks: Default, Guest, IoT, Camera, Secure, AI-Quarantine
3. Keep same threat categories (already comprehensive)
4. Consider enabling Honeypot: creates a fake device that alerts on any scan/probe

---

### 🟠 REC-07 — Decommission Bobcat Helium miner
**Priority: HIGH | Effort: Low (15 min)**

**What:** The Bobcat Helium miner (`bobcatminer`, `192.168.10.65`) is still connected to the Default network. The Helium IoT network effectively ceased to be a viable earning opportunity in 2022-2023. The miner has:
- Active port forwards still punched through firewall (handled by REC-01)
- A device still pulling DHCP on your network
- Unknown firmware/software state — may not have received updates in years

**Why it matters:** An unmanaged, internet-connected device with years of potentially unpatched firmware is a liability. If it's earning $0, there's no reason to keep it online.

**Fix:**
1. Physical: Unplug the Bobcat miner from network and power
2. UniFi UI → Clients → find `bobcatminer` (192.168.10.65) → Forget/Remove
3. Delete DHCP reservation if one exists
4. Port forwards already deleted in REC-01

---

### 🟠 REC-08 — Resolve AC Mesh offline + wrong subnet situation
**Priority: HIGH | Effort: Low-Medium (30 min)**

**What:** The AC Mesh (`U7MSH`, `192.168.0.112`) is offline and on the old 192.168.0.x subnet — completely isolated from the rest of the network which migrated to 192.168.10.x. It's running ancient firmware `5.43.56.12784` (U6/AC-era, many security patches behind). Zero clients, completely orphaned.

**Why it matters:** If this AP ever comes back online on the old subnet, it's running years-old firmware with known vulnerabilities and is unmanaged by the controller. If it's permanently dead, it's phantom hardware in your inventory causing confusion.

**Fix (Option A — Recover it):**
1. Physically locate the AC Mesh
2. If wired uplink: verify which switch port it's on and that port's VLAN config
3. Factory reset: hold reset button 10+ seconds
4. Adopt it fresh in UniFi controller — it will get the current subnet
5. Update firmware immediately after adoption

**Fix (Option B — Retire it):**
1. Physically disconnect and store/sell
2. UniFi UI → Devices → AC Mesh → Forget Device
3. Document removal in project log

Given you have 6 functional APs including the U7 Outdoor (WiFi 6E), the AC Mesh is likely redundant. Recommend Option B unless there's a specific coverage gap it was filling.

---

### 🟠 REC-09 — Enable PMF on all SSIDs and upgrade to WPA3
**Priority: HIGH | Effort: Low (20 min)**

**What:** PMF (Protected Management Frames / 802.11w) is disabled on all three SSIDs (Dream Machine, Guest, IoT DMelsh). PMF prevents deauthentication attacks — a trivially-executed attack where an adversary forces clients to disconnect, enabling evil twin attacks or denial-of-service.

**Why it matters:** Without PMF, any nearby attacker with aircrack-ng can kick all clients off your network at will. WPA3 requires PMF, and WPA3 provides forward secrecy — meaning past traffic can't be decrypted even if your PSK is later compromised.

**Fix:**
1. UniFi UI → WiFi → "Dream Machine" → Edit → Advanced
2. Security Protocol: Change from **WPA2** to **WPA2/WPA3 (Mixed)** — this allows legacy WPA2-only devices while enabling WPA3 for capable ones
3. PMF: Set to **Optional** (compatible mode) initially, then **Required** once all devices support it
4. Repeat for "Guest" and "IoT DMelsh" SSIDs
5. Test that all devices reconnect (most modern devices handle WPA3 mixed mode fine)

**Note:** The UAP-AC-Lite on AI-Quarantine doesn't support WPA3 — but PMF Optional still works on WPA2.

---

### 🟠 REC-10 — Enable IGMP Snooping to fix Sonos multicast flooding
**Priority: HIGH | Effort: Low (10 min)**

**What:** IGMP Snooping is disabled on all 6 networks. You have 7 Sonos devices on the network (including `SonosZP` at `192.168.70.113` on AI-Quarantine). Sonos uses multicast for device discovery. Without IGMP snooping, every multicast packet floods every port on every switch — unnecessary traffic overhead, and a known cause of Sonos discovery/grouping failures.

**Why it matters:** Multicast flooding degrades switch performance and is likely causing your Sonos devices to have intermittent connectivity/grouping issues. Also causes higher CPU on wireless APs.

**Fix:**
1. UniFi UI → Settings → Networks → Default → Edit → IGMP Snooping: **Enable**
2. Repeat for all 6 networks (Guest, IoT, Secure, Camera, AI-Quarantine)
3. If Sonos is spread across VLANs: consolidate all Sonos devices onto Default, or use mDNS reflector (UniFi → Settings → Services → mDNS) to bridge discovery across VLANs

**Sonos-specific note:** Sonos strongly recommends all speakers on the same VLAN. Having `SonosZP` on AI-Quarantine (70) while others are on Default (10) will cause grouping failures. Move the AI-Quarantine Sonos to Default (or dedicated Sonos sub-VLAN off Default).

---

### 🟡 REC-11 — Activate DNS filtering (configured but set to "none")
**Priority: MEDIUM | Effort: Low (15 min)**

**What:** DNS filtering is configured for 5 networks but the filter is set to "none" on all — it's a no-op. The UDM Pro supports DNS Shield (Cloudflare) and custom DNS filter lists. This is a quick win for blocking malware C2 domains, especially useful on the IoT VLAN.

**Why it matters:** DNS-based filtering is the single most effective cheap security control — it blocks malware before a TCP connection is even established. IoT devices that get compromised often call home to known malicious domains immediately.

**Fix:**
1. UniFi UI → Settings → Security → DNS Shield
2. Enable for all networks. Options:
   - **Cloudflare Malware Blocking** — blocks known malware/phishing (free, no privacy tradeoff)
   - **Cloudflare Family** — also blocks adult content
   - **Custom** — point to your own Pi-hole if desired
3. Recommended: Apply "Cloudflare Malware" to IoT, Camera, AI-Quarantine, Guest
4. Apply "Cloudflare" standard (1.1.1.1) to Default/Secure for speed + privacy

---

### 🟡 REC-12 — Clarify and fix Dan's workstation VLAN placement
**Priority: MEDIUM | Effort: Low (15 min, decision required)**

**What:** `DESKTOP-AU0QVMV` (Dan's main workstation, `192.168.70.112`) is wired into the AI-Quarantine VLAN (70). This is where OpenClaw runs. Meanwhile, `Melsh_Computer` (`192.168.10.143`) is on Default. There appear to be two different "main workstation" entries, or the wired vs. wireless interfaces are on different VLANs.

**Why it matters:** If OpenClaw is running on `DESKTOP-AU0QVMV` and it's on AI-Quarantine with no firewall rules allowing outbound internet (just yet), it may be getting out via a gap in the current (absent) ruleset. Once REC-04 firewall rules are applied, this machine may lose internet or LAN access unintentionally.

**Fix:**
1. Determine intent: Is AI-Quarantine isolation intentional for the workstation?
   - **If yes (intentional sandbox):** Create explicit allow rules: AI-Quarantine → Internet (already allowed by default), AI-Quarantine → NAS (`192.168.10.108`), AI-Quarantine → Home Assistant (`192.168.10.242`)
   - **If no (accidental):** Move `DESKTOP-AU0QVMV` to Default VLAN — check which switch port it's wired into and update the port profile in UniFi
2. Check if `DESKTOP-AU0QVMV` and `Melsh_Computer` are the same physical machine (different NICs?) or two separate machines
3. Document the decision in this project file

---

### 🟡 REC-13 — Fix Nano HD running at 100Mbps + channel conflicts
**Priority: MEDIUM | Effort: Low-Medium (30 min)**

**What:** The Nano HD AP (`192.168.10.56`) is connected to the Family Room switch (`US8P150`) at 100Mbps instead of Gigabit, and its uplink port shows 52 rx_errors. Additionally, three APs are on 2.4GHz channel 6 simultaneously (IW HD, U7 Outdoor, AC-Lite), creating co-channel interference.

**Why it matters:** A 100Mbps AP uplink caps every client on that AP at 100Mbps max throughput — painful on a gigabit ISP. With 4 clients and WiFi 6 speeds, you're leaving 900Mbps on the table. The rx_errors suggest a bad cable or connector.

**Fix — Nano HD 100Mbps:**
1. UniFi UI → Devices → Switch Family Room (US8P150) → Ports → Port 5 (where Nano HD is connected)
2. Check port config — should be auto-negotiate or forced 1000FDX
3. Likely cause: bad ethernet cable or connector. Re-terminate or replace the cable run to the Nano HD
4. If cable is fine: try forcing 1000FDX on the port and Nano HD side

**Fix — 2.4GHz channel conflicts:**
1. UniFi UI → WiFi → RF Environment tool to check actual interference
2. Current: IW HD=6, U7 Outdoor=6, AC-Lite=6 — three APs on same channel
3. Reassign: IW HD → **1**, U7 Outdoor → **11** (or auto), keep AC-Lite on 6
4. Enable Auto-Optimize in UniFi AI or manually assign non-overlapping channels: 1, 6, 11 spread evenly across 6 APs

---

### 🟡 REC-14 — Enable fast roaming (802.11r) and band steering
**Priority: MEDIUM | Effort: Low (15 min)**

**What:** Fast roaming is disabled on all SSIDs. Band steering is null/unconfigured. With 74 clients and 6 active APs spanning a full home, clients don't roam efficiently — they cling to far APs and refuse to switch. Band steering forces capable clients to 5GHz (less congested, faster).

**Why it matters:** The U6 LR carrying 21 clients and U6 Pro with 15 clients suggests uneven load distribution — some APs are overloaded while others have 1-3 clients. Poor roaming = latency spikes, reconnection drops, and the "my phone is slow in the kitchen even though it was fine in the office" experience.

**Fix:**
1. UniFi UI → WiFi → "Dream Machine" → Edit → Advanced
2. **Fast Transition (802.11r):** Enable — set to "Open" (compatible with most clients) vs. "OKC" (opportunistic key caching, stricter)
3. **Band Steering:** Enable — set to "Prefer 5GHz" (not force, for compatibility)
4. **BSS Transition (802.11v):** Already on for IoT and Guest SSIDs — enable for main SSID too
5. **Min RSSI:** Set minimum RSSI to -75 or -70 dBm — forces clients to roam when signal is weak
6. Repeat for IoT DMelsh SSID

---

### 🟡 REC-15 — Rename main SSID from "Dream Machine" and enable 6GHz on U7 Outdoor
**Priority: MEDIUM (low risk, low effort) | Effort: Low (10 min)**

**What:** The main SSID is literally named "Dream Machine" — the factory default of the UDM Pro. This is a passive fingerprint telling anyone scanning WiFi that you haven't touched the default config. Separately, the U7 Outdoor (`UKPW`) is WiFi 6E capable (`is_11be: true`) but is likely only broadcasting 2.4/5GHz.

**Why it matters:** "Dream Machine" SSID is a minor but unnecessary fingerprint. Enabling 6GHz on the U7 Outdoor gives the highest-throughput band to outdoor devices and reduces 5GHz congestion (but 6GHz range is shorter, so this is most useful for covered patio/outdoor seating areas close to the AP).

**Fix:**
1. Rename SSID: UniFi UI → WiFi → "Dream Machine" → Edit → Name: something non-identifying (e.g., "Melsheimer", "WhyFi", personal preference)
2. Enable 6GHz: UniFi UI → Devices → U7 Outdoor → Radio Configuration → enable 6GHz band if available in firmware 8.5.21
3. Create a separate 6GHz-only SSID or enable tri-band on existing SSID

---

## Part 2: Phased Project Plan

---

### Phase 1 — Critical Security (Week 1, ~2 hours total, zero disruption)
*Do these first. They fix real vulnerabilities with minimal risk of breaking anything.*

| Task | Steps | Time | Disruption Risk | Rollback |
|------|-------|------|-----------------|---------|
| **REC-01**: Delete Helium port forwards (SSH/443/44158) | Settings → Port Forwarding → Delete 3 rules | 5 min | None — dead IPs | Re-add rules |
| **REC-02**: Fix Plex port forward to 192.168.10.108 | Settings → Port Forwarding → Edit Plex | 5 min | None — Plex was broken already | Revert IP |
| **REC-03**: SSH key auth on UDM Pro | Generate key → Add to UniFi Settings → Disable password auth | 30 min | None if key tested first | Re-enable password auth |
| **REC-05**: Fix Guest VLAN purpose to "guest" | Settings → Networks → Guest → Change purpose | 5 min | Guests get more isolated (intended) | Revert to corporate |
| **REC-07**: Decommission Bobcat miner | Unplug → Forget in UniFi | 15 min | None | Replug |
| **REC-06**: Enable IPS on all VLANs | Security → Threat Management → add all networks | 10 min | Slight CPU uptick on UDM Pro | Disable added networks |
| **REC-10**: Enable IGMP Snooping | Networks → each VLAN → enable IGMP Snooping | 10 min | Possible brief Sonos hiccup | Disable |
| **REC-11**: Enable DNS filtering | Security → DNS Shield → enable Cloudflare Malware | 15 min | None — opt-in | Disable |

**Phase 1 Completion Criteria:**
- [ ] No port 22 in port forwards
- [ ] Plex remote access working in Plex dashboard
- [ ] SSH to UDM Pro works with key only
- [ ] Guest VLAN purpose = "guest"
- [ ] Bobcat unplugged and forgotten
- [ ] IPS active on all 6 networks
- [ ] IGMP Snooping on, Sonos still working
- [ ] DNS filtering active

---

### Phase 2 — VLAN Enforcement (Weeks 2-3, ~4-6 hours, moderate disruption)
*This is the hard part. Moving devices and adding firewall rules will cause temporary disconnections. Do over a weekend morning.*

#### 2A — Inter-VLAN Firewall Rules (REC-04)
**Time estimate: 2 hours | Risk: HIGH — can break things if rules are wrong**

1. Before adding drop rules, verify all critical device IPs:
   - NAS: `192.168.10.108` ✅
   - Home Assistant: `192.168.10.242` ✅
   - UDM Pro gateway: `192.168.10.1` ✅
   - JetKVM: `192.168.10.26`
2. Add rules in this exact order (Allow before Drop):
   - Allow Established/Related traffic (first rule — prevents breaking existing sessions)
   - Allow IoT → Home Assistant (192.168.10.242) on ports 8123, 8124
   - Drop IoT → Default LAN
   - Drop IoT → Secure LAN
   - Allow AI-Quarantine → Internet (auto-allowed, just verify)
   - Drop AI-Quarantine → Default LAN (unless DESKTOP-AU0QVMV issue resolved in REC-12)
   - Block Guest → All LAN (should be handled by "guest" purpose from REC-05, but add explicit rule)
3. **Test each rule before proceeding to next** using a device on that VLAN
4. **Rollback:** Delete all custom rules to return to current state (no rules = everything works, just insecure)

#### 2B — Move IoT devices to IoT VLAN (REC-04 prerequisite)
**Time estimate: 2-3 hours | Risk: MEDIUM — devices may need re-pairing**

The following devices are on Default (192.168.10.x) and should be on IoT VLAN 30 (192.168.30.x):
- Dyson VS3, TP-Link HS300, GE Appliance module
- Owlet cameras (baby monitors) — move to Camera VLAN (50)
- EcoFlow (multiple), Espressif devices
- Roborock vacuum, LG Smart Fridge
- Rachio irrigation controller
- MyQ garage door
- Tonal fitness system
- EcoNet water heater
- RestPlus sound machines
- DeLonghi coffee machine
- Levoit purifiers/humidifier (not already on AI-Quarantine)

**Process for each device:**
1. Check if device connects via MAC address (if so, update DHCP assignment for new subnet)
2. For WiFi IoT devices: ensure they're connecting to "IoT DMelsh" SSID (not "Dream Machine")
3. Devices connecting to wrong SSID: either configure them to use IoT SSID, or tag their switch port to IoT VLAN
4. After moving: verify device still works (Rachio, MyQ, etc. — cloud-connected devices usually survive VLAN moves fine)

**Devices that need Home Assistant access after move:**
Make sure the firewall allow rule for IoT → HA (192.168.10.242:8123) is in place before migrating devices that HA integrates.

#### 2C — Move cameras to Camera VLAN
**Time estimate: 1 hour | Risk: LOW — UniFi cameras can be moved via controller**

Move to Camera VLAN (50):
- front-porch-dome: 192.168.10.32 → 192.168.50.x
- backyard: 192.168.10.180 → 192.168.50.x  
- g4-doorbell-pro: 192.168.10.159 → 192.168.50.x
- Video-Recorder (NVR): 192.168.10.51 → 192.168.50.x (or keep on Default — NVR needs LAN access for management)

**UniFi Protect cameras:** Can be reassigned in UniFi controller without physical re-pairing.
**Firewall:** Camera VLAN should only reach internet (for cloud features) and the NVR. Block Camera → Default except NVR IP.

#### 2D — Resolve AI-Quarantine workstation situation (REC-12)
**Time estimate: 30 min | Risk: LOW**

Determine intent and implement appropriate firewall exceptions before dropping AI-Quarantine → Default. Do this before the drop rules go in.

---

### Phase 3 — Performance & Optimization (Month 2, weekends)

| Task | Detail | Time | Risk |
|------|--------|------|------|
| **REC-13**: Fix Nano HD cable | Re-run or re-terminate cable to Family Room Nano HD | 1-2 hours | Low — AP briefly offline |
| **REC-09**: Enable WPA3 mixed + PMF | Per SSID, one at a time | 30 min | Low — clients reconnect |
| **REC-14**: Enable fast roaming + band steering | Per SSID, test roaming with phone | 30 min | Low |
| **REC-13**: Fix 2.4GHz channel plan | Reassign to 1/6/11, re-run RF scan | 15 min | None |
| **REC-15**: Rename SSID + enable 6GHz | Cosmetic + U7 Outdoor 6GHz | 15 min | Brief disconnect during rename |
| **REC-08**: Recover or retire AC Mesh | Physical locate + factory reset or disconnect | 1 hour | None |
| UDM Pro firmware check | Verify 5.1.8.32785 is current vs. available — UniFi release notes | 15 min | Brief outage if upgrading |

**UDM Pro firmware note:** Version `5.1.8.32785` is UniFi OS 5.x. Check https://community.ui.com/releases for latest stable. UniFi OS 4.x → 5.x was a major jump; 5.x → 5.x patches are generally safe with auto-upgrade already on.

---

### Phase 4 — Ongoing Operations (Monthly)

| Cadence | Task |
|---------|------|
| Monthly | Review UniFi threat log for flagged events (Settings → Security → Threat Log) |
| Monthly | Check for firmware updates on all devices (auto-upgrade catches APs/switches, but UDM Pro needs manual review) |
| Monthly | Audit client list — look for unknown devices, old static leases |
| Quarterly | Review port forwards — remove anything stale |
| Quarterly | Review firewall rules — prune anything no longer needed |
| Quarterly | Check IPS threat category updates in UniFi |
| As-needed | Update SSH keys if workstation changes |

**Automation ideas:**
- OpenClaw cron: Weekly UniFi API check for offline devices, new unknown clients, firmware update available
- Home Assistant: Alert when a new device joins the IoT or Default network (UniFi integration → `device_tracker`)
- Alert on any port forward to 192.168.0.x (old subnet) — canary for subnet confusion

---

## Part 3: Hardware Upgrade Considerations

---

### 🔴 Replace: AC-Lite on AI-Quarantine VLAN
**Device:** UAP-AC-Lite (`U7LT`, `192.168.70.36`)
**Issue:** This is 802.11ac (WiFi 5) era hardware serving 9 clients including Dan's work laptop (`dmelsheime-ltw2`). No WPA3 support, no WiFi 6, lower max throughput. It's the oldest active AP in the fleet.

**Recommendation:** Replace with a **U6 Lite** (~$99) or repurpose one of the existing U6 APs to cover AI-Quarantine if coverage allows. If AI-Quarantine is specifically for quarantining certain devices (and they don't need high throughput), the AC-Lite is acceptable short-term but should be on the replacement list.

---

### 🟠 Consider: 10G uplink for NAS (current: 2×1G LAG = 2Gbps)
**Device:** Synology NAS `dmelsh1019` (`192.168.10.108`)
**Current:** LAG on ports 5+6 of Main Switch (US24PRO2) — 2×1G aggregated ≈ 2Gbps max
**Issue:** The NAS is serving Plex (transcoding), file storage, and likely Time Machine. With a 10G SFP+ port on the Basement Switch Pro Max 24 PoE and potentially on the Main Switch (US24PRO2 has SFP+ ports), a direct 10G link to the NAS would be a significant throughput improvement.

**Recommendation:** 
1. Check if Synology `dmelsh1019` has a 10GbE NIC (DS models from 2019+ often support expansion)
2. If NAS supports 10G: add a 10GbE NIC (~$80-150 for Synology E10G22-T1-Mini), run direct SFP+ or DAC to Main Switch or Basement Switch
3. If NAS doesn't support 10G: current 2Gbps LAG is fine for most home use

---

### 🟠 Retire or Replace: Power Distribution Pro at 100Mbps
**Device:** Power Distribution Pro (`USPPDUP`, `192.168.10.198`)
**Issue:** Connected at 100Mbps to UDM Pro port 3. This is a smart PDU with network monitoring — the 100Mbps is probably fine for management traffic, but suggests a cable/port issue rather than hardware limitation. Still worth investigating.

**Action:** Check the cable and port. If it's a cable issue (likely), fix it. If it's an inherent device limitation, the USPPDUP should still only need 100Mbps for management.

---

### 🟡 Consider: U7 Pro or U7 Pro Max for high-density areas
**Context:** U6 LR (`192.168.10.39`) carrying 21 clients, U6 Pro (`192.168.10.103`) carrying 15 clients. These are the two busiest APs.

**If performance degrades:** The U7 Pro Max (~$299) supports WiFi 7 with 6GHz band and higher multi-client throughput. Not urgent now, but if you're seeing slow speeds during evening peak (everyone home, multiple 4K streams, work calls), it would be the highest-impact single AP upgrade.

**Note:** The U7 Outdoor already has WiFi 6E (6GHz capable), which is a good sign for the outdoor coverage. Once 6GHz is enabled on it, it'll offload some 5GHz congestion.

---

### 🟡 Upgrade Path: UDM Pro → UDM Pro Max or UDM SE (future consideration)
**Current:** UDM Pro (`UDMPRO`, `192.168.1.226` — note: different from the .10.x subnet, likely a management interface)
**Future consideration:** If you add UniFi Protect cameras beyond current count, run VPN at high throughput, or want built-in 10G WAN, the **UDM Pro Max** or **UDM SE** offer more headroom. Not urgent — the UDM Pro handles a home network fine.

---

### 🟢 Keep: Everything Else Is Fine
- **Basement Switch Pro Max 24 PoE** (USPM24P): Excellent aggregation switch, 10G uplink, PoE+ — no replacement needed
- **Switch Main (US24PRO2)**: Solid 24-port with SFP+ — keep
- **U6 Pro, U6 LR**: Good WiFi 6 APs — keep, optimize with fast roaming
- **U7 Outdoor**: Excellent — enable 6GHz when configured
- **USW Flex**: Keep for edge use cases
- **UDM Pro firmware**: Stay on current auto-upgrade schedule

---

## Quick Reference: Device → Correct VLAN Mapping

| Device | Current | Correct | Priority |
|--------|---------|---------|---------|
| Dyson VS3 | Default | IoT (30) | High |
| TP-Link HS300 | Default | IoT (30) | High |
| GE Appliance | Default | IoT (30) | High |
| Owlet cameras | Default | Camera (50) | High |
| Roborock vacuum | Default | IoT (30) | High |
| LG Smart Fridge | Default | IoT (30) | High |
| Rachio irrigation | Default | IoT (30) | High |
| MyQ garage | Default | IoT (30) | High |
| EcoNet water heater | Default | IoT (30) | High |
| Tonal | Default | IoT (30) | Medium |
| EcoFlow (×2+) | Default | IoT (30) | Medium |
| DeLonghi coffee | Default | IoT (30) | Medium |
| RestPlus sound machines | Default | IoT (30) | Medium |
| Bobcat miner | Default | Retire | Critical |
| front-porch-dome | Default | Camera (50) | High |
| backyard camera | Default | Camera (50) | High |
| g4-doorbell-pro | Default | Camera (50) | High |
| Video-Recorder (NVR) | Default | Camera (50) or Default* | High |
| JetKVM | Default | Secure (40) | Medium |
| SonosZP (AI-Q VLAN) | AI-Quarantine | Default | Medium |

*NVR may need Default access for management — evaluate based on access needs

---

## Summary Scorecard

| Category | Current State | Target State |
|----------|--------------|--------------|
| Internet exposure | 🔴 Port 22 + 443 exposed | ✅ Only Plex + Overseerr |
| SSH security | 🔴 Password only, weak | ✅ Key-based only |
| VLAN enforcement | 🔴 Zero firewall rules | ✅ Inter-VLAN drop rules |
| Device segmentation | 🔴 IoT/cameras on Default | ✅ All devices on correct VLANs |
| Guest isolation | 🟠 "corporate" purpose | ✅ "guest" purpose |
| IDS/IPS coverage | 🟠 Default only | ✅ All networks |
| Wireless security | 🟠 WPA2 only, no PMF | ✅ WPA3 mixed + PMF |
| Roaming | 🟠 None configured | ✅ 802.11r + band steering |
| DNS filtering | 🟡 Configured but inactive | ✅ Malware blocking active |
| IGMP/Multicast | 🟡 Flooding all ports | ✅ Snooping enabled |
| Dead hardware | 🟡 AC Mesh + miner online | ✅ Retired/recovered |
| Physical links | 🟡 Nano HD at 100Mbps | ✅ Gigabit everywhere |
