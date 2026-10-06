# Attacker Flow: Container Escape and Runtime Compromise

```mermaid
sequenceDiagram
  participant Attacker as Attacker with code execution in pod
  participant Container as Container Runtime
  participant Kyverno as Kyverno Admission
  participant Node as Host Node
  participant Falco as Falco eBPF Sensor
  participant SIEM as OpenSearch SIEM
  participant Cluster as Kubernetes Control Plane

  activate Attacker
  Attacker->>Container: Enumerate user, capabilities, mounts, privileges
  Container->>Attacker: Running as non-root, no added capabilities, read-only filesystem
  deactivate Attacker

  activate Attacker
  Attacker->>Kyverno: Attempt to deploy a privileged helper pod
  Kyverno->>Attacker: REJECTED - policy disallows privileged containers
  deactivate Attacker

  activate Attacker
  Attacker->>Container: Attempt to write a malicious binary to disk
  Container->>Attacker: DENIED - read-only root filesystem
  deactivate Attacker

  activate Attacker
  Attacker->>Node: Attempt to mount host filesystem or reach the runtime socket
  Node->>Attacker: DENIED - dropped capabilities, no host mounts, non-privileged
  deactivate Attacker

  activate Attacker
  Attacker->>Container: Spawn a shell to continue the attack
  Container->>Falco: execve syscall observed
  Falco->>SIEM: ALERT - shell spawned in a container, with pod and image detail
  deactivate Attacker

  Note over Falco,Cluster: Runtime detection catches what admission and hardening could not
```

## Narrative

This scenario begins with the attacker already executing code inside a container. The defence is layered to make escape as hard as possible and to detect it if attempted.

Pod hardening removes the tools an attacker needs: the container runs as a non-root user, has its Linux capabilities dropped, and has a read-only root filesystem, so writing a malicious binary fails. Kyverno admission control prevents the attacker from deploying a privileged helper pod to escalate. Attempts to mount the host filesystem or reach the container runtime socket fail because the pod is non-privileged with no host mounts. And when the attacker spawns a shell to continue, Falco, watching kernel syscalls via eBPF, catches the execve immediately and alerts with full pod and image context. Admission control and hardening raise the bar; Falco catches what gets through.
