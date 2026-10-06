# STRIDE Analysis: Container Escape and Runtime Compromise

```mermaid
graph TD
  subgraph Pod["Hardened Pod"]
    App[Container: non-root]
    RO[Read-only FS]
    Caps[Dropped capabilities]
  end

  subgraph Node["Host Node"]
    Runtime[Container Runtime]
    OtherPods[Co-located Pods]
  end

  subgraph Controls["Controls"]
    Kyverno[Kyverno Admission]
    Falco[Falco eBPF]
    NetPol[Network Policies]
    SIEM[("OpenSearch SIEM")]
  end

  App --> RO
  App --> Caps
  Kyverno -.admits.-> App
  App -.escape blocked.-> Runtime
  App -.isolated from.-> OtherPods
  Falco -.watches.-> App
  Falco --> SIEM

  T1["Spoofing - run as a more privileged identity"] -.-> App
  T2["Tampering - write malware or modify the node"] -.-> Runtime
  T3["Repudiation - act at OS level invisible to app logs"] -.-> App
  T4["Information Disclosure - read co-located secrets and data"] -.-> OtherPods
  T5["Denial of Service - crash the node or exhaust resources"] -.-> Node
  T6["Elevation of Privilege - escape the container to the host"] -.-> Runtime

  M1["Non-root execution, dropped capabilities, no privileged context"] --> T1
  M2["Read-only root filesystem, Kyverno denies host-path mounts"] --> T2
  M3["Falco eBPF syscall monitoring, alerts shipped to the SIEM"] --> T3
  M4["Workload isolation, network policies, least-privilege identity"] --> T4
  M5["Resource limits, node monitoring, autoscaling"] --> T5
  M6["Pod hardening plus Kyverno admission, Falco detects escape attempts"] --> T6
```

## STRIDE Breakdown

| STRIDE Category | Threat | SecureStack Mitigation |
| --- | --- | --- |
| **S**poofing | Run as a more privileged identity to aid escape | Non-root execution, dropped capabilities, no privileged context |
| **T**ampering | Write malware to disk or modify the node | Read-only root filesystem; Kyverno denies host-path mounts |
| **R**epudiation | Act at the OS level, invisible to application logs | Falco eBPF syscall monitoring with alerts shipped to the SIEM |
| **I**nformation Disclosure | Read secrets and data of co-located workloads | Workload isolation, zero-trust network policies, least-privilege identity |
| **D**enial of Service | Crash the node or exhaust its resources | Resource limits, node monitoring, autoscaling |
| **E**levation of Privilege | Escape the container to gain host-level control | Pod hardening plus Kyverno admission control; Falco detects escape attempts |
