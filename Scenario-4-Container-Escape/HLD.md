# High-Level Design: SecureStack (Container Escape view)

```mermaid
flowchart TD
  subgraph External
    Attacker[Attacker with code execution in pod]
  end

  subgraph "Admission Control"
    Kyverno[Kyverno Policies]
  end

  subgraph "Host Node"
    subgraph "Hardened Pod"
      App[Container: non-root]
      RO[Read-only root filesystem]
      Caps[Dropped Linux capabilities]
    end
    Runtime[Container Runtime]
    OtherPods[Other Co-located Pods]
  end

  subgraph "Detection"
    Falco[Falco eBPF Sensor]
    SIEM[OpenSearch SIEM]
    Prom[Prometheus / Grafana]
  end

  ControlPlane[Kubernetes Control Plane]

  Attacker --> App
  Kyverno -.admits only compliant pods.-> App
  App --> RO
  App --> Caps
  App -.cannot reach.-> Runtime
  App -.cannot reach.-> OtherPods
  App -.escape blocked.-> Runtime
  Falco -.watches syscalls.-> App
  Falco --> SIEM
  App -.cannot reach.-> ControlPlane
```

## Key Architectural Controls (relevant to this scenario)

- **Pod hardening**: containers run as a non-root user, with Linux capabilities dropped and a read-only root filesystem, removing the primitives an attacker needs to escape or persist.
- **Kyverno admission control**: rejects privileged containers, root execution, host-path mounts, and unapproved images, so an attacker cannot deploy an escalation helper (proven live).
- **Pod Security Standards**: baseline/restricted enforcement as a second, platform-level admission layer (defence in depth with Kyverno).
- **Falco runtime detection**: watches kernel syscalls via eBPF and alerts on escape-indicative behaviour such as a shell spawned in a container (proven live).
- **Network policies**: zero-trust segmentation limits what a compromised pod can reach across the cluster.
- **Centralised SIEM**: Falco and application signals are correlated for investigation.
