# Azure Virtual Machine Deployment via ARM Template

## Overview

This project demonstrates Infrastructure as Code (IaC) on Microsoft Azure using Azure Resource Manager (ARM) templates. Rather than provisioning resources manually through the Azure Portal, this project defines a Virtual Machine and its supporting network infrastructure declaratively in JSON, enabling consistent, repeatable, and automated deployments.

## Project Goals

- Understand the structure of ARM templates: parameters, variables, resources, and outputs.
- Apply declarative infrastructure principles for consistent, repeatable deployments.
- Parameterize templates for environment reproducibility and scalability.
- Use the Azure CLI for cloud resource management and deployment.
- Troubleshoot deployment validation errors and manage resource dependencies.
- Implement security best practices, including Network Security Groups (NSGs) and secure authentication.

## Architecture

The template provisions the following resources:

| Resource | Purpose |
|---|---|
| Virtual Network (VNet) | Isolated network for the VM |
| Subnet | Address space within the VNet |
| Public IP Address | External connectivity to the VM |
| Network Security Group (NSG) | Controls inbound/outbound traffic (e.g., SSH/RDP) |
| Network Interface (NIC) | Connects the VM to the subnet and public IP |
| Virtual Machine | The compute resource (OS, size, disk, credentials) |

## Repository Structure

```
.
├── azuredeploy.json       # Main ARM template
├── azuredeploy.parameters.json  # Parameter values for deployment
├── screenshots/           # Deployment and connectivity verification proof
├── outputs.log            # Deployment output log (Public IP, Resource IDs)
└── README.md
```

## Prerequisites

- An active Azure subscription
- [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli) installed
- Logged in via `az login`

## Deployment Steps

### 1. Create a Resource Group

```bash
az group create \
  --name myResourceGroup \
  --location eastus
```

### 2. Validate the Template

```bash
az deployment group validate \
  --resource-group myResourceGroup \
  --template-file azuredeploy.json \
  --parameters azuredeploy.parameters.json
```

### 3. Deploy the Template

```bash
az deployment group create \
  --name vmDeployment \
  --resource-group myResourceGroup \
  --template-file azuredeploy.json \
  --parameters azuredeploy.parameters.json
```

### 4. Retrieve Deployment Outputs

```bash
az deployment group show \
  --resource-group myResourceGroup \
  --name vmDeployment \
  --query properties.outputs
```

### 5. Connect to the VM

```bash
ssh <adminUsername>@<publicIPAddress>
```

## Key ARM Template Concepts Applied

- **Parameters** – Dynamic inputs such as VM name, admin username, authentication type, and location, avoiding hardcoded values.
- **Variables** – Computed or reused values (e.g., resource naming conventions) derived from parameters.
- **Resources** – Declarative definitions of the VNet, subnet, NIC, public IP, NSG, and VM.
- **dependsOn** – Explicit dependency management to ensure, for example, the NIC and public IP exist before the VM is created.
- **Outputs** – Returns useful post-deployment values such as the VM's public IP address and resource ID.

## Security Considerations

- An NSG is attached to restrict inbound traffic to only the necessary ports (e.g., 22 for SSH or 3389 for RDP).
- SSH key-based authentication is used in place of password authentication where possible.
- Sensitive parameters (e.g., admin password) are marked with the `secureString` type to prevent exposure in logs or the portal.

## Troubleshooting Notes

Common issues encountered and resolved during this project included:

- **Validation errors** from malformed JSON or missing required properties — resolved by checking template syntax against ARM schema documentation.
- **Dependency errors** caused by resources referencing others that hadn't yet been created — resolved using `dependsOn`.
- **Deployment failures** due to naming conflicts or region/SKU availability — resolved by adjusting parameter values.

## Verification

Deployment success was verified by:

1. Confirming a `Succeeded` provisioning state via `az deployment group show`.
2. Connecting to the VM over SSH using its public IP address.
3. Capturing screenshots of the deployed resources in the Azure Portal (see `/screenshots`).

## Author

**Abdulsalam Olarinoye Bashir**
GitHub: [BashLaw-Cyber](https://github.com/BashLaw-Cyber) / [abdulsalamnetwork](https://github.com/abdulsalamnetwork)
Part of the 3MTT (3 Million Technical Talent) learning journey.

## License

This project is for educational purposes as part of the 3MTT program.
