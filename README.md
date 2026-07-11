# CloudEval AWS CloudFormation Review Example

Public reference repository for CloudEval pull-request review on AWS CloudFormation infrastructure as code.

This repo mirrors the Azure ARM review example, but uses AWS CloudFormation templates and AWS-specific scanners:

- AWS Guard reviewed rules for static Well-Architected signals where mappings exist.
- Checkov for supplemental IaC security and configuration findings.
- cfn-lint for CloudFormation deployment-quality validation.
- CloudEval canonical evidence, GitHub PR comments, PDF artifacts, and merge-gate status.

AWS CloudFormation support is currently beta. CloudEval reports unsupported AWS Well-Architected pillars as `Not assessed` instead of inventing a score, and this demo does not configure AWS retail cost gates.

## Repository layout

| Path | Purpose |
| --- | --- |
| `.cloudeval/config.yaml` | CloudEval stack selection, review outputs, and release-gate policy. |
| `.github/workflows/cloudeval-review.yml` | GitHub Actions workflow that runs CloudEval on every PR. |
| `templates/webapp.yaml` | Primary CloudFormation YAML template for topology, posture, and PR gating. |
| `templates/storage.json` | Secondary CloudFormation JSON template used to verify JSON parsing. |

## Required secrets

Create or import this repository as a CloudEval GitHub project, then add these repository secrets:

| Secret | Value |
| --- | --- |
| `CLOUDEVAL_ACCESS_KEY` | A CloudEval access key created from the GitHub Actions CI template. |
| `CLOUDEVAL_PROJECT_ID` | The CloudEval project id linked to this GitHub repository. |

The workflow intentionally skips review when either secret is missing so forks can run without exposing credentials.

## Demo PRs

Use these PRs to verify the complete gating behavior:

| PR | Expected result | What it demonstrates |
| --- | --- | --- |
| Passing baseline | Pass | CloudFormation YAML and JSON sync, posture review, PR comment, and artifacts. |
| Public access regression | Fail | High-risk S3, security group, and database exposure findings block the gate. |
| Deployment-quality regression | Fail | cfn-lint catches invalid CloudFormation references and blocks the gate. |

The passing baseline PR intentionally keeps infrastructure unchanged. Use it to verify that GitHub sync, PR-head commit review, comments, and artifacts work before tightening gates.

## Local smoke tests

Run these before pushing template changes:

```bash
cfn-lint templates/webapp.yaml templates/storage.json
checkov -f templates/webapp.yaml --framework cloudformation --quiet --skip-download
checkov -f templates/storage.json --framework cloudformation --quiet --skip-download
```

Checkov may report non-blocking supplemental findings. The release gate is controlled by `.cloudeval/config.yaml` and the CloudEval project policy.
