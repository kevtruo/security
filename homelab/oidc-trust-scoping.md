# OIDC trust scoping for CI roles

Status: **Design** (planned, not authorized, as of 2026-10-22).
Initiative: quartermaster.

## Problem

CI workflows get cloud credentials by exchanging a GitHub OIDC token for a short-lived role session. If a role's trust policy matches broad subjects, such as "any workflow on `main`" or "any pull request", then any workflow that can produce such a token can assume the role. That includes a workflow file edited on a PR branch.

## Goal

No cloud role can be assumed by any workflow except the named infrastructure workflows, in their intended context:

- **Apply roles**: the named workflow, on `main`, in a protected apply environment.
- **Plan roles**: the named workflow, in an environment that requires my approval.

No pipeline outage during the change.

## Design

1. **Pin the workflow in the subject claim.** Customize the repository's OIDC subject template to include `job_workflow_ref`. Trust policies then match the exact workflow file and ref, not just the repository and branch.
2. **Gate plan runs server-side.** Every plan job runs in an environment with a required reviewer, and the plan role's trust requires that environment. A PR branch can edit its own workflow file, so a gate that lives in the file does not hold. An environment's protection rules live in repository settings and do.
   Cost: one approval click per plan run.
3. **Cut over in three phases with dual-accept.**
   - Phase A: trust accepts both the current subjects and the new pinned subjects.
   - Phase B: switch the subject template and verify plan and apply still work.
   - Phase C: remove the legacy subjects, including the generic branch and pull request matches.
   At every step at least one valid path exists, so there is no lockout.
4. **Rollback** goes through a separate administrative path outside CI.

## Verification plan

- Readback of every role's trust policy after each phase.
- Negative tests from a throwaway branch: a test workflow tries to assume each role and must be denied.
- Positive tests: the real pipelines plan and apply with no changes.

## Follow-on

- Review plan-role permissions, including read access to Terraform state, because state can hold sensitive values.
- Renaming the repository changes the subject strings again, so it must reuse the dual-accept pattern.

## References

- [About security hardening with OpenID Connect](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/about-security-hardening-with-openid-connect)
- [Configuring OpenID Connect in Amazon Web Services](https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services)
- [Managing environments for deployment](https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-deployments/managing-environments-for-deployment)
- [IAM: Creating a role for OIDC federation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-idp_oidc.html)
