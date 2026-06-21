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
├── azuredeploy.json               # Main ARM template
├── azuredeploy.parameters.json    # Parameter values for deployment
├── screenshots/                   # Deployment and connectivity verification proof
├── outputs.log                    # Deployment output log (Public IP, Resource IDs)
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
  --location eastus2
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
ssh azureuser@<publicIPAddress>
```

## Key ARM Template Concepts Applied

- **Parameters** – Dynamic inputs such as VM name, admin username, authentication type, and location, avoiding hardcoded values.
- **Variables** – Computed or reused values (e.g., resource naming conventions) derived from parameters.
- **Resources** – Declarative definitions of the VNet, subnet, NIC, public IP, NSG, and VM.
- **dependsOn** – Explicit dependency management to ensure, for example, the NIC and public IP exist before the VM is created.
- **Outputs** – Returns useful post-deployment values such as the VM's public IP address and resource ID.

## Security Considerations

- An NSG is attached to restrict inbound traffic to only the necessary ports (port 22 for SSH).
- Sensitive parameters (e.g., admin password/key) are marked with the `secureString` type to prevent exposure in logs or the portal.
- Inbound rules were reviewed in the Azure Portal to confirm only the intended SSH rule is open, with all other inbound traffic denied by default.

## Troubleshooting Notes

Issues encountered and resolved during this project:

- **`SkuNotAvailable` error** — `Standard_B1s` had no available capacity in `eastus`. Resolved by switching the deployment region to `eastus2`.
- **Dependency ordering** — handled using `dependsOn` so the NIC and public IP exist before the VM is created.
- **SSH host key verification prompt** — expected on first connection to a new VM; accepted the host fingerprint to proceed.

## Verification

Deployment success was verified through the following steps and supporting screenshots in `/screenshots`:

### 1. Resource Group Created

`screenshots/resource_group_screen.png`

### 2. Template Validation

Template validated successfully with no errors before deployment.

`screenshots/validate_deployment_screen.png`

### 3. Deployment Execution

ARM template deployed via `az deployment group create`, showing dependency resolution (NSG → VNet → NIC → VM) and a `Succeeded` provisioning state.

`screenshots/deployment_with_arm_screen.png`
`screenshots/deployment_with_arm1_screen.png`

### 4. Deployment Outputs

Retrieved via `az deployment group show --query properties.outputs`, confirming the public IP address, generated SSH command, and VM resource ID.

```json
{
  "publicIPAddress": { "value": "20.110.158.73" },
  "sshCommand": { "value": "ssh azureuser@20.110.158.73" },
  "vmResourceId": { "value": "/subscriptions/.../resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/bashlaw-vm" }
}
```

### 5. Resources Provisioned in Azure Portal

All six resources (VM, NIC, NSG, public IP, VNet, OS disk) confirmed in `myResourceGroup` under region East US 2.

`screenshots/resource_group_screen.png`

### 6. Connect Blade — SSH Command

Azure Portal's **Connect** blade confirming the public IP, port 22, and the native SSH command for the VM.

`screenshots/connect_methods_screen.png`

### 7. Successful SSH Connection

Connected to the VM over SSH from the local machine, confirming host key acceptance, authentication, and a live Ubuntu 22.04.5 LTS session.

`screenshots/ssh_connect_screen.png`
`screenshots/ssh_connect1_screen.png`

### 8. NSG Inbound/Outbound Rules

Confirmed the NSG attached to the subnet allows only SSH (port 22) inbound, with default deny-all rules otherwise in place.

`screenshots/ngs_screen.png`

## Author

**Abdulsalam Olarinoye Bashir**
GitHub: [BashLaw-Cyber](https://github.com/BashLaw-Cyber) / [abdulsalamnetwork](https://github.com/abdulsalamnetwork)
Part of the 3MTT (3 Million Technical Talent) learning journey.

## License

This project is for educational purposes as part of the 3MTT program.
