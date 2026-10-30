# Central Terraform GitHub Actions templates

Reusable workflows shared by Terraform module repositories and the platform-infrastructure deployment repository.

## Repository layout

```text
.github/workflows/
├── terraform-validate.yml
├── terraform-security.yml
├── terraform-plan.yml
├── terraform-apply.yml
└── terraform-release-nexus.yml
```

## Workflow responsibilities

| Workflow | Responsibility |
|---|---|
| terraform-validate.yml | Terraform fmt, init without backend, and validate |
| terraform-security.yml | Checkov infrastructure-as-code security scan |
| terraform-plan.yml | Initialize and create a Terraform execution plan |
| terraform-apply.yml | Apply Terraform with an optional approval/environment gate |
| terraform-release-nexus.yml | Validate, package, checksum, and publish a module to Nexus Raw |

## Caller examples

Validation and security for a reusable module repository:

```yaml
jobs:
  validate:
    uses: manojrasia/terraform-github-actions-templates/.github/workflows/terraform-validate.yml@master
    with:
      terraform_version: "1.9.8"
      working_directory: "."

  security:
    uses: manojrasia/terraform-github-actions-templates/.github/workflows/terraform-security.yml@master
    with:
      working_directory: "."
      soft_fail: false
```

Release to Nexus:

```yaml
jobs:
  release:
    uses: manojrasia/terraform-github-actions-templates/.github/workflows/terraform-release-nexus.yml@master
    with:
      module_name: terraform-google-gcs
      terraform_version: "1.9.8"
      working_directory: "."
      nexus_repository: ${{ vars.NEXUS_RAW_REPOSITORY }}
    secrets:
      NEXUS_URL: ${{ secrets.NEXUS_URL }}
      NEXUS_USERNAME: ${{ secrets.NEXUS_USERNAME }}
      NEXUS_PASSWORD: ${{ secrets.NEXUS_PASSWORD }}
```

## Security and platform setup

- Set repository Actions access so approved caller repositories can use these reusable workflows.
- Configure Nexus credentials as caller-repository secrets; do not hardcode credentials.
- The release workflow requires a self-hosted Linux runner labelled `self-hosted`, `linux`, and `nexus-local`, with `zip`, `curl`, and network access to Nexus.
- Configure the `production` GitHub Environment and required reviewers in the platform deployment repository before using the apply workflow.
- Configure Google Cloud Workload Identity Federation in the caller workflow and pass only short-lived identity configuration. Avoid service-account key JSON.
- Pin production callers to a protected version tag or immutable commit SHA instead of `@master`.

## Important usage boundary

Use validate/security/release from module repositories. Use plan/apply from `platform-infrastructure`, where root Terraform configurations, backend state, environment-specific variables, and cloud authentication are defined. A module repository by itself generally does not contain enough input values to plan or apply infrastructure.
