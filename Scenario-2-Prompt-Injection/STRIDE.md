# STRIDE Analysis: Prompt Injection

```mermaid
graph TD
  subgraph Interaction["User Interaction"]
    U[Healthcare User]
    A[Attacker]
  end

  subgraph Pod["MediTriage Pod"]
    API["API Container"]
    GIN["llm-guard: Input Scan"]
    GOUT["llm-guard: Output Scan"]
  end

  subgraph Ext["Model and Services"]
    Model["OpenAI gpt-3.5-turbo"]
    ESO["External Secrets Operator"]
    SM[("Secrets Manager + KMS")]
    OS[("OpenSearch SIEM")]
    Prom["Prometheus / Grafana"]
  end

  U --> API
  A --> API
  API --> GIN
  GIN -->|clean| Model
  Model --> GOUT
  GOUT --> API
  API --> ESO
  ESO --> SM
  API -.-> OS
  API -.-> Prom

  T1["Spoofing - impersonate a legitimate user / prompt"] -.-> API
  T2["Tampering - manipulate the model via injected instructions"] -.-> Model
  T3["Repudiation - deny sending malicious input"] -.-> API
  T4["Information Disclosure - leak system prompt / secrets / PHI"] -.-> Model
  T5["Denial of Service - flood with heavy prompts; exhaust model quota"] -.-> Model
  T6["Elevation of Privilege - code execution via insecure output handling"] -.-> API

  M1["Input validation; guard scans every request; rate limiting"] --> T1
  M2["llm-guard input scan blocks injection before the model (fail-closed)"] --> T2
  M3["App logs + CloudTrail in SIEM; blocked-injection metrics"] --> T3
  M4["Output scan for leakage; key kept out of prompt context; data-classification ceilings"] --> T4
  M5["Rate limiting; request size limits; model quota controls"] --> T5
  M6["Semgrep blocks dangerous output sinks at build time; Falco + Kyverno at runtime"] --> T6
```

## STRIDE Breakdown

| STRIDE Category | Threat | SecureStack Mitigation |
| --- | --- | --- |
| **S**poofing | Attacker submits malicious input disguised as a legitimate patient request | Every request is scanned by the guard regardless of source; input validation; rate limiting on the endpoint |
| **T**ampering | Attacker manipulates the model's behaviour via injected instructions | llm-guard input scan detects and blocks prompt injection before the model (proven live); fail-closed |
| **R**epudiation | Attacker denies submitting the malicious input | Application logs and CloudTrail feed the OpenSearch SIEM; blocked-injection metrics provide an audit trail |
| **I**nformation Disclosure | Model leaks its system prompt, the API key, or patient data | Output scan checks for leakage; the OpenAI key is never placed in the prompt context; AI-BOM data-classification ceilings |
| **D**enial of Service | Attacker floods the API with heavy prompts to exhaust model quota or availability | Rate limiting, request-size limits, and model quota controls (operational hardening) |
| **E**levation of Privilege | Insecure output handling (e.g. eval of model output) leads to code execution | Custom Semgrep rule blocks dangerous sinks at build time; Falco and Kyverno constrain runtime; least-privilege IRSA |
