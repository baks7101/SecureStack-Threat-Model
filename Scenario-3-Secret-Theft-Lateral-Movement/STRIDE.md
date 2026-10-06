# STRIDE Analysis: Secret Theft and Cloud Lateral Movement

```mermaid
graph TD
  subgraph Workload["Compromised Workload"]
    Pod[Compromised Pod]
    SAT[Service Account Token]
  end

  subgraph Identity["Identity and Metadata"]
    IMDS[IMDSv2]
    IRSA[Scoped IRSA Role]
    RBAC[Kubernetes RBAC]
  end

  subgraph AWS["AWS Account"]
    SM[("Secrets Manager + KMS")]
    OtherAWS[("Other AWS Resources")]
    CT[CloudTrail]
    GD[GuardDuty]
    SOAR[SOAR Lambda]
    OS[("OpenSearch SIEM")]
  end

  Pod --> SAT
  SAT --> RBAC
  Pod --> IMDS
  Pod --> IRSA
  IRSA --> SM
  IRSA --> OtherAWS
  IRSA --> CT
  CT --> OS
  GD --> SOAR

  T1["Spoofing - impersonate the pod identity or node role"] -.-> IRSA
  T2["Tampering - modify IAM policies or disable logging"] -.-> CT
  T3["Repudiation - operate via stolen legitimate credentials"] -.-> IRSA
  T4["Information Disclosure - steal secrets and patient data"] -.-> SM
  T5["Denial of Service - exhaust or disrupt account resources"] -.-> OtherAWS
  T6["Elevation of Privilege - steal node creds via metadata"] -.-> IMDS

  M1["Scoped IRSA per pod, short-lived credentials, no broad node role"] --> T1
  M2["Immutable CloudTrail, disabling logs is a high-signal GuardDuty finding"] --> T2
  M3["All API calls logged and correlated in the SIEM, GuardDuty anomaly detection"] --> T3
  M4["ESO scoped to 2 secret ARNs, KMS CMK scoped decrypt, encryption at rest"] --> T4
  M5["Least-privilege IAM limits reachable resources, quotas and monitoring"] --> T5
  M6["IMDSv2 with hop limit blocks metadata credential theft"] --> T6
```

## STRIDE Breakdown

| STRIDE Category | Threat | SecureStack Mitigation |
| --- | --- | --- |
| **S**poofing | Impersonate the pod's identity or assume the node IAM role | Scoped IRSA per pod with short-lived credentials; no reliance on a broad node role |
| **T**ampering | Modify IAM policies or disable CloudTrail to hide activity | Immutable audit trail; disabling logs is itself a high-signal GuardDuty finding |
| **R**epudiation | Operate using stolen legitimate credentials to avoid attribution | Every API call logged in CloudTrail and correlated in the SIEM; GuardDuty flags anomalous use |
| **I**nformation Disclosure | Steal the OpenAI key, application secrets, or patient data | ESO scoped to two secret ARNs, KMS CMK scoped decrypt, encryption at rest |
| **D**enial of Service | Exhaust or disrupt account resources with stolen access | Least-privilege IAM limits reachable resources; quotas and monitoring |
| **E**levation of Privilege | Steal the node IAM role via the metadata service | IMDSv2 with a hop limit prevents containers reaching the metadata endpoint |
