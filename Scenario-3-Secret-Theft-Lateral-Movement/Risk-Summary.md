# Risk Summary: Secret Theft and Cloud Lateral Movement

Each risk is mapped to a mitigation that is actually implemented in SecureStack and, where noted, proven live. This is the residual-risk view: the inherent High risks reduced by containment controls.

| Risk ID | Description | Severity | Likelihood | Impact | Mitigation (implemented in SecureStack) |
| --- | --- | --- | --- | --- | --- |
| R1 | A compromised pod steals node IAM credentials via the metadata service | High | Medium | High | IMDSv2 enforced with a hop limit so containers cannot reach the metadata endpoint; closes the Capital One attack path. |
| R2 | The pod identity is over-permissioned and enables broad AWS actions | High | Medium | High | Scoped IRSA per pod: a minimal cloud identity bound to the service account, not the broad node role. |
| R3 | The service-account token is abused against the Kubernetes API | High | Medium | High | Least-privilege Kubernetes RBAC limits what the service account can do. |
| R4 | The attacker reads secrets beyond what the pod needs | High | Medium | High | The External Secrets Operator role is scoped to two specific secret ARNs; KMS CMK adds a scoped-decrypt requirement. |
| R5 | A single pod compromise cascades into account-wide compromise | High | Medium | High | Defence in depth: IMDSv2, scoped IRSA, RBAC, and scoped secrets together contain the blast radius to one workload. |
| R6 | The attacker reaches patient data (PII / PHI) in the account | High | Medium | High | Least-privilege identity plus encryption at rest (KMS); access is logged and alertable. |
| R7 | Malicious credential use goes undetected | High | Medium | High | Every API call is logged in CloudTrail and correlated in the OpenSearch SIEM; GuardDuty detects anomalous credential use. |
| R8 | A credential compromise is not responded to quickly | High | Medium | High | SOAR Lambda auto-disables the compromised credential on a GuardDuty finding in seconds (proven live). |
| R9 | Runtime credential theft or enumeration goes unseen | Medium | Medium | Medium | Falco (eBPF) detects the anomalous syscalls associated with credential access (proven live). |
| R10 | The attacker disables logging to cover their tracks | Medium | Low | High | CloudTrail is a managed, append-oriented audit trail; disabling it is itself a high-signal GuardDuty finding. |

## Residual Risk Statement

With SecureStack's containment controls applied, the dominant risk, a single pod compromise cascading into full account compromise, is reduced from High to Low-to-Medium, because every escalation path is independently blocked (IMDSv2, scoped IRSA, RBAC, scoped secrets) and any attempt is detected and auto-remediated (CloudTrail, GuardDuty, SOAR). The highest residual exposure is a vulnerability in the scoped IRSA role's own permitted actions being chained toward a sensitive resource, which is mitigated by keeping IRSA roles minimal and monitoring their use in the SIEM.
