# Insider Threat Response Playbook

**Primary MITRE Techniques:** T1005 (Data from Local System), T1530 (Data from Cloud Storage), T1078 (Valid Accounts), T1052 (Exfiltration Over Physical Medium)
**Source Cases:** Case_010 (Alert D — whoami/RDP context), Case_015 (Alert V — USB device, Ambiguous reasoning), Case_017 (Alert AD — self-add to lower-privilege group), drill practice (after-hours mass download, out-of-role data access, badge/VPN mismatch)

---

## 1. Preparation

- **User and Entity Behavior baselines documented** — normal working hours, typical data access volume, and department-appropriate file access patterns per role
- **Least-privilege access reviewed regularly** — technical permission should not be assumed to equal expected/legitimate access (see verdict discipline below)
- **DLP (Data Loss Prevention) tooling** configured to flag USB writes and cloud-sync of sensitive file categories
- **HR/Legal coordination process established in advance** — insider threat investigations require HR and Legal involvement earlier than most other incident types, due to employment-law sensitivity

---

## 2. Detection & Analysis

**Primary indicators — note that insider threat alerts are disproportionately Ambiguous and require more context than other categories:**

| Indicator | Verdict Guidance | Reference |
|---|---|---|
| USB device connected, no authorized-device baseline | Ambiguous by default — normal timing is not positive proof of safety; needs follow-up file-copy evidence | Case_015, Alert V |
| User accesses a permitted file/folder for the first time in months | Ambiguous — technical permission ≠ expected access; check for a legitimate reason (new project, role change) before escalating | Drill practice |
| After-hours mass download (200+ files) with no established pattern for the account | TP — sudden, large-volume access at an unusual hour with no baseline is a strong indicator regardless of stated permission | Drill practice |
| Physical badge entry and a remote VPN session for the same account, same time, different location | TP — physically impossible for one legitimate user; indicates shared or compromised credentials | Drill practice |
| Self-add to a lower-privilege group (e.g. Remote Desktop Users) by an existing user | Ambiguous — genuinely dual-use (legitimate self-service vs. compromised-session persistence); needs corroborating context | Case_017, Alert AD |
| `whoami` or similar recon command run immediately following an RDP/remote logon | Ambiguous, but treat as elevated concern — combine with other signals before escalating to TP | Case_010, Alert D |

**Verdict discipline — this is the single most important principle for this playbook category:** do not default to FP just because an event "looks normal" (business hours, technically permitted access). Equally, do not default to TP just because an action is unusual. **Positive evidence is required in either direction** — a documented baseline, a corroborating HR record (e.g. a confirmed role change), or a follow-up suspicious action (file copy after a folder access, external connection after a mass download) is what actually resolves these cases, not the surface appearance of the single event.

---

## 3. Containment

**Insider threat containment differs from external-attacker containment — actions must be more measured, given the employment and legal context:**

1. **Do not immediately confront or terminate access** without HR/Legal sign-off, unless there is confirmed active data exfiltration in progress (e.g. large file transfer actively occurring) — premature action can compromise an investigation or create legal exposure.
2. If active exfiltration is confirmed and urgent, restrict access to the specific sensitive data source (revoke share access, disable the specific account's remote access) rather than a full account lockout, where possible, to avoid tipping off the individual before evidence is preserved.
3. Preserve logs and evidence discreetly — avoid actions that would alert the individual that they are under investigation before HR/Legal has a plan in place.
4. Loop in HR and Legal at this stage, not after — insider cases carry different obligations (employment law, potential criminal referral) than external-attacker incidents.

---

## 4. Eradication

1. Once the investigation and any required HR/Legal process concludes, revoke access as determined appropriate (may range from no action, to access restriction, to termination and account deactivation, depending on findings).
2. Review and correct any overly broad access permissions identified during the investigation (e.g. the out-of-role folder access case — should that permission have existed at all?).
3. If external transfer occurred (USB, cloud sync), determine what data left the organization and assess exposure/notification obligations.

---

## 5. Recovery

1. Restore any legitimately-needed access for unaffected team members if broader access was temporarily restricted during investigation.
2. Review and tighten the specific permission/DLP gap that allowed the questionable access in the first place.
3. Document the resolution (whether the activity was ultimately determined legitimate or not) for future reference — many insider-threat alerts resolve as FP once context is gathered, and that outcome should be documented as thoroughly as a confirmed TP.

---

## 6. Lessons Learned

- Was the access technically permitted but not actually expected for the role? If so, is a broader access-review needed across similar roles?
- Did the investigation confirm or rule out malicious intent? Either way, was the evidence-gathering process sufficient to reach that conclusion confidently?
- Were HR/Legal engaged at the appropriate stage, not too late (risking evidence loss) or too early (risking premature/unfounded action)?
- Update `Noise-Baselines/known_legitimate_accounts.md` if a new legitimate self-service access pattern (like Case_017's self-add to RDP Users) is confirmed as a recurring, authorized behavior for certain roles.

---

## What a Real Enterprise Version Would Add

- **Named HR/Legal escalation contacts and required approval chain** before any account action is taken, specific to the organization's employment law jurisdiction
- **Formal UEBA (User and Entity Behavior Analytics) tooling integration** — real enterprises use dedicated behavioral-baseline platforms rather than manual pattern review
- **Defined thresholds for "reasonable suspicion"** that trigger different response tiers (monitoring only vs. active investigation vs. immediate access restriction), set by Legal/Compliance in advance
- **Whistleblower/reporting-channel protections** — ensuring the investigation process doesn't inadvertently violate protections for employees who report concerns
- **Criminal referral procedure** — the specific process and threshold for involving law enforcement, which varies significantly by jurisdiction and data type involved

This distinction is intentional — this repo documents the technically correct response framework, while the company-specific layer above it depends on organizational and legal context not available in a personal portfolio project.
