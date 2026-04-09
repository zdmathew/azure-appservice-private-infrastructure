# azure-appservice-private-infrastructure

> Production-grade Terraform module for deploying isolated Azure App Service infrastructure into existing virtual network environments with full private endpoint coverage and NSG enforcement.

---

## Overview

This module provisions a secure, fully private Azure application hosting environment by deploying App Service Plans, Web Apps, and Function Apps into an existing VNet and subnet. All public access is disabled. Resources communicate exclusively over private endpoints with integrated Private DNS resolution.

Built to be replicable across environments (dev, test, prod) using configurable Terraform variables — no hardcoded regions, resource names, or connection strings.

---

## Architecture

```
Existing VNet / Subnet
│
├── App Service Plan A
│   ├── Web App A  ──────── Private Endpoint ──── Private DNS Zone
│   └── Function App A ──── Private Endpoint ──── Private DNS Zone
│
├── App Service Plan B
│   ├── Web App B  ──────── Private Endpoint ──── Private DNS Zone
│   └── Function App B ──── Private Endpoint ──── Private DNS Zone
│
└── Existing Storage Account
        └── Private Endpoint ──── Private DNS Zone

NSG Rules applied per resource
```

---

## Resources Provisioned

| Resource | Count | Notes |
|---|---|---|
| App Service Plan | 2 | Configurable SKU |
| Web App | 2 | Linux, configurable runtime |
| Function App | 2 | Integrated with Storage Account |
| Private Endpoints | 6 | One per resource |
| Private DNS Zones | 6 | Linked to existing VNet |
| Network Security Groups | 4 | Applied per App Service and Function App |

---

## Prerequisites

- Existing Azure Virtual Network and Subnet
- Existing Storage Account (for Function App backend)
- Terraform >= 1.3
- AzureRM Provider >= 3.0
- Appropriate Azure RBAC permissions (Contributor or custom role)

---

## Usage

```hcl
module "appservice_private" {
  source = "./modules/appservice-private"

  resource_group_name  = var.resource_group_name
  location             = var.location
  environment          = var.environment

  vnet_name            = var.vnet_name
  vnet_resource_group  = var.vnet_resource_group
  subnet_id            = var.subnet_id

  storage_account_id   = var.storage_account_id
  storage_account_name = var.storage_account_name

  app_service_plan_sku = var.app_service_plan_sku
  runtime_stack        = var.runtime_stack
  runtime_version      = var.runtime_version

  tags = var.tags
}
```

---

## Variables

| Variable | Description | Type | Required |
|---|---|---|---|
| `resource_group_name` | Target resource group | `string` | Yes |
| `location` | Azure region | `string` | Yes |
| `environment` | Environment tag (dev/test/prod) | `string` | Yes |
| `vnet_name` | Name of existing VNet | `string` | Yes |
| `vnet_resource_group` | Resource group of existing VNet | `string` | Yes |
| `subnet_id` | Subnet ID for VNet integration | `string` | Yes |
| `storage_account_id` | Resource ID of existing Storage Account | `string` | Yes |
| `storage_account_name` | Name of existing Storage Account | `string` | Yes |
| `app_service_plan_sku` | SKU for App Service Plans (e.g. P1v3) | `string` | No |
| `runtime_stack` | Runtime stack (e.g. PYTHON, NODE, DOTNET) | `string` | No |
| `runtime_version` | Runtime version | `string` | No |
| `tags` | Tags to apply to all resources | `map(string)` | No |

---

## Security Considerations

- All public network access disabled on Web Apps and Function Apps
- Private endpoints provisioned for each resource with dedicated Private DNS zone links
- NSGs restrict inbound and outbound traffic at the subnet and resource level
- VNet integration uses delegated subnets — no overlap with existing address space
- Storage Account access restricted to private endpoint only

---

## Deployment

```bash
terraform init
terraform plan -var-file="environments/dev.tfvars"
terraform apply -var-file="environments/dev.tfvars"
```

---

## Author

**Zachary Mathew** — Senior Cloud & Infrastructure Engineer  
[linkedin.com/in/zachary-mathew](https://linkedin.com/in/zachary-mathew)

---

## License

MIT
