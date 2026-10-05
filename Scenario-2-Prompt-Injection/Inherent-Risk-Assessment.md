# Inherent Risk Assessment: Prompt Injection

Inherent risk is assessed BEFORE SecureStack's controls are applied, to establish the baseline exposure a prompt-injection attack would present against an AI healthcare application with no guardrails.

| Category | Description | Likelihood | Impact | Risk Rating | Scenario |
| --- | --- | --- | --- | --- | --- |
| Prompt Injection (LLM01) | User input is passed to the LLM, which cannot separate instructions from data | High | High | High | Instruction Override |
| Information Disclosure (LLM06) | The model may reveal its system prompt, secrets, or other patients' data | High | High | High | Sensitive Data Leakage |
| Insecure Output Handling (LLM02) | Trusting the model's output can lead to harmful actions or code execution | Medium | High | High | Output Exploitation |
| Data Sensitivity | The application processes patient symptoms (PII / PHI) | High | High | High | Healthcare Data Exposure |
| Accessibility of Attack | The attack needs only normal API access, no exploit or credentials | High | Medium | High | Low Barrier to Entry |
| Clinical Safety | Manipulated triage output could misclassify urgency and cause patient harm | Medium | High | High | Harmful Clinical Output |
| Detection Difficulty | Malicious input looks like ordinary natural-language text, not an exploit | Medium | Medium | Medium | Stealthy Input |
| Compliance | Leakage of regulated healthcare data triggers regulatory exposure | Medium | High | High | Regulatory Exposure |

## Critical Assets

- The LLM system prompt and application configuration.
- The OpenAI API key and application secrets.
- Patient data (PII / PHI) processed by the triage API.
- The integrity and clinical safety of the triage output.
- The model's output stream (must be treated as untrusted).

## Summary

Before controls, prompt injection against this system is predominantly **High** risk. The drivers are unique to AI applications: the model's inability to separate instructions from data, the sensitivity of healthcare data, the very low barrier to entry (any user can attempt it through the normal interface), and the potential for manipulated clinical output to cause real-world harm. These inherent risks are what SecureStack's layered AI-security controls, the llm-guard, AI-BOM governance, and custom Semgrep rules, are designed to reduce.
