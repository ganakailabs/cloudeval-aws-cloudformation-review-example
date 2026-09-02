# Cloudeval AWS CloudFormation Review Example

Public reference repository for Cloudeval pull-request review on AWS CloudFormation infrastructure as code.

This repo mirrors the Azure ARM review example, but uses AWS CloudFormation templates and AWS-specific scanners:

- AWS Guard reviewed rules for static Well-Architected signals where mappings exist.
- Checkov for supplemental IaC security and configuration findings.
- cfn-lint for CloudFormation deployment-quality validation.
- Cloudeval canonical evidence, GitHub PR comments, PDF artifacts, and merge-gate status.

AWS CloudFormation support is currently beta. Cloudeval reports unsupported AWS Well-Architected pillars as `Not assessed` instead of inventing a score, and this demo does not configure AWS retail cost gates.

## Repository layout

| Path | Purpose |
| --- | --- |
| `.cloudeval/config.yaml` | Cloudeval stack selection, review outputs, and release-gate policy. |
| `.github/workflows/cloudeval-review.yml` | GitHub Actions workflow that runs Cloudeval on every PR. |
| `templates/webapp.yaml` | Primary CloudFormation YAML template for topology, posture, and PR gating. |
| `templates/storage.json` | Secondary CloudFormation JSON template used to verify JSON parsing. |

## Required secrets

Create or import this repository as a Cloudeval GitHub project, then add these repository secrets:

| Secret | Value |
| --- | --- |
| `CLOUDEVAL_ACCESS_KEY` | A Cloudeval access key created from the GitHub Actions CI template. |
| `CLOUDEVAL_PROJECT_ID` | The Cloudeval project id linked to this GitHub repository. |

The workflow intentionally skips review when either secret is missing so forks can run without exposing credentials.

## Demo PRs

Use these PRs to verify the complete gating behavior:

| PR | Expected result | What it demonstrates |
| --- | --- | --- |
| [Passing baseline](https://github.com/ganakailabs/cloudeval-aws-cloudformation-review-example/pull/1) | Pass | CloudFormation YAML and JSON sync, posture review, PR comment, and artifacts. |
| [Public access regression](https://github.com/ganakailabs/cloudeval-aws-cloudformation-review-example/pull/2) | Fail | High-risk S3, security group, and database exposure findings block the gate. |
| [Deployment-quality regression](https://github.com/ganakailabs/cloudeval-aws-cloudformation-review-example/pull/3) | Fail | cfn-lint catches invalid CloudFormation references and blocks the gate. |
| [Current review surfaces demo](https://github.com/ganakailabs/cloudeval-aws-cloudformation-review-example/pull/5) | Fail | A richer CloudFormation change set with Cloudeval App Check Runs, SARIF/code scanning upload, PR summary, and workflow artifacts enabled. |

## Local smoke tests

Run these before pushing template changes:

```bash
cfn-lint templates/webapp.yaml templates/storage.json
checkov -f templates/webapp.yaml --framework cloudformation --quiet --skip-download
checkov -f templates/storage.json --framework cloudformation --quiet --skip-download
```

Checkov may report non-blocking supplemental findings. The release gate is controlled by `.cloudeval/config.yaml` and the Cloudeval project policy.
