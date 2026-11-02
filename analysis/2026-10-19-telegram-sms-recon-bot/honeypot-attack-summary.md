# Honeypot Attack Summary: Telegram Session and SMS Modem Reconnaissance Bot

| | |
|---|---|
| **Date** | 2026-10-26 (attacks observed 2026-10-19 and 2026-10-21 UTC) |
| **Sensor** | DShield honeypot (Cowrie) on a cloud VM |
| **Data sources** | Cowrie JSON logs, Cowrie TTY recordings, DShield firewall log, SANS ISC, RIPE WHOIS/RIPEstat, AbuseIPDB |

## 1. Summary

- Two home-broadband IPs logged in over SSH with default credentials: the UK on 2026-10-19 (`baikal/baikal`) and South Korea on 2026-10-21 (`root/admin`).
- Both ran the same nine commands in the same order within 4–6 seconds, using the same SSH client (`libssh2_1.11.1`).
- The script looks for accounts and phone infrastructure rather than CPU: Telegram Desktop sessions, cellular modems and SMS gateways, and MikroTik routers.
- Its last command checks shell behavior, and Cowrie's reply differs from a real Linux system's.
- The same sequence appears in a SANS ISC diary from April 2026 [1] and an AbuseIPDB report from 2026-09-02 [5], so the campaign has run for months.

## 2. Attacker attribution

| Field | 81.97.108.152 | 125.140.145.62 |
|---|---|---|
| ASN | AS5089 | AS4766 |
| ISP / Org | NTL Virgin Media Limited | Korea Telecom (KORNET) |
| Usage type | Residential/consumer broadband | Residential/consumer broadband |
| Geolocation | GB | KR |
| Reputation | SANS ISC [2]: no prior reports | SANS ISC [2]: no prior reports |

- Both sources are consumer ISPs with no prior SANS ISC reports, consistent with compromised home routers or PCs.

## 3. Timeline (UTC)

| Time | Source | Event |
|---|---|---|
| 2026-10-19 00:47:41 | Firewall | First packet from 81.97.108.152 to TCP/22 |
| 00:47:43 – 00:47:46 | Cowrie | 4 failed logins as `pi` |
| 00:47:49 | Cowrie | Successful login: `baikal` / `baikal` (session `c5f4f73f7da5`) |
| 00:47:49 – 00:47:53 | Cowrie | 9 commands, each over its own SSH channel (section 6) |
| 00:49:10 | Cowrie | Session `c5f4f73f7da5` closed |
| 2026-10-21 04:30:15 | Firewall | First packet from 125.140.145.62 to TCP/22 |
| 04:30:23 | Cowrie | Failed login `root/root` |
| 04:30:24 | Cowrie | Successful login: `root` / `admin` (session `913abb59974b`) |
| 04:30:24 – 04:30:30 | Cowrie | The same 9 commands, in the same order |
| 04:31:21 | Cowrie | Session `913abb59974b` closed |

## 4. Activity statistics

| Metric | 81.97.108.152 | 125.140.145.62 |
|---|---|---|
| Firewall packets (all TCP/22) | 6 | 4 |
| Cowrie sessions | 2 | 1 |
| Failed logins | 4 | 1 |
| Successful logins | 1 | 1 |
| Commands executed | 9 | 9 |
| Files downloaded / uploaded | 0 | 0 |
| Days observed (firewall, since August 2026) | 1 (2026-10-19 only) | 1 (2026-10-21 only) |

- Low volume: each bot got in within five attempts, ran the script and left.

## 5. Tooling fingerprint

| Indicator | Value | Sessions |
|---|---|---|
| SSH client version | `SSH-2.0-libssh2_1.11.1` | 3 / 3 |
| HASSH | `f45fb203c31069bb280067b71ed92ccb` | 3 / 3 |

- `libssh2` indicates a scripted client, not OpenSSH.
- No other IP in the retained logs (2026-10-19 onward) used this HASSH.

## 6. Credentials and commands

**Credentials attempted:**

| Source | Failed | Success |
|---|---|---|
| 81.97.108.152 | `pi/pi`, `pi/raspberry`, `pi/5nWt3P-fF4WosQm5O`, `pi/FAqY7=MZk66k-ob3Rmk` | `baikal/baikal` |
| 125.140.145.62 | `root/root` | `root/admin` |

- `pi/raspberry` and `root/admin` are common defaults; the random-looking `pi` passwords are consistent with leaked credential lists.

**Commands executed:**

| # | Command | Purpose |
|---|---|---|
| 1 | `/ip cloud print` | Is this a MikroTik router? |
| 2–4 | `ifconfig`, `uname -a`, `cat /proc/cpuinfo` | Network, OS and CPU profile |
| 5–6 | `ps \| grep '[Mm]iner'`, `ps -ef \| grep '[Mm]iner'` | Is a competing cryptominer running? |
| 7 | `ls -la` over 15 paths | Telegram Desktop sessions, cellular modems, SMS gateways |
| 8 | `locate D877F783D5D3EF8Cs` | Telegram session data outside the default path |
| 9 | `echo Hi \| cat -n` | Honeypot check |

- The commands were identical in both sessions, and every check came back negative on the sensor.
- Full commands and responses: Exhibit 3.

## 7. MITRE ATT&CK mapping

| Tactic | Technique | Evidence |
|---|---|---|
| Credential Access | T1110.001: Brute Force: Password Guessing | 1–4 failed attempts before success |
| Initial Access / Persistence | T1078.001: Valid Accounts: Default Accounts | `baikal/baikal`, `root/admin` |
| Discovery | T1082: System Information Discovery | `uname -a`, `cat /proc/cpuinfo` |
| Discovery | T1016: System Network Configuration Discovery | `ifconfig`, `/ip cloud print` |
| Discovery | T1057: Process Discovery | `ps \| grep '[Mm]iner'` |
| Discovery | T1083: File and Directory Discovery | `ls -la` over Telegram/SMS paths, `locate D877F783D5D3EF8Cs` |
| Credential Access *(targeted, not achieved)* | T1552.001: Unsecured Credentials: Credentials In Files | Search for Telegram `tdata` session files |
| Defense Evasion *(probable)* | T1497.001: Virtualization/Sandbox Evasion: System Checks | `echo Hi \| cat -n` |

## 8. Analysis

**Relevance:**
- A copied Telegram session lets an attacker sign in as the victim without the password or 2FA code.
- An SMS-capable device can send spam or receive password-reset codes.
- Any Linux host, router or IoT device with default SSH credentials is a target; no exploit is needed.

**Vulnerability:**
- No CVE applies; this is a configuration weakness: CWE-1392 (default credentials) and CWE-521 (weak passwords) [3].

**Purpose:**
- Inventory the target for Telegram account takeover, SMS and cellular abuse, router recruitment and cryptomining value [1].

**Automation:**
- Fixed command order at a steady interval (0.43–0.68 seconds), one SSH channel per command, identical across IPs, dates and credentials.
- The script carried on after `/ip cloud print` failed.

**Would it succeed against a vulnerable system?**
- Yes, for reconnaissance: on a device with Telegram Desktop or an SMS modem, the script would find them.
- The next step, exfiltrating the session data, was not reached [1].

**Honeypot detection:**
- Cowrie's `cat -n` reply uses two spaces where real `cat` uses a tab (Exhibit 3b).
- Identical `ifconfig` and `/proc/cpuinfo` output for both visitors also gives the honeypot away (Exhibit 3).
- Both bots disconnected after the check; with every file check empty, there was nothing to pursue.

**Prevention:**
- Configuration is the fix here; no software upgrade applies.

| Layer | Control | Prevents | MITRE mitigation [4] |
|---|---|---|---|
| Account | Change every default credential on first boot. Remove or lock unused accounts such as `pi`, and accounts with names like `baikal`. | The initial login | M1027 Password Policies, M1018 User Account Management |
| SSH | `PasswordAuthentication no` (keys only); `PermitRootLogin no`; `AllowUsers` limited to named admins | Password guessing entirely | M1042 Disable or Remove Feature, M1026 Privileged Account Management |
| SSH | Require MFA (FIDO2 `sk` keys or TOTP) for interactive logins | Use of a guessed or leaked credential alone | M1032 Multi-factor Authentication |
| Network | Don't expose SSH to the internet; use a VPN, bastion host or session manager, or restrict source IPs | The bot ever reaching the login prompt | M1035 Limit Access to Resource Over Network |
| Routers / IoT | MikroTik: disable unused services and restrict management access to trusted networks. Cellular routers: change defaults and keep firmware current. | Router and modem recruitment | M1042, M1051 Update Software |
| Telegram | Review Settings > Devices (active sessions) and end unknown sessions; set a cloud password (2FA); monitor non-Telegram processes reading `tdata` [1] | Reuse of a stolen session | M1032 |
| Detection | Alert on a new login followed within seconds by `tdata`, `ttyGSM`/`smsd`, `[Mm]iner` or `/ip cloud print` | Unnoticed reconnaissance | — |

## 9. Indicators of compromise

| Type | Value |
|---|---|
| IPv4 | `81.97.108.152` (AS5089, GB) |
| IPv4 | `125.140.145.62` (AS4766, KR) |
| IPv4 (related, AbuseIPDB [5]) | `106.114.105.59` |
| SSH client banner | `SSH-2.0-libssh2_1.11.1` |
| HASSH | `f45fb203c31069bb280067b71ed92ccb` |
| Credentials | `baikal/baikal`, `root/admin` |
| Command string | `locate D877F783D5D3EF8Cs` |
| Command string | `/ip cloud print` followed by `ls -la ~/.local/share/TelegramDesktop/tdata …` |
| Cowrie sessions | `c5f4f73f7da5`, `913abb59974b` |

- Residential IPs may be reassigned; the HASSH and command strings are more durable indicators.

## 10. Confidence and gaps

**Verified from logs:**
- Credentials, commands and their order, timing, the SSH fingerprint, firewall totals and the byte-level `cat -n` response (Appendix A).

**Assessed from indirect evidence:**
- Compromised home devices as the sources, based on ISP type and no prior reports.
- Recognition of the honeypot, based on the `cat -n` response.
- Exfiltration as the next stage, from the ISC diary [1]; not observed.

**Retention:**
- About 8 days of Cowrie logs, so the campaign search (Exhibit 6) covers 2026-10-19 onward; firewall history (Exhibit 5) goes back to August 2026.

**Third-party data:**
- WHOIS, RIPEstat, SANS ISC and AbuseIPDB data retrieved 2026-10-26; may change.

## 11. References

1. L. Carty, "[Guest Diary] Beyond Cryptojacking: Telegram tdata as a Credential Harvesting Vector, Lessons from a Honeypot Incident," SANS Internet Storm Center, 2026-04-22. https://isc.sans.edu/diary/32888
2. SANS ISC IP info: https://isc.sans.edu/ipinfo.html?ip=81.97.108.152 and https://isc.sans.edu/ipinfo.html?ip=125.140.145.62
3. MITRE CWE: [CWE-1392](https://cwe.mitre.org/data/definitions/1392.html), [CWE-521](https://cwe.mitre.org/data/definitions/521.html)
4. MITRE ATT&CK: [T1110.001](https://attack.mitre.org/techniques/T1110/001/), [T1078.001](https://attack.mitre.org/techniques/T1078/001/), [T1082](https://attack.mitre.org/techniques/T1082/), [T1016](https://attack.mitre.org/techniques/T1016/), [T1057](https://attack.mitre.org/techniques/T1057/), [T1083](https://attack.mitre.org/techniques/T1083/), [T1552.001](https://attack.mitre.org/techniques/T1552/001/), [T1497.001](https://attack.mitre.org/techniques/T1497/001/)
5. AbuseIPDB report of the same command sequence from 106.114.105.59, 2026-09-02: https://www.abuseipdb.com/user/344195
6. Cowrie SSH/Telnet honeypot: https://github.com/cowrie/cowrie
7. SANS ISC DShield sensor: https://isc.sans.edu/honeypot.html
8. HASSH SSH client fingerprinting: https://github.com/salesforce/hassh

## Appendix A: Evidence

Each exhibit links to the raw text output of the query beneath it, captured on the sensor. The queries assume the default Cowrie log directory, `/srv/cowrie/var/log/cowrie`. [`SHA256SUMS`](SHA256SUMS) lists a hash for each output file.

### Exhibit 1: Event log for sessions `c5f4f73f7da5` and `913abb59974b`

Output: [`ex1-sessions.txt`](ex1-sessions.txt)

```bash
jq -c --arg a c5f4f73f7da5 --arg b 913abb59974b 'select(.session==$a or .session==$b)|{timestamp,session,eventid,src_ip,username,password,input,version,hassh,duration}' cowrie.json*
```

### Exhibit 2: Activity totals and credential attempts

Output: [`ex2-activity.txt`](ex2-activity.txt)

```bash
for ip in 81.97.108.152 125.140.145.62; do echo "== $ip"
  for e in cowrie.session.connect cowrie.login.failed cowrie.login.success cowrie.command.input; do
    echo "$e: $(jq -r --arg ip $ip --arg e $e 'select(.src_ip==$ip and .eventid==$e)|.eventid' cowrie.json* | wc -l)"; done
  jq -r --arg ip $ip 'select(.src_ip==$ip and (.eventid|startswith("cowrie.login")))|"\(.timestamp[0:19]) \(.eventid|sub("cowrie.login.";"")) \(.username)/\(.password)"' cowrie.json*
done
```

### Exhibit 3: Commands and the honeypot's responses (TTY recordings)

Output: [`ex3-tty.txt`](ex3-tty.txt). Network identifiers are redacted.

```bash
jq -r 'select((.session=="c5f4f73f7da5" or .session=="913abb59974b") and .eventid=="cowrie.log.closed")|"\(.session) \(.shasum)"' cowrie.json* |
  while read s h; do echo "===== session $s (tty $h)"; strings -n 2 /srv/cowrie/var/lib/cowrie/tty/$h; done
```

### Exhibit 3b: Raw bytes of the `echo Hi | cat -n` response

Output: [`ex3b-cat-n-bytes.txt`](ex3b-cat-n-bytes.txt). Offset `0x5e` holds `20 20` (two spaces) where GNU `cat -n` writes `09` (a tab).

```bash
xxd /srv/cowrie/var/lib/cowrie/tty/3e8341eade715d716ae9fe37db380fb9cab5f953782fe691fcf3af29041a344e
```

### Exhibit 4: SSH client fingerprint, and other IPs using it

Output: [`ex4-fingerprint.txt`](ex4-fingerprint.txt)

```bash
jq -r 'select(.hassh=="f45fb203c31069bb280067b71ed92ccb")|"\(.timestamp[0:10]) \(.src_ip)"' cowrie.json* | sort | uniq -c
```

### Exhibit 5: Firewall history of both IPs since August 2026

Output: [`ex5-firewall.txt`](ex5-firewall.txt). Only the timestamp and destination port are kept.

```bash
grep -oE '"time": [0-9]+, "flags": "[A-Z]+", "sip": "<ip>"[^}]*' dshield-local.log |
  sed -E 's/.*"time": ([0-9]+).*"dport": "([0-9]+)".*/\1 \2/'
```

### Exhibit 6: Every retained session that ran part of the script

Output: [`ex6-campaign-scope.txt`](ex6-campaign-scope.txt)

```bash
jq -r 'select(.eventid=="cowrie.command.input" and (.input|test("ip cloud print|TelegramDesktop|ttyGSM|smsd|qmuxd|D877F783")))|"\(.timestamp[0:10]) \(.src_ip) \(.session)"' cowrie.json* | sort | uniq -c
```
