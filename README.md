
# Terraform and Azure CLI Setup Guide for Azure Infrastructure

This guide walks you through everything from installing Terraform and Azure CLI to provisioning Azure resources using CSV-based configurations.

---

## Table of Contents

1. [Install Terraform](#1-install-terraform)
2. [Install Azure CLI](#2-install-azure-cli)
3. [Login to Azure Account](#3-login-to-azure-account)
4. [Terraform Configuration](#4-understand-csv-files-and-variables)
5. [Role Assignments (IAM) in Azure](#5-role-assignments-iam-in-azure)
6. [Execution Guide](#6-execute-the-deployment-script)
7. [Common Issues and Tips](#7-common-issues-and-tips)

---

## 1. Install Terraform

Terraform is used to **automate Azure infrastructure deployment** using simple configuration files.

### 🔧 Installation Steps

1. Visit the official site: [Terraform Downloads](https://developer.hashicorp.com/terraform/install)
2. Download the Windows version.
3. Extract the zip file.
4. Copy `terraform.exe` to a folder like `C:\Terraform`.
5. Add `C:\Terraform` to your system `PATH`:

   * Open Windows Search → Type "Environment Variables"
   * Edit the `Path` variable in system settings
   * Add `C:\Terraform`

### ✅ Verify Installation

```bash
terraform -version
```

Expected Output:

```
Terraform v1.6.x
```

---

## 2. Install Azure CLI

Azure CLI is a command-line tool to **manage and configure Azure resources**.

### 🔧 Installation via PowerShell

```powershell
winget install --exact --id Microsoft.AzureCLI
```

If you don’t have `winget`, download manually:
👉 [Azure CLI Manual Install](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows?view=azure-cli-latest)

### ✅ Verify Installation

```bash
az version
```

---

## 🔐 3. Login to Azure Account

After installing the Azure CLI, **you must log in to access your Azure resources**.

### 👤 Login to Azure

Run this command in Command Prompt or PowerShell:

```bash
az login
```

* This opens a browser and prompts you to sign in with your **Azure AD (Active Directory)** credentials.
* After successful login, it shows your subscriptions and tenant details.

### 📍 Purpose

* Required for Terraform to access Azure.
* Your credentials are stored locally for CLI and Terraform usage.

### 💡 Useful Azure AD CLI Commands

* View logged-in user:

  ```bash
  az account show
  ```

---

## 4. Terraform Configuration:

* Navigate to the `terraform-configs` folder in fabric starter kit.
* After setting the values below, upload the following metadata files containing the configuration below.
Each module in the Terraform setup reads **CSV files** to create resources dynamically.

---

### 4.1 Resource Group Module:

### 🔹 resource\_group\_variables.csv

| Column Name               | Description                                          |
| ------------------------- | ---------------------------------------------------- |
| `environment`             | Name of the environment: `dev`, `test`, `prod`       |
| `project`                 | Name of the Project                                 |
| `resource_group_name`     | Name of the Azure resource group to be created       |
| `resource_group_location` | Azure region (e.g., `East US`, `West Europe`)        |
| `costcenter`              | Tagname               |
| `container_name`          | Container in the storage account for Terraform state |
| `storage_account_name`    | Azure Storage Account for storing Terraform state    |

🔎 **Purpose**: This defines your resource group, and the backend state configuration needed for Terraform.

---

### 4.2 Fabric Capacity Module:

### 🔹 fabric\_variables.csv

| Column Name   | Description   |
| ------------- | ------------- |
| `environment` | Same as above |
| `project`     | Same as above |
| `costcenter`  | Same as above |

🔎 **Purpose**: Used to tag resources and group them logically by environment or project.

---

### 🔹 capacities.csv

| Column Name | Description                                                                        |
| ----------- | ---------------------------------------------------------------------------------- |
| `name`      | Name of the capacity (e.g., `cap-dev`)                                             |
| `location`  | Azure region                                                                       |
| `size`      | Capacity size (e.g., `F64`, `F32`)                                                 |
| `members`   | Comma-separated list of Azure AD users (e.g., `"[""user1@jmangroup.com"", ""user2@jmangroup.com""]"`) |

🔎 **Purpose**: Creates and configures Fabric Capacity with assigned users.

> **Note:**
> If you need to create a capacity larger than 64 units, please contact your Azure subscription manager to upgrade your subscription tier accordingly.

---

### 4.3 Application Module:

### 🔹 application\_variables.csv

| Column Name                | Description                              |
| -------------------------- | ---------------------------------------- |
| `application_display_name` | Name of the Azure AD App                 |
| `current_user_upn`         | User email (e.g., `john.doe@domain.com`) |

🔎 **Purpose**: Used for creating Azure AD apps and assigning the User.

---

### 4.4 Key vault Module:

### 🔹 key\_vault\_variables.csv

| Column Name                            | Description                  |
| -------------------------------------- | ---------------------------- |
| `key_vault_name`                       | Azure Key Vault name         |
| `location`                             | Region                       |
| `resource_group_name`                  | RG where Key Vault is placed |
| `environment`, `project`, `costcenter` | Same as above                |



### 🔹 vault\_access\_policies.csv

| Column Name               | Description                        |
| ------------------------- | ---------------------------------- |
| `email`                   | Azure AD User or Service Principal |
| `key_permissions`         | The specific permissions granted to the user or service principal for managing keys in the Azure Key Vault e.g., `"[""Get"", ""List""]"`     |
| `secret_permissions`      | The specific permissions granted to the user or service principal for managing secrets in the Azure Key Vault e.g., `"[""Get"", ""list"", ""set""]"`    |
| `certificate_permissions` | The specific permissions granted to the user or service principal for managing certificates in the Azure Key Vault e.g., `"[""Get"", ""import""]"`      |

🔎 **Purpose**: Defines access to Key Vault secrets, keys, and certificates.

---

### Roles Assignment Module:

### 🔹 role\_assignment\_variables.csv

| Column Name | Description                                          |
| ----------- | ---------------------------------------------------- |
| `resource_group_name`     | The name of the Azure resource group(Same as you Resource Group name). |


### 🔹 roles\_and\_users.csv

| Column Name | Description                                          |
| ----------- | ---------------------------------------------------- |
| `role`      | Role (e.g., `Contributor`, `Reader`)                 |
| `username`  | Mail address of the user                             |

---

### Role Assignments (IAM) in Azure

**IAM (Identity and Access Management)** helps define **who** has access to **what** in Azure.

### 🔑 Common IAM Roles

| Role Name                     | Permissions                             |
| ----------------------------- | --------------------------------------- |
| **Owner**                     | Full access + assign roles              |
| **Contributor**               | Full access except managing permissions |
| **Reader**                    | View-only access                        |
| **User Access Administrator** | Manage access permissions               |
| **Key Vault Contributor**     | Manage Key Vault content                |

📖 **Reference**:
👉 [Azure Built-in Roles List](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles)

### 🔗 Example Role Assignment via Terraform

```hcl
resource "azurerm_role_assignment" "example" {
  scope                = azurerm_resource_group.example.id
  role_definition_name = "Contributor"
  principal_id         = data.azuread_user.user.object_id
}
```

---

## 6. Execution Guide

Follow the steps below to configure and run the deployment script for setting up infrastructure and workspaces:

### 6.1 Execute the Script

Once the configuration is complete, run the following command in Git Bash to start the script:

 ```bash
   ./run_teraform_and_python.sh
   ```

### 6.2 Select an Option

After running the script, you will be prompted to select one of the following options:

* **1 → Infrastructure Setup + Fabric Workspace**
* **2 → Fabric Workspace Only**

Choose the appropriate option based on your project requirements.

---

## Script Functionality Options

### 1. Create Workspace and Onboard Users

Select this option to create workspaces and onboard users.

**Steps**:

* Choose "Create Workspace and Onboard User" from the menu.
* Enter:

  * The capacity name for the connection.
  * A workspace name.
  * For every workspace enter the capacity number from the capacities list.
* The script will automatically create four workspaces: dev, prod, landing-dev, landing-prod.
* Users will be assigned to these workspaces based on the roles defined in the `workspace_role_assignment.csv` file located at:

  ```
  Fabric/templates/workspace_role_assignment.csv
  ```

**User Role Guidelines**:

* Assign Project Manager access.
* Provide Contributor or Viewer access to team members.

> **Note:**
> Ensure you retrieve the user ID for each user from the Microsoft Entra ID section in the Azure portal and fill it in the CSV accordingly.

### 2. Exit

* Select "Exit" to return to the previous menu or terminate the process.

---

## 🛠️ 7. Common Issues and Fixes

| Issue                             | Fix                                                               |
| --------------------------------- | ----------------------------------------------------------------- |
| `az login` stuck or fails         | Run in PowerShell (not Git Bash), try `az logout` then login      |
| `terraform: command not found`    | Ensure Terraform path is set in environment variables             |
| Incorrect subscription being used | Use `az account set --subscription <id>`                          |
| CSV file not read                 | Ensure correct path and format (no extra commas or empty headers) |

---

## 🎓 Summary

By completing these steps, you'll have:

✅ Azure CLI and Terraform installed
✅ Authenticated access to Azure
✅ All resources defined via CSV
✅ IAM roles assigned correctly
✅ Infrastructure provisioned automatically

---

Would you like this saved as a downloadable `README.md` or `.pdf` file for easy sharing?
