# `azure-sap-automation-devops` — operator guide

## Purpose

`azure-sap-automation-devops` is the plugin shipped by this repository. It
installs two grounded AI skills into your CLI so that GitHub Copilot, Claude
Code, or Gemini CLI can help you set up and operate SDAF on **Azure DevOps**
using only what this repository actually documents.

It is the **Azure DevOps platform-specific plugin** in the SDAF AI-skills
family. A separate **hub plugin** in
[`Azure/sap-automation`](https://github.com/Azure/sap-automation) covers
framework-level, platform-independent SDAF content.

> **All SDAF AI plugins are optional and independently installable.** For
> complete Azure DevOps coverage, install the hub plugin from
> `Azure/sap-automation` alongside this plugin, following that repository's
> own README. **This plugin does not install the hub** and does not
> duplicate any hub skill. Installing this plugin on its own works; you
> simply won't have the hub's framework-level content loaded at the same
> time.
>
> Recommended pairings for complete platform coverage:
>
> - **Local execution** — hub plugin only.
> - **Azure DevOps** — hub plugin + this plugin.
> - **GitHub Actions** — hub plugin +
>   [`azure-sap-automation-github`](https://github.com/Azure/sap-automation-gh-bootstrap)
>   from [`Azure/sap-automation-gh-bootstrap`](https://github.com/Azure/sap-automation-gh-bootstrap).

## What the plugin ships

Two skills, both real directories under [`skills/`](../skills):

- **[`sdaf-ado-project-bootstrap`](../skills/sdaf-ado-project-bootstrap/SKILL.md)** —
  drives the documented `New-SDAFUserAssignedIdentity` +
  `New-SDAFADOProject` + `New-SDAFADOWorkloadZone` PowerShell path from
  [`docs/02-00-bootstrap.md`](02-00-bootstrap.md),
  [`docs/02-10-configure-devops-project.md`](02-10-configure-devops-project.md),
  and [`docs/02-20-configure-workload-zone-artifacts.md`](02-20-configure-workload-zone-artifacts.md).
  **Use it when** you are standing up the Azure DevOps project for the first
  time, or adding another workload zone (variable group + service connection)
  to an existing SDAF project.
- **[`sdaf-ado-pipeline-catalogue`](../skills/sdaf-ado-pipeline-catalogue/SKILL.md)** —
  context primer over [`docs/pipeline-reference.md`](pipeline-reference.md)
  and [`docs/07-00-operations.md`](07-00-operations.md). **Use it when** you
  need to know what a specific wrapper pipeline does, what parameters it
  takes, which variable groups it reads, or how the `20` / `21` / `22`
  level-up path works. It does not restate the catalogue here — the skill
  reads the source docs directly.

### How to pick between the two skills

In one sentence: **`sdaf-ado-project-bootstrap` creates the Azure DevOps
scaffolding you need before you can run any pipeline; `sdaf-ado-pipeline-catalogue`
explains the pipelines you run afterwards.**

- If the question is *"how do I get an Azure DevOps project ready for SDAF?"*
  or *"how do I onboard a new workload zone?"* — that is
  `sdaf-ado-project-bootstrap`. It walks the PowerShell utilities that create
  the project, repos, pipelines, cleanup task, agent pool, managed identity,
  service connections, variable groups, and (optionally) the Web App.
- If the question is *"what does pipeline `03` do?"*, *"which variable group
  does `05` read?"*, *"what parameters does the workload-zone pipeline take?"*,
  or *"what is `20` vs `21` vs `22`?"* — that is
  `sdaf-ado-pipeline-catalogue`. It never runs a pipeline for you; it
  explains them.

The two skills have **disjoint triggers by design**. The bootstrap skill owns
"create / add / onboard"; the catalogue skill owns "what / which / when to
run".

## Install

Prerequisite: the CLI you plan to use is already installed and signed in
(Copilot CLI, Claude Code, or Gemini CLI). Then pick the section below.

### GitHub Copilot CLI

```bash
copilot plugin marketplace add Azure/sap-automation-bootstrap
copilot plugin install azure-sap-automation-devops@sap-automation-bootstrap
```

### Claude Code

Run inside a Claude Code session:

```text
/plugin marketplace add Azure/sap-automation-bootstrap
/plugin install azure-sap-automation-devops@sap-automation-bootstrap
```

### Gemini CLI

```bash
gemini extensions install https://github.com/Azure/sap-automation-bootstrap
```

Use `gemini extensions install`, not `gemini skills install`. The extension
reads the root [`gemini-extension.json`](../gemini-extension.json) and
auto-loads the root [`skills/`](../skills) directory.

## Verify the install

You should see exactly two skills registered:
`sdaf-ado-project-bootstrap` and `sdaf-ado-pipeline-catalogue`.

- **Copilot CLI:** `copilot plugin list` should show
  `azure-sap-automation-devops` as installed.
- **Claude Code:** open the `/plugin` menu; the plugin should appear as
  installed and both skill names should be visible under it.
- **Gemini CLI:** `gemini extensions list` should show
  `azure-sap-automation-devops`.

If your CLI version's subcommand names differ, run `--help` on its plugin or
extension command and use the equivalent.

### Smoke-test with a prompt

Ask your agent:

> *"Which SDAF skills are loaded, and when should I use each?"*

A working install answers with the two skill names above and the
pick-between rule from the previous section.

## Example prompts

For `sdaf-ado-project-bootstrap`:

- *"Set up SDAF on Azure DevOps for a new control plane."*
- *"Bootstrap the SDAF Azure DevOps project — walk me through
  `New-SDAFADOProject`."*
- *"Onboard a new workload zone: create the `SDAF-<env>` variable group and
  workload service connection."*
- *"Create the control-plane managed identity with
  `New-SDAFUserAssignedIdentity`."*

For `sdaf-ado-pipeline-catalogue`:

- *"What does pipeline `01-deploy-control-plane.yml` do, and what variable
  groups does it read?"*
- *"What parameters does `02-sap-workload-zone.yml` take, and how does
  `test: true` behave?"*
- *"Explain `10` vs `11` vs `12` for removal."*
- *"How do I level up my ADO pipelines? What is `20` vs `21` vs `22`?"*
- *"Which pipeline downloads SAP software — and how do I choose between
  `04-sap-software-download.yml` and `04-sap-software-download_v2.yml`?"*
  (The skill surfaces this as a documented gap rather than invent a
  selection rule.)

## Capability ownership across the SDAF AI-skills family

Ownership is disjoint by design so nothing is duplicated:

- **Hub plugin** ([`Azure/sap-automation`](https://github.com/Azure/sap-automation)) —
  framework-level, platform-independent SDAF content.
- **This plugin** — Azure DevOps project bootstrap and the wrapper pipelines
  shipped in this repository.
- **[`azure-sap-automation-github`](https://github.com/Azure/sap-automation-gh-bootstrap)** —
  the GitHub Actions execution path.

Consistent with [`README.md`](../README.md) *Current capability boundaries*,
Azure DevOps and GitHub Actions implement the same SDAF lifecycle but their
automation is **not identical**. There is no verified full Azure DevOps
equivalent for GitHub configuration workflows `02` and `04`, and
`07-sap-cal-installation.yml` references a core template that was not
present in the validated `sap-automation` checkout. The skills reflect those
boundaries and will not invent equivalents.

## The 13 pipeline wrappers

The catalogue skill is scoped to the **13** wrapper pipelines in
[`pipelines/`](../pipelines):

`01`, `02`, `03`, `04`, `04_v2`, `05`, `07`, `10`, `11`, `12`, `20`, `21`,
`22`.

The two `pipelines/resources*.yml` files are repository-resource declarations
(they wire in `sap-automation` and, where applicable, `sap-samples`), not
wrappers, and are not counted.

**This document does not restate the per-pipeline catalogue.** Content is
owned by [`docs/pipeline-reference.md`](pipeline-reference.md) and
[`docs/07-00-operations.md`](07-00-operations.md); the catalogue skill reads
those files directly so a single source stays authoritative.

## Update and uninstall

Update and uninstall subcommand names vary across CLI versions and this
repository does not ship its own update tooling. Use each runtime's own
plugin (or extension) manager:

- **GitHub Copilot CLI** — run `copilot plugin --help` and use the update /
  uninstall subcommand your CLI version exposes for the
  `azure-sap-automation-devops` plugin.
- **Claude Code** — open the interactive `/plugin` menu inside a session.
  It lists installed plugins under the `sap-automation-bootstrap`
  marketplace and exposes the actions your Claude Code version supports.
- **Gemini CLI** — run `gemini extensions --help` and use the update /
  uninstall subcommand your CLI version exposes for the
  `azure-sap-automation-devops` extension.

If your CLI does not expose an update path, uninstall (via its plugin
manager) and then reinstall using the verified commands in the **Install**
section above.

## Troubleshooting

**"Marketplace or extension not found."** Confirm the source is spelled
`Azure/sap-automation-bootstrap` (Copilot / Claude) or the full URL
`https://github.com/Azure/sap-automation-bootstrap` (Gemini). GitHub
authentication for the CLI must be in place for private-network mirrors.

**"Plugin installed but no skills load."** Both skills live in the root
[`skills/`](../skills) directory as real folders (no symlinks, no mirror
script). Confirm your CLI resolved this repository's root, not a
sub-directory. For Gemini, the loader is driven by the root
[`gemini-extension.json`](../gemini-extension.json); for Claude, by
[`.claude-plugin/plugin.json`](../.claude-plugin/plugin.json) and
[`.claude-plugin/marketplace.json`](../.claude-plugin/marketplace.json).

**"Only one skill triggers."** Trigger phrasing is disjoint by design. Ask
the agent *"which SDAF skill fits my question?"* to route explicitly, or
use the pick-between rule above.

**"The skill says the answer isn't in the docs."** By ground rule, both
skills teach only what this repository's `docs/` documents. Undocumented
behaviour — for example `04` vs `04_v2` selection criteria, `07` core
template availability, exhaustive per-variable catalogues — is surfaced as
a **gap**, not reconstructed from source. Treat it as a prompt to review
[`README.md`](../README.md) *Current capability boundaries* or file an
issue upstream.

**"I need to change how a pipeline behaves."** Skills describe and drive
existing SDAF behaviour; they do not modify pipelines, PowerShell utilities,
playbooks, or Terraform. Edit the pipeline file directly (or file the fix
upstream in `Azure/sap-automation` for core-template defects); the skills
pick up your edits on next invocation.

**"Which platform should I use — Azure DevOps or GitHub Actions?"**
That decision is framework-level. The hub plugin at
[`Azure/sap-automation`](https://github.com/Azure/sap-automation) covers it.
This plugin covers the Azure DevOps side only.