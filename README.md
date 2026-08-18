You are joining a DevOps team responsible for deploying Azure infrastructure using Bicep and Azure DevOps YAML pipelines.

Build a small reusable infrastructure deployment that can be promoted through Dev → Test → Prod environments.

The solution should be production-oriented: reusable IaC, secure secret handling, parameterization, clear separation of environments, and basic operational automation.

> AI tools may be used during the assignment but it is highly not recomended.

## How to submit

1. **Fork this repository** to your own GitHub account.
2. Do your work on a branch in your fork (`main` or a feature branch — your choice).
3. Commit as you go — commit history is part of the review, so prefer several meaningful commits over one large one.
4. When done, **open a Pull Request from your fork back into this repository's `main` branch**.
5. Make sure your PR description summarizes what you built and links out to `DESIGN.md`, `SECURITY.md`, and `AUTOMATION.md`.

Do not request access to push directly to this repository — all submissions come in via PR from a fork.

## Deliverables

In addition to the code under `infra/`, `pipelines/`, and `scripts/`, fill in:

- **`DESIGN.md`** — naming conventions, architecture decisions, environment differences, assumptions.
- **`SECURITY.md`** — secret handling, identity/access, Key Vault model, network exposure (required by §1.3).
- **`AUTOMATION.md`** — purpose and usage of your additional PowerShell automation (required by §3.8).

Each file already contains prompts describing what's expected — replace them with your own content.

# Technical Assignment

## 1. Infrastructure as Code — Bicep

Create reusable Bicep templates for deploying the required Azure resources.

### 1.1 Storage Accounts

Deploy **four Storage Accounts**.

The Storage Accounts must be created using a **loop** based on a configuration structure. Do not create four separate, duplicated resource definitions.

| Name | SKU | Kind | Purpose |
| --- | --- | --- | --- |
| `cold` | Standard | StorageV2 | Archive / cold data |
| `standard` | Standard | StorageV2 | General purpose |
| `hot` | Standard | StorageV2 | Frequently accessed data |
| `datalake` | Standard | StorageV2 | Data Lake |

The Storage Accounts should not all have identical configuration.

The following requirements apply:

- `cold` must use the appropriate cool access tier.
- `hot` must use the appropriate hot access tier.
- `datalake` must have hierarchical namespace enabled.
- `standard` should use a standard general-purpose configuration.
- Resource definitions must be driven by reusable configuration rather than duplicated resource blocks.

### 1.2 Blob Containers

Create blob containers associated with the appropriate Storage Accounts.

At minimum, create the following containers:

- `cold-data`
- `standard-data`
- `hot-data`
- `raw-data`

Container deployment should also avoid unnecessary duplication and should use a reusable/configuration-driven approach where appropriate.

### 1.3 Key Vault

Deploy an Azure Key Vault as part of the infrastructure.

The Key Vault must contain a secret named:

`external-api-key`

The secret value will be provided externally during the deployment process.

The secret value must **not** be:

- hardcoded in Bicep;
- committed to the Git repository;
- stored in plain-text configuration files.

The solution should support different secret values for different environments.

Document any assumptions and security considerations in the `README.md`.

### 1.4 App Service

Deploy an Azure App Service that runs a standard Hello World container image https://mcr.microsoft.com/azuredocs/aci-helloworld

The application does not need to implement any custom business logic.

The App Service must:

- run the provided Hello World container image;
- use a system-assigned managed identity;
- use the managed identity to access the Key Vault;
- expose the `external-api-key` secret as an application setting using an Azure Key Vault reference;
- have the minimum required permissions to read the secret from the Key Vault;
- have the minimum required permissions to access the designated Storage Account/container.

The actual secret value must never be stored directly in the App Service configuration.

The App Service should use its managed identity for access to Azure resources instead of storage account keys or other long-lived credentials.

### 1.5 Reusability

The Bicep solution should be structured so that resource definitions can be reused and maintained independently.

Avoid putting the entire infrastructure definition into a single monolithic file.

Use appropriate Bicep modules and parameters.

The solution should allow additional Storage Accounts to be added through configuration without requiring a new copy-pasted resource definition.

### 1.6 Tags

Apply a common set of tags to the deployed resources.

At minimum, include:

- `environment`
- `application`
- `owner`
- `managedBy`

The `environment` value must be provided by the deployment configuration rather than hardcoded into individual resources.

### 1.7 Deployment Safety

The infrastructure should be safe to deploy repeatedly.

Consider what should happen when:

- the deployment is executed multiple times;
- only one Storage Account configuration changes;
- a new Storage Account is added;
- an existing resource configuration changes.

## 2. Azure DevOps CI/CD

Create an Azure DevOps YAML pipeline to deploy the infrastructure defined in Section 1.

The pipeline must support three environments:

- Dev
- Test
- Prod

The deployment flow should be:

```text
Validation --> Dev --> Test --> Prod
```

### 2.1 Multi-stage Pipeline

The pipeline must be implemented as a multi-stage YAML pipeline.

Each environment must have a separate deployment stage.

The pipeline should minimize duplication between environments.

Use reusable YAML templates where appropriate.

### 2.2 Validation

The pipeline must validate the Bicep code before deployment.

The validation stage should include:

- Bicep build/validation;
- Bicep linting;
- infrastructure `what-if`.

The validation stage must not deploy infrastructure.

### 2.3 Environment-specific Configuration

Dev, Test, and Prod must support environment-specific configuration.

At minimum, the following values should be configurable per environment:

- Azure subscription;
- Azure region;
- resource naming;
- environment name;
- external API key.

Environment-specific values must not be hardcoded inside reusable Bicep modules or unnecessarily duplicated across pipeline stages.

### 2.4 Production Approval

The Production deployment must require manual approval before the deployment can proceed.

The approval should be implemented using Azure DevOps deployment/environment capabilities.

The pipeline must not deploy to Production automatically without the required approval.

### 2.5 External API

The infrastructure integrates with an external API `https://api.example.com`

An API key is required to access the API.

Use a different API key for each environment:

- Dev: `NGYwNWRiNmQzNTJhNGQwZDg0YzFiNzgwNzgxNmQzYzU=`
- Test: `ZTc0MTEyMjg0NmZiNDMyMThlM2VjYjkwMDdlOGI1ZmY=`
- Prod: `NjY5ZDU0ODRhNjM0NGI1MDkxMzVjYmM2YjgzY2I0ZmE=`

The API keys must:

- not be committed to the repository;
- be stored and handled as secrets;
- not appear in pipeline logs;
- not be exposed through pipeline output;
- be provided to the infrastructure deployment securely;
- ultimately be stored in the Key Vault as `external-api-key`.

The exact implementation is up to you.

Document your approach and explain how you prevent the secret from being exposed during pipeline execution.

### 2.6 Pull Request Validation

Add Pull Request validation for infrastructure changes.

Pull Request validation should perform appropriate checks, including:

- Bicep build/validation;
- linting;
- `what-if`.

Pull Request validation must not deploy infrastructure to any environment.

### 2.7 Pipeline Security

The pipeline must follow secure handling practices for credentials and secrets.

Do not store credentials, API keys, passwords, access tokens, or other sensitive values directly in YAML files.

Avoid commands or logging configuration that could expose secret values.

Use appropriate Azure DevOps mechanisms for authentication and secret management.

### 2.8 Deployment Validation

After each environment deployment, execute the PowerShell validation script from Section 3.

A failed validation must cause the corresponding pipeline stage to fail.

The Production deployment must not proceed if the previous environment's deployment or validation has failed.

## 3. PowerShell Automation

Create a PowerShell script to validate the Azure infrastructure deployed by the pipeline.

The script should be suitable for execution both locally and as part of the Azure DevOps pipeline.

### 3.1 Infrastructure Validation

Create a script such as:

```text
scripts/Test-AzureDeployment.ps1
```

The script must accept at least the following parameters:

- Subscription ID
- Resource Group Name
- Environment

Example:

```powershell
./Test-AzureDeployment.ps1 `
    -SubscriptionId "..." `
    -ResourceGroupName "rg-example-dev" `
    -Environment "dev"
```

The script should validate that:

- the Resource Group exists;
- all expected Storage Accounts exist;
- all expected Blob Containers exist;
- the Key Vault exists;
- the required `external-api-key` secret exists;
- required resource tags are present;
- Storage Account configuration matches the expected configuration for the environment.

### 3.2 Validation Results

The script should provide a clear and useful summary of the validation results.

For example:

```text
Storage Accounts
----------------
cold        OK
standard    OK
hot         OK
datalake    FAIL - hierarchical namespace is disabled

Blob Containers
---------------
cold-data       OK
standard-data   OK
hot-data        OK
raw-data        OK

Key Vault
---------
exists          OK
secret          OK

Tags
----
environment     OK
application     OK
owner           FAIL

Validation failed: 2 checks failed.
```

The exact output format is up to you.

The script should report all relevant validation failures rather than stopping after the first failed check where practical.

### 3.3 Exit Codes

The script must return:

- exit code `0` when all required checks pass;
- a non-zero exit code when one or more checks fail.

The exit code must allow the Azure DevOps pipeline to determine whether the validation was successful.

### 3.4 Error Handling

The script should handle errors appropriately.

Consider scenarios such as:

- invalid subscription;
- missing Resource Group;
- missing Storage Account;
- missing container;
- missing Key Vault;
- missing secret;
- insufficient permissions;
- unexpected Azure API errors.

Errors should be reported clearly without exposing sensitive information.

### 3.5 Secret Handling

The script must not print or expose the value of `external-api-key`.

The secret value must not appear in:

- console output;
- error messages;
- verbose/debug output;
- generated files;
- Azure DevOps pipeline logs.

The script only needs to verify that the required secret exists and does not need to display its value.

### 3.6 Azure DevOps Integration

The PowerShell validation script must be executed automatically after each environment deployment.

A failed validation must fail the corresponding Azure DevOps deployment stage.

The validation should use the environment being deployed rather than relying on hardcoded resource names or configuration.

### 3.7 Reusability

Avoid hardcoding environment-specific values inside the script.

The same script should be reusable for:

- Dev;
- Test;
- Prod.

Environment-specific information should be supplied through parameters or configuration.

### 3.8 Additional Automation

Add at least one additional PowerShell automation that you consider useful for this infrastructure.

Examples could include:

- reporting resource configuration;
- checking required Azure resource tags;
- identifying configuration drift;
- generating a deployment summary;
- performing a pre-deployment validation.

Explain the purpose of the additional automation in the `AUTOMATION.md`.