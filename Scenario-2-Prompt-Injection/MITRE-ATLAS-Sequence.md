# MITRE ATLAS Sequence Summary: Prompt Injection

For AI-specific attacks, the appropriate framework is MITRE ATLAS (Adversarial Threat Landscape for Artificial-Intelligence Systems), the AI-focused companion to MITRE ATT&CK. The kill-chain stages below are mapped to ATLAS tactics and techniques, with the equivalent OWASP LLM Top 10 reference.

```mermaid
flowchart TD
    style Reconnaissance fill:#F4D03F,stroke:#000,stroke-width:2px
    style Weaponization fill:#F5B041,stroke:#000,stroke-width:2px
    style Delivery fill:#EB984E,stroke:#000,stroke-width:2px
    style Exploitation fill:#E59866,stroke:#000,stroke-width:2px
    style Impact fill:#BA4A00,stroke:#000,stroke-width:2px
    style ATLAS fill:#85C1E9,stroke:#000,stroke-width:2px

    Reconnaissance[Reconnaissance] -->|Probe API and model behaviour| Weaponization[Weaponization]
    Weaponization -->|Craft injection / jailbreak payload| Delivery[Delivery]
    Delivery -->|Submit malicious input via symptoms field| Exploitation[Exploitation]
    Exploitation -->|Model obeys injected instructions| Impact[Impact]
    Impact -->|Leak system prompt / secrets / patient data| Impact
    Impact -->|Produce harmful clinical output| Impact

    subgraph ATLAS_Attack[MITRE ATLAS Techniques]
        Reconnaissance -->|AML.T0040 - ML Model Inference API Access| ATLAS
        Weaponization -->|AML.T0051 - LLM Prompt Injection| ATLAS
        Delivery -->|AML.T0054 - LLM Jailbreak| ATLAS
        Exploitation -->|AML.T0051.000 - Direct Prompt Injection| ATLAS
        Impact -->|AML.T0057 - LLM Data Leakage| ATLAS
        Impact -->|AML.T0048 - Societal / External Harm| ATLAS
    end
```

## Technique Mapping

| Kill Chain Stage | MITRE ATLAS Technique | OWASP LLM | Application to SecureStack |
| --- | --- | --- | --- |
| Reconnaissance | AML.T0040 ML Model Inference API Access | - | Probing the public triage API to learn model behaviour |
| Weaponization | AML.T0051 LLM Prompt Injection | LLM01 | Crafting instruction-override and jailbreak payloads |
| Delivery | AML.T0054 LLM Jailbreak | LLM01 | Submitting the payload through the symptoms field |
| Exploitation | AML.T0051.000 Direct Prompt Injection | LLM01 | The model obeys injected instructions (if ungated) |
| Impact | AML.T0057 LLM Data Leakage | LLM06 | Leaking the system prompt, secrets, or patient data |
| Impact | AML.T0048 External Harm | LLM02 | Manipulated clinical output causing patient harm |

## Note

Where a prompt injection is coupled with insecure output handling (for example, the model's output is passed to a dangerous sink such as eval), the chain extends beyond the AI layer into traditional ATT&CK techniques (T1059 Command and Scripting Interpreter), which is why insecure output handling is treated as a severe, coupled risk rather than a purely AI-layer concern.
