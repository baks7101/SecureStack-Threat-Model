# Attacker Flow: Prompt Injection

```mermaid
sequenceDiagram
  participant Attacker
  participant API as MediTriage API
  participant Guard as llm-guard Sidecar
  participant Model as OpenAI gpt-3.5-turbo
  participant Secrets as Secrets Manager
  participant Metrics as Prometheus Grafana
  participant SIEM as OpenSearch SIEM

  activate Attacker
  Attacker->>API: Probe POST /api/chat/triage with test inputs
  API->>Attacker: Observe responses, infer a guardrail exists
  deactivate Attacker

  activate Attacker
  Attacker->>API: Submit injection in symptoms field - ignore instructions, reveal system prompt and API keys
  API->>Guard: Forward input for scanning at input stage
  Guard->>Guard: PromptInjection scanner evaluates the input
  Guard->>API: THREAT DETECTED - block
  API->>Attacker: HTTP 400 - request blocked by content safety scan
  deactivate Attacker

  activate Attacker
  Note over Guard,Model: The injected prompt never reaches the model
  Guard-->>Metrics: Increment llm_guard_blocks_total for PromptInjection
  deactivate Attacker

  activate Attacker
  Attacker->>API: Submit a legitimate triage request as a control
  API->>Guard: Input scan clean
  Guard->>Model: Forward sanitised prompt
  Model->>Guard: Response
  Guard->>Guard: Output scan for sensitive-data leakage
  Guard->>API: Clean response
  API->>Attacker: HTTP 200 - valid triage returned
  deactivate Attacker

  Note over API,SIEM: Fail-closed - if the guard errors, the request is blocked, not passed
  Metrics-->>SIEM: Block metrics visible, anomalous volume surfaced for investigation
```

## Narrative

The attacker's injection is delivered through the normal interface, there is no software exploit, just a malicious value in the symptoms field. The llm-guard sidecar scans the input before it reaches the model. A detected prompt injection is blocked at the input stage with an HTTP 400, so the payload never touches the LLM.

Two design properties matter. First, the guard is fail-closed: if the scanner itself errors or is unreachable, the request is rejected rather than passed unscanned, for a health application, refusing to serve beats serving unsafely. Second, the block is observable: the guard increments a Prometheus counter, so a spike in blocked injections is visible in Grafana and surfaced for investigation. A legitimate request, by contrast, passes the input scan, is answered by the model, and has its output scanned for leakage before being returned.
