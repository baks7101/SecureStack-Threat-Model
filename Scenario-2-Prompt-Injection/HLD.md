# High-Level Design: SecureStack (Prompt Injection view)

```mermaid
flowchart TD
  subgraph External
    User[Healthcare User]
    Attacker[Attacker - malicious input]
  end

  subgraph "Build-Time Controls (pipeline)"
    Semgrep[Custom Semgrep AI Rules]
    AIBOM[AI-BOM Validation]
    ClaudeMD[CLAUDE.md Governance]
  end

  subgraph "EKS Cluster - MediTriage Pod"
    API[API Container]
    GuardIn[llm-guard: Input Scan]
    GuardOut[llm-guard: Output Scan]
  end

  subgraph "AWS / External"
    Model[OpenAI gpt-3.5-turbo]
    ESO[External Secrets Operator]
    SM[Secrets Manager + KMS]
    Prom[Prometheus]
    Graf[Grafana]
    OS[OpenSearch SIEM]
    Falco[Falco Runtime Detection]
  end

  User --> API
  Attacker --> API
  API --> GuardIn
  GuardIn -->|clean| Model
  GuardIn -->|threat| API
  Model --> GuardOut
  GuardOut -->|clean| API
  GuardOut -->|leak| API
  API --> ESO
  ESO --> SM
  API --> Prom
  Prom --> Graf
  API --> OS
  Falco -.watches.-> API

  Semgrep -.defends build time.-> API
  AIBOM -.governs model.-> Model
  ClaudeMD -.governs AI agents.-> API
```

## Defence in Depth for Prompt Injection

Prompt injection is defended at all three layers, so a single bypass does not grant the objective:

- **Build time**: custom Semgrep rules flag injection-prone code patterns (for example, gluing raw user input into a prompt, or calling eval on model output). The AI-BOM validator governs which models are even permitted and for what data classification. The CLAUDE.md check governs AI coding agents.
- **Runtime (primary)**: the llm-guard sidecar scans input for injection before the model sees it (blocking at HTTP 400), and scans output for sensitive-data leakage before returning it. The guard is fail-closed.
- **Detection and response**: blocked-injection metrics are visible in Grafana; Falco watches for any runtime anomaly if a coupled attack reached code execution; the SIEM correlates activity.

## Key Design Properties

- The model cannot distinguish trusted instructions from untrusted user data, so input must be treated as hostile and scanned.
- The OpenAI key is used to authenticate the API call but is not placed in the prompt context, limiting what a successful injection can leak.
- Fail-closed design: a broken or unavailable guard blocks requests rather than silently disabling the control.
