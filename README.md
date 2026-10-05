# SecureStack Threat Model

A threat-modelling exercise for the SecureStack platform and its showcase application, ai-vibecode-lab (MediTriage, an AI patient-triage API). This repository documents the attack scenarios, risk assessments, and the controls that mitigate them, mapped to the controls actually implemented and, where noted, proven live on AWS.

## Introduction

This threat model details runbook scenarios of attacks against SecureStack, a DevSecOps platform securing an AI healthcare application across the full lifecycle (build time, deploy time, and runtime). It assesses both a traditional software-supply-chain attack and an AI-specific attack, reflecting that modern applications must defend against both classic and emerging, AI-native threats.

## Scope

Two scenarios are modelled:

1. CI/CD Pipeline Compromise - a software-supply-chain attack that attempts to inject malicious code or configuration through the pipeline and have it deployed as a trusted workload.
2. Prompt Injection - an AI-specific attack that abuses the application's normal input to manipulate the Large Language Model behind the triage API.

## Methodology

Each scenario is assessed against the cyber kill chain, mapped to MITRE ATT&CK (for the supply-chain scenario) and MITRE ATLAS (for the AI scenario), with STRIDE used for control-gap analysis. Each identified risk is mapped to a mitigation that exists in SecureStack, distinguishing controls that are implemented, proven live, or planned as next steps. This keeps the model honest: it documents the real security posture, not an aspirational one.

## System Under Assessment

- ai-vibecode-lab (MediTriage) - an AI patient-triage API (API container + llm-guard sidecar) on AWS EKS, owning its own infrastructure.
- SecureStack-platform - the reusable 13-stage security pipeline, policies, AI-BOM validator, and governance consumed by the app via workflow_call.
- Supporting AWS services: ECR, ArgoCD (GitOps), Secrets Manager + KMS, External Secrets Operator (IRSA), OpenSearch SIEM, CloudTrail, GuardDuty, a SOAR Lambda, Prometheus/Grafana, Kyverno, and Falco.

## Conclusion

Across both scenarios, the inherent (pre-control) risk is predominantly High, driven by the sensitivity of healthcare data, the ability of a supply-chain compromise to act in the cloud account, and the AI-native risk that a model cannot distinguish instructions from data. With SecureStack's layered, defence-in-depth controls applied, the dominant risk paths are reduced to Low-to-Medium, because an attacker must defeat multiple independent controls rather than any single one.

## Overall Risk Overview

| Scenario | Primary Threat | Inherent Risk | Key Controls | Residual Risk |
| --- | --- | --- | --- | --- |
| 1. CI/CD Pipeline Compromise | Malicious code deployed via the supply chain | High | 13-stage gated pipeline, keyless OIDC, GitOps, IRSA, Kyverno, SIEM/SOAR | Low-Medium |
| 2. Prompt Injection | Model manipulation via adversarial input | High | llm-guard (fail-closed), AI-BOM ceilings, custom Semgrep rules, output scanning | Low-Medium |

## Top Controls Required (consolidated)

- A merge-blocking 13-stage security pipeline (secret scanning, SAST, SCA, SBOM, IaC, policy, AI-BOM, DAST).
- Keyless OIDC authentication with least-privilege, short-lived roles; IRSA per-pod identity; IMDSv2.
- GitOps delivery (ArgoCD) so only reviewed, git-declared manifests deploy.
- Runtime AI guardrails (llm-guard, fail-closed) blocking prompt injection and scanning output.
- AI governance: AI-BOM data-classification ceilings and CLAUDE.md agent governance.
- Admission control (Kyverno) and runtime detection (Falco).
- Centralised SIEM (OpenSearch correlating CloudTrail, GuardDuty, app logs) with automated response (SOAR).

## Threat Modelling Process

```mermaid
mindmap
  root((SecureStack Threat Model))
    Methodology
      Kill Chain
      MITRE ATTACK
      MITRE ATLAS
      STRIDE
    Per Scenario
      Attacker Flow
      Data Flow Diagram
      HLD
      Inherent Risk
      Sequence
      STRIDE
      Risk Summary
      Controls Required
    Scenarios
      CICD Pipeline Compromise
      Prompt Injection
    Outcome
      Risks mapped to real controls
      Implemented or Proven live or Next step
```

## Repository Structure

Scenario 1 (CICD-Compromise) and Scenario 2 (Prompt-Injection) each contain nine files: a README (kill chain description), Attacker-Flow, Data-Flow-Diagram, HLD, Inherent-Risk-Assessment, a MITRE sequence (ATT&CK for Scenario 1, ATLAS for Scenario 2), STRIDE, Risk-Summary, and Controls-Required.

## Related Repositories

- SecureStack-platform - the security platform (pipeline, policies, governance): https://github.com/baks7101/SecureStack-platform
- ai-vibecode-lab - the AI application being protected: https://github.com/baks7101/ai-vibecode-lab
