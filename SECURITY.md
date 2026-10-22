# Security policy

This repository holds notes, not software. Two kinds of report are still useful:

1. **Something sensitive was published here.** For example: a credential, a real internal address or hostname, an account identifier, or detail that maps a real network.
2. **A note is wrong in a way that could lead someone to an insecure setup.**

## How to report

Use GitHub's private vulnerability reporting: open the **Security** tab of this repository and choose **Report a vulnerability**. Please do not open a public Issue for case 1, because that republishes the detail.

For case 2, a public Issue is fine.

## What happens next

- Case 1: the content is removed and any affected credential is revoked or rotated. Removing a file from the latest commit does not remove it from git history, so a leaked secret is always treated as compromised.
- Case 2: the note is corrected, or marked with a caveat until it can be.
