# security

Public security notes. Two kinds of material live here:

- **Homelab security**: public-safe write-ups of the security work in my homelab, such as change control, cloud identity, and detection. These build on the initiatives in [kevtruo/homelab](https://github.com/kevtruo/homelab).
- **Security analysis**: standalone notes on tools, vulnerabilities, techniques, and standards.

Nothing here is a live configuration. Notes describe patterns and decisions, not the exact setup of any system.

## Layout

| Path | Contents |
|---|---|
| [`homelab/`](homelab/) | Homelab security write-ups, grouped by initiative |
| [`analysis/`](analysis/) | Standalone security analysis notes |
| [`templates/`](templates/) | Starting points for new notes |
| [`docs/publication-policy.md`](docs/publication-policy.md) | What may and may not be published here, and the placeholders to use |

## Status markers

Every note states one status near the top:

- **Design**: decided or proposed, not built.
- **Implemented**: built, not yet independently verified.
- **Verified**: built and checked, with the method of verification stated.
- **Superseded**: kept for history; links to its replacement.

## Before you publish

Read [`docs/publication-policy.md`](docs/publication-policy.md). Run the checks in it before every push.

## Reporting a problem

See [`SECURITY.md`](SECURITY.md).
