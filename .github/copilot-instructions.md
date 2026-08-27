# Copilot / Claude / Gemini instructions for sap-automation-bootstrap

This repository ships the `azure-sap-automation-devops` plugin — the Azure DevOps
**surface** node of the SDAF cross-agent skills graph. It is installed by an
operator only after the `Azure/sap-automation` hub plugin has directed them
here. It does **not** install the hub, and it does **not** duplicate any hub
skill.

## What the plugin ships

Two skills under `skills/` (real directories; no symlinks, no mirror script):

- `sdaf-ado-project-bootstrap` — action-loop skill that drives the documented
  `New-SDAFUserAssignedIdentity` + `New-SDAFADOProject` +
  `New-SDAFADOWorkloadZone` path from `docs/02-00-bootstrap.md`,
  `docs/02-10-configure-devops-project.md`, and
  `docs/02-20-configure-workload-zone-artifacts.md`.
- `sdaf-ado-pipeline-catalogue` — context-primer skill covering the 13 pipeline wrappers in `pipelines/`, their documented parameters and preconditions,
  the variable groups they read, and the documented `20`/`21`/`22` level-up
  path (`docs/pipeline-reference.md`, `docs/07-00-operations.md`).

## Ground rules

1. **Documented-only.** Skills teach only what this repository's `docs/`
   already documents. Undocumented behaviour reconstructed from source is out
   of scope — flag the gap, do not invent the procedure.
2. **No production code changes.** Skills describe and drive existing SDAF
   behaviour; they never modify pipelines, PowerShell utilities, playbooks, or
   Terraform.
3. **Two skills, disjoint triggers.** "Set up SDAF on ADO" / "onboard a
   workload zone" → `sdaf-ado-project-bootstrap`. "What does pipeline N do"
   / "which variable group does N read" / "level up" → catalogue.
4. **This plugin is a surface node.** It does not orient the operator across
   surfaces; that is the hub plugin's job. When an operator has not decided
   between Azure DevOps and GitHub Actions, refer them back to the hub.
5. **Never install another plugin from these skills.** There is no runtime
   dependency and no auto-install machinery in this repo.

## Install (all three runtimes)

See `docs/PLUGINS.md`.