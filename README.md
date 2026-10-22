# security

Public security notes. Two kinds of material live here:

- **Homelab security**: public-safe write-ups of the security work in my homelab, such as change control, cloud identity, and detection. These build on the initiatives in [kevtruo/homelab](https://github.com/kevtruo/homelab).
- **Security analysis**: attack observations, one folder per attack, each holding a summary and its evidence.

Nothing here is a live configuration. Notes describe patterns and decisions, not the exact setup of any system.

## Layout

| Path | Contents |
|---|---|
| [`homelab/`](homelab/) | Homelab security write-ups, grouped by initiative |
| [`analysis/`](analysis/) | One folder per attack observation, holding its `honeypot-attack-summary.md` and all supporting evidence |
| [`templates/`](templates/) | Starting points for new notes |

## Status markers

Every note states one status near the top:

- **Design**: decided or proposed, not built.
- **Implemented**: built, not yet independently verified.
- **Verified**: built and checked, with the method of verification stated.
- **Superseded**: kept for history; links to its replacement.

## Before you publish

Work through the public-safety checklist in the [pull request template](.github/pull_request_template.md) before every merge.

## Reporting a problem

See [`SECURITY.md`](SECURITY.md).
