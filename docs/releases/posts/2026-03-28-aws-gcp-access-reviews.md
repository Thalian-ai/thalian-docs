---
date: 2026-03-28
slug: aws-gcp-access-reviews
---

# AWS IAM, GCP IAM, access review campaigns, and AI remediation planner

Two new cloud IAM integrations, structured access review campaigns with ITSM integration, and Claude-driven remediation planning with reasoning.

<!-- more -->

## AWS IAM integration

Connect AWS Identity and Access Management to detect IAM users that exist outside your corporate IDP lifecycle controls. Thalian syncs all IAM users via `GetAccountAuthorizationDetails`, detects admin-level accounts (AdministratorAccess / PowerUserAccess) with no matching IDP identity, IAM users without MFA enrolled, and stale IAM users whose IDP account has been suspended or deprovisioned. Auth uses Access Key ID + Secret Access Key, no OAuth required.

## GCP IAM integration

Connect Google Cloud Platform to detect IAM access gaps between your GCP projects and your corporate identity provider. Thalian maps every human member across all your GCP projects and cross-references against Okta, Entra ID, Google Workspace, JumpCloud, and OneLogin. Four new findings:

- **GCP project owner not in IDP** (critical)
- **GCP member not in IDP** (high)
- **Public IAM binding** via `allUsers` or `allAuthenticatedUsers` (critical)
- **Stale IAM binding** for a suspended or deprovisioned user (high)

## Access review campaigns

IT and security teams can now run structured user access certification campaigns directly in Thalian. Create a campaign, scope it by department or application, and work through a paginated list of entitlements — approving access, revoking it, or granting a time-limited exception. Revoke decisions automatically open an ITSM ticket in Jira, ServiceNow, Freshservice, or Zendesk. Completed campaigns export to CSV as audit evidence for SOC 2, ISO 27001, and HIPAA reviews.

## Claude-driven agentic remediation planner

After every sync, Claude Sonnet now reviews all new findings and decides which ones need action, what action to take, and why. Queued actions include a "Claude's reasoning" block — e.g., "This account hasn't logged into the IDP in 47 days but retains admin roles in Salesforce and GitHub. Suspending reduces blast radius while the offboarding gap is investigated." The planner sequences actions correctly, groups multi-finding actions on the same identity, and skips low-priority findings.

[View on GitHub](https://github.com/Thalian-ai/thalian/releases/tag/2026-03-28)
