# Inherent Risk Assessment: Secret Theft and Cloud Lateral Movement

Inherent risk is assessed BEFORE SecureStack's controls are applied, to establish the baseline exposure if an attacker with an initial foothold attempted to escalate and move laterally.

| Category | Description | Likelihood | Impact | Risk Rating | Scenario |
| --- | --- | --- | --- | --- | --- |
| Credential Theft | A compromised pod steals node or role credentials via the metadata service | Medium | High | High | Metadata Credential Theft |
| Privilege Escalation | An over-permissioned pod identity enables broad AWS actions | Medium | High | High | Identity Abuse |
| Lateral Movement | A single compromised workload reaches other resources in the account | Medium | High | High | Account-Wide Movement |
| Secret Exposure | The attacker reads the OpenAI key and application secrets | Medium | High | High | Secret Exfiltration |
| Data Sensitivity | The attacker reaches patient data (PII / PHI) in the account | Medium | High | High | Healthcare Data Exposure |
| Blast Radius | One pod compromise cascades into full account compromise | Medium | High | High | Uncontained Breach |
| Anti-Forensics | The attacker disables logging to cover their tracks | Low | High | Medium | Log Tampering |
| Detection Difficulty | Activity via stolen legitimate credentials blends with normal traffic | Medium | Medium | Medium | Stealth Operation |

## Critical Assets

- The node IAM role credentials (reachable via the metadata service if unprotected).
- The pod service-account token and its RBAC permissions.
- The OpenAI API key and application secrets in Secrets Manager.
- Patient data (PII / PHI) and other resources across the AWS account.
- CloudTrail logs and the integrity of the audit trail.

## Summary

Before controls, post-foothold escalation is predominantly **High** risk. The core danger is blast radius, whether a single compromised pod can cascade into full account compromise through stolen credentials and lateral movement. This is precisely the failure mode behind major cloud breaches such as Capital One, where a compromised component reached the metadata service and stole credentials. SecureStack's controls, IMDSv2, scoped IRSA, least-privilege RBAC, scoped secrets access, and automated response, are designed to contain the blast radius to the single compromised workload.
