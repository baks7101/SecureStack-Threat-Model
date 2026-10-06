# Controls Required: Container Escape and Runtime Compromise

The following controls mitigate the identified risks. Controls marked [IMPLEMENTED] exist in SecureStack today; those marked [IMPLEMENTED - PROVEN LIVE] were demonstrated on a running cluster; [NEXT STEP] are planned hardening.

## Pod and Container Hardening

- [IMPLEMENTED] Containers run as a non-root user.
- [IMPLEMENTED] Linux capabilities are dropped (no unnecessary elevated privileges).
- [IMPLEMENTED] Read-only root filesystem prevents writing malware or backdoors.
- [IMPLEMENTED] No privileged containers and no host-path mounts.
- [IMPLEMENTED] Checkov independently verifies these settings in the manifests at build time.

## Admission Control

- [IMPLEMENTED - PROVEN LIVE] Kyverno rejects privileged containers, root execution, host-path mounts, and unapproved images before they start.
- [IMPLEMENTED] Pod Security Standards (baseline/restricted) as a second, platform-level admission layer (defence in depth with Kyverno).

## Isolation and Segmentation

- [IMPLEMENTED] Zero-trust Kubernetes network policies limit what a compromised pod can reach.
- [IMPLEMENTED] Workload isolation so one compromised pod does not expose co-located workloads.
- [IMPLEMENTED] Least-privilege RBAC and scoped IRSA reduce what a pod can do if compromised.

## Runtime Detection and Response

- [IMPLEMENTED - PROVEN LIVE] Falco (eBPF) detects escape-indicative behaviour such as a shell spawned in a container, with full pod and image context.
- [IMPLEMENTED] Application and system signals correlated in the OpenSearch SIEM.
- [IMPLEMENTED] GuardDuty provides managed threat detection at the account level.
- [IMPLEMENTED] Prometheus / Grafana observability of cluster and workload behaviour.
- [NEXT STEP] Forward Falco alerts into the SIEM (Falcosidekick) for unified correlation.
- [NEXT STEP] Automated response to critical Falco alerts (for example, isolate or kill the offending pod).
- [NEXT STEP] Keep nodes patched against kernel and runtime vulnerabilities (node lifecycle management).
