# MITRE ATT&CK Sequence Summary: Container Escape and Runtime Compromise

```mermaid
flowchart TD
    style Execution fill:#F4D03F,stroke:#000,stroke-width:2px
    style Discovery fill:#F5B041,stroke:#000,stroke-width:2px
    style PrivEsc fill:#EB984E,stroke:#000,stroke-width:2px
    style Escape fill:#E59866,stroke:#000,stroke-width:2px
    style Persistence fill:#DC7633,stroke:#000,stroke-width:2px
    style Lateral fill:#BA4A00,stroke:#000,stroke-width:2px
    style MITRE fill:#85C1E9,stroke:#000,stroke-width:2px

    Execution[Execution in Container] -->|Enumerate runtime, caps, mounts| Discovery[Discovery]
    Discovery -->|Identify escape path| PrivEsc[Privilege Escalation]
    PrivEsc -->|Abuse privileged context or capability| Escape[Escape to Host]
    Escape -->|Persist on the node| Persistence[Persistence]
    Persistence -->|Reach other nodes or control plane| Lateral[Lateral Movement]
    Lateral -->|Access co-located secrets and data| Lateral

    subgraph MITRE_Attack[MITRE ATT&CK Techniques]
        Discovery -->|T1613 - Container and Resource Discovery| MITRE
        PrivEsc -->|T1611 - Escape to Host| MITRE
        PrivEsc -->|T1548 - Abuse Elevation Control| MITRE
        Escape -->|T1610 - Deploy Container| MITRE
        Persistence -->|T1543 - Create or Modify System Process| MITRE
        Lateral -->|T1021 - Remote Services| MITRE
    end
```

## Technique Mapping

| Kill Chain Stage | MITRE Technique | Application to SecureStack |
| --- | --- | --- |
| Discovery | T1613 Container and Resource Discovery | Enumerating the container runtime, capabilities, and mounts |
| Privilege Escalation | T1611 Escape to Host | Attempting to break out of the container to the node |
| Privilege Escalation | T1548 Abuse Elevation Control Mechanism | Abusing a privileged context or capability |
| Escape | T1610 Deploy Container | Deploying a privileged helper container (blocked by Kyverno) |
| Persistence | T1543 Create or Modify System Process | Installing a backdoor process on the node |
| Lateral Movement | T1021 Remote Services | Reaching other nodes or the control plane |
