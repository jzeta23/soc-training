# Case: Brute Force leading to Potential Credential Compromise (SSH)

> Simulated SOC case written to be **replicable**: includes thresholds, baseline comparison, timestamps, false positive considerations, and operational actions.


## Source

- **Event source:** Linux SSH authentication
- **Log source:** `/var/log/auth.log`
- **Collection:** Filebeat → SIEM
- **Detection:** SIEM alert rule `SSH_Bruteforce_Threshold_v1`
- **Alert name:** "SSH brute force: failed logins exceed threshold"
- **Initial priority:** Medium (auto) → adjusted during triage


## Events Detected

**Host:** `web-01` (Ubuntu)  
**User targeted:** `root` and `deploy`  
**Source IP:** `203.0.113.45` (untrusted ASN / not in allowlist)

Time window: **2025-03-12 02:13:11 – 02:21:02 UTC**

| Timestamp (UTC) | Event | Username | Result | Source IP |
|---|---|---:|---|---|
| 02:13:11 | sshd | root | Failed password | 203.0.113.45 |
| 02:14:03 | sshd | root | Failed password | 203.0.113.45 |
| 02:14:58 | sshd | root | Failed password | 203.0.113.45 |
| 02:16:09 | sshd | root | Failed password | 203.0.113.45 |
| 02:17:40 | sshd | deploy | Failed password | 203.0.113.45 |
| 02:18:12 | sshd | root | Failed password | 203.0.113.45 |
| 02:19:55 | sshd | deploy | Failed password | 203.0.113.45 |
| 02:21:02 | sshd | root | Failed password | 203.0.113.45 |

**Count:** 8 failed SSH logins from the same IP in ~8 minutes.


## Baseline / Threshold

### Threshold (configured in detection rule)
- **Brute force threshold:** **≥ 5 failed attempts within 10 minutes** from the same source IP to the same host

### Baseline (normal behavior for this host)
From last 14 days (summarized baseline):
- Typical failed SSH attempts: **0–1 per day**
- Typical source IPs: internal VPN range only
- Typical access hours: **10:00–22:00 UTC**
- No history of `203.0.113.45` observed previously


## Triage Checks Performed (Before Escalation)

1) **Correlated success events**
- Checked for `Accepted password` / `Accepted publickey` events on `web-01` around the same timeframe  
- Result: **No successful SSH login** from `203.0.113.45` during 02:00–03:00 UTC

2) **Host activity after attempts**
- Quick scan for process spikes / suspicious auth events (e.g., new users, sudo attempts) in the next 30 minutes  
- Result: **No immediate post-auth anomalies found** (limited SOC L1 visibility)

3) **Known legitimate sources**
- Compared source IP against allowlist (VPN, office ranges)  
- Result: **Not allowlisted**

4) **Time-of-day risk**
- Events occurred at **02:13–02:21 UTC**, which is **outside normal access hours** for this host


## False Positive Considerations

Potential benign causes considered:
- Misconfigured automation or health-check repeatedly trying wrong credentials  
- A developer connecting from a new IP

Why this is unlikely (based on evidence):
- Baseline for this host is near-zero failed SSH attempts
- Targeting `root` + `deploy` suggests credential guessing, not a single-user typo
- Source IP not in allowlist and not seen historically
- Burst pattern (8 failures in 8 minutes) matches brute-force behavior more than human error


## Analysis

- Failed attempts **exceed threshold** (8 in 8 min vs threshold 5 in 10 min) → meets brute force criteria
- Anomalous timing (outside normal access hours) increases risk
- Multiple usernames targeted (`root`, `deploy`) suggests systematic guessing
- No success observed → **attempted compromise**, not confirmed access

This is **not just a suspicious event**: it is an attack attempt that triggers a response playbook due to threshold + baseline deviation.


## Decision

**Classification:** ✅ **Incident – Credential Brute Force Attempt (No Confirmed Access)**  
**Severity:** 🟠 Medium-High (P2)

Why not P1:
- No confirmed successful authentication
- No evidence of post-compromise behavior within available telemetry

Why not P3:
- Clear threshold breach + baseline deviation + multi-username pattern


## Actions Taken (Operational)

**Playbook reference:** `IR-AUTH-SSH-001 – SSH Brute Force`

1) **Block the source IP at firewall (temporary containment)**
- Tool: `ufw` (host-based firewall)  
- Action:
  - `sudo ufw deny from 203.0.113.45 to any port 22`

*(If cloud environment instead of host firewall, example alternative: AWS Security Group / NACL deny rule for 203.0.113.45/32 on port 22.)*

2) **Add IOC for correlation**
- IOC: `203.0.113.45`
- Action: add to SIEM watchlist / threat list so future hits correlate across hosts

3) **Document and escalate to L2**
- Escalation includes:
  - timestamps table
  - threshold vs observed count
  - baseline summary (14d)
  - containment performed
  - recommendation: review SSH hardening + fail2ban/rate limiting


## Learning / Takeaways

- Thresholds reduce subjectivity: **>=5 failed/10min** is a clear trigger
- Baseline matters: a burst on a normally quiet host is higher signal
- “Incident” can be an **attack attempt** even without confirmed access (depending on policy)
- Actions must be operational: tool + instruction, not generic advice
