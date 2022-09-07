> L01-03

In this module we will set up the following Azure resources

| Resource | Description |
| -- | -- |
| Managed Identity | Identity Azure SQL will use to access the Managed HSM |
| Azure SQL Logical Server | Server to host the SQL database |
| Azure SQL Database | Database to store the encrypted HR data |

> 💡 Azure SQL will encrypt and decrypt our sensitive data with two platform features:
> - [Transparent Data Encryption](https://docs.microsoft.com/en-us/azure/azure-sql/database/transparent-data-encryption-tde-overview?view=azuresql) for data at rest
> - [Always Encrypted with secure enclaves](https://docs.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-enclaves) for data in use

## Step 1 - Create a Managed Identity with access to the encryption keys

First we will create a new [Managed Identity](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview) that Azure SQL will use to access the encryption keys stored in the Managed HSM.

```powershell
# Create a new Managed Identity

$miSqlName = "miAzureSql"

az identity create `
  --name $miSqlName `
  --resource-group $rgName `
  --location $azLocation
```

> ⚠️ note the **appID** and **password** of the new managed identity for a future step

Assign the managed identity to the _Crypto Service Encryption User_ built-in role granting it permission to use the encryption keys.

```powershell
# Get the Managed Identity Service Principal ID
$miPrincipalId = az identity show --name $miSqlName --resource-group $rgName --query principalId

# Grant the Managed Identity permission to use the encryption keys
az keyvault role assignment create `
  --hsm-name $hsmName `
  --role "Managed HSM Crypto Service Encryption User" `
  --scope "/" `
  --assignee $miPrincipalId
```

> 📖 See more about [Managed HSM Access Control](https://docs.microsoft.com/en-us/azure/key-vault/managed-hsm/access-control)

## Step 2 - Create a logical Azure SQL server

First set the name of the server:

```powershell
# Make sure the name is unique
$sqlServerName = "sqlCelAccHol"
```

Now deploy the Azure SQL logical server:

```powershell
# Get the ID of the SQL encryption key from the HSM
$sqlKeyId = az keyvault key show --hsm-name $hsmName --name $sqlKeyName --query "key.kid"

# Get the ID of the Managed Identity
$managedIdentityId = az identity list --query "[?name == 'miAzureSql'].id | [0]"

# Set the admin credentials
$sqlAdmin = "celSqlAdmin"
$sqlPassword = "nooneWillEverGuessThis0ne!+$"

# Create the logical server
az sql server create `
   --name $sqlServerName `
   --resource-group $rgName `
   --location $azLocation `
   --admin-user $sqlAdmin `
   --admin-password $sqlPassword `
   --assign-identity `
   --user-assigned-identity-id $managedIdentityId `
   --primary-user-assigned-identity-id $managedIdentityId `
   --identity-type UserAssigned `
   --key-id $sqlKeyId
```

> 💡 The primary user assigned identity will be used when Azure SQL needs to access keys stored in the Managed HSM

## Step 3 - Create an Azure SQL database

> 😖 At the moment the Azure CLI does not yet support the creation of Databases using confidential compute so we will use the PowerShell Azure Commandlets here.

```Powershell
$databaseName = "ContosoHR"

New-AzSqlDatabase `
  -ResourceGroupName $rgName `
  -ServerName $sqlServerName `
  -DatabaseName $databaseName `
  -Edition GeneralPurpose `
  -Vcore 2 `
  -ComputeGeneration DC
```

> 💡 Note that we're using the **DC compute generation** to make sure the database is hosted on processors with Intel SGX support.

---

Now that the database is ready we can create the Confidential Ledger: [L01 05 Confidential Ledger](L01-04-CreateAzureSql.md)