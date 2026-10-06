# Attacker Flow: Secret Theft and Cloud Lateral Movement

```mermaid
sequenceDiagram
  participant Attacker as Attacker in compromised pod
  participant IMDS as Instance Metadata Service
  participant SAToken as Service Account Token
  participant K8sAPI as Kubernetes API
  participant IAM as AWS IAM
  participant Secrets as Secrets Manager
  participant CloudTrail as CloudTrail
  participant SOAR as SOAR Lambda
  participant SIEM as OpenSearch SIEM

  activate Attacker
  Attacker->>IMDS: Query metadata endpoint for node IAM credentials
  IMDS->>Attacker: BLOCKED - IMDSv2 requires token handshake, hop limit stops containers
  deactivate Attacker

  activate Attacker
  Attacker->>SAToken: Read the pod service-account token
  SAToken->>Attacker: Token retrieved
  Attacker->>K8sAPI: Attempt actions beyond the pod scope
  K8sAPI->>Attacker: DENIED - least-privilege RBAC limits the service account
  deactivate Attacker

  activate Attacker
  Attacker->>IAM: Attempt to use the pod IRSA role for broad AWS actions
  IAM->>Attacker: DENIED - IRSA role scoped to minimal actions and resources
  deactivate Attacker

  activate Attacker
  Attacker->>Secrets: Attempt to read secrets beyond the two the pod needs
  Secrets->>Attacker: DENIED - ESO IRSA role scoped to two specific secret ARNs
  deactivate Attacker

  activate Attacker
  Note over IAM,CloudTrail: Every API call, allowed or denied, is recorded
  IAM->>CloudTrail: Log the access attempts
  CloudTrail->>SIEM: Ship events to the SIEM
  CloudTrail->>SOAR: GuardDuty finding triggers automated response
  SOAR->>IAM: Auto-disable the compromised credential
  deactivate Attacker
```

## Narrative

This scenario begins where others end: the attacker already has a foothold in a pod. The defence is about containment, ensuring one compromised workload cannot cascade into full account compromise.

Each escalation path is blocked by a specific control. The instance metadata attack, the technique behind the Capital One breach, is stopped by IMDSv2 and a hop limit that prevents containers reaching the endpoint. The Kubernetes API is constrained by least-privilege RBAC. The pod's cloud identity is a tightly scoped IRSA role, not a broad node role, so it cannot act widely in AWS. The secrets operator can read only the two specific secrets the app needs. And critically, every attempt, allowed or denied, is recorded in CloudTrail, surfaced in the SIEM, and a GuardDuty finding triggers the SOAR Lambda to auto-disable the compromised credential. The blast radius of a single pod compromise is deliberately tiny.
