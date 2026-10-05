# Controls Required: Prompt Injection

The following controls mitigate the identified AI-security risks. Controls marked [IMPLEMENTED] exist in SecureStack today; those marked [IMPLEMENTED - PROVEN LIVE] were demonstrated on a running cluster; [NEXT STEP] / [OPERATIONAL] are planned or process controls.

## Runtime AI Guardrails (primary defence)

- [IMPLEMENTED - PROVEN LIVE] llm-guard sidecar scans every input for prompt injection and hidden text; a detected injection is blocked at HTTP 400 before the model is reached.
- [IMPLEMENTED] llm-guard scans every model response for sensitive-data leakage before it is returned to the user.
- [IMPLEMENTED] Fail-closed design: if the guard errors or is unreachable, the request is rejected, not passed unscanned.
- [IMPLEMENTED] The OpenAI API key is used to authenticate the call but is kept out of the prompt context, so it cannot be leaked by the model.
- [OPERATIONAL] Keep the guard's detection models updated to cover new jailbreak techniques.

## Build-Time AI Controls

- [IMPLEMENTED] Custom Semgrep rules flag injection-prone patterns (raw user input glued into a prompt) and dangerous output sinks (eval on model output); hard-fail the pipeline.
- [IMPLEMENTED] AI-BOM validation enforces data-classification ceilings: a model approved only for internal data is blocked when declared for PHI.
- [IMPLEMENTED] CLAUDE.md governance check enforces rules of engagement for AI coding agents.

## Input / Output Handling

- [IMPLEMENTED] Treat all model output as untrusted; never pass it to a dangerous sink without validation.
- [NEXT STEP] Enforce rate limiting and request-size limits on the triage endpoint to limit abuse and quota exhaustion.
- [NEXT STEP] Structured output constraints / schema validation on the model response.

## Detection and Response

- [IMPLEMENTED - PROVEN LIVE] Blocked-injection metrics (llm_guard_blocks_total) exposed to Prometheus and visible in Grafana; a spike is an indicator of active probing.
- [IMPLEMENTED] Application logs shipped to the OpenSearch SIEM for correlation.
- [IMPLEMENTED - PROVEN LIVE] Falco (eBPF) detects runtime anomalies such as a shell in a container, covering a coupled attack that reached code execution.
- [IMPLEMENTED - PROVEN LIVE] Kyverno admission control limits what can run; least-privilege IRSA limits blast radius.
- [NEXT STEP] Forward Falco alerts into the SIEM (Falcosidekick).
- [NEXT STEP] Alerting rule on an anomalous volume of blocked injections.

## Clinical Safety (operational)

- [OPERATIONAL] Treat AI triage output as advisory, not authoritative; maintain human-in-the-loop clinical oversight.
- [OPERATIONAL] Log and review misclassifications to monitor for manipulation or model drift.
