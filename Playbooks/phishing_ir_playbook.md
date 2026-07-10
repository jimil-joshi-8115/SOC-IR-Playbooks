# Phishing Incident Response Playbook

**Primary MITRE Techniques:** T1566 (Phishing), T1204.002 (User Execution: Malicious File), T1078 (Valid Accounts), T1566.002 (Spearphishing Link / BEC)
**Source Cases:** Case_014 (Alert Q — Outlook→PowerShell), Case_017 (Alert AC — Mimikatz download following execution chain), drill practice (BEC payment fraud, lookalike domains)

---

## 1. Preparation

- **Email security controls in place:** SPF, DKIM, DMARC enforcement on the organization's domain
- **User awareness training** current and measurable (phishing simulation click-rate tracked)
- **Baseline documented** for which applications legitimately spawn script interpreters (reference `Noise-Baselines/known_legitimate_processes.md` — Office/email apps should never appear here)
- **Reporting mechanism** clearly available to all staff (e.g. "Report Phishing" button) and monitored in real time

---

## 2. Detection & Analysis

**Primary indicators:**

| Indicator | Reference |
|---|---|
| Office/email application (`OUTLOOK.EXE`, `WINWORD.EXE`, `EXCEL.EXE`) spawning `powershell.exe` or `cmd.exe` | Case_014, Alert Q — the abnormal parent process itself is the primary red flag, weighted above payload content |
| Encoded/obfuscated PowerShell command executed shortly after an email attachment was opened | Combines with the above for a decisive TP call |
| Lookalike/spoofed sender domain (character substitution, e.g. `cornpany.com` vs `company.com`) | Deliberate phishing infrastructure, no legitimate explanation |
| Urgent payment-change request purportedly from a vendor (Business Email Compromise) | Verdict depends on outcome: did the change actually occur, or was it verified before action? |
| User self-reports a suspicious email with no execution | Correctly triaged as FP — the awareness process working as intended, not an incident |

**Verdict discipline:**
- **Self-reported, no execution, no links clicked** = FP, close quickly. Do not treat every reported email as an incident — this rewards good behavior and avoids alert fatigue for genuinely urgent cases.
- **Attachment opened + abnormal parent-child execution chain observed** = TP, escalate immediately regardless of what the payload's specific content turns out to be — the execution chain itself is the evidence (same reasoning as Case_014).
- **BEC-style request** — verdict hinges on whether financial/data impact actually occurred, not just whether the email was sent. A verified-before-acting outcome is FP for the *organization's* response, even though the email itself was a genuine phishing attempt.

---

## 3. Containment

**Short-term:**
1. Isolate the affected host if malicious execution occurred (see `Ransomware-Response-Playbook.md` containment steps if follow-on activity like Mimikatz or C2 is confirmed, as in Case_017's chain).
2. Block the sender domain and any malicious URLs/attachments at the email gateway.
3. Search and quarantine the same email from all other inboxes it was delivered to (not just the reporting user's).
4. If credentials were entered on a phishing page, force an immediate password reset and session revocation for the affected account.

**Long-term:**
5. Identify how many users received the email and whether any others clicked/executed it.
6. If a BEC/payment-fraud email led to an actual financial transaction, engage finance and legal immediately to attempt fund recall and determine reporting obligations.

---

## 4. Eradication

1. Remove any dropped malware, scheduled tasks, or persistence mechanisms resulting from the malicious execution (cross-reference `Triage-Playbooks/scheduled_task_triage.md` and `process_creation_triage.md`).
2. Confirm no credential-dumping tools (e.g. Mimikatz-pattern, as seen in Case_017) were successfully staged or executed.
3. Close the specific delivery vector if it exploited a technical gap (e.g. an email gateway rule that allowed a lookalike domain through).

---

## 5. Recovery

1. Restore affected accounts/systems from confirmed-clean state.
2. Re-enable normal email flow once the malicious sender/domain is fully blocked organization-wide.
3. Communicate a brief awareness notice to staff who received the email, without over-alarming, reinforcing the correct reporting behavior.

---

## 6. Lessons Learned

- Did the abnormal parent-child execution pattern (Office/email app spawning a script interpreter) get flagged automatically, or was it caught manually? If manual, consider a detection rule addition.
- How many users received the phishing email, and what fraction reported it correctly versus clicked/executed?
- If a BEC attempt succeeded, what verification step was missing that should be added to the finance approval process (e.g. mandatory callback verification for any payment-detail change request)?
- Update `Noise-Baselines/known_legitimate_processes.md`'s "abnormal parent process" reference table if a new execution chain pattern was observed.

---

## What a Real Enterprise Version Would Add

- **Named escalation matrix** and SLAs specific to the organization's IR team structure
- **Legal/Compliance triggers** — particularly for BEC incidents involving actual financial loss, which may require law enforcement (cybercrime cell) reporting and cyber insurance notification
- **Email gateway/tool-specific commands** — exact steps for the organization's actual email security platform (e.g. Proofpoint, Mimecast, Microsoft Defender for Office 365) to search-and-quarantine across all mailboxes
- **Finance department playbook integration** — a formal fund-recall and bank-notification procedure specific to the organization's banking relationships, for BEC cases with completed fraudulent transfers
- **Pre-approved staff communication templates** for phishing-campaign awareness notices

This distinction is intentional — this repo documents the technically correct response framework, while the company-specific layer above it depends on organizational context not available in a personal portfolio project.
