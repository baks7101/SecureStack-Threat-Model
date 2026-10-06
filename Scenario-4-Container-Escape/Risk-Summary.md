# Risk Summary: Container Escape and Runtime Compromise

Each risk is mapped to a mitigation that is actually implemented in SecureStack and, where noted, proven live. This is the residual-risk view: the inherent High risks reduced by hardening, admission control, and runtime detection.

| Risk ID | Description | Severity | Likelihood | Impact | Mitigation (implemented in SecureStack) |
| --- | --- | --- | --- | --- | --- |
| R1 | Code execution in a container escapes to the host node | High | Medium | High | Pod hardening: non-root user, dropped Linux capabilities, read-only root filesystem, removing the primitives needed to escape. |
| R2 | A privileged container is used to break out | High | Low | High | Kyverno admission control rejects privileged containers before they start (proven live); Pod Security Standards as a second layer. |
| R3 | The attacker writes malware or a backdoor to disk | High | Medium | High | Read-only root filesystem prevents writing executables inside the container. |
| R4 | The attacker mounts the host filesystem or reaches the runtime socket | High | Low | High | Non-privileged context, no host-path mounts, and dropped capabilities block host access; Kyverno denies host-path pods. |
| R5 | A compromised pod reaches other workloads on the node | High | Medium | High | Workload isolation plus zero-trust network policies limit what a pod can reach. |
| R6 | Escape or persistence happens invisibly at the OS level | High | Medium | High | Falco (eBPF) observes kernel syscalls and alerts on escape-indicative behaviour such as a shell in a container (proven live). |
| R7 | The attacker reaches the control plane or other nodes | High | Low | High | Least-privilege RBAC, network policies, and Falco detection constrain and surface lateral movement. |
| R8 | Runtime compromise goes undetected and unresponded | High | Medium | High | Falco alerts feed investigation; GuardDuty and the SIEM provide correlation; metrics in Grafana. |
| R9 | Patient data on the node is exposed | High | Medium | High | Containment to the single pod, encryption at rest, and least-privilege identity limit reachable data. |

## Residual Risk Statement

With SecureStack's controls applied, the dominant risk, a container escape leading to host or cluster compromise, is reduced from High to Low-to-Medium, because hardening removes the escape primitives, admission control blocks privileged escalation helpers, and Falco detects any breakout attempt at the kernel level. The highest residual exposure is a novel kernel or runtime zero-day that defeats hardening; this is mitigated by keeping nodes patched, minimising node privileges, and relying on Falco for behaviour-based detection that does not depend on knowing the specific exploit.
