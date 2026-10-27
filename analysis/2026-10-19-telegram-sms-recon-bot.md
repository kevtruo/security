# Honeypot Attack Summary: Telegram Session and SMS Modem Reconnaissance Bot

Date: 2026-10-26 (attacks observed 2026-10-19 and 2026-10-21 UTC)
Status: Verified (from the sensor's Cowrie and firewall logs; see Appendix A)
Tags: honeypot, Cowrie, DShield, SSH, default credentials, Telegram tdata, SMS modem, MikroTik, honeypot detection, MITRE ATT&CK

| | |
|---|---|
| **Sensor** | DShield honeypot (Cowrie) on a cloud VM |
| **Data sources** | Cowrie JSON logs, Cowrie TTY recordings, DShield firewall log, SANS ISC, RIPE WHOIS/RIPEstat, AbuseIPDB |

## 1. Why this attack matters

This attack doesn't want the server's CPU. It's looking for **accounts and phone infrastructure**. Once logged in, the bot checks for exactly three things: a Telegram Desktop session it could copy, a cellular modem or SMS gateway it could abuse, and a MikroTik router. A copied Telegram `tdata` folder lets an attacker sign in as the victim without the password or 2FA code. An SMS-capable device can send spam or receive one-time codes for password resets.

The attack is cheap to run and hard to notice. It needs no exploit, only a default password. It finishes in 4–6 seconds, and it ran from home-broadband IP addresses that don't look like attack infrastructure. Any Linux box, router or IoT device left with default SSH credentials is a candidate, and losing a messaging account or an SMS channel is harder to undo than removing a cryptominer.

## 2. What happened

Two different home-broadband IP addresses logged in to the honeypot over SSH with default credentials. The UK address on 2026-10-19 used `baikal/baikal`; the South Korean address on 2026-10-21 used `root/admin`. Each one ran the **same nine commands in the same order** within 4–6 seconds. Both used the same SSH client (`libssh2_1.11.1`, same HASSH). Cowrie sent both of them byte-identical responses. Neither installed anything. The last command, `echo Hi | cat -n`, checks how the shell responds, and Cowrie's reply differs from a real Linux system's (section 7).

The same command sequence was published in a SANS ISC guest diary in April 2026 [1] and reported against a third IP on AbuseIPDB on 2026-09-02 [5]. That makes this a long-running, automated campaign, not a one-off.

## 3. Timeline (UTC)

| Time | Source IP | Event |
|---|---|---|
| **2026-10-19 00:47:41** | 81.97.108.152 | First packet to TCP/22 |
| 00:47:43 – 00:47:46 | 81.97.108.152 | 4 failed logins as `pi`: `pi`, `raspberry`, and two random-looking passwords |
| 00:47:49 | 81.97.108.152 | **Login succeeds: `baikal/baikal`** (session `c5f4f73f7da5`) |
| 00:47:49 – 00:47:53 | 81.97.108.152 | 9 commands, each over its own SSH channel (section 6) |
| 00:49:10 | 81.97.108.152 | Session closed |
| **2026-10-21 04:30:15** | 125.140.145.62 | First packet to TCP/22 |
| 04:30:23 | 125.140.145.62 | Failed login `root/root` |
| 04:30:24 | 125.140.145.62 | **Login succeeds: `root/admin`** (session `913abb59974b`) |
| 04:30:24 – 04:30:30 | 125.140.145.62 | The same 9 commands, in the same order |
| 04:31:21 | 125.140.145.62 | Session closed |

The firewall recorded only 6 and 4 packets from these IPs, all to TCP/22, and nothing else since August 2026 (Exhibit 5). Each bot showed up with a short credential list, got in within 5 attempts, ran the script and left. This was not a high-volume brute force.

## 4. Where the attack came from

| | 81.97.108.152 | 125.140.145.62 |
|---|---|---|
| ASN / ISP | AS5089, NTL Virgin Media Limited | AS4766, Korea Telecom (KORNET) |
| Country | GB | KR |
| Network type | Residential/consumer broadband | Residential/consumer broadband |
| SANS ISC ipinfo [2] | No prior reports | No prior reports |
| SSH client | `SSH-2.0-libssh2_1.11.1` | `SSH-2.0-libssh2_1.11.1` |
| HASSH | `f45fb203c31069bb280067b71ed92ccb` | `f45fb203c31069bb280067b71ed92ccb` |

**Automated, not hands-on-keyboard.** Several things point to a script:
- The nine commands ran at a steady interval: about 0.43 s apart in one session and 0.68 s in the other.
- Each command ran over its own SSH exec channel; a person typing would use one interactive shell.
- The command list was identical across IPs, dates and credentials.
- The bot ran the `tdata` and modem checks even after `/ip cloud print` had already failed.

**Likely compromised home devices.** Unlike scanners rented from hosting providers, both sources sit on large consumer ISPs, and neither IP had prior SANS ISC reports. That pattern fits a botnet running on infected home routers or PCs. *This is an inference; the ISP data doesn't confirm it.*

## 5. What vulnerability does it exploit?

**No CVE applies.** The attack exploits a configuration weakness, not a software flaw:

- **CWE-1392: Use of Default Credentials** [3]. `root/admin` is a common vendor default, `baikal/baikal` is a username-as-password guess, and `pi/raspberry` is the historic Raspberry Pi OS default.
- **CWE-521: Weak Password Requirements** [3]. The system accepted a trivially guessable password.

The technique is documented as MITRE ATT&CK **T1078.001: Valid Accounts: Default Accounts** [4].

## 6. What the attacker was looking for

| # | Command | What it's for | Honeypot's response |
|---|---|---|---|
| 1 | `/ip cloud print` | MikroTik RouterOS command (cloud/DDNS status). It checks whether the target is a MikroTik router. | Command not found |
| 2 | `ifconfig` | Network interfaces and addresses | Emulated interface list (redacted in evidence) |
| 3 | `uname -a` | OS, kernel, architecture | Emulated Debian x86_64 kernel |
| 4 | `cat /proc/cpuinfo` | CPU model and core count, to judge value for mining | Emulated 2-core Xeon |
| 5–6 | `ps \| grep '[Mm]iner'`, `ps -ef \| grep '[Mm]iner'` | Is a competing cryptominer already running? | No match |
| 7 | `ls -la` over 15 paths | **Telegram Desktop session folders** (`~/.local/share/TelegramDesktop/tdata`); **SMS and cellular modem artifacts** (`/dev/ttyGSM*`, `/dev/ttyUSB-mod*`, `/dev/modem*`, `/var/spool/sms/*`, `/etc/smsd.conf*`, `/var/log/smsd.log`, `/usr/bin/qmuxd`, `/var/qmux_connect_socket`, `/etc/config/simman`, `/var/config/sms/*`) | Every path: "No such file or directory" |
| 8 | `locate D877F783D5D3EF8Cs` | A file name associated with Telegram Desktop `tdata` session data; `locate` would find it outside the default path | No output |
| 9 | `echo Hi \| cat -n` | Checks the shell's response; see section 7 | `     1  Hi` (see section 7) |

## 7. Would the attack succeed?

**Against a vulnerable system: the reconnaissance would succeed, and theft would be the next step.** On a device that accepts these default credentials and has Telegram Desktop or an SMS modem, the script would find them. The ISC diary [1] concludes that the likely next step is to compress and upload the session data, which this bot never reached. The script itself is well built. It probes MikroTik, Linux servers and cellular IoT devices in one pass and tolerates failed commands. Nothing indicates a mistake by the attacker.

**Against this honeypot: the bot probably recognized it.** The last command, `echo Hi | cat -n`, has no use except checking the shell's behavior. Real GNU `cat -n` prints the line number, a **tab** and the text (`     1\tHi`). Cowrie's recorded reply (Exhibit 3b) uses **two spaces** instead of the tab (`20 20` rather than `09`) and sends the number and the text as separate writes. A script comparing the reply byte for byte would see the difference. Two other giveaways appear in the evidence: Cowrie returned byte-identical `ifconfig` and `/proc/cpuinfo` output to both visitors, and the emulated IPv6 address uses `fe77::`, which isn't a valid link-local prefix (real ones start with `fe80::`). Both bots disconnected without any further action.

*Caveat:* the ISC diary describes this command as shell verification [1]. I can't tell from the logs whether the bot judged the reply wrong or simply had nothing left to do. Its file checks all came back empty, so it had no reason to continue either way.

## 8. What was the goal?

**Account and infrastructure theft, supported by resource reconnaissance:**
1. **Telegram account takeover:** copy `tdata` and reuse the session from the attacker's machine. The diary calls this a long-term asset, compared with the short-term gain of cryptojacking [1].
2. **SMS/cellular abuse:** find GSM/LTE modems and SMS daemons to send spam or intercept SMS codes, which can defeat SMS-based 2FA and password resets [1].
3. **Router recruitment:** identify MikroTik devices, which are common botnet targets.
4. **Resource triage:** check the CPU and look for competing miners before deciding whether the machine is worth using for mining.

No payload, persistence or data theft occurred on the honeypot.

## 9. How a defender prevents it

| Layer | Control | Prevents | MITRE mitigation [4] |
|---|---|---|---|
| Account | Change every default credential on first boot. Remove or lock unused accounts such as `pi`, and accounts with names like `baikal`. | The initial login | M1027 Password Policies, M1018 User Account Management |
| SSH | `PasswordAuthentication no` (keys only); `PermitRootLogin no`; `AllowUsers` limited to named admins | Password guessing entirely | M1042 Disable or Remove Feature, M1026 Privileged Account Management |
| SSH | Require MFA (FIDO2 `sk` keys or TOTP) for interactive logins | Use of a guessed or leaked credential alone | M1032 Multi-factor Authentication |
| Network | Don't expose SSH to the internet; use a VPN, bastion host or session manager, or restrict source IPs | The bot ever reaching the login prompt | M1035 Limit Access to Resource Over Network |
| Routers / IoT | MikroTik: disable services you don't use and restrict management access to trusted networks. Cellular routers: change defaults and keep firmware current. | Router and modem recruitment | M1042, M1051 Update Software |
| Telegram | Review **Settings > Devices** (active sessions) and end unknown sessions; set a cloud password (2FA); monitor non-Telegram processes reading `tdata` [1] | Reuse of a stolen session | M1032 |
| Detection | Alert on a new login followed within seconds by `tdata`, `ttyGSM`/`smsd`, `[Mm]iner` or `/ip cloud print` | Unnoticed reconnaissance | — |

**No software upgrade fixes this.** The vulnerable component is the configuration, not a version.

## 10. MITRE ATT&CK mapping

| Tactic | Technique | Evidence |
|---|---|---|
| Initial Access / Persistence | T1078.001: Valid Accounts: Default Accounts | `baikal/baikal`, `root/admin` |
| Credential Access | T1110.001: Brute Force: Password Guessing | 1–4 failed attempts before success |
| Discovery | T1082: System Information Discovery | `uname -a`, `cat /proc/cpuinfo` |
| Discovery | T1016: System Network Configuration Discovery | `ifconfig`, `/ip cloud print` |
| Discovery | T1057: Process Discovery | `ps \| grep '[Mm]iner'` |
| Discovery | T1083: File and Directory Discovery | `ls -la` over Telegram/SMS paths, `locate D877F783D5D3EF8Cs` |
| Credential Access *(targeted, not achieved)* | T1552.001: Unsecured Credentials: Credentials In Files | Search for Telegram `tdata` session files |
| Defense Evasion *(probable)* | T1497.001: Virtualization/Sandbox Evasion: System Checks | `echo Hi \| cat -n` |

## 11. Indicators of compromise

| Type | Value |
|---|---|
| IPv4 | `81.97.108.152` (AS5089, GB) |
| IPv4 | `125.140.145.62` (AS4766, KR) |
| SSH client banner | `SSH-2.0-libssh2_1.11.1` |
| HASSH | `f45fb203c31069bb280067b71ed92ccb` |
| Credentials | `baikal/baikal`, `root/admin` |
| Command string | `locate D877F783D5D3EF8Cs` |
| Command string | `/ip cloud print` followed by `ls -la ~/.local/share/TelegramDesktop/tdata …` |
| Cowrie sessions | `c5f4f73f7da5`, `913abb59974b` |

The source IPs are residential and may be reassigned, so the HASSH and command strings are more durable indicators than the IP addresses.

## 12. Confidence and gaps

- **Verified from logs:** the credentials, the commands and their order, the timing, the SSH fingerprint, the firewall totals, and the byte-level `cat -n` response.
- **Inferred:** that the sources are compromised home devices; that the bot recognized the honeypot; what the attacker would have done next (from the ISC diary [1], not observed here).
- **Retention limits the retrospective view.** The sensor keeps about 8 days of Cowrie logs, so the search for other sessions running this script (Exhibit 6) covers only 2026-10-19 onward. The firewall history (Exhibit 5) goes back to August 2026.
- **Third-party data** (WHOIS, RIPEstat, SANS ISC, AbuseIPDB) was retrieved on 2026-10-26 and may change.

## 13. References

1. L. Carty, "[Guest Diary] Beyond Cryptojacking: Telegram tdata as a Credential Harvesting Vector, Lessons from a Honeypot Incident," SANS Internet Storm Center, 2026-04-22. https://isc.sans.edu/diary/32888
2. SANS ISC IP info: https://isc.sans.edu/ipinfo.html?ip=81.97.108.152 and https://isc.sans.edu/ipinfo.html?ip=125.140.145.62
3. MITRE CWE: [CWE-1392](https://cwe.mitre.org/data/definitions/1392.html), [CWE-521](https://cwe.mitre.org/data/definitions/521.html)
4. MITRE ATT&CK: [T1078.001](https://attack.mitre.org/techniques/T1078/001/), [T1110.001](https://attack.mitre.org/techniques/T1110/001/), [T1082](https://attack.mitre.org/techniques/T1082/), [T1016](https://attack.mitre.org/techniques/T1016/), [T1057](https://attack.mitre.org/techniques/T1057/), [T1083](https://attack.mitre.org/techniques/T1083/), [T1552.001](https://attack.mitre.org/techniques/T1552/001/), [T1497.001](https://attack.mitre.org/techniques/T1497/001/)
5. AbuseIPDB report of the same command sequence from 106.114.105.59, 2026-09-02: https://www.abuseipdb.com/user/344195
6. Cowrie SSH/Telnet honeypot: https://github.com/cowrie/cowrie
7. HASSH SSH client fingerprinting: https://github.com/salesforce/hassh

## Appendix A: Evidence

Each exhibit links to the raw text output of the query beneath it, captured on the sensor. The queries assume the default Cowrie log directory, `/srv/cowrie/var/log/cowrie`. [`SHA256SUMS`](2026-10-19-telegram-sms-recon-bot/SHA256SUMS) lists a hash for each output file.

### Exhibit 1: Full event log of both sessions

Output: [`ex1-sessions.txt`](2026-10-19-telegram-sms-recon-bot/ex1-sessions.txt)

```bash
jq -c --arg a c5f4f73f7da5 --arg b 913abb59974b 'select(.session==$a or .session==$b)|{timestamp,session,eventid,src_ip,username,password,input,version,hassh,duration}' cowrie.json*
```

### Exhibit 2: Logins attempted by each source IP

Output: [`ex2-activity.txt`](2026-10-19-telegram-sms-recon-bot/ex2-activity.txt)

```bash
for ip in 81.97.108.152 125.140.145.62; do echo "== $ip"
  for e in cowrie.session.connect cowrie.login.failed cowrie.login.success cowrie.command.input; do
    echo "$e: $(jq -r --arg ip $ip --arg e $e 'select(.src_ip==$ip and .eventid==$e)|.eventid' cowrie.json* | wc -l)"; done
  jq -r --arg ip $ip 'select(.src_ip==$ip and (.eventid|startswith("cowrie.login")))|"\(.timestamp[0:19]) \(.eventid|sub("cowrie.login.";"")) \(.username)/\(.password)"' cowrie.json*
done
```

### Exhibit 3: Commands and the honeypot's responses (TTY recordings)

Output: [`ex3-tty.txt`](2026-10-19-telegram-sms-recon-bot/ex3-tty.txt). Network identifiers are redacted.

```bash
jq -r 'select((.session=="c5f4f73f7da5" or .session=="913abb59974b") and .eventid=="cowrie.log.closed")|"\(.session) \(.shasum)"' cowrie.json* |
  while read s h; do echo "===== session $s (tty $h)"; strings -n 2 /srv/cowrie/var/lib/cowrie/tty/$h; done
```

### Exhibit 3b: Raw bytes of the `echo Hi | cat -n` response

Output: [`ex3b-cat-n-bytes.txt`](2026-10-19-telegram-sms-recon-bot/ex3b-cat-n-bytes.txt). Offset `0x5e` holds `20 20` (two spaces) where GNU `cat -n` writes `09` (a tab).

```bash
xxd /srv/cowrie/var/lib/cowrie/tty/3e8341eade715d716ae9fe37db380fb9cab5f953782fe691fcf3af29041a344e
```

### Exhibit 4: SSH client fingerprint, and other IPs using it

Output: [`ex4-fingerprint.txt`](2026-10-19-telegram-sms-recon-bot/ex4-fingerprint.txt)

```bash
jq -r 'select(.hassh=="f45fb203c31069bb280067b71ed92ccb")|"\(.timestamp[0:10]) \(.src_ip)"' cowrie.json* | sort | uniq -c
```

### Exhibit 5: Firewall history of both IPs since August 2026

Output: [`ex5-firewall.txt`](2026-10-19-telegram-sms-recon-bot/ex5-firewall.txt). Only the timestamp and destination port are kept.

```bash
grep -oE '"time": [0-9]+, "flags": "[A-Z]+", "sip": "<ip>"[^}]*' dshield-local.log |
  sed -E 's/.*"time": ([0-9]+).*"dport": "([0-9]+)".*/\1 \2/'
```

### Exhibit 6: Every retained session that ran part of the script

Output: [`ex6-campaign-scope.txt`](2026-10-19-telegram-sms-recon-bot/ex6-campaign-scope.txt)

```bash
jq -r 'select(.eventid=="cowrie.command.input" and (.input|test("ip cloud print|TelegramDesktop|ttyGSM|smsd|qmuxd|D877F783")))|"\(.timestamp[0:10]) \(.src_ip) \(.session)"' cowrie.json* | sort | uniq -c
```
