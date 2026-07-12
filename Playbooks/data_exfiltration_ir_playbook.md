# Data Exfiltration Response Playbook

**Primary MITRE Techniques:** T1048 (Exfiltration Over Alternative Protocol), T1052 (Exfiltration Over Physical Medium), T1567 (Exfiltration Over Web Service), T1071.004 (DNS)
**Source Cases:** Case_009 (Alert C — DNS beaconing), Case_015 (Alert U — DNS DGA pattern), Case_019 (Alert BD — high-volume DNS TXT tunneling), drill practice (mass USB copy, personal cloud sync, archive staging)

---

## 1. Preparation

- **DLP tooling configured** to flag mass file copies to removable media, personal cloud storage sync, and large archive creation in unusual locations (e.g. Temp directories)
- **DNS query volume baselines established** — so abnormal query rates (e.g. Case_019's 340 TXT queries in 5 minutes) stand out against normal traffic
- **Egress filtering reviewed** — outbound firewall rules should restrict unnecessary protocols/destinations, reducing available exfiltration channels
- **Sensitive data classification in place** — HR/finance/IP-critical files tagged so DLP and access alerts can prioritize correctly

---

## 2. Detection & Analysis

**Primary indicators:**

| Indicator | Verdict Guidance | Reference |
|---|---|---|
| High-volume DNS TXT queries (100+ in a short window) to a single domain | TP — volume/rate alone is decisive, independent of how legitimate the domain name sounds; classic DNS tunneling signature | Case_019, Alert BD |
| Repeated queries to varying subdomains of the same base domain | Ambiguous unless a resolved-IP reputation check or follow-up connection confirms outcome — stronger pattern match than simple repetition, but still needs confirmation | Case_015, Alert U |
| Simple repeated identical DNS query (2-3x) to one domain | Ambiguous — could be manual/normal retry behavior; insufficient volume alone | Case_009, Alert C |
| Large archive created in a Temp directory, followed by an outbound connection shortly after | TP — the *sequence* (collect then transmit) is decisive, not either action alone | Drill practice |
| Multiple sensitive files copied to USB by an account with no role-based need for that data | TP — decisive when data classification and role are both incompatible with access | Drill practice |
| Confidential files detected syncing to an unmanaged personal cloud storage account | TP — no legitimate business reason for personal cloud sync of classified data | Drill practice |

**Verdict discipline:** DNS-based exfiltration is easy to under-triage because each individual query looks harmless. Always evaluate **volume and rate**, not single events — this is the same principle applied throughout the DNS-related cases in `SOC-Triage-Practice`. When rate is ambiguous, look for a second corroborating signal (resolved IP reputation, a follow-up connection, or a preceding data-staging action like archive creation) before closing as Ambiguous versus escalating to TP.

---

## 3. Containment

**Short-term:**
1. Block the destination (domain, IP, or cloud service) at the DNS/firewall/proxy level immediately upon confirmation.
2. If a specific host is actively transmitting data, isolate it from the network — do not wait for full scope assessment.
3. If exfiltration is via removable media, physically secure or image the device if still accessible; revoke the account's USB write permissions going forward if policy allows.
4. If via personal cloud storage, work with the cloud provider (where possible, via legal request) or the organization's CASB (Cloud Access Security Broker) to restrict or reverse the sync where feasible.

**Long-term:**
5. Determine what specific data left the organization — this drives downstream notification obligations (see enterprise-gap section) and severity assessment.
6. Check for the same exfiltration pattern (same destination, same technique) across other hosts in the environment.

---

## 4. Eradication

1. Remove any tooling responsible for the exfiltration channel (e.g. a script generating the DNS tunnel traffic, an unauthorized cloud-sync client installed on the endpoint).
2. Close the technical gap that allowed the channel to exist (e.g. tighten egress filtering to block DNS-over-HTTPS to unmanaged resolvers, restrict USB write access for the account/role going forward).
3. If the exfiltration was credential-driven (an attacker exfiltrating via a compromised account rather than an insider), see `Credential-Compromise-Playbook.md` for the account-remediation steps.

---

## 5. Recovery

1. Restore normal DNS/network operation once the malicious channel is confirmed closed.
2. Re-enable any temporarily restricted USB/cloud-sync permissions for unaffected users once the specific risk is addressed.
3. Monitor the previously-used exfiltration channel and technique for recurrence over the following weeks.

---

## 6. Lessons Learned

- What data left the organization, and what is the actual business/regulatory impact of that specific data being exposed?
- Was the exfiltration technique (DNS tunneling, USB, cloud sync) one that existing DLP/egress controls should have caught earlier? If so, what specific rule/threshold needs adjustment?
- If DNS volume was the detecting signal, was the baseline sensitive enough, or did it take an unusually high volume (like Case_019's 340-query spike) before triggering? Consider lowering alert thresholds if so.
- Update `Noise-Baselines/` with any newly-confirmed legitimate high-volume DNS pattern discovered during investigation, to reduce future false positives.

---

## What a Real Enterprise Version Would Add

- **Data breach notification legal obligations** — specific requirements under applicable law (e.g. India's DPDP Act, GDPR, sector-specific regulations for financial/healthcare data) triggered by confirmed exfiltration of regulated data categories
- **CASB and DLP tool-specific procedures** — exact steps for the organization's actual cloud security and data-loss-prevention platforms
- **Threat intelligence integration** — checking exfiltration destinations against the organization's paid threat intel feeds for known-APT infrastructure attribution
- **Customer/partner notification templates** — where exfiltrated data includes third-party information requiring their notification
- **Forensic data-volume quantification tooling** — enterprise DLP platforms can often precisely quantify what data left (file names, sizes, timestamps) rather than requiring manual reconstruction

This distinction is intentional — this repo documents the technically correct response framework, while the company-specific layer above it depends on organizational and legal context not available in a personal portfolio project.
