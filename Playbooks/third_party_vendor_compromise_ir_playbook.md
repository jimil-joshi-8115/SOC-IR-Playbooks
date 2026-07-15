# Third-Party / Vendor Compromise Response Playbook

**Primary MITRE Techniques:** T1195 (Supply Chain Compromise), T1199 (Trusted Relationship), T1078.002 (Domain/Cloud Accounts via trusted third party)
**Source Cases:** Case_010 (Alert F — legitimate-platform script download reasoning, applied here to vendor-supplied tooling), drill practice (new OAuth app broad-permission grant — a common vendor-integration compromise vector)

**Note on scope:** This is the least commonly covered incident type in fresher SOC portfolios, but increasingly one of the most consequential in real enterprises — a compromise doesn't need to originate inside the organization if a trusted vendor, software supplier, or integrated third-party service is compromised first.

---

## 1. Preparation

- **Vendor/third-party inventory maintained** — a documented list of what third-party software, integrations, and service providers have access to internal systems or data, and what level of access each has
- **Least-privilege access for third-party integrations** — vendor tools/OAuth apps should only be granted the minimum permissions needed, reviewed periodically (same principle as `Cloud-Account-Compromise-Playbook.md`)
- **Software supply chain awareness** — where feasible, verify software update sources and consider code-signing verification for critical third-party software
- **Vendor security contact information documented in advance** — knowing who to call at a vendor during an active incident should not be discovered for the first time during the incident

---

## 2. Detection & Analysis

**Primary indicators:**

| Indicator | Verdict Guidance | Reference |
|---|---|---|
| A trusted, previously-legitimate software update or vendor tool begins exhibiting anomalous behavior (unexpected network connections, unusual process spawning) shortly after an update | TP-leaning — supply-chain compromises often manifest as a previously-trusted binary suddenly behaving abnormally; do not dismiss anomalous behavior just because the source was previously trusted |
| A third-party OAuth-integrated application's access pattern changes suddenly (accessing data types or volumes inconsistent with its stated function) | TP-leaning — could indicate the vendor's own systems were compromised and the integration is being abused, distinct from a first-party account compromise | Related to drill practice OAuth scenario |
| Vendor publicly discloses a breach or security incident affecting a product/service your organization uses | Requires proactive investigation, not just passive alert monitoring — check your own environment for indicators even without an internal alert firing first |
| A script/tool downloaded from a legitimate, known platform (e.g. a vendor's GitHub) but from an undocumented, unverified specific path | Ambiguous — legitimate platform, but no baseline confirms this specific resource is the authentic, unmodified vendor content | Case_010, Alert F (same reasoning pattern applied to vendor context) |

**Verdict discipline:** The core challenge in this category is that indicators often look like "normal" activity from an already-trusted source — this is precisely what makes supply-chain and trusted-relationship compromises effective. Weight *behavioral change* from a trusted source more heavily than you would weight the same behavior from an unknown source, since trust itself is the exploited factor.

---

## 3. Containment

**Short-term:**
1. If a specific vendor integration or tool is confirmed compromised, immediately revoke or suspend its access/credentials — do not wait for the vendor's own confirmation if internal evidence is strong.
2. Isolate any systems that interacted with the suspected-compromised vendor software/update.
3. Notify the vendor's security team using the pre-documented contact process, but do not treat vendor response time as a blocker to your own containment actions.

**Long-term:**
4. Assess what data or access the compromised third-party integration had, to scope potential impact independent of what the vendor discloses.
5. Review all other integrations/access from the same vendor if multiple products/services are used from them.

---

## 4. Eradication

1. Remove or replace the compromised vendor software/integration once a clean version or confirmed remediation is available from the vendor.
2. Rotate any credentials, API keys, or tokens that were shared with or accessible to the compromised third party.
3. Re-verify that the reinstated integration only has the minimum necessary access, reassessing scope rather than restoring identical permissions by default.

---

## 5. Recovery

1. Restore the vendor integration/service only after independent verification of remediation, not solely on the vendor's assurance.
2. Monitor the restored integration closely for a defined period for any recurrence of anomalous behavior.
3. Update the vendor risk assessment based on this incident.

---

## 6. Lessons Learned

- Did the organization have visibility into this vendor's access/behavior sufficient to detect the compromise, or was it discovered externally (e.g. public disclosure)? If the latter, this is a monitoring gap.
- Was the vendor's access scoped appropriately (least privilege), or did the compromise have a larger blast radius than necessary due to over-broad permissions?
- How quickly did the vendor communicate and remediate on their end? Does this affect the ongoing vendor relationship/risk rating?
- Update the vendor inventory and risk documentation with findings from this incident.

---

## What a Real Enterprise Version Would Add

- **Contractual security requirements and SLAs with vendors** — specific breach-notification timelines and security-standard requirements written into vendor contracts, which determine legal recourse and expected vendor cooperation
- **Third-party risk management (TPRM) program integration** — formal vendor risk scoring and periodic security assessment process, typically owned by a dedicated procurement/vendor-risk team
- **Software Bill of Materials (SBOM) tracking** — enterprise supply-chain security programs increasingly require SBOMs for critical software to enable rapid impact assessment when a component vulnerability is disclosed
- **Legal review of vendor liability and indemnification clauses** relevant to the specific incident
- **Industry information-sharing participation** (ISACs/ISAOs) to receive and contribute early warning on vendor/supply-chain compromises affecting the sector

This distinction is intentional — this repo documents the technically correct response framework, while the company-specific layer above it depends on organizational and contractual context not available in a personal portfolio project.
