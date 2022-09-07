> L01-05

# Create Azure Attestation provider
Next we will create and Azure Attestation provider to verify the secure enclave our Azure SQL database uses.

[Microsoft Azure Attestation](https://docs.microsoft.com/en-us/azure/attestation/overview) is a solution for remotely verifying the trustworthiness of a platform and integrity of the binaries running inside it.

## Step 1 - Configure Azure CLI to work with Attestation Service

Since the service is still in Preview, we need to register the resource provider to allow the Azure subscription to allocate Attestation Service resources.

```powershell
az provider register --name Microsoft.Attestation
```

The resource provider registration is asynchronous and may take few minutes. You can check when it's completed by running this command and checking that registrationState switches from "Registering" to "Registered":

```powershell
az provider show --name Microsoft.Attestation
```

Once the provider has been registered, install the _Azure CLI extension for Attestation_ so we can create a new attestation service using the CLI.

```powershell
az extension add --name attestation
```

## Step 2 - Create Attestation Service

Now we can create our attestation service.

```powershell
# make sure the name is unique
$attestName = "apcelacchol"

az attestation create `
   --name $attestName `
   --resource-group $rgName `
   --location $azLocation

# Get the URL for future use
$attestUrl = az attestation show --name $attestName --resource-group $rgName --query attestUri
$attestUrl = $attestUrl.Replace('"','')
```

## Step 3 - Import the attestation policy

Once the attestation service is created, assign the current user account permission to set policy:

```powershell
$apId = az attestation show --resource-group $rgName --name $attestName --query id

az role assignment create `
   --assignee $currentUserId `
   --role "Attestation Contributor" `
   --scope $apId
```

Next we add the policy for attesting Intel SGX enclaves.

> 😖 The experimental Azure CLI extension has a text encoding / decoding problem using the latest Python version so we will use Azure PowerShell here.

```Powershell
# Get the contents of the policy file
$apPolicy=Get-Content -path ".\Code\Lab 1 - Azure Resources\attestation-policy.txt" -Raw

# Set the SGX Enclave attestation policy
Set-AzAttestationPolicy `
   -Name $attestName `
   -ResourceGroupName $rgName `
   -Tee SgxEnclave `
   -PolicyFormat Text `
   -Policy $apPolicy
```

You can review the imported SGX Enclave policy.

```powershell
Get-AzAttestationPolicy `
   -Name $attestName `
   -ResourceGroupName $rgName `
   -Tee SgxEnclave
```

> 📖 Learn more about [authoring attestation policies](https://docs.microsoft.com/en-us/azure/attestation/author-sign-policy).

---

Next we will deploy the Confidential VM: [L1-06 Confidential VM](./L01-06-CreateConfidentialVm.md)