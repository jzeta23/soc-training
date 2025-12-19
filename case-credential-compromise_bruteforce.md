# Case: Possible Credential Compromise via Brute Force

> Simulated SOC case based on common authentication attack patterns.  
> Analysis performed from a SOC L1 / Junior perspective.

## Event Source

- **SIEM:** Generic SIEM (simulated environment)
- **Log source:** Authentication logs (auth.log / Azure AD Sign-in Logs equivalent)
- **Alert type:** SIEM Alert – Multiple Failed Logins followed by Success
- **Priority (initial):** Medium

## Events Detected

Time window analyzed: **2025-03-12 02:10 – 02:25 UTC**

- 12 failed login attempts for user `j.sanchez@company.com`
- Source IP: `185.231.xxx.xxx`
- Attempts occurred within **15 minutes**
- Login attempts originated from **Eastern Europe**
- **1 successful login** from the same IP at **02:24 UTC**
- Login occurred **outside business hours**
- No previous login history from this IP or region for this user


## Triage & Evidence Analysis

### Quantitative indicators
- Failed login count exceeds threshold  
  - Threshold: **10 failed attempts within 10 minutes**
  - Observed: **12 failed attempts within 15 minutes**
- Successful login occurred shortly after failures
- Same source IP used across attempts

### Contextual checks performed
- User role: Standard corporate user (non-admin)
- Historical login behavior:
  - Normal business hours: 13:00–21:00 UTC
  - Usual locations: Argentina
- MFA status: Enabled, but push-based (possible MFA fatigue not confirmed)

### Indicators of Compromise (IOCs)
- Source IP: `185.231.xxx.xxx`
- Unusual geolocation
- Brute force pattern followed by success


## Analysis Summary

While failed logins alone could be considered a security event,  
the combination of:
- high failure volume,
- successful authentication,
- anomalous location,
- and deviation from normal behavior

raises confidence that this is **not a benign event**.

The evidence supports **potential credential compromise**.


## Decision

**Classification:**  
✅ **Incident – Potential Credential Compromise**

**Severity:**  
🔴 High (P1)

Rationale:
- Confirmed successful login after brute-force pattern
- Risk of account misuse or lateral movement
- Possible data exposure


## Actions Taken (SOC L1 Scope)

- Collected and documented relevant logs
- Preserved timestamps, IPs, and user context
- Escalated to **SOC L2 / Incident Response** with evidence
- Referenced internal playbook:  
  `IR-ACC-001 – Suspicious Authentication Activity`

⚠️ No containment actions were taken directly (per L1 responsibilities)


## Recommended Actions (Escalated)

- Force password reset for affected user
- Revoke active sessions
- Review mailbox rules and recent activity
- Block source IP if confirmed malicious
- Validate MFA integrity


## Learning & Takeaways

- Failed logins alone are **events**
- Failed logins + success + anomaly = **incident**
- Quantitative thresholds help remove subjectivity
- Clear documentation enables fast escalation
- Evidence-based decisions are critical in SOC work

This case reinforced the importance of structured triage and avoiding assumptions without data.
