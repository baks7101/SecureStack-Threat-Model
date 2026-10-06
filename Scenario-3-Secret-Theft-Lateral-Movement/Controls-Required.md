# Controls Required: Secret Theft and Cloud Lateral Movement

The following controls mitigate the identified risks. Controls marked [IMPLEMENTED] exist in SecureStack today; those marked [IMPLEMENTED - PROVEN LIVE] were demonstrated on a running cluster; [NEXT STEP] are planned hardening.

## Identity and Metadata Protection

- [IMPLEMENTED] IMDSv2 enforced with a hop limit so containers cannot query the metadata service for node credentials (closes the Capital One attack path).
- [IMPLEMENTED] Scoped IRSA per pod: each workload has a minimal cloud identity bound to its service account, not the broad node IAM role.
- [IMPLEMENTED] Least-privilege Kubernetes RBAC constraining what each service-account token can do.
- [NEXT STEP] Periodic IAM access-analyzer review to catch permission creep on IRSA roles.

## Secrets Protection

- [IMPLEMENTED] The External Secrets Operator role is scoped to only two specific secret ARNs; it cannot enumerate or read others.
- [IMPLEMENTED] Secrets encrypted with a KMS customer-managed key; decrypt permission is scoped to that specific key.
- [IMPLEMENTED] Secrets are never in git, never in container images, and never seen by a human.
- [NEXT STEP] Automatic scheduled secret rotation.

## Blast-Radius Containment

- [IMPLEMENTED] Defence in depth: IMDSv2, scoped IRSA, RBAC, and scoped secrets together ensure a single pod compromise cannot cascade into account-wide compromise.
- [IMPLEMENTED] Encryption at rest (KMS) and in transit (TLS) across the account.
- [IMPLEMENTED] Network segmentation via Kubernetes network policies.

## Detection and Response

- [IMPLEMENTED] CloudTrail logs every AWS API call (allowed or denied) across IAM, Secrets Manager, S3, and more.
- [IMPLEMENTED - PROVEN LIVE] Centralised OpenSearch SIEM correlating CloudTrail, GuardDuty, and application logs.
- [IMPLEMENTED] GuardDuty detects anomalous credential use, metadata access, and reconnaissance behaviour.
- [IMPLEMENTED - PROVEN LIVE] SOAR Lambda auto-disables a compromised credential on a GuardDuty finding in seconds.
- [IMPLEMENTED - PROVEN LIVE] Falco (eBPF) detects the anomalous syscalls associated with credential theft and enumeration.
- [NEXT STEP] Forward Falco alerts into the SIEM (Falcosidekick).
- [NEXT STEP] Alerting on CloudTrail logging being disabled (a high-signal anti-forensics indicator).
