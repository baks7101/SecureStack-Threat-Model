# High-Level Design: SecureStack (Secret Theft and Lateral Movement view)

```mermaid
flowchart TD
  subgraph External
    Attacker[Attacker with initial foothold]
  end

  subgraph "EKS Cluster"
    Pod[Compromised Pod]
    SAToken[Service Account Token]
    RBAC[Kubernetes RBAC]
    Kyverno[Kyverno]
    Falco[Falco Runtime Detection]
    ESO[External Secrets Operator]
  end

  subgraph "Node and Identity"
    IMDS[IMDSv2 Metadata Service]
    NodeRole[Node IAM Role]
    IRSA[Scoped IRSA Pod Role]
  end

  subgraph "AWS Account"
    SM[Secrets Manager]
    KMS[KMS CMK]
    OtherAWS[Other AWS Resources]
    CloudTrail[CloudTrail]
    GuardDuty[GuardDuty]
    SOAR[SOAR Lambda]
    OpenSearch[OpenSearch SIEM]
  end

  Attacker --> Pod
  Pod --> SAToken
  SAToken --> RBAC
  RBAC --> K8sAPI[Kubernetes API]
  Pod --> IMDS
  IMDS -.hop limit blocks.-> NodeRole
  Pod --> IRSA
  IRSA --> SM
  SM --> KMS
  IRSA -.scoped.-> OtherAWS
  Falco -.watches.-> Pod

  IRSA --> CloudTrail
  K8sAPI --> CloudTrail
  CloudTrail --> OpenSearch
  GuardDuty --> OpenSearch
  GuardDuty --> SOAR
  SOAR --> IRSA
```

## Key Architectural Controls (relevant to this scenario)

- **IMDSv2 with hop limit**: prevents a container from querying the metadata service to steal the node IAM role (closes the Capital One attack path).
- **Scoped IRSA per pod**: each pod has a minimal cloud identity bound to its service account, not the broad node role, so a compromised pod cannot act widely in AWS.
- **Least-privilege Kubernetes RBAC**: the service-account token can perform only narrowly defined actions against the Kubernetes API.
- **Scoped secrets access**: the External Secrets Operator role can read only two specific secret ARNs; it cannot enumerate or read others.
- **KMS CMK with scoped decrypt**: reading a secret also requires permission to use the specific encryption key.
- **Immutable audit and automated response**: every API call is logged in CloudTrail and surfaced in the OpenSearch SIEM; GuardDuty findings trigger a SOAR Lambda that auto-disables the compromised credential.
- **Runtime detection**: Falco watches for the anomalous syscalls associated with credential theft and enumeration.
