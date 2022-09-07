# Getting Started

Welcome to the Confidential Computing lab! This page will help you get your development environment ready.

## Azure:

- Azure subscription
- Available quota for the DC family of VMs

## Install software

- Winget
- Windows Terminal
- PowerShell
- Azure CLI
- OpenSSL
- Git
- Visual Studio Code
- SQL Server Management Studio

You can install these manually or using **Windows Terminal** and winget:

```powershell

# Install required software
winget install --id=Microsoft.DotNet.SDK.5 -e && `
winget install --id=Microsoft.PowerShell -e && `
winget install --id=Microsoft.AzureCLI -e && `
winget install --id=ShiningLight.OpenSSL -e && `
winget install --id=Git.Git -e && `
winget install --id=Microsoft.VisualStudioCode -e && `
winget install --id=Microsoft.SQLServerManagementStudio -e

# Reload the current PowerShell session path
$Env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine")

### FIX NUGET DOTNET PUBLISH PROBLEM HERE

```
> ⚠️ Note: If the winget shininglight install does not work get it here: https://slproweb.com/download/Win64OpenSSL-3_0_5.msi

Make sure you have the latest [Azure PowerShell module](https://docs.microsoft.com/en-us/powershell/azure/new-azureps-module-az?view=azps-8.1.0) installed:

```powershell

# Allow installation of signed software for the current user
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Install the latest version of the Azure PowerShell module
Install-Module -Name Az -Scope CurrentUser -Repository PSGallery -Force -AllowClobber

```

## Copy the lab files to your PC

Open **Windows Terminal** and use Git to clone the repository to a folder on your PC.

```powershell
git clone https://github.com/r4lfvb/az-confidential-computing-hol.git
```

## Lab overview

1. Deploy and configure resources using the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/), [Azure PowerShell](https://docs.microsoft.com/en-us/powershell/azure/new-azureps-module-az?view=azps-8.1.0) and [ARM Templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/) ☁️
2. Build and deploy the _Contoso HR_ application 👩🏽‍💻
3. Attempt to access sensitive HR data 🔥
3. Remove the Azure resources used in the lab ♻️

---

Let's get started by creating our Azure resources: [L01-01 Create Azure Resources](./Lab%201%20-%20Azure%20Resources/L01-01-CreateAzureResources.md)