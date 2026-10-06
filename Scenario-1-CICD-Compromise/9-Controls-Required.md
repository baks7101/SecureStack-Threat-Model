# Controls Required: CI/CD Pipeline Compromise

The following controls mitigate the identified risks. Controls marked [IMPLEMENTED] exist in SecureStack today; those marked [IMPLEMENTED - PROVEN LIVE] were demonstrated on a running cluster; [NEXT STEP] are planned hardening.

## Pipeline and Supply Chain

- [IMPLEMENTED] Enforce a 13-stage security pipeline with a merge-blocking gate on every pull request.
- [IMPLEMENTED] Secret scanning (Gitleaks) plus GitHub push protection at the git layer.
- [IMPLEMENTED] SAST with CodeQL (data-flow) and custom Semgrep rules (AI-specific patterns).
- [IMPLEMENTED] SCA (Trivy) failing on High/Critical, and a dependency pin-check enforcing exact versions.
- [IMPLEMENTED] SBOM generation (Syft) and vulnerability scanning (Grype) failing on High+.
- [IMPLEMENTED] Enforce protected branches and required status checks so the gate cannot be bypassed by a normal merge.
- [IMPLEMENTED] The platform runs a trimmed version of its own pipeline on itself (dogfooding).
- [NEXT STEP] Artifact signing and provenance verification (e.g. cosign / SLSA) before deployment.
- [NEXT STEP] Pin third-party GitHub Actions to commit SHAs rather than tags.

## Identity and Access

- [IMPLEMENTED] Keyless authentication to AWS via GitHub OIDC; no long-lived cloud keys exist.
- [IMPLEMENTED] The pipeline's assumed role is scoped read-only and short-lived.
- [IMPLEMENTED] The persistent CI identity role is isolated in a separate bootstrap Terraform stack.
- [IMPLEMENTED] Per-pod identity via IRSA; the External Secrets Operator role can read only two specific secrets.
- [IMPLEMENTED] IMDSv2 enforced with a hop limit so containers cannot steal node credentials.
- [NEXT STEP] Enforce MFA on all developer and maintainer accounts.

## Infrastructure and Policy

- [IMPLEMENTED] IaC scanning (Checkov) on Terraform and Kubernetes config.
- [IMPLEMENTED] Custom OPA/Rego policy evaluated against a real Terraform plan (e.g. deny SSH open to the internet).
- [IMPLEMENTED] Deny-by-default KMS encryption at rest; TLS in transit.
- [IMPLEMENTED] Remote Terraform state in an encrypted, versioned S3 bucket with DynamoDB locking.

## Deployment and Runtime

- [IMPLEMENTED] GitOps with ArgoCD; only git-declared manifests deploy, and drift is auto-reverted (proven live self-heal).
- [IMPLEMENTED - PROVEN LIVE] Kyverno admission control rejects privileged, root, and unapproved-registry pods.
- [IMPLEMENTED - PROVEN LIVE] Falco (eBPF) detects anomalous runtime behaviour such as a shell in a container.
- [IMPLEMENTED] Hardened pods: non-root, dropped capabilities, read-only root filesystem.

## Detection and Response

- [IMPLEMENTED - PROVEN LIVE] Centralised OpenSearch SIEM correlating CloudTrail, GuardDuty, and application logs.
- [IMPLEMENTED - PROVEN LIVE] SOAR Lambda auto-disables compromised credentials on a GuardDuty finding.
- [IMPLEMENTED] CloudTrail logging for S3, Lambda, IAM, and pipeline actions.
- [IMPLEMENTED] Prometheus / Grafana observability of security metrics.
- [IMPLEMENTED] Sigma detection rules authored and mapped to MITRE ATT&CK.
- [NEXT STEP] Forward Falco alerts into the SIEM (Falcosidekick).
