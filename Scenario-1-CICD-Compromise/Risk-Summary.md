# Risk Summary: CI/CD Pipeline Compromise

Each risk is mapped to a mitigation that is actually implemented in SecureStack and, where noted, proven live. This is the residual-risk view: the inherent High risks reduced by the layered controls.

| Risk ID | Description | Severity | Likelihood | Impact | Mitigation (implemented in SecureStack) |
| --- | --- | --- | --- | --- | --- |
| R1 | A poisoned dependency or vulnerable code reaches production through the pipeline | High | Medium | High | 13-stage pipeline with a merge-blocking gate: Gitleaks, CodeQL, custom Semgrep rules, Trivy SCA, dependency pin-check (exact versions), Syft/Grype SBOM scan. Branch protection requires the gate to pass. |
| R2 | A tampered image or Dockerfile is deployed | High | Medium | High | Container scan stage (Trivy), pinned base images, and GitOps: ArgoCD deploys only git-declared manifests, so an attacker who pushes an image cannot deploy it without a reviewed commit. |
| R3 | A compromised pipeline assumes cloud credentials and acts in AWS | High | Medium | High | Keyless OIDC with a short-lived, read-only scoped role (no long-lived keys to steal). The persistent identity role is isolated in a separate bootstrap Terraform stack. |
| R4 | Infrastructure misconfiguration is introduced via IaC | High | Medium | High | Checkov IaC scan and custom OPA/Rego policy evaluated against a real Terraform plan (e.g. blocks SSH open to the internet). Both must pass the gate. |
| R5 | A malicious or unapproved AI component is introduced | High | Low | High | AI-BOM validation enforcing data-classification ceilings, plus a CLAUDE.md governance check for AI coding agents. Build fails on an unapproved model. |
| R6 | A deployed workload reads secrets or patient data it should not | High | Medium | High | Keyless secrets via ESO with an IRSA role scoped to only two secrets; KMS CMK encryption with tightly scoped kms:Decrypt; IMDSv2 and per-pod IRSA limit lateral movement. |
| R7 | A privileged or non-compliant workload reaches the cluster | High | Medium | High | Kyverno admission control rejects privileged / root / unapproved-registry pods before they start (proven live). |
| R8 | Malicious runtime behaviour goes undetected | High | Medium | High | Falco (eBPF) detects anomalous syscalls such as a shell in a container (proven live); CloudTrail + GuardDuty feed the OpenSearch SIEM; a SOAR Lambda auto-responds to findings (proven live). |
| R9 | A change to the central platform affects every consuming app | High | Low | High | Platform changes are themselves scanned by the pipeline, gated by branch protection, and the platform runs a trimmed pipeline on itself (dogfooding). |
| R10 | Prolonged recovery / uncertain integrity after compromise | Medium | Medium | Medium | Everything is code (Terraform, manifests, pipeline) and reproducible; immutable audit trail in CloudTrail; S3 state versioning. |

## Residual Risk Statement

With SecureStack's layered controls applied, the dominant failure path, a single malicious change reaching production, is reduced from High to Low-to-Medium, because an attacker must defeat multiple independent controls (the gate, GitOps, scoped OIDC, IRSA, admission control, and runtime detection) rather than any one. The highest residual exposure is a determined insider with valid credentials combined with weak branch protection, which is why branch protection, least-privilege identity, and the SIEM are emphasised.
