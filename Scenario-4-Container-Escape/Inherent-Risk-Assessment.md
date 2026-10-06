# Inherent Risk Assessment: Container Escape and Runtime Compromise

Inherent risk is assessed BEFORE SecureStack's controls are applied, to establish the baseline exposure if an attacker with code execution in a container attempted to escape.

| Category | Description | Likelihood | Impact | Risk Rating | Scenario |
| --- | --- | --- | --- | --- | --- |
| Container Escape | Code execution in a container breaks out to the host node | Medium | High | High | Host Compromise |
| Privilege Escalation | An over-privileged or privileged container enables escape | Medium | High | High | Privileged Breakout |
| Workload Isolation | One compromised pod reaches secrets or data of co-located pods | Medium | High | High | Cross-Workload Exposure |
| Cluster Compromise | From a node, the attacker reaches the control plane or other nodes | Low | High | High | Cluster-Wide Impact |
| Persistence | The attacker installs a backdoor on the node | Medium | High | High | Node Persistence |
| Data Sensitivity | Patient data processed on the node is exposed | Medium | High | High | Healthcare Data Exposure |
| Detection Difficulty | Escape and persistence occur at the OS level, invisible to app logs | Medium | High | High | Blind Spot |
| Resource Abuse | The attacker mines resources using node access | Low | Medium | Medium | Cryptojacking |

## Critical Assets

- The host node and the container runtime.
- Secrets and data of other workloads co-located on the node.
- The Kubernetes control plane and the broader cluster.
- Patient data (PII / PHI) processed by any node workload.

## Summary

Before controls, container escape is predominantly **High** risk. The danger is that a single application compromise becomes host and potentially cluster compromise, because many workloads share a node, and because escape and persistence happen at the operating-system level, invisible to application logging. SecureStack's controls, pod hardening, admission control, and kernel-level runtime detection, are designed to make escape very difficult and to detect it immediately if attempted.
