# Inherent Risk Assessment: CI/CD Pipeline Compromise

Inherent risk is assessed BEFORE SecureStack's controls are applied, to establish the baseline exposure a CI/CD supply-chain attack would present against an AI healthcare application.

| Category | Description | Likelihood | Impact | Risk Rating | Scenario |
| --- | --- | --- | --- | --- | --- |
| Supply Chain | A poisoned dependency or tampered artifact can reach production through the pipeline | Medium | High | High | Pipeline Poisoning |
| Privilege / Identity | A compromised pipeline can assume cloud credentials and act in the AWS account | Medium | High | High | Credential Abuse |
| Blast Radius (Platform) | A change to the central platform pipeline is inherited by every consuming app at once | Low | High | High | Platform-Wide Impact |
| Data Sensitivity | The app processes patient symptoms (PII / PHI) and holds an OpenAI key and secrets | Medium | High | High | Data / Secret Exposure |
| Detection Difficulty | Malicious code deployed through the trusted pipeline looks like a legitimate release | Medium | High | High | Stealth Persistence |
| Integrity | Attacker code running as a trusted workload can tamper with triage logic or outputs | Medium | High | High | Workload Tampering |
| Compliance | Breach of regulated healthcare data would trigger regulatory exposure | Medium | High | High | Regulatory Exposure |
| Recovery Complexity | Restoring trust after a supply-chain compromise requires auditing the whole pipeline | Medium | Medium | Medium | Prolonged Recovery |

## Critical Assets

- The OpenAI API key and application secrets in Secrets Manager.
- Patient data (PII / PHI) processed by the triage API.
- The pipeline's OIDC-assumed AWS role and the persistent bootstrap identity role.
- The integrity of the central SecureStack platform pipeline and policies.
- The ECR image registry and the EKS workloads.

## Summary

Before controls, a CI/CD compromise against this system is predominantly **High** risk, driven by the sensitivity of healthcare data, the ability of a compromised pipeline to act in the cloud account, and the amplified blast radius of a shared platform pipeline. These inherent risks are what SecureStack's layered controls (assessed in the Risk Summary and Controls Required) are designed to reduce.
