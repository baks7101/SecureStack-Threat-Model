# MITRE ATT&CK Sequence Summary: CI/CD Pipeline Compromise

```mermaid
flowchart TD
    style Reconnaissance fill:#F4D03F,stroke:#000,stroke-width:2px
    style Weaponization fill:#F5B041,stroke:#000,stroke-width:2px
    style Delivery fill:#EB984E,stroke:#000,stroke-width:2px
    style Exploitation fill:#E59866,stroke:#000,stroke-width:2px
    style Installation fill:#DC7633,stroke:#000,stroke-width:2px
    style Command_Control fill:#CA6F1E,stroke:#000,stroke-width:2px
    style Actions_Objectives fill:#BA4A00,stroke:#000,stroke-width:2px
    style MITRE fill:#85C1E9,stroke:#000,stroke-width:2px

    Reconnaissance[Reconnaissance] -->|Enumerate repos, workflow, OIDC trust| Weaponization[Weaponization]
    Weaponization -->|Craft poisoned dependency / tampered artifact| Delivery[Delivery]
    Delivery -->|Merge via weak branch protection or stolen creds| Exploitation[Exploitation]
    Exploitation -->|Pipeline executes attacker-controlled steps| Installation[Installation]
    Installation -->|Deploy malicious workload to EKS via ECR| Command_Control[Command and Control]
    Command_Control -->|Establish covert outbound communication| Actions_Objectives[Actions on Objectives]
    Actions_Objectives -->|Exfiltrate secrets / PHI| Actions_Objectives
    Actions_Objectives -->|Tamper with triage logic / pivot in AWS| Actions_Objectives

    subgraph MITRE_Attack[MITRE ATT&CK Techniques]
        Reconnaissance -->|T1593 - Search Open Websites/Domains| MITRE
        Delivery -->|T1195.001 - Compromise Software Dependencies| MITRE
        Delivery -->|T1078 - Valid Accounts| MITRE
        Exploitation -->|T1059 - Command and Scripting Interpreter| MITRE
        Exploitation -->|T1552 - Unsecured Credentials| MITRE
        Installation -->|T1610 - Deploy Container| MITRE
        Command_Control -->|T1071 - Application Layer Protocol| MITRE
        Command_Control -->|T1105 - Ingress Tool Transfer| MITRE
        Actions_Objectives -->|T1552.007 - Container API / Cloud Secrets| MITRE
        Actions_Objectives -->|T1530 - Data from Cloud Storage| MITRE
    end
```

## Technique Mapping

| Kill Chain Stage | MITRE Technique | Application to SecureStack |
| --- | --- | --- |
| Reconnaissance | T1593 Search Open Websites/Domains | Reading the public app and platform repos to map the pipeline and OIDC trust |
| Delivery | T1195.001 Compromise Software Dependencies | Poisoning a dependency the app pulls in |
| Delivery | T1078 Valid Accounts | Using stolen developer or CI credentials to merge |
| Exploitation | T1059 Command and Scripting Interpreter | Running attacker commands inside the CI runner |
| Exploitation | T1552 Unsecured Credentials | Harvesting secrets or the OIDC role from the runner |
| Installation | T1610 Deploy Container | Deploying a tampered image as a trusted workload |
| Command and Control | T1071 / T1105 | Covert outbound comms and tool transfer from the workload |
| Actions on Objectives | T1552.007 / T1530 | Stealing cloud secrets and patient data from the environment |
