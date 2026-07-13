# 🛡️ SOC-IR-Playbooks

Formal Incident Response runbooks for SOC L1/L2 use, mapped to **NIST SP 800-61** phases and **MITRE ATT&CK**. Built directly from real triage patterns confirmed across [SOC-Triage-Practice](https://github.com/jimil-joshi-8115/SOC-Triage-Practice) — a 20-case, 61-alert practical triage repo — rather than generic templates.

Where `SOC-Triage-Practice/Triage-Playbooks/` answers *"how do I triage this one alert type?"*, this repo answers *"once I've confirmed an incident, what's the full response lifecycle?"* — the two are companion references covering tactical and strategic response.

---

## 📂 Repo Structure

```
SOC-IR-Playbooks/
├── README.md
└── Playbooks/
    ├── ransomware_ir_playbook.md
    ├── phishing_ir_playbook.md
    ├── insider_threat_ir_playbook.md
    ├── data_exfiltration_ir_playbook.md
    ├── credential_compromise_ir_playbook.md
    ├── ddos_ir_playbook.md
    ├── cloud_account_compromise_ir_playbook.md
    ├── malware_endpoint_ir_playbook.md
    ├── third_party_vendor_compromise_ir_playbook.md
    └── physical_security_ir_playbook.md
```

---

## 🧭 Playbook Format

Every playbook follows the same six-phase structure from **NIST SP 800-61 Rev. 2** (Computer Security Incident Handling Guide):

1. **Preparation** — what should already be in place before this incident type occurs
2. **Detection & Analysis** — indicators to watch for, mapped to MITRE ATT&CK techniques
3. **Containment** — short-term and long-term containment actions
4. **Eradication** — removing the root cause
5. **Recovery** — safely restoring normal operations
6. **Lessons Learned** — post-incident review questions

Each playbook also links back to the specific `SOC-Triage-Practice` case(s) that informed its detection criteria, so every recommendation is traceable to real, previously-triaged evidence rather than generic best practice alone.

---

## 📋 Playbook Index

| Playbook | Primary MITRE Tactics Covered | Source Cases |
|---|---|---|
| [Ransomware](Playbooks/ransomware_ir_playbook.md) | Impact (T1486), Inhibit System Recovery (T1490) | Case_010 (Alert E), Case_016 (Alert Z context) |
| [Phishing](Playbooks/phishing_ir_playbook.md) | Initial Access (T1566), User Execution (T1204) | Case_014 (Alert Q), Case_019/020 (Final Exam chain) |
| [Insider Threat](Playbooks/insider_threat_ir_playbook.md) | Collection (T1005/T1530), Exfiltration (T1052/T1567) | Case_017 (Alert AD), Case_015 (Alert V) |
| [Data Exfiltration](Playbooks/data_exfiltration_ir_playbook.md) | Exfiltration (T1041/T1048), C2 (T1071) | Case_017 (Alert AE), Case_015 (Alert U) |
| [Credential Compromise](Playbooks/credential_compromise_ir_playbook.md) | Credential Access (T1003/T1110/T1558) | Case_005, Case_006, Case_013 (Alert N), Case_019/020 |
| [DDoS](Playbooks/ddos_ir_playbook.md) | Impact — Availability (T1498/T1499) | Case_016 (Alert AA context), drill practice |
| [Cloud Account Compromise](Playbooks/cloud_account_compromise_ir_playbook.md) | Cloud Accounts (T1078.004), Additional Roles (T1098.003) | Drill practice (cloud OAuth/admin-role scenarios) |
| [Malware / Endpoint Infection](Playbooks/malware_endpoint_ir_playbook.md) | LOLBins (T1218), Process Injection (T1055) | Case_008, Case_011 (Alert I), Case_016 (Alert Y) |
| [Third-Party / Vendor Compromise](Playbooks/third_party_vendor_compromise_ir_playbook.md) | Supply Chain (T1195), Trusted Relationship (T1199) | Case_010 (Alert F reasoning), drill practice |
| [Physical Security Incident](Playbooks/physical_security_ir_playbook.md) | Valid Accounts via physical misuse (T1078), Hardware Additions (T1200) | Case_015 (Alert V), drill practice |

---

## 👤 Analyst

**Jimil Joshi** — SOC L1 Analyst (Fresher)
TryHackMe SOC L1 · Deloitte Cyber Job Simulation · TATA Cybersecurity Analyst Simulation (IAM) · Ministry of Home Affairs "Cyber Smart"
[LinkedIn](https://linkedin.com/in/jimil-joshi-soc-analyst) · [GitHub](https://github.com/jimil-joshi-8115)

Companion repos: [SOC-Triage-Practice](https://github.com/jimil-joshi-8115/SOC-Triage-Practice) · [SOC-Lab-Splunk](https://github.com/jimil-joshi-8115/SOC-Lab-Splunk) · [Rakshak-SOC](https://github.com/jimil-joshi-8115/Rakshak-SOC) · [Rakshak-EMAIL](https://github.com/jimil-joshi-8115/Rakshak-_EMAIL)
