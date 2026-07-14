# DDoS (Distributed Denial of Service) Response Playbook

**Primary MITRE Techniques:** T1498 (Network Denial of Service), T1499 (Endpoint Denial of Service)
**Source Cases:** Case_016 (Alert AA — external RDP brute force at scale, related connection-flood pattern), drill practice (port scan sweep — precursor reconnaissance pattern)

**Note on scope:** DDoS response differs fundamentally from the other playbooks in this repo — the goal is **availability restoration**, not evidence preservation or credential remediation. Speed of mitigation matters more than forensic depth in the initial response window.

---

## 1. Preparation

- **Baseline normal traffic volume/patterns documented** per service, so a genuine DDoS spike is distinguishable from a legitimate traffic surge (e.g. marketing campaign, product launch)
- **DDoS mitigation service/capability in place** — CDN-based scrubbing (Cloudflare, AWS Shield, Akamai), or ISP-level mitigation agreement, configured and tested before an actual event
- **Rate limiting and connection-throttling rules pre-configured** at the load balancer/WAF level
- **Communication plan ready** — internal stakeholders and, if customer-facing, a status-page process, so this doesn't need to be improvised mid-incident

---

## 2. Detection & Analysis

**Primary indicators:**

| Indicator | Verdict Guidance |
|---|---|
| Sudden, sustained spike in inbound connection attempts/requests from many distinct source IPs | TP — volumetric DDoS, distinguish from a single-source burst (which may be a different issue, e.g. a misconfigured client retry loop) |
| High-volume connection attempts from a single source across many internal hosts/ports | More consistent with a scan/brute-force precursor (see `Credential-Compromise-Playbook.md`) than true DDoS — verify source diversity before classifying |
| Application-layer symptoms (slow response times, service unavailability) without a corresponding network-layer traffic spike | Consider application-layer DoS (e.g. resource-exhaustion via expensive queries) rather than network-layer flood — different mitigation approach |
| Traffic spike correlating with a known legitimate event (product launch, marketing push) | Ambiguous/FP-leaning — verify against planned business activity before treating as an attack |

**Verdict discipline:** Confirm genuine attack traffic (many distinct sources, abnormal request patterns, no legitimate business correlation) before committing to full DDoS response — misclassifying a legitimate traffic surge as an attack and aggressively rate-limiting can itself cause a self-inflicted outage.

---

## 3. Containment

**This phase is time-critical — the priority is service restoration, executed in parallel with attack confirmation, not sequentially after it:**

1. Engage DDoS mitigation service/CDN scrubbing immediately upon confirmation (or even upon strong suspicion, given the low cost of enabling a pre-configured mitigation versus prolonged downtime).
2. Apply rate limiting and connection throttling at the load balancer/WAF for the affected service.
3. If the attack targets a specific application endpoint (application-layer DoS), consider temporarily disabling or caching that specific endpoint rather than the whole service.
4. Communicate status internally and, if customer-facing impact is occurring, activate the prepared external communication plan.

---

## 4. Eradication

1. Once mitigation is active and traffic is stabilized, identify the attack pattern/signature (source ASN ranges, request patterns, protocol used) for longer-term filtering rules.
2. If the attack leveraged a specific application vulnerability (e.g. an expensive, unauthenticated API endpoint), patch or add authentication/rate-limiting to that specific endpoint.
3. Review whether any legitimate infrastructure was compromised and used as part of a reflection/amplification attack against a third party (in which case, remediate that compromise as well).

---

## 5. Recovery

1. Gradually reduce mitigation intensity/rate limits as attack traffic subsides, monitoring closely for resumption.
2. Confirm full service restoration and validate application functionality, not just that traffic is flowing.
3. Review and adjust baseline traffic thresholds if the incident revealed the existing baseline was too permissive to trigger early detection.

---

## 6. Lessons Learned

- How long was the detection-to-mitigation window? What could shorten it (earlier alerting thresholds, pre-authorized mitigation activation)?
- Was the mitigation service/capability sufficient for the attack volume experienced, or does capacity/contract need reassessment?
- If application-layer DoS was involved, does the affected endpoint need architectural hardening (caching, stricter rate limits, authentication)?
- Was any customer-facing communication timely and accurate? Update the communication template/process if gaps were found.

---

## What a Real Enterprise Version Would Add

- **Named SLA with the DDoS mitigation/CDN provider** — specific response-time commitments and escalation contacts
- **Automated traffic-anomaly-triggered mitigation activation** — many enterprises pre-authorize automatic scrubbing activation above a defined threshold, removing human decision latency from the critical path
- **Legal/PR involvement** for customer-facing outages exceeding a defined duration or severity threshold
- **Cross-functional "war room" activation procedure** — specific to the organization's incident severity classification system
- **Post-incident capacity planning integration** with infrastructure/finance teams to right-size mitigation capacity going forward

This distinction is intentional — this repo documents the technically correct response framework, while the company-specific layer above it depends on organizational context not available in a personal portfolio project.
