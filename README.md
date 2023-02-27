# Project

This repository contains the core pipelines for calling the SDAF Azure DevOps pipelines.

It will also contain the Terraform configuration files used for deploying the infrastructure required for the SAP Deployment Automation Framework deployments.

## How it works

During pipeline execution the two repositories will be present on the deployer-agent. Each of the repositories will be mapped using the following:

- sap-automation will be mapped to ```/sap-automation```
- customer configuration repository will be mapped to ```/config```

During execution the repositories will interact with each other by using the following:

```mermaid
flowchart LR
    subgraph deployer-agent
        pipeline--uses-->templated-pipelines
        templated-pipelines--uses-->workspace-configuration
        subgraph customer repository
            workspace-configuration
            pipeline
        end
        subgraph sap-automation
        templated-pipelines--executes-->sap-installation
        end
    end
```

## Trademarks

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft 
trademarks or logos is subject to and must follow 
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.
