# MITRE ATT&CK Sequence Summary: Secret Theft and Cloud Lateral Movement

```mermaid
flowchart TD
    style Foothold fill:#F4D03F,stroke:#000,stroke-width:2px
    style Discovery fill:#F5B041,stroke:#000,stroke-width:2px
    style CredAccess fill:#EB984E,stroke:#000,stroke-width:2px
    style PrivEsc fill:#E59866,stroke:#000,stroke-width:2px
    style Lateral fill:#DC7633,stroke:#000,stroke-width:2px
    style Exfil fill:#BA4A00,stroke:#000,stroke-width:2px
    style MITRE fill:#85C1E9,stroke:#000,stroke-width:2px

    Foothold[Initial Foothold] -->|Enumerate pod, identity, network| Discovery[Discovery]
    Discovery -->|Query metadata, read token, probe secrets| CredAccess[Credential Access]
    CredAccess -->|Steal node creds or abuse scoped role| PrivEsc[Privilege Escalation]
    PrivEsc -->|Use stolen identity in the account| Lateral[Lateral Movement]
    Lateral -->|Reach other resources and data| Exfil[Exfiltration]
    Exfil -->|Exfiltrate secrets and PHI| Exfil
    Exfil -->|Disable logging to hide| Exfil

    subgraph MITRE_Attack[MITRE ATT&CK Techniques]
        Discovery -->|T1580 - Cloud Infrastructure Discovery| MITRE
        CredAccess -->|T1552.005 - Cloud Instance Metadata API| MITRE
        CredAccess -->|T1528 - Steal Application Access Token| MITRE
        PrivEsc -->|T1548 - Abuse Elevation Control| MITRE
        Lateral -->|T1550 - Use Alternate Authentication Material| MITRE
        Exfil -->|T1530 - Data from Cloud Storage| MITRE
        Exfil -->|T1562.008 - Disable Cloud Logs| MITRE
    end
```

## Technique Mapping

| Kill Chain Stage | MITRE Technique | Application to SecureStack |
| --- | --- | --- |
| Discovery | T1580 Cloud Infrastructure Discovery | Enumerating what the compromised pod can see and reach |
| Credential Access | T1552.005 Cloud Instance Metadata API | Querying IMDS to steal node credentials (blocked by IMDSv2) |
| Credential Access | T1528 Steal Application Access Token | Reading the pod service-account token |
| Privilege Escalation | T1548 Abuse Elevation Control Mechanism | Attempting to act beyond the pod's scoped identity |
| Lateral Movement | T1550 Use Alternate Authentication Material | Using stolen credentials to move through the account |
| Exfiltration | T1530 Data from Cloud Storage | Reading patient data and secrets from AWS |
| Exfiltration | T1562.008 Disable Cloud Logs | Attempting to disable CloudTrail to cover tracks |
