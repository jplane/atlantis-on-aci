# Running Atlantis (https://www.runatlantis.io) on Azure Container Instances

Requirements:

1. A Bash shell (on Windows you'll probably want [WSL](https://docs.microsoft.com/en-us/windows/wsl/about))

1. [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)

1. [openssl](https://www.openssl.org/)


Run the following commands from the same location as the master [script](atlantis-on-aci.sh).

```bash
#! /bin/bash

GITHUB_USER=YOUR-USER
GITHUB_TOKEN=YOUR-TOKEN
GITHUB_WEBHOOK_SECRET=YOUR-SECRET
REPO_WHITELIST=github.com/YOUR-ORG/YOUR-REPO

ATLANTIS_LOCATION=YOUR-AZURE-LOCATION
SUFFIX=123

. ./atlantis-on-aci.sh
```

Change the argument values as needed, and note that you can remove the 'SUFFIX' argument entirely and have a unique suffix generated on your behalf, if you wish. See the master [script](atlantis-on-aci.sh) for additional arguments you can optionally set if you don't want the defaults.

Notes:

1. [Authenticate](https://docs.microsoft.com/en-us/cli/azure/authenticate-azure-cli?view=azure-cli-latest) from the Azure CLI to a valid [Azure Active Directory tenant](https://docs.microsoft.com/en-us/azure/active-directory/develop/)

1. [Set](https://docs.microsoft.com/en-us/cli/azure/manage-azure-subscriptions-azure-cli?view=azure-cli-latest) your target Azure subscription

   ```bash
   az account set --subscription "my sub"
   ```

1. The script creates the following resources on your behalf:

    * A new resource group
    * A new storage account within the RG
    * A new blob container within the storage account (used for durable storage of [Terraform execution state](https://www.terraform.io/docs/backends/types/azurerm.html))
    * A new file share within the storage account (used to hold SSL self-signed certs for Atlantis HTTPS support)
    * A new [container group](https://docs.microsoft.com/en-us/azure/container-instances/container-instances-container-groups) within the RG configured to execute the [Atlantis Docker image](https://hub.docker.com/r/runatlantis/atlantis/)
    * An [MSI identity](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/) configured with [Contributor role access](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#contributor) to your Azure subscription, under which the ACI group executes. This identity has sufficient permission to create new resources on your behalf, within the Atlantis workflow
    
1. The script generates a self-signed certificate using [openssl.exe](https://www.openssl.org/) and uploads it to an [Azure File share](https://docs.microsoft.com/en-us/azure/storage/files/storage-files-introduction). The certificate is then configured within Atlantis to provide HTTPS support. **You should switch to a real (non-self-signed) cert in any production scenario.**

1. The script configures [MSI support](https://www.terraform.io/docs/providers/azurerm/auth/managed_service_identity.html) for Atlantis/Terraform execution as well as configuration of an ambient [ARM_SUBSCRIPTION_ID](https://www.terraform.io/docs/providers/azurerm/index.html#subscription_id) so that you don't need to provide one in each Terraform script you apply using the Atlantis workflow

1. For convenience, the script also configures [SKIP_CREDENTIALS_VALIDATION](https://www.terraform.io/docs/providers/azurerm/index.html#skip_credentials_validation) = false and [SKIP_PROVIDER_REGISTRATION](https://www.terraform.io/docs/providers/azurerm/index.html#skip_provider_registration) = false, but you can change these by setting environment variables prior to script execution
