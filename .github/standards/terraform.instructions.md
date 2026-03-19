---
applyTo: "**/*.tf,**/*.tfvars"
---

# Terraform / HCL Standards

- **Formatter:** `terraform fmt` before committing.
- **Naming:** `snake_case` for all resource names, variables, outputs, locals. Module directories: `snake_case` or `kebab-case` (match existing convention).
- **Variables:** Always include `description` and `type`. Use `default` only when a sensible safe default exists.
- **Sensitive values:** Mark with `sensitive = true`. Never put secrets in `.tfvars` — use Secret Manager.
- **Outputs:** Provide for any value downstream modules or the console need.
- **File layout per module:** `main.tf`, `variables.tf`, `outputs.tf`. Large modules split into logical files (`iam.tf`, `networking.tf`).
- **Target dev first.** Never apply to prod without a successful dev deployment.
- **State:** Use GCS remote state.
- **Auth:** Impersonate service account with `gcloud auth application-default login`.
