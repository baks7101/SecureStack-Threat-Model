# Data Flow Diagram: CI/CD Pipeline Compromise

```mermaid
flowchart TD
  A["Attacker"] --> R1["ai-vibecode-lab Repo (app)"]
  A --> R2["SecureStack-platform Repo (pipeline + policies)"]

  R1 --> P["SecureStack 13-Stage Pipeline (workflow_call)"]
  R2 --> P
  P --> GATE["Security Gate (blocks on any failure)"]

  GATE --> IMG["Docker Image Build"]
  IMG --> ECR["ECR (Private Image Registry)"]

  DEV["Developer"] --> R1
  R1 --> ARGO["ArgoCD (GitOps reconcile)"]
  ECR --> KUBELET["EKS kubelet pulls image"]
  ARGO --> POD["MediTriage Pod (API + llm-guard sidecar)"]
  KUBELET --> POD

  POD --> ESO["External Secrets Operator (IRSA)"]
  ESO --> SM["AWS Secrets Manager (OpenAI key, tokens)"]
  POD --> OAI["OpenAI API"]

  POD --> FB["Fluent Bit"]
  FB --> OS["OpenSearch SIEM"]
  CT["CloudTrail"] --> OS
  GD["GuardDuty"] --> OS
```

## Trust Boundaries

| Boundary | Separates | Why it matters |
| --- | --- | --- |
| Repo boundary | App repo vs platform repo | The app team consumes the pipeline but cannot alter the security controls; the security team owns them centrally. |
| CI boundary | GitHub Actions runner vs AWS account | The runner authenticates to AWS via short-lived OIDC credentials, not long-lived keys. A compromised runner gets a scoped, read-only role. |
| Registry boundary | CI build vs the cluster | The cluster pulls only from the private ECR; images are not pulled from arbitrary sources. |
| GitOps boundary | Git (desired state) vs the live cluster | ArgoCD deploys only what is declared in git. An attacker who pushes an image cannot deploy it without a reviewed git commit referencing it. |
| Secrets boundary | The vault vs the workload | Secrets never touch git. ESO reads them via an IRSA role scoped to exactly two secrets; no human handles the values. |
| Account boundary | The workload vs the wider AWS account | IRSA per-pod identity and IMDSv2 prevent a compromised pod from assuming broad node or account permissions. |

## Sensitive Data in Scope

- Patient-submitted symptoms (potential PII / PHI) processed by the triage API.
- The OpenAI API key and application secrets held in Secrets Manager.
- The pipeline's OIDC-assumed AWS credentials.
- Stored triage logs (held in memory for the lab; would be a database in production).
