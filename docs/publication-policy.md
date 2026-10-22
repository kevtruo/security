# Publication policy

This repository is public. Anything pushed here should be treated as permanent, even if it is later deleted, because forks, caches, and clones may keep it.

## Never publish

- Secrets of any kind: tokens, keys, passwords, private keys, session cookies, OAuth or API credentials. Also do not publish the **names** of secrets when the name reveals how a system is wired.
- Real IP addresses, subnets, VLAN IDs, MAC addresses, or DNS names from the homelab or any network I operate.
- Hostnames, device names, and serial numbers.
- Cloud account IDs, organization IDs, role ARNs, resource IDs, and bucket names.
- GitHub App IDs, installation IDs, webhook URLs, and private repository names, Issue numbers, PR numbers, branches, or comment links.
- Ruleset, environment, or policy names that are specific to a private system.
- Local filesystem paths and usernames.
- Packet captures, logs, or screenshots that have not been scrubbed of the above.
- Detail that maps a live network: which sensor sits where, what is exposed, and what is not monitored.
- Anything about systems I do not own or do not have permission to test.

## How to refer to private things

- Private evidence is referred to only as "recorded in a private repository". No links, numbers, or names.
- Use role names instead of real names: "the infrastructure repository", "the sandbox repository", "the management account", "the plan role".

## Placeholders

Use reserved values so that examples can never collide with something real.

| Kind | Placeholder | Reserved by |
|---|---|---|
| IPv4 | `192.0.2.0/24`, `198.51.100.0/24`, `203.0.113.0/24` | RFC 5737 |
| IPv6 | `2001:db8::/32` | RFC 3849 |
| Domain | `example.com`, `example.net`, `example.org`, `*.example` | RFC 2606 |
| AWS account ID | `123456789012` | Common documentation convention |
| Anything else | `<redacted>` or `<placeholder: what it is>` | — |

## Checks before every push

1. Read the full diff: `git diff --cached`.
2. Run a secret scan over the staged changes. With [gitleaks](https://github.com/gitleaks/gitleaks) installed and the hook in [`.pre-commit-config.yaml`](../.pre-commit-config.yaml) enabled, this runs on every commit.
3. Grep the diff for addresses and paths:
   ```sh
   git diff --cached | grep -nE '([0-9]{1,3}\.){3}[0-9]{1,3}|/Users/|/home/|C:\\\\|arn:aws:|[0-9]{12}'
   ```
   Every hit must be a reserved placeholder from the table above.
4. Check images and attachments by hand. Scanners do not read screenshots.

## Repository settings that back this up

These are set in the GitHub web UI, not in files:

- Secret scanning and push protection: on.
- Private vulnerability reporting: on.
- Branch protection or a ruleset on `main`: changes arrive by pull request.

## If something leaks

1. Revoke or rotate the credential first. Assume it is compromised.
2. Remove the content.
3. If history must be rewritten, do it deliberately and record that it happened. A rewrite does not undo exposure.
