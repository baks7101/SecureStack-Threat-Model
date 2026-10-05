# Risk Summary: Prompt Injection

Each risk is mapped to a mitigation that is actually implemented in SecureStack and, where noted, proven live. This is the residual-risk view: the inherent High risks reduced by the layered AI-security controls.

| Risk ID | Description | Severity | Likelihood | Impact | Mitigation (implemented in SecureStack) |
| --- | --- | --- | --- | --- | --- |
| R1 | Prompt injection overrides the model's instructions (LLM01) | High | High | High | llm-guard input scan detects prompt injection and blocks the request at HTTP 400 before the model is reached (proven live). Fail-closed. |
| R2 | The model leaks its system prompt, secrets, or patient data (LLM06) | High | Medium | High | llm-guard output scan checks the response for sensitive-data leakage before return; the OpenAI key is kept out of the prompt context so it cannot be leaked. |
| R3 | Insecure output handling leads to harmful action or code execution (LLM02) | High | Low | High | Custom Semgrep rule flags dangerous sinks (e.g. eval on model output) at build time and hard-fails the pipeline; output is never blindly executed. |
| R4 | An unapproved or unsafe model is used for sensitive data | High | Low | High | AI-BOM validation enforces data-classification ceilings: a model approved only for internal data is blocked when declared for PHI. Build fails. |
| R5 | Manipulated triage output misclassifies urgency and harms a patient | Medium | Medium | High | Guardrails constrain model behaviour; output scanning; clinical output is advisory. (Human-in-the-loop review is a recommended operational control.) |
| R6 | The guardrail fails open and silently stops protecting | High | Low | High | Fail-closed design: if the guard errors or is unreachable, the request is rejected, not passed unscanned (observed live during a cold-start). |
| R7 | Attacks go undetected, enabling repeated probing | Medium | Medium | Medium | Blocked-injection metrics (llm_guard_blocks_total) surfaced in Grafana; app logs in the OpenSearch SIEM; a volume spike is visible for investigation. |
| R8 | A coupled attack reaches runtime code execution | Medium | Low | High | Falco (eBPF) detects anomalous syscalls such as a shell in a container (proven live); Kyverno limits what can run; least-privilege IRSA limits blast radius. |
| R9 | Governance gap allows AI components to drift unmanaged | Medium | Low | Medium | CLAUDE.md governance check enforces rules for AI coding agents; AI-BOM maintains an approved-components inventory. |

## Residual Risk Statement

With SecureStack's layered AI-security controls applied, the dominant risk, a prompt injection reaching and manipulating the model, is reduced from High to Low-to-Medium, because the primary runtime guard blocks injections before the model, the key is kept out of prompt context, dangerous output sinks are caught at build time, and the guard is fail-closed. The highest residual exposure is a novel jailbreak that evades the input scanner's current detection; this is mitigated operationally by monitoring blocked-injection metrics for anomalies, keeping the guard's models updated, and treating clinical output as advisory with human oversight.
