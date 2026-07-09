# Ransomware Incident Response Playbook

**Primary MITRE Techniques:** T1486 (Data Encrypted for Impact), T1490 (Inhibit System Recovery), T1489 (Service Stop)
**Source Cases:** Case_010 (Alert E — vssadmin), Case_016 (Alert Z — Startup-folder malware delivery), Case_019/020 (Final Exam — full multi-stage compromise chain)

---

## 1. Preparation

Before a ransomware incident occurs, the following should already be in place:

- **Backup verification** — confirm offline/immutable backups exist and are tested for restoration, not just present
- **Baseline documentation** — `Noise-Baselines/known_legitimate_services.md`-style reference so `vssadmin`, backup software, and legitimate encryption tools (BitLocker, disk utilities) are distinguishable from malicious use
- **Detection rules in place for:**
  - `vssadmin delete shadows` / `wbadmin delete catalog` / `wmic shadowcopy delete`
  - Mass file rename/extension-change events in a short window
  - Print/file-share services generating unusual write volume
- **Defined isolation procedure** — network segmentation or VLAN isolation capability ready to execute quickly, tested in advance

---

## 2. Detection & Analysis

**Primary indicators (any one is high-confidence; two or more is near-certain):**

| Indicator | Reference |
|---|---|
| `vssadmin delete shadows /all /quiet` or equivalent (wbadmin, wmic) | Case_010, Alert E — confirmed TP, decisive on command content alone |
| Mass, rapid file extension changes across multiple directories (500+ files in under 2 minutes) | Matches T1486 encryption-in-progress signature |
| A downloaded executable placed in the Startup folder or other persistence location shortly before encryption begins | Case_016, Alert Z — confirms the delivery stage that often precedes encryption |
| Security services (Defender, Firewall, Event Logging) disabled shortly beforehand | Case_011 (Alert G), Case_013 (Alert M) — common pre-encryption defense evasion |
| Ransom note file(s) appearing in multiple directories (e.g. `README_RESTORE.txt`) | Direct confirmation |

**Verdict discipline:** `vssadmin delete shadows` is decisive TP on its own — do **not** wait for encryption to actually begin before escalating. Treat shadow-copy deletion as an active-incident trigger, not a "wait and see" indicator (same principle applied in Case_010's verdict.md).

**Triage note from Case_019/020:** in the Final Exam scenario, shadow-copy-style destructive actions were part of a larger chain (download-execute → Mimikatz confirmation → SYSTEM-level C2 callback). Always check for correlated alerts in the same session/timeframe before assuming an isolated event — a ransomware indicator occurring alongside other confirmed TPs should elevate the entire session to "active incident," not be triaged individually.

---

## 3. Containment

**Short-term (within minutes of confirmation):**
1. Isolate the affected host(s) from the network immediately (disable network adapter, VLAN quarantine, or EDR network-isolation feature) — do not wait for full scope assessment first.
2. Do **not** power off the machine — memory forensics may be needed; disconnect network only.
3. Identify and disable the malicious process/service if still running (reference `SOC-Triage-Practice/Noise-Baselines/known_legitimate_services.md` to avoid disabling a legitimate service by mistake under pressure).
4. Block outbound traffic to any confirmed C2 IP at the firewall/proxy level.

**Long-term (within the hour):**
5. Identify all hosts on the same network segment and check for lateral spread (same file-rename pattern, same C2 IP contacted).
6. Disable or reset credentials for any account involved in the initial compromise (see `Credential-Compromise-Playbook.md` if credential theft is also confirmed, as in Case_019/020).
7. Preserve forensic evidence (memory dump, disk image) before any remediation that could overwrite evidence.

---

## 4. Eradication

1. Remove the ransomware binary and any persistence mechanisms (Startup folder entries, scheduled tasks, registry Run keys — cross-reference `Triage-Playbooks/scheduled_task_triage.md` and `process_creation_triage.md` for how these were identified in Case_007/015).
2. Re-enable any security services that were disabled (Defender, Firewall, Event Logging).
3. Rotate credentials for all accounts active on the affected host during the incident window.
4. Patch or close the initial access vector (phishing link, vulnerable service, exposed RDP) once identified.

---

## 5. Recovery

1. Restore affected files from verified, clean (pre-incident) backups — never restore from backups without first confirming they were not also encrypted or tampered with.
2. Rebuild the affected host from a known-clean image where full confidence in eradication cannot be established, rather than trusting an in-place cleanup.
3. Gradually reconnect to the network in stages, monitoring closely for any re-encryption or beaconing activity before fully restoring access.
4. Confirm shadow copy service and backup schedules are re-enabled and functioning.

---

## 6. Lessons Learned

- What was the confirmed initial access vector? (Phishing, exposed service, credential compromise?)
- How long between initial compromise and shadow-copy deletion? Could earlier alerts (e.g. Defender-disable, AV-enumeration task) have triggered isolation sooner?
- Were backups sufficient and unaffected? If not, what backup-hardening changes are needed (immutability, offline copies, more frequent snapshots)?
- Did any detection gaps (e.g. missing 4698/4720-style audit logs, as recurringly documented across `SOC-Triage-Practice`) delay confirmation? Escalate as a logging-configuration finding separate from this specific incident.
- Update `Noise-Baselines/` and detection rules with any new indicators observed in this incident for faster triage next time.

---

## What a Real Enterprise Version Would Add

This playbook follows the same NIST SP 800-61 framework and MITRE ATT&CK mapping used in real SOC operations, but a company's production playbook would layer the following company-specific elements on top of this same skeleton:

- **Named escalation matrix** — specific SLAs (e.g. "L1 escalates to L2 within 15 min, IR Manager notified within 30 min, CISO within 1 hour") tied to the organization's actual team structure
- **Legal/Compliance/PR triggers** — ransomware often requires breach notification under applicable law (e.g. India's DPDP Act, GDPR for EU data), cyber insurance claim initiation, and potential law enforcement contact — these are legal decisions requiring company counsel, not something a technical playbook can prescribe generically
- **Tool-specific commands** — a real playbook names the exact EDR/SIEM console actions (e.g. "isolate via CrowdStrike Falcon console → Network Containment," or "trigger Splunk ES notable event workflow") tied to the organization's actual security stack
- **Communication templates** — pre-approved internal stakeholder notification emails and, where required, customer/regulator notification templates
- **RTO/RPO targets** — Recovery Time/Point Objectives set by business continuity planning (e.g. "critical systems restored within 4 hours"), specific to each organization's risk tolerance
- **Ransom negotiation policy** — most enterprises have an explicit stance (commonly "we do not pay ransoms" or a defined executive approval chain) set in advance, not decided during the incident

This distinction is intentional — this repo documents the technically correct response *framework*, while the company-specific layer above it depends on organizational context not available in a personal portfolio project.

