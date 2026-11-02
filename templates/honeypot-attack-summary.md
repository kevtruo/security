<!--
Template for analysis/YYYY-MM-DD-short-topic/honeypot-attack-summary.md

- Folder name: the first-seen attack date plus a short topic. Evidence files sit in the same folder.
- Voice: finished public report. No draft notes such as "this is an inference", no first or second person.
  State assessments directly ("consistent with", "likely") and list their basis under Confidence and gaps.
- Public safety: attacker IPs may be published as indicators. Nothing that identifies the sensor:
  its address, hostname, cloud account, or log shipping. Work through the PR template checklist before merging.
- Delete these comments and any section that does not apply.
-->

# Honeypot Attack Summary: <Short Description of the Attack>

| | |
|---|---|
| **Date** | YYYY-MM-DD (attack observed YYYY-MM-DD UTC) |
| **Sensor** | DShield honeypot (<Cowrie / web honeypot>) on a cloud VM |
| **Data sources** | <Cowrie JSON logs, web honeypot logs, DShield firewall log, TTY recordings, SANS ISC, RIPE WHOIS/RIPEstat, AbuseIPDB> |

## 1. Summary

<Two or three short paragraphs: who attacked, what they did, what happened, and the single most important finding.>

## 2. Attacker attribution

| Field | Value |
|---|---|
| IP address | <attacker IP> |
| ASN | <ASxxxx> |
| ISP / Org | <organisation> |
| Usage type | <hosting / residential / cloud> |
| Geolocation | <country; note disagreement between sources> |
| Reputation | <SANS ISC / AbuseIPDB results> |

<One or two sentences interpreting the source network. For several sources, use one column per IP or one row per IP.>

## 3. Timeline (UTC)

| Time | Source | Event |
|---|---|---|
| <HH:MM:SS> | <Firewall / Cowrie / Web> | <event> |

## 4. Activity statistics

| Metric | Count |
|---|---|
| <Firewall packets / requests> | <n> |
| <Sessions> | <n> |
| <Failed / successful logins> | <n> |
| <Commands executed> | <n> |
| <Files downloaded / uploaded> | <n> |
| <Days observed> | <n> |

## 5. Tooling fingerprint

| Indicator | Value | Sessions / requests |
|---|---|---|
| <SSH client version / user agent> | <value> | <n / n> |
| <HASSH / other fingerprint> | <value> | <n / n> |

<One sentence on what the fingerprint indicates.>

## 6. <Credentials attempted / Commands executed / Exploits attempted>

<Table of what the attacker tried, with a short explanation of each item and, for exploits, the CVE.>

## 7. MITRE ATT&CK mapping

| Tactic | Technique | Evidence |
|---|---|---|
| <tactic> | <Txxxx: name> | <what was observed> |

## 8. Analysis

**Relevance:**
- <Why this attack matters to a defender.>
- <Who is exposed and what they stand to lose.>

**Vulnerability:**
- <The CVE(s), or "No CVE applies" with the CWE for a configuration weakness.>
- <What the flaw does and which version fixes it.>

**Purpose:** <One or two sentences on the attacker's goal, citing a source where the goal is not directly observed.>

**Automation:**
- <Evidence that the attack is scripted or hands-on: timing, ordering, fingerprints, repetition across sources.>

**Would it succeed against a vulnerable system?**
- <Yes/no for a vulnerable target, and what that target looks like.>
- <Whether the attacker made mistakes, and why it did not succeed against the sensor.>

**<Notable detail, for example Honeypot detection or Request in detail>:**
- <Optional block for a finding specific to this attack.>

**Prevention:**
- <One-line framing, for example whether patching or configuration is the fix.>

| Layer | Control | Prevents | MITRE mitigation |
|---|---|---|---|
| <Account / SSH / Patch / Network / Detection> | <control> | <what it stops> | <Mxxxx: name> |

## 9. Indicators of compromise

| Type | Value |
|---|---|
| <IPv4 / user agent / HASSH / hash / credential / command string> | <value> |

## 10. Confidence and gaps

- **Verified from logs:** <what the evidence shows directly. Appendix A has the query and output for each.>
- **Assessed from indirect evidence:** <each assessment and its basis.>
- **<Retention or capture limits>:** <what the logs could not show.>
- **Third-party data:** <sources and retrieval date; may change.>

## 11. References

1. <Primary sources first: vendor advisories, CVE/NVD entries, SANS ISC diaries, research write-ups.>
2. MITRE ATT&CK: <technique links>

## Appendix A: Evidence

Each exhibit links to the text output of the query beneath it, captured on the sensor. <State the log directory assumed and any scrubbing applied.> [`SHA256SUMS`](SHA256SUMS) lists a hash for each output file.

### Exhibit 1: <Title>

Output: [`ex1-<name>.txt`](ex1-<name>.txt)

```bash
<query that produced the output>
```
