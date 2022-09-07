# Clean up ♻️

Once you've finished with the lab, delete the Azure resources to stop billing 💸

## Step 1 - Remove app registration

Remove the Contoso HR app registration from Azure AD.
```powershell
az ad app delete --id $clientId
```

## Step 2 - Remove remaining resources

Then remove the remaining resources:

```powershell
# Delete the resource group
az group delete --name $rgName
```

> ⚠️ Note that the Managed HSM was deleted but because purge protection is enabled it will remain recoverable (and be charged for) for the retention period which we set at 7 days (which is the minimum) in the lab. At the end of the retention period it will be purged. Once purged it will be unrecoverable and no longer charged.

> 📖 Learn more about [Managed HSM soft delete and purge protection](https://docs.microsoft.com/en-gb/azure/key-vault/managed-hsm/recovery)

---

All the Azure resources should now be removed from your subscription!