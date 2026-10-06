# Attacker Flow: CI/CD Pipeline Compromise

```mermaid
sequenceDiagram
  participant Attacker
  participant Repos as GitHub Repos (app + platform)
  participant Pipeline as SecureStack Pipeline (13 stages)
  participant OIDC as GitHub OIDC -> AWS Role
  participant ECR as ECR (Image Registry)
  participant ArgoCD as ArgoCD (GitOps)
  participant EKS as EKS Cluster
  participant Secrets as Secrets Manager
  participant SIEM as OpenSearch SIEM

  activate Attacker
  Attacker->>Repos: Enumerate ai-vibecode-lab + SecureStack-platform
  Repos->>Attacker: Reusable workflow, OIDC trust, branch rules identified
  deactivate Attacker

  activate Attacker
  Attacker->>Repos: Open PR with poisoned dependency / tampered Dockerfile
  Repos->>Pipeline: Pull request triggers the 13-stage security pipeline
  Pipeline->>Pipeline: Secret scan, SAST, SCA, SBOM, IaC, policy, AI-BOM, DAST
  Pipeline->>Attacker: Malicious change BLOCKED at the security gate (expected)
  deactivate Attacker

  activate Attacker
  Attacker->>Repos: Attempt bypass - weak branch protection or stolen CI creds
  Repos->>OIDC: Compromised workflow assumes the AWS role
  OIDC->>Attacker: Scoped read-only credentials (least privilege limits blast radius)
  deactivate Attacker

  activate Attacker
  Attacker->>ECR: Push tampered image (if registry write is reachable)
  ECR->>ArgoCD: Image referenced by a git-committed manifest
  ArgoCD->>EKS: Reconcile cluster to git (deploys only what git declares)
  deactivate Attacker

  activate Attacker
  EKS->>Secrets: Malicious workload attempts to read the OpenAI key / app secrets
  Secrets->>EKS: ESO IRSA role scoped to only two secrets (least privilege)
  EKS->>SIEM: CloudTrail + GuardDuty record the API calls
  SIEM->>SIEM: Anomalous activity surfaced for investigation
  deactivate Attacker
```

## Narrative

The attacker first maps the two-repository supply chain. Their primary delivery attempt, a malicious pull request, is designed to be caught by the pipeline, and in the normal case it is: the 13-stage gate blocks it. The attacker therefore pivots to bypass techniques (weak branch protection, stolen credentials, abusing the OIDC-assumed role).

Even on a successful bypass, defence in depth constrains the blast radius at every step: the OIDC role is scoped read-only, GitOps means only git-declared manifests deploy (not arbitrary attacker pushes), the External Secrets Operator's IRSA role can read only two specific secrets, and every AWS API call lands in CloudTrail and GuardDuty for the SIEM to surface. No single compromise grants the full objective.
