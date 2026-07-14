# Physical Security Incident Response Playbook

**Primary MITRE Techniques:** T1078 (Valid Accounts — via physical credential misuse), T1200 (Hardware Additions), T1052.001 (Exfiltration via Removable Media)
**Source Cases:** Drill practice (badge/VPN location mismatch), Case_015 (Alert V — USB device connection reasoning, applied here to physical device handling)

**Note on scope:** This playbook bridges physical and logical security — most SOC L1 alerting is purely digital, but badge systems, lost/stolen devices, and physical tampering all generate logical indicators (logs, alerts) that a SOC analyst needs to recognize and correlate with physical facts.

---

## 1. Preparation

- **Badge/access control logs integrated with SIEM** where feasible, so physical entry events can be correlated against logical access (VPN, workstation logon) for the same account
- **Lost/stolen device reporting process clearly communicated** to all staff, with a fast, low-friction reporting channel
- **Device encryption enforced** (BitLocker or equivalent) on all portable endpoints, so a lost/stolen device does not automatically equal a data breach
- **USB port control policy defined** — whether removable media is restricted, monitored, or freely permitted, and for which roles

---

## 2. Detection & Analysis

**Primary indicators:**

| Indicator | Verdict Guidance | Reference |
|---|---|---|
| Physical badge entry logged for an account, while a VPN/remote session for the same account is simultaneously active from a different city | TP — physically impossible for one legitimate user; indicates shared or compromised credentials | Drill practice |
| A device reported lost or stolen | Requires immediate action regardless of "verdict" framing — treat as a confirmed incident requiring response, not something to triage as Ambiguous | — |
| An unrecognized USB/peripheral device connected, no authorized-device baseline | Ambiguous by default — same reasoning as Case_015's Alert V; requires follow-up (file-copy activity, device identification) to resolve | Case_015, Alert V |
| Evidence of physical tampering with a workstation or network hardware (e.g. an unfamiliar USB device physically implanted, a keylogger dongle) | TP — hardware addition with no legitimate explanation, decisive | — |
| Badge access attempted outside an employee's normal working hours/location pattern, with no VPN conflict | Ambiguous — could be legitimate (early/late work), needs correlation with other factors before escalating | — |

**Verdict discipline:** Physical indicators should always be correlated with logical (digital) activity for the same account/timeframe where possible — a badge/VPN mismatch is far more decisive as a *combination* than either fact would be alone. This mirrors the cross-alert correlation principle applied throughout `SOC-Triage-Practice`.

---

## 3. Containment

**Short-term:**
1. **Lost/stolen device:** Remotely lock and, if policy permits and data sensitivity warrants, remote-wipe the device immediately upon report. Disable the associated account's active sessions if credentials were cached/accessible on the device.
2. **Badge/VPN mismatch:** Disable the account's remote access immediately while investigating; do not wait for full confirmation given the strong indicator of credential sharing or compromise.
3. **Physical tampering/hardware addition:** Physically secure the affected area/device, isolate the affected system from the network, and do not immediately remove suspicious hardware without photographing/documenting it first for evidence purposes.

**Long-term:**
4. Review physical access logs for the affected badge/account over a wider window to identify any other anomalous entry patterns.
5. If credential sharing is confirmed (not compromise), this becomes a policy/HR matter — coordinate with `Insider-Threat-Playbook.md` guidance on HR/Legal involvement timing.

---

## 4. Eradication

1. For lost/stolen devices: confirm remote wipe completed successfully; reissue a new device and credentials.
2. For badge/VPN mismatch confirmed as compromise: rotate the affected account's credentials and re-issue physical badge credentials if badge cloning is suspected.
3. For hardware tampering: remove the malicious hardware (after evidence documentation) and inspect the system for any software-based follow-on compromise the hardware may have enabled (e.g. a keylogger capturing credentials later used for logical access).

---

## 5. Recovery

1. Restore normal access for the affected account/employee with new credentials/device as applicable.
2. Reinforce physical security controls if a gap was identified (e.g. tailgating allowing unauthorized facility access, badge cloning vulnerability).
3. Monitor the affected account for a period following restoration.

---

## 6. Lessons Learned

- Was the physical/logical correlation (e.g. badge + VPN) available and reviewed in time, or was this manually pieced together after the fact? If the latter, consider integrating badge logs into the SIEM if not already done.
- For lost/stolen devices, was encryption confirmed active, limiting actual data exposure risk? If not, this is a policy enforcement gap.
- If hardware tampering occurred, how was physical access to the affected area obtained? Address the physical access control gap, not just the resulting digital indicator.
- Update `Noise-Baselines/` if a legitimate explanation for an unusual access pattern is confirmed (e.g. a documented exception for an employee's unusual working pattern).

---

## What a Real Enterprise Version Would Add

- **Physical security team integration** — most enterprises have a dedicated physical security/facilities team that owns badge systems and physical incident response, requiring a defined handoff/collaboration process with the SOC
- **Remote device management platform-specific procedures** — exact steps for the organization's actual MDM (Mobile Device Management) platform to execute remote lock/wipe
- **Legal chain-of-custody procedures** for any physically tampered hardware that may be used as evidence in an investigation or prosecution
- **Badge system vendor-specific forensic capabilities** — some badge systems support more granular logging/anti-cloning detection than others, affecting what's actually knowable during an investigation

This distinction is intentional — this repo documents the technically correct response framework, while the company-specific layer above it depends on organizational context not available in a personal portfolio project.
