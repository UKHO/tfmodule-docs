# Terraform Module: for Azure vnet with nsg

## Required Resources

- `Resource Group` exists or is created external to the module.

## Usage

```terraform
variable "address" {
  default = "10.0.1.0/24"
}

variable "subnets" {
  default = [
    {
      name = "service-subnet"
      number = 0
    }
  ]
}

variable "endpoints" {
  default = ["Microsoft.Sql", "Microsoft.Storage", "Microsoft.KeyVault"]
}

variable "tags" {
  type = map
  default = {
    ENVIRONMENT      = "Dev"
  }
}

resource "azurerm_resource_group" "gg" {
  name = "existing-rg"
  location = "UKSouth"
  tags = var.tags
}

module "setup" {
  source                        = "github.com/ukho/tfmodule-azure-vnet-with-nsg?ref=1.0.0"
  prefix                        = "Prefix"
  tags                          = "${var.tags}"
  resource_group                = azurerm_resource_group.gg
  address                       = "${var.address}"
  subnets                       = "${var.subnets}"
  newbits                       = "4"
  service_endpoints             = "${var.endpoints}"
}
```

if you arent woried about the version you use, latest can be retrieved by removing `?ref=x.y.z` from source path

## Example for subnets

subnets are created using an array expecting a `name` and a `number`, number should increment from 0.

It is also worth noting, the addition of newbits to the base address should not exceed /29. Azure has the habit of absorbing 5 ip addresses per subnet. so the smallest you could go it a range of 8 ips (/29). with a newbits of 4, this would imply a minimum base of /25 is needed.

```terraform
[{
  name = "subnet1-subnet"
  number = 0
},
{
  name = "subnet2-subnet"
  number = 1
}]
```

## Service Endpoints

An example of `service_endpoints` is ["Microsoft.Sql", "Microsoft.Storage", "Microsoft.KeyVault"]
