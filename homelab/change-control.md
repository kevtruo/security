# Change control with an AI contributor

Status: **Design**, with a sandbox pilot authorized and not yet run (as of 2026-10-22).
Initiative: yeoman.

## Problem

I want an AI agent to open pull requests in my repositories without being able to merge them, change repository settings, or reach cloud credentials. The agent's GitHub App has broad permissions that cannot be accepted partially, so the controls have to come from elsewhere.

## Approach

- **Identity**: the agent acts as a GitHub App bot account, separate from my own account. Every change it makes is attributable to the bot.
- **Trigger**: only my account can trigger the agent's workflow. The workflow checks the actor, and the action itself rejects triggers from users without write access.
- **Least privilege in the workflow**: the workflow requests only the permissions it needs (contents, pull requests, issues) plus the OIDC token permission the action requires to authenticate.
- **Restricted tools**: the agent gets an explicit allow-list of tools. No unrestricted shell.
- **Bounded runs**: the action is pinned to a commit SHA, with a maximum turn count and a job timeout.
- **Merge controls**: a ruleset on `main` requires my approval, dismisses stale approvals on new commits, and gives the bot no bypass. The bot never merges.
- **Cloud isolation**: no cloud trust policy may accept a token from the agent's workflow. This is enforced on the cloud side, see [OIDC trust scoping](oidc-trust-scoping.md).

## Known risk

The App holds write permission on workflows and on repository webhooks. A bot-authored branch could therefore change a workflow file. Two consequences follow:

1. Any control that lives in a workflow file on a PR branch can be edited by that branch. Server-side controls (rulesets, environments with required reviewers, cloud trust conditions) are the ones that hold.
2. The pilot tests whether the bot can create a webhook or change settings, in a throwaway sandbox only.

## Test plan (pilot)

Pass/fail:
1. A comment from me makes the bot open a PR, authored by the bot.
2. CI runs on the bot's PR.
3. I can be requested as reviewer and approve.
4. The bot cannot merge before approval.
5. The bot cannot merge after approval.
6. A new bot commit after approval dismisses the approval.
7. I can merge through the PR.

Record only:
8. The effective permissions of the runtime token, and whether a workflow change on a bot branch runs.
9. Trigger restriction, verified by configuration.
10. Whether the bot can create a webhook or change settings.

## Results

Not run yet. Results will be added here as a summary; detailed evidence stays in a private repository.

## References

- [Claude Code GitHub Actions](https://docs.anthropic.com/en/docs/claude-code/github-actions)
- [About rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets)
- [Security hardening for GitHub Actions](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions)
