# STRIDE Analysis: CI/CD Pipeline Compromise

```mermaid
graph TD
  subgraph User_And_Dev["User / Dev Interaction"]
    Dev[Developer / Maintainer]
    U[Healthcare User]
    A[Attacker]
  end

  subgraph CI_CD["CI/CD Layer"]
    AppRepo[ai-vibecode-lab Repo]
    PlatRepo[SecureStack-platform Repo]
    Pipeline[13-Stage Pipeline]
    Gate[Security Gate]
    OIDC[GitHub OIDC -> AWS]
  end

  subgraph AWS["SecureStack on AWS"]
    ECR[("ECR Registry")]
    ArgoCD[ArgoCD GitOps]
    Pod[MediTriage Pod: API + llm-guard]
    Kyverno[Kyverno]
    Falco[Falco]
    ESO[External Secrets Operator]
    SM[("Secrets Manager + KMS")]
    OS[("OpenSearch SIEM")]
    CT[CloudTrail]
    GD[GuardDuty]
  end

  Dev --> AppRepo
  AppRepo --> Pipeline
  PlatRepo --> Pipeline
  Pipeline --> Gate
  Gate --> ECR
  Pipeline --> OIDC
  ECR --> Pod
  AppRepo --> ArgoCD
  ArgoCD --> Pod
  Pod --> ESO
  ESO --> SM
  U --> Pod
  Pod -.-> OS
  CT --> OS
  GD --> OS

  T1["Spoofing - stolen CI token / maintainer creds"] -.-> OIDC
  T2["Tampering - poison dependency / tamper Dockerfile / edit shared policy"] -.-> Pipeline
  T2 -.-> ECR
  T3["Repudiation - deny changes / hide in build logs"] -.-> Pipeline
  T4["Information Disclosure - exfiltrate secrets / PHI"] -.-> SM
  T5["Denial of Service - break the pipeline or flood the gate"] -.-> Gate
  T6["Elevation of Privilege - abuse OIDC role / privileged pod"] -.-> Pod

  M1["OIDC keyless auth, short-lived scoped role, branch protection, MFA"] --> T1
  M2["Gate blocks poisoned deps/code; GitOps deploys only git-declared manifests; platform dogfoods its own pipeline"] --> T2
  M3["Immutable CloudTrail audit, required PR reviews, SIEM correlation"] --> T3
  M4["ESO IRSA scoped to 2 secrets, KMS CMK, no secrets in git, IMDSv2"] --> T4
  M5["Independent parallel stages, pinned scanner versions, fail-closed gate"] --> T5
  M6["Least-privilege IRSA per pod, Kyverno admission control, Falco runtime detection"] --> T6
```

## STRIDE Breakdown

| STRIDE Category | Threat | SecureStack Mitigation |
| --- | --- | --- |
| **S**poofing | Attacker uses stolen CI tokens or maintainer credentials to act as a trusted identity | Keyless OIDC (short-lived, scoped role), branch protection, MFA on accounts, no long-lived keys |
| **T**ampering | Poison a dependency, tamper the Dockerfile, or edit a shared policy / composite action | 13-stage gate (SCA, pin-check, SBOM, container scan), GitOps deploys only git-declared manifests, platform runs its own pipeline on itself |
| **R**epudiation | Deny making a malicious change, or hide activity in build logs | Immutable CloudTrail audit trail, required pull-request reviews, centralised SIEM correlation |
| **I**nformation Disclosure | Exfiltrate the OpenAI key, secrets, or patient PII / PHI | ESO IRSA role scoped to two secrets, KMS CMK encryption, secrets never in git, IMDSv2 closing metadata theft |
| **D**enial of Service | Break the pipeline or overwhelm the gate to force a bypass | Independent parallel stages, pinned scanner versions, a fail-closed gate (a broken check blocks, it does not pass) |
| **E**levation of Privilege | Abuse an over-permissioned role or deploy a privileged pod | Least-privilege IRSA per pod, Kyverno rejecting privileged pods at admission (proven live), Falco detecting runtime escalation (proven live) |
