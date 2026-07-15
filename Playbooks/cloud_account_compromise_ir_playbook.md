# Cloud Account Compromise Response Playbook

**Primary MITRE Techniques:** T1078.004 (Valid Accounts: Cloud Accounts), T1098.003 (Additional Cloud Roles), T1528 (Steal Application Access Token), T1114.003 (Email Forwarding Rule)
**Source Cases:** Drill practice — cloud impossible-travel login, new OAuth app broad-permission grant, cloud admin role granted without record, hidden mailbox auto-forward rule

**Note on scope:** Cloud identity compromise differs from on-premises credential compromise (`Credential-Compromise-Playbook.md`) in blast radius and containment mechanics — a compromised cloud admin account can affect every connected SaaS service simultaneously, and containment relies on the identity provider's controls rather than local host isolation.

---

## 1. Preparation

- **Conditional Access / risk-based sign-in policies configured** in the identity provider (Azure AD, Okta, Google Workspace) to auto-challenge or block anomalous logins
- **OAuth app consent restricted** — admin consent required for apps requesting broad permissions (Mail.ReadWrite, Files.ReadWrite.All, Directory.Read.All), rather than allowing user self-consent
- **Privileged role assignment monitored** — alerts configured for any Global Admin / high-privilege role grant, with automatic notification to security team
- **Mailbox rule auditing enabled** — many identity providers can flag or block auto-forward rules to external domains by default

---

## 2. Detection & Analysis

**Primary indicators:**

| Indicator | Verdict Guidance |
|---|---|
| Cloud console login from two geographically distant locations within an implausible timeframe | TP — impossible travel on an admin console carries elevated severity versus a standard workstation login, given broader blast radius |
| New OAuth app granted broad permissions (Mail, Files, Directory) by a user, app not on the approved allowlist | TP — scope requested vastly exceeds what a typical productivity add-on needs; classic OAuth-consent-phishing pattern |
| Global Administrator (or equivalent) role granted with no corresponding change ticket or approval record | TP — undocumented high-privilege grant is decisive, especially if the granting account is itself non-baseline |
| Hidden mail-forwarding rule created, forwarding to an external address, not visible in the normal rules UI | TP — a well-documented post-compromise mailbox-monitoring persistence technique; no legitimate reason to hide a forwarding rule from the account owner's own view |
| OAuth token used from a new, unrecognized device with no corresponding fresh MFA challenge | TP — valid token used without a new authentication event suggests token theft, which can bypass password/MFA controls entirely |

**Verdict discipline:** Cloud identity alerts often look individually minor (a permission grant, a login from a new location) but should be evaluated for **compounding effect** — a new OAuth app grant *combined with* a subsequent hidden forwarding rule is a much stronger signal together than either alone, similar to the cross-alert correlation principle applied in `SOC-Triage-Practice` Case_017/019.

---

## 3. Containment

**Short-term:**
1. Immediately revoke all active sessions and refresh tokens for the affected account via the identity provider console — a simple password reset is insufficient, since stolen OAuth tokens/refresh tokens can persist access even after a password change.
2. Revoke consent for any suspicious OAuth application immediately.
3. Remove any unauthorized privileged role grants.
4. Delete or disable any hidden/unauthorized mailbox forwarding rules.

**Long-term:**
5. Review the affected account's full activity log (sign-ins, file access, sent mail, admin actions) for the entire suspected compromise window, not just the triggering alert.
6. Check whether the compromised account's privileges were used to grant further access, create additional accounts, or modify security policies — cloud compromises often escalate quickly given the interconnected nature of SaaS permissions.

---

## 4. Eradication

1. Force a full credential reset (password + re-registration of MFA factors) for the affected account — do not simply reset the password if MFA factors may also be compromised.
2. Audit and remove any persistence mechanisms established during the compromise window (forwarding rules, OAuth grants, delegated mailbox access, newly created service principals/API keys).
3. Review Conditional Access and consent policies for the gap that allowed the initial compromise (e.g. was admin consent for OAuth apps not enforced? Was impossible-travel detection not configured?).

---

## 5. Recovery

1. Restore normal access for the account with fresh credentials and MFA registration.
2. Re-verify all cloud service integrations and delegated permissions the account previously held are correctly scoped, not restored with excess privilege by default.
3. Monitor the account closely for a defined period following restoration.

---

## 6. Lessons Learned

- How was initial access to the cloud account obtained (phishing, password reuse, OAuth consent phishing)? Address the entry vector.
- Did Conditional Access / risk-based sign-in policies catch this, or was it identified through manual review? If manual, what policy configuration gap should be closed?
- Was the OAuth app consent process too permissive? Consider requiring admin consent for all non-Microsoft/Google-verified publishers going forward.
- Update `Noise-Baselines/known_legitimate_accounts.md`-style documentation with any newly confirmed legitimate cloud app or admin-grant pattern discovered during investigation.

---

## What a Real Enterprise Version Would Add

- **Identity provider-specific runbooks** — exact console steps for the organization's actual IdP (Azure AD/Entra, Okta, Google Workspace), since revocation and audit-log procedures differ meaningfully between platforms
- **SaaS-application-specific containment** — if the compromised account had access to multiple connected SaaS tools (Salesforce, Slack, GitHub), each may require separate session revocation and audit review
- **Regulatory notification assessment** — cloud compromises affecting regulated data may trigger the same breach-notification obligations discussed in `Data-Exfiltration-Playbook.md`
- **CASB (Cloud Access Security Broker) integration** — enterprise environments often have dedicated tooling for automated OAuth app risk scoring and anomalous cloud activity detection beyond native IdP logging

This distinction is intentional — this repo documents the technically correct response framework, while the company-specific layer above it depends on organizational context not available in a personal portfolio project.
