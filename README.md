# AWS Cloud Security Baseline Assessment

A security assessment of a fresh AWS account using **Prowler**, with prioritized
remediation and a measured before/after. The goal wasn't to click "enable" on
every service — it was to **assess the account like an analyst**: scan it,
interpret the findings by severity and compliance framework, remediate the
highest-impact gaps, and document what remained and why.

**Headline result:** one well-chosen remediation (enabling encrypted,
multi-region CloudTrail) cleared **all 17 High-severity findings** and raised the
**CIS AWS Benchmark pass rate from 38.9% to 67.3%**, improving every one of 40+
compliance frameworks Prowler evaluates.

---

## Scope & methodology

| Item | Detail |
| --- | --- |
| Tool | [Prowler](https://docs.prowler.com/) v5.45 (open-source CSPM) |
| Scope | Single AWS account, all regions, run from AWS CloudShell |
| Checks | 655 security checks across 40+ compliance frameworks (CIS, NIST, SOC 2, HIPAA, MITRE ATT&CK, PCI, ISO 27001, …) |
| Approach | Baseline scan → prioritize by severity/framework → remediate → re-scan → document |

---

## Results: before vs. after

| Metric | Baseline | After remediation |
| --- | --- | --- |
| **Checks passed** | 28 | **84** |
| **Checks failed** | 74 | 65 |
| **CIS 2.0 AWS** | 38.9% | **67.3%** |
| CIS 3.0 AWS | 37.1% | 68.8% |
| AWS Foundational Security Best Practices | 27.8% | 58.2% |
| NIST CSF 2.0 | 46.7% | 72.2% |
| SOC 2 | 36.4% | 68.6% |
| HIPAA | 27.8% | 63.6% |
| ISO 27001:2013 | 9.7% | 65.7% |
| MITRE ATT&CK | 55.2% | 56.3% |
| **CloudTrail High-severity fails** | 17 | **0** |
| CloudTrail total fails | 35 | 8 |

Full reports in [`reports/`](reports/) (baseline and after), and screenshots in
[`screenshots/`](screenshots/).

---

## Baseline findings (prioritized)

The baseline scan surfaced **74 failed checks**. Ranked by risk:

| Severity | Service | Finding theme |
| --- | --- | --- |
| **High (17)** | CloudTrail | No account-wide audit logging — no record of API activity, no log encryption, no log-file validation |
| **Critical/High (3)** | IAM | Account/root-level controls (see "Remaining findings") |
| Medium | Config, VPC, Bedrock | Config recorder off, no VPC flow logs, Bedrock logging off |
| Low (19) | SageMaker | Default service-level checks with no actual resources deployed |

**The analysis:** the single biggest risk was the complete absence of audit
logging. Without CloudTrail, nothing in the account is attributable or
investigable — which is also why it drags down nearly every compliance
framework at once. That made it the clear first remediation.

---

## Remediation performed

### Enabled CloudTrail (multi-region, encrypted, validated)

Created an account-wide CloudTrail trail with:
- **Multi-region** coverage (captures activity in every region)
- **SSE-KMS encryption** of log files at rest
- **Log-file validation** (tamper detection)
- Management events (read + write) delivered to a dedicated S3 bucket

**Impact:** cleared **all 17 High-severity findings**, reduced CloudTrail fails
from 35 to 8, and lifted every compliance framework — CIS jumped ~28 points.

### A note on second-order effects

Enabling CloudTrail created an S3 log bucket, which Prowler then evaluated,
adding **19 new S3 findings** (the bucket lacked server-access logging, a
lifecycle policy, etc.). This is a deliberate call-out: **remediation can
introduce new resources with their own security posture**, so an assessment is
iterative, not one-and-done. Hardening that bucket is listed in Recommendations.

---

## Remaining findings & why

Not every finding was remediable in this environment, and distinguishing
"won't fix" from "can't fix" is part of the job:

| Finding group | Status | Reason |
| --- | --- | --- |
| IAM Critical/High (root, password policy) | **AWS-managed** | This account uses federated access (AWS Builder ID + an assumed role), so root and account-password controls are managed by AWS, not the account operator |
| GuardDuty, AWS Config | **Gated** | Require the AWS paid plan; unavailable on the free-tier sandbox used here |
| SageMaker (19, Low) | **Not applicable** | Default service checks against resources that don't exist — candidates to mute in Prowler to cut noise |
| S3 log bucket (19) | **Accepted / backlog** | Introduced by the CloudTrail remediation; medium/low severity, hardening listed below |

---

## Recommendations (prioritized)

1. **Harden the CloudTrail S3 bucket** — enable server-access logging, versioning, a lifecycle policy, and block public access.
2. **In a production-capable account**, enable **GuardDuty** (threat detection), **AWS Config** (config drift), and **IAM Access Analyzer** (external-access detection).
3. **Address the IAM Critical/High** in an account type that grants real root access (a standalone account rather than the federated free-tier sandbox).
4. **Tune Prowler** — mute the not-applicable SageMaker defaults so the signal-to-noise ratio of future scans stays high.

---

## How to reproduce

```bash
# From AWS CloudShell
pip install prowler
prowler aws                    # baseline
# ... remediate (enable CloudTrail, etc.) ...
prowler aws                    # re-scan
```

Prowler writes CSV, JSON (OCSF), and HTML reports to `~/output/`.

---

## Skills demonstrated

Cloud security posture management (Prowler) · interpreting findings by severity
and compliance framework (CIS, NIST, SOC 2, HIPAA, MITRE ATT&CK) · AWS CloudTrail
(multi-region, KMS encryption, log validation) · prioritized remediation ·
distinguishing remediable vs. managed/gated risk · security reporting.
