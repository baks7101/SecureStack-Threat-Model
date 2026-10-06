# High-Level Design: SecureStack (CI/CD Compromise view)

```mermaid
flowchart TD
  subgraph External
    Attacker[Attacker]
    Dev[Developer / Maintainer]
    User[Healthcare User - submits symptoms]
  end

  subgraph "Source and CI/CD"
    AppRepo[ai-vibecode-lab Repo]
    PlatRepo[SecureStack-platform Repo]
    Pipeline[13-Stage Security Pipeline]
    Gate[Security Gate]
    OIDC[GitHub OIDC Provider]
  end

  subgraph "AWS Account"
    ECR[ECR Image Registry]
    ArgoCD[ArgoCD GitOps]

    subgraph "EKS Cluster"
      Pod[MediTriage Pod: API + llm-guard]
      Kyverno[Kyverno Admission Control]
      Falco[Falco Runtime Detection]
      ESO[External Secrets Operator]
    end

    SM[Secrets Manager + KMS]
    OpenSearch[OpenSearch SIEM]
    CloudTrail[CloudTrail]
    GuardDuty[GuardDuty]
    SOAR[SOAR Lambda]
    Bootstrap[Bootstrap Stack: persistent OIDC role]
  end

  OpenAI[OpenAI API]

  Dev --> AppRepo
  AppRepo --> Pipeline
  PlatRepo --> Pipeline
  Pipeline --> Gate
  Gate --> ECR
  Pipeline --> OIDC
  OIDC --> Bootstrap

  ECR --> Pod
  AppRepo --> ArgoCD
  ArgoCD --> Pod
  Kyverno -.admits.-> Pod
  Falco -.watches.-> Pod
  Pod --> ESO
  ESO --> SM
  Pod --> OpenAI

  Pod --> OpenSearch
  CloudTrail --> OpenSearch
  GuardDuty --> OpenSearch
  GuardDuty --> SOAR

  User --> Pod

  Attacker --> AppRepo
  Attacker --> PlatRepo
  Attacker --> OIDC
```

## Key Architectural Controls (relevant to this scenario)

- **Two-repository separation**: the security pipeline and policies live in the platform repo, owned centrally; the app consumes them via `workflow_call` and cannot weaken them.
- **13-stage pipeline + security gate**: blocks insecure code, poisoned dependencies, and misconfigurations before merge.
- **Keyless OIDC authentication**: the pipeline assumes a short-lived, scoped AWS role; no long-lived keys exist to steal.
- **Bootstrap stack**: the persistent CI identity role is isolated in its own Terraform state, separate from the ephemeral cluster.
- **GitOps (ArgoCD)**: git is the single source of truth; only declared manifests deploy, and drift is auto-reverted.
- **Admission + runtime**: Kyverno gates what can start; Falco watches what runs.
- **Keyless secrets**: Secrets Manager + ESO via a tightly scoped IRSA role.
- **Centralised SIEM + SOAR**: CloudTrail, GuardDuty, and app logs converge in OpenSearch; GuardDuty findings trigger automated response.
