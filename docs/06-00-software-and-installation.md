# Acquire SAP software and run installation

## Outcome

You have downloaded the selected SAP software into the SAP library and run the
approved operating-system, database, and SAP installation stages.

## Before you begin

Complete [Deploy SAP-system infrastructure](05-00-sap-system.md). Confirm
network access to SAP download endpoints and target hosts. Set `S-Username` and
secret `S-Password` in `SDAF-General`.

The canonical SAP definitions and BOM files are owned by
[`Azure/SAP-automation-samples`](https://github.com/Azure/SAP-automation-samples).

## Inputs

- Control-plane environment and region.
- Approved application, database, kernel, or combined BOM name.
- SAP-system configuration name and workload environment.
- Database platform.
- Installation-stage selections.
- Optional Ansible extra parameters.

## Select a software-download wrapper

Two wrappers call the same core software-download template. Their public
support status is not confirmed, so do not infer a preferred option from the
filenames.

- `04-sap-software-download.yml` selects one combined BOM from its list, accepts
  an override name, and requires an explicit region.
- `04-sap-software-download_v2.yml` selects separate application, database, and
  kernel BOMs, constructs a combined BOM name, and derives the region from the
  environment name.

Use the wrapper identified by your reviewed release guidance. Verify that its
listed BOM values exist in the checked-out samples repository.

## What the automation does

Both `04` wrappers extend `resources_including_samples.yml`, which declares
`sap-automation` and `sap-samples`, and call
`deploy/pipelines/04-sap-software-download.yaml`. The core template checks out
the core, samples, and configuration repositories. It installs Terraform and
Ansible, prepares the selected BOM with SAP credentials, and runs the software
download. The download job has no pipeline timeout.

`05-DB-and-SAP-installation.yml` extends the core installation template. It can
run parameter validation, base OS configuration, SAP OS configuration, BOM
processing, database installation, central services, database load, high
availability, primary and additional application servers, Web Dispatcher,
post-configuration actions, quality checks, ACSS registration, and AMS
provider creation.

## Review before execution

SAP software licensing and media access remain your responsibility. Never put
SAP credentials in BOM or Terraform files. Review all installation booleans;
most stages default to `true`, while some optional post-installation stages
default to `false`.

## Download software

1. Confirm the selected BOM definitions exist in the samples repository ref
   used by Azure DevOps.
2. Confirm the wrapper can resolve the `sap-samples` checkout through its
   repository resources.
3. Queue the approved `04` wrapper with the control-plane environment, BOM
   selections, platform, and `re_download: false`.
4. Monitor **Preparation** and **Download software**. The preparation stage
   resolves the BOM, Key Vault, and SAP username for the download stage.
5. Verify the expected media is present in the SAP library storage and that the
   run contains no credential values.
6. Set `re_download: true` only when you deliberately need the core downloader
   to acquire the media again.

## Run installation

1. Queue `05-DB-and-SAP-installation.yml` with the exact SAP-system name,
   workload environment, and BOM.
2. Disable every stage that is outside the approved change. The selected
   booleans determine which Ansible playbooks run.
3. Review the parameter-validation result before later installation tasks
   continue.
4. Monitor each selected task and record the first failed play and host if the
   run stops.
5. Verify operating-system configuration, database services, SAP central
   services, application instances, and optional components selected for the
   run.

## SAP CAL wrapper

`07-sap-cal-installation.yml` references
`deploy/pipelines/07-sap-cal-installation.yaml`, which was not present in the
validated core checkout. Do not queue this wrapper until the `sap-automation`
repository ref used by Azure DevOps contains the template and your release has
validated the CAL path.

## If it fails

Do not rerun every installation stage by default. Correct the failed host,
credential, media, or playbook input. Then queue pipeline `05` with completed
stages disabled and only the required recovery stages enabled.

## Next step

[Operate and remove SDAF deployments](07-00-operations.md).
