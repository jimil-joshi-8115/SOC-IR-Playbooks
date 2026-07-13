# Credential Compromise Response Playbook

**Primary MITRE Techniques:** T1110 (Brute Force), T1110.003 (Password Spraying), T1558.003 (Kerberoasting), T1003 (OS Credential Dumping), T1078 (Valid Accounts)
**Source Cases:** Case_005 (single-account brute force), Case_006 (password spray), Case_016 (Alert AA — external RDP brute force), Case_017 (Alert AC — Mimikatz download), Case_019/020 (Final Exam — Tor login, Kerberoasting, full credential-theft chain)

---

## 1. Preparation

- **Account lockout policy configured and verified** — Case_005 documented a real gap where 5+ failed attempts triggered no lockout (EventCode 4740 absent); confirm this control actually functions before it's needed in a real incident
- **MFA enforced** on all accounts, especially privileged and remote-access accounts
- **Baseline of normal login patterns documented** — typical hours, typical source locations/IPs, typical logon types per account, so anomalies (Tor exit node, impossible travel, off-hours access) stand out clearly
- **Kerberoasting exposure minimized** — service accounts should use long, complex passwords or managed service accounts (gMSA) resistant to offline cracking after ticket extraction

---

## 2. Detection & Analysis

**Primary indicators:**

| Indicator | Verdict Guidance | Reference |
|---|---|---|
| 5+ failed logon attempts, same account, tight window | TP (T1110) — decisive on volume alone, regardless of whether any attempt succeeded | Case_005 |
| 3+ distinct accounts attempted rapidly (password spray) | TP (T1110.003) — decisive regardless of internal/external origin | Case_006 |
| High-volume failed RDP attempts from an external IP against a high-value account | TP, elevated severity — active outside attack surface exposure | Case_016, Alert AA |
| Successful login from a Tor exit node | TP (T1090/T1078) — no legitimate everyday reason for an account to authenticate through anonymization infrastructure | Case_019 (drill-derived, consistent with Final Exam reasoning) |
| Multiple Kerberos TGS-REQ events for distinct service accounts within a short window | TP (T1558.003) — offline credential-cracking preparation, no everyday justification | Case_019/020, Alert BH |
| Download of a tool with a Mimikatz-style filename/signature, or an EDR-confirmed credential-dumping detection | TP (T1003) — decisive on tool identification alone; if EDR-confirmed, treat as direct evidence, not a theoretical technique | Case_017, Alert AC; Case_019, Alert BG |
| 1-2 failed attempts only | Ambiguous — insufficient volume to confirm either direction | Case_015, Alert S |

**Verdict discipline:** Never treat "the attempt failed" as reassuring — a failed brute-force or Kerberoasting attempt is still TP-worthy malicious behavior (same principle established in Case_005). Conversely, a *successful* follow-on login after a failed-attempt pattern should immediately escalate the incident's severity, since it indicates the attack succeeded.

---

## 3. Containment

**Short-term:**
1. Immediately disable or force a password reset on the compromised/targeted account(s) — do not wait for full scope confirmation if a successful unauthorized login is confirmed.
2. Terminate any active sessions for the affected account across all systems.
3. Block the source IP at the firewall if the attack originated externally (e.g. external RDP brute force, Tor-sourced login).
4. If Kerberoasting is confirmed, assume the requested service account passwords may be crackable offline — treat as compromised even before cracking is confirmed, and begin credential rotation for the targeted service accounts.

**Long-term:**
5. Check for lateral use of the compromised credential across other systems (same account attempting logins elsewhere).
6. If a credential-dumping tool (Mimikatz-pattern) was confirmed, assume **all** credentials cached in memory on that host at the time may be compromised, not just the account initially targeted — this includes any admin credentials that may have been used to log into that host previously.

---

## 4. Eradication

1. Rotate credentials for the directly compromised account **and** any other accounts that may have had cached credentials exposed via dumping (see containment step 6).
2. Remove any credential-dumping tools, downloaded payloads, or persistence mechanisms established using the compromised access.
3. Re-enable/verify account lockout policy is functioning if the incident revealed it was not (as found in Case_005's environment).
4. For Kerberoasting exposure, rotate the affected service account passwords to long, random values, and evaluate migrating to managed service accounts.

---

## 5. Recovery

1. Restore account access with new credentials and confirm MFA is enforced before re-enabling.
2. Monitor the previously-compromised account closely for a period following restoration for any signs of continued unauthorized access attempts.
3. Confirm no persistence mechanisms remain that could allow credential re-theft after rotation.

---

## 6. Lessons Learned

- Did the account lockout policy function as expected? If not (as in Case_005), this is a control gap requiring remediation independent of this specific incident.
- How did the attacker obtain initial access to attempt credential theft (phishing, exposed RDP, prior compromise)? Address the entry vector, not just the credential-theft symptom.
- If Kerberoasting succeeded, were service account passwords compliant with complexity/rotation policy? Update policy if not.
- Were correlated indicators (e.g. Tor login + Kerberoasting + Mimikatz download in the same session, as in the Final Exam scenario) recognized as one incident, or triaged as separate unrelated alerts? Reinforce cross-alert correlation practices where gaps are found.

---

## What a Real Enterprise Version Would Add

- **Privileged Access Management (PAM) integration** — real enterprises often have dedicated PAM solutions that can automatically rotate and vault credentials upon suspected compromise, rather than manual reset procedures
- **Identity provider-specific containment steps** — exact procedures for the organization's actual IdP (Azure AD, Okta, Ping Identity) to force session termination and conditional access blocks
- **Threat intelligence-driven IP/infrastructure blocking** — automated blocklist integration with known malicious infrastructure feeds, rather than manual IP blocking per incident
- **Formal password/credential policy enforcement audit trail** — documentation proving compliance with regulatory password policy requirements where applicable
- **Cross-account/cross-domain correlation tooling** — enterprise SIEM/UEBA platforms can automatically flag the same credential being used across multiple systems in a way manual review cannot easily replicate at scale

This distinction is intentional — this repo documents the technically correct response framework, while the company-specific layer above it depends on organizational context not available in a personal portfolio project.
