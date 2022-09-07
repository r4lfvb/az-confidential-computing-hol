> L01-04

# Create Confidential Ledger

To store our application logs securely and prevent tampering we will create an [Azure Confidential Ledger](https://docs.microsoft.com/en-us/azure/confidential-ledger/overview).

## Step 1 - Configure the Azure CLI to work with Confidential Ledger

Since the Confidential Ledger is still in Preview, we need to register the resource provider to allow the Azure subscription to allocate Confidential Ledger resources.

```powershell
az provider register --namespace Microsoft.ConfidentialLedger
```

The resource provider registration is asynchronous and may take few minutes. You can check when it's completed by running this command and checking that registrationState switches from "Registering" to "Registered":

```powershell
az provider show --name Microsoft.ConfidentialLedger
```

Once the provider has been registered, install the _Azure CLI extension for Confidential Ledger_ so we can create a new ledger using the CLI.

```powershell
az extension add --name confidentialledger
```

## Step 2 - Create a confidential ledger

Create a new Confidential Ledger and assign the current user as the administrator:

```powershell
# Create a Confidential Ledger

# Make sure the name is unique
$clName = "clCelAccHol"

# Note we are using westeurope here, as Confidential Ledger is currently not available in North Europe
az confidentialledger create `
  --name $clName `
  --resource-group $rgName `
  --location westeurope `
  --ledger-type Public `
  --aad-based-security-principals ledger-role-name=Administrator principal-id=$currentUserId
```

---

Now that your Confidential Ledger is ready we can deploy an Azure Attestation Service to validate the Azure SQL enclave before decryption data in use: [L01-05 Create Attestation Servce](./L01-05-CreateAttestationService.md)