# Honeypot Attack Summary: SSH Brute Force from 77.239.124.204

| | |
|---|---|
| **Date** | 2026-10-21 (attack observed 2026-10-18 UTC)
| **Sensor** | DShield honeypot (Cowrie) on a cloud VM |
| **Data sources** | Cowrie JSON logs, DShield firewall log, Cowrie TTY recording, AbuseIPDB, RIPE WHOIS |

## 1. Summary

On 2026-10-18, 77.239.124.204 ran an automated SSH brute-force attack against the honeypot. It tried 46 username/password pairs over about 10 minutes. The 46th, `mysql/mysql`, succeeded. Six seconds later the attacker ran a single system-fingerprinting command (`uname -s -v -n -r -m`) and disconnected. It downloaded no malware and set up no persistence.

The attack used a Go-based SSH client with a constant fingerprint, tried each credential pair once from a mixed wordlist, and left right after reconnaissance. Together these point to a large, automated credential-validation botnet, not a targeted, single-operator intrusion. A neighboring address in the same /24 subnet (77.239.124.64) attacked with almost identical volume the next day.

## 2. Attacker attribution

| Field | Value |
|---|---|
| IP address | 77.239.124.204 |
| ASN | AS198364 |
| ISP / Org | ROCKET & MARINICA LTD ("RocketCloud", rocket-cloud.org) |
| Usage type | Data center / web hosting / transit |
| Geolocation | RIPE inetnum lists **FR**, the org address is London **GB**, and AbuseIPDB places it in Kerkrade **NL** |
| Network age | RIPE inetnum and organisation objects were created **July 2026**, about 3 months before the attack |
| Reputation | AbuseIPDB: **3,391 reports, 100% abuse confidence** |

Because geolocation sources disagree, the country of origin can't be determined with confidence. The source is a recently registered hosting range with a heavy abuse history, which is typical of infrastructure rented for scanning.

## 3. Timeline (UTC, 2026-10-18)

| Time | Source | Event |
|---|---|---|
| 05:26:12 | Firewall | First packet to TCP/22 |
| 05:27:22 | Cowrie | First connection; closes immediately (port check) |
| 05:27:47 | Cowrie | First failed login (`root`) |
| 05:30:04 onward | Cowrie | Parallel SSH sessions opened every 2–5 seconds |
| **05:30:23** | Cowrie | **Successful login: `mysql` / `mysql`** (session `cadf2304c7ee`) |
| 05:30:29 | Cowrie | Command executed: `uname -s -v -n -r -m` |
| 05:30:33 | Cowrie | Session `cadf2304c7ee` closed (10 seconds after login) |
| 05:30:32 – 05:37:03 | Cowrie | Brute force continues on other sessions (`root`, `admin`, `devops`, …) |
| 05:37:03 | Cowrie | Last Cowrie event from this IP |
| 08:10:21 | Firewall | Last packet to TCP/22 |

## 4. Activity statistics

| Metric | Count |
|---|---|
| Firewall packets (all TCP/22) | 1,485 |
| Cowrie sessions | 47 |
| Failed logins | 45 |
| Successful logins | 1 |
| Commands executed | 1 |
| Files downloaded / uploaded | 0 |
| Days observed in Cowrie | 1 (2026-10-18 only) |

The firewall saw traffic until 08:10, but Cowrie recorded nothing after 05:37. Most firewall packets were probably SYNs or retries that never completed an SSH handshake.

## 5. Tooling fingerprint

| Indicator | Value | Sessions |
|---|---|---|
| SSH client version | `SSH-2.0-Go` | 46 / 46 |
| HASSH | `16443846184eafde36765c9bab2f4397` | 46 / 46 |

The `SSH-2.0-Go` banner shows a custom tool built on Go's `golang.org/x/crypto/ssh` library rather than a standard client such as OpenSSH. The identical HASSH across every handshake shows a single automated tool.

## 6. Credentials attempted

Each of the 46 pairs was tried exactly once with no retries, a pattern consistent with a shared wordlist split across bot nodes.

| Category | Examples |
|---|---|
| Service accounts | **`mysql/mysql` (success)**, `ftpuser/qwer1234`, `ftpuser2/ftpuser2`, `cacti/cacti`, `deploy/123123`, `devops/admin`, `www/123456` |
| Embedded / IoT boards | `pi/toor` (Raspberry Pi), `admin/linaro` and `newuser/linaro` (Linaro ARM images), `forlinx/root` (Forlinx SoMs), `Tronlong/default` (Tronlong boards) |
| Weak root passwords | `root/P@ssw0rd123`, `root/Admin@123456`, `root/1qaz@WSX`, `root/123qweASD`, `root/root1234`, `root/kali`, `root/jboss` |
| Generic / other | `ubuntu/ubuntu2026`, `user/user@123`, `admin/123456789`, `tom/tom`, `tactical/tactical` |

## 7. MITRE ATT&CK mapping

| Tactic | Technique | Evidence |
|---|---|---|
| Reconnaissance | T1595: Active Scanning | TCP/22 probes starting 05:26:12 |
| Credential Access | T1110.001: Brute Force: Password Guessing | 45 failed logins |
| Initial Access / Persistence | T1078.001: Valid Accounts: Default Accounts | Successful `mysql/mysql` login |
| Discovery | T1082: System Information Discovery | `uname -s -v -n -r -m` |

## 8. Analysis

- **Purpose:** The attacker left right after `uname`. It recorded OS, kernel, hostname and CPU architecture, which a later stage could use to pick a matching payload; access could also be resold.
- **Automation:** The parallel sessions, constant 2–5 second cadence, single-attempt wordlist and Go SSH library fingerprint indicate that an automated tool was used.
- **Coordinated infrastructure:** The neighboring IP **77.239.124.64** sent almost the same firewall traffic (1,498 packets, all TCP/22) and generated 4,848 Cowrie log lines on 2026-10-19. Two addresses in the same newly allocated /24 attacking on consecutive days indicates a single operation. This attribution rests on network and traffic similarity; the SSH client fingerprints of the two addresses were not compared.
- **Separate actor:** The `sshd` malware drop and the 30 MB upload on the same day came from **114.130.253.3**, a different actor, and are not covered in this report.

## 9. Indicators of compromise

| Type | Value |
|---|---|
| IPv4 | `77.239.124.204` |
| IPv4 (related) | `77.239.124.64` |
| ASN | AS198364 (ROCKET & MARINICA LTD) |
| SSH client banner | `SSH-2.0-Go` |
| HASSH | `16443846184eafde36765c9bab2f4397` |
| Credential | `mysql` / `mysql` |
| Cowrie session | `cadf2304c7ee` |
| TTY log SHA-256 | `7ab552f01de999cb12092166cdc36fd68a0edbb33927e0ef3d26f4ee6449f804` |

## 10. Confidence and gaps

- **Verified from logs:** the timeline, session counts, credential list, the command run, the SSH fingerprint and the firewall totals. Appendix A has the query and raw output for each.
- **Third-party data:** the attribution and reputation fields come from RIPE WHOIS and AbuseIPDB as retrieved on 2026-10-21, and may change.

## 11. References

- MITRE ATT&CK: [T1595](https://attack.mitre.org/techniques/T1595/), [T1110.001](https://attack.mitre.org/techniques/T1110/001/), [T1078.001](https://attack.mitre.org/techniques/T1078/001/), [T1082](https://attack.mitre.org/techniques/T1082/)
- Cowrie SSH/Telnet honeypot: https://github.com/cowrie/cowrie
- SANS ISC DShield sensor: https://isc.sans.edu/honeypot.html
- HASSH SSH client fingerprinting: https://github.com/salesforce/hassh
- AbuseIPDB: https://www.abuseipdb.com/check/77.239.124.204

## Appendix A: Evidence

Each exhibit links to the raw text output of the query beneath it, captured on the sensor. The queries assume the default Cowrie log directory, `/srv/cowrie/var/log/cowrie`. [`SHA256SUMS`](SHA256SUMS) lists a hash for each output file.

### Exhibit 1: Event log for session `cadf2304c7ee`

Output: [`ex1-session.txt`](ex1-session.txt)

```bash
jq -c 'select(.session=="cadf2304c7ee")|{timestamp,eventid,src_ip,username,password,input}' cowrie.json.2026-10-18
```

### Exhibit 2: Activity totals and credential attempts

Output: [`ex2-credentials.txt`](ex2-credentials.txt)

```bash
IP=77.239.124.204
for e in cowrie.session.connect cowrie.login.failed cowrie.login.success cowrie.command.input; do echo "$e: $(jq -r --arg ip $IP --arg e $e 'select(.src_ip==$ip and .eventid==$e)|.eventid' cowrie.json.2026-10-18 | wc -l)"; done
jq -r --arg ip $IP 'select(.src_ip==$ip and (.eventid|startswith("cowrie.login")))|"\(.eventid|sub("cowrie.login.";"")) \(.username)/\(.password)"' cowrie.json.2026-10-18 | sort | uniq -c | sort -rn
```

### Exhibit 3: SSH client fingerprint

Output: [`ex3-fingerprint.txt`](ex3-fingerprint.txt)

```bash
jq -r --arg ip $IP 'select(.src_ip==$ip and (.eventid=="cowrie.client.version" or .eventid=="cowrie.client.kex"))|(.version // .hassh)' cowrie.json.2026-10-18 | sort | uniq -c
```

### Exhibit 4: Firewall summary (ports, first and last seen)

Output: [`ex4-firewall.txt`](ex4-firewall.txt)

```bash
zcat -f /var/log/dshield.log* | grep "SRC=$IP " | grep -oE "DPT=[0-9]+" | sort | uniq -c | sort -rn
zcat -f /var/log/dshield.log* | grep "SRC=$IP " | awk '{print $1}' | sort -n | sed -n '1p;$p' | xargs -I{} date -u -d @{}
```

### Exhibit 5: TTY replay of session `cadf2304c7ee`

Output: [`ex5-tty-replay.txt`](ex5-tty-replay.txt)

The recording is 202 bytes and holds two records. The first is the attacker's input, `uname -s -v -n -r -m`. The second is the emulated response Cowrie returned:

```text
Linux svr04 6.1.0-21-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.90-1 (2024-05-03) x86_64
```

This is fake system information generated by the honeypot, not the sensor's real kernel or hostname. `playlog` replays only the input record, so the exhibit also includes a hex dump of the raw recording that shows the response.

```bash
T=/srv/cowrie/var/lib/cowrie/tty/7ab552f01de999cb12092166cdc36fd68a0edbb33927e0ef3d26f4ee6449f804
sudo -u cowrie /srv/cowrie/cowrie-env/bin/python3 /srv/cowrie/bin/playlog -m 0 $T
xxd $T
```

### Exhibit 6: AbuseIPDB reputation

Source: https://www.abuseipdb.com/check/77.239.124.204 (retrieved 2026-10-21: 3,391 reports, 100% abuse confidence, AS198364, data center/web hosting/transit)
